---
title: "[UnrealEngine] Subsystem을 활용한 ObjectPool_1"
description: ObjectPool 초기화
date: 2025-08-15 00:00:00 +09:00
categories: [GameDev, UnrealEngine]
tags:
  [
    UnrealEngine,
    C++,
    Subsystem
  ]
---

> 작업한 내용을 정리한 글입니다.

## 개요
팀 프로젝트를 진행하면서 **싱글톤 패턴**과 **템플릿**을 이용해 매니저 클래스를 만든 적 있다. 하지만 이는 유니티 방식을 이용한 것이기에, 언리얼에는 적합하지 않았는데, 가장 큰 이유는 **객체의 생명 주기 관리** 문제 때문이다. 그렇기에 언리얼에서는 `GameInstance`나 `Subsystem`을 이용하는 것이 적합한데, 이번 포스팅에서는 이를 활용한 ObjectPool 구현 과정을 정리하고자 한다.

> 글은 작업하면서 겪은 우여곡절을 따라 진행됩니다.
{: .prompt-warning }

## DataTable
우선 풀을 통해 초기화될 객체들을 **테이블로 관리**하고자 했다. 이렇게 하면 데이터와 코드의 역할 분담이 명확해져 유지 보수적 측면에서 유리할 뿐더러 테스트도 용이하기 때문이다.

```c++
USTRUCT(BlueprintType)
struct FObjectPoolData : public FTableRowBase
{
	GENERATED_BODY()

public:
	UPROPERTY(EditAnywhere)
	FGameplayTag ObjectTag;

	UPROPERTY(EditAnywhere)
	TSubclassOf<AActor> ActorClass = nullptr;

	UPROPERTY(EditAnywhere, BlueprintReadOnly)
	int32 PoolSize = 10;
};
```

ObjectPool은 `Map`을 이용해 관리할 예정이다. 키를 이용해 배열에 접근해 요소를 꺼내오는 방식인 것이다. 여기서 키로 사용되는 것이 `FGameplayTag`이다.

## ObjectPoolSubsystem
`Subsystem`은 **특정 기능이나 데이터를 관리하는 모듈형 객체**이다. 쉽게 말해 언리얼 환경의 매니저 클래스인 셈이다. ObjectPool을 관리하는 Subsystem은 `UWorldSubsystem`을 상속받아 만들었는데, 월드와 생명 주기가 같기 때문에 목적에 적합하다 판단했다.

### 테이블 로드
```c++
UCLASS()
class OBJECTPOOL_API UObjectPoolSubsystem : public UWorldSubsystem
{
	GENERATED_BODY()

public:
	UObjectPoolSubsystem();
	
	virtual void OnWorldBeginPlay(UWorld& InWorld) override;

protected:
	UPROPERTY()
	TSoftObjectPtr<UDataTable> ObjectPoolDataTable = nullptr;

private:
	void LoadDataTable();
};
```

멤버로 `TSoftObjectPtr<UDataTable>` 객체를 선언했다. 데이터 테이블을 비동기로 로드하기 위함이다. 물론 원시 포인터나 `TObjectPtr`을 사용할 수 있지만, 초기화 때 한 번 사용할 객체를 하드하게 참조하는 것은 낭비라고 생각했다.

```c++
UObjectPoolSubsystem::UObjectPoolSubsystem()
{
	ObjectPoolDataTable = TSoftObjectPtr<UDataTable>(FSoftObjectPath(TEXT("/Script/Engine.DataTable'/Game/_DataTables/DT_ObjectPool.DT_ObjectPool'")));
}

void UObjectPoolSubsystem::OnWorldBeginPlay(UWorld& InWorld)
{
	Super::OnWorldBeginPlay(InWorld);

	if (GetWorld()->GetNetMode() == NM_Client)
		return;

	LoadDataTable();
}
```

생성자에서는 에셋의 경로를 직접 지정해 변수를 초기화했다. `Subsystem`의 블루프린트를 따로 만들 생각이 없었기에 단순히 하드 코딩으로 할당해줬다.

`OnWorldBeginPlay` 함수에서는 서버 체크 후 테이블 로드를 진행하도록 했다. 풀에 초기화되는 객체들을 서버에서 관리하기 위함이었다.

```c++
void UObjectPoolSubsystem::LoadDataTable()
{
	if (ObjectPoolDataTable.IsNull())
		return;

	FStreamableDelegate InitPoolDelegate = FStreamableDelegate::CreateLambda([this]
	{
		UDataTable* LoadedDataTable = ObjectPoolDataTable.Get();
		if (IsValid(LoadedDataTable))
			// InitPool(LoadedDataTable);
	});

	UAssetManager::GetStreamableManager().RequestAsyncLoad(ObjectPoolDataTable.ToSoftObjectPath(), InitPoolDelegate);
}
```

앞서 언급한 것처럼 데이터 테이블은 비동기로 로드한다. 그리고 로드 완료 시 호출될 콜백 함수를 **람다**로 만들어 전달했다.

### 풀 초기화
```c++
USTRUCT()
struct FActorPool
{
	GENERATED_BODY()

	UPROPERTY()
	TArray<TObjectPtr<AActor>> AllObjects;

	UPROPERTY()
	TArray<TObjectPtr<AActor>> AvailableObjects;
};

UCLASS()
class OBJECTPOOL_API UObjectPoolSubsystem : public UWorldSubsystem
{
	GENERATED_BODY()

	// ...

	UPROPERTY()
	TMap<FGameplayTag, FActorPool> ObjectPoolMap;

	void InitPool(const UDataTable* InDataTable);
};
```

다음으로 `FActorPool`을 정의했다. 멤버에 대한 설명은 다음과 같다.

- `AllObjects`
	- 모든 객체를 관리하기 위한 배열

- `AvailableObjects`
	- 사용 가능한 객체에 대한 배열
	- 객체를 꺼내거나 반납할 때 사용

```c++
void UObjectPoolSubsystem::InitPool(const UDataTable* InDataTable)
{
	TArray<FObjectPoolData*> AllData;
	InDataTable->GetAllRows<FObjectPoolData>(__FUNCTION__, AllData);

	for (const FObjectPoolData* Data : AllData)
	{
		if (Data->ObjectTag == FGameplayTag::EmptyTag || IsValid(Data->ActorClass) == false)
			continue;

		FActorPool& ActorPool = ObjectPoolMap.FindOrAdd(Data->ObjectTag);
		for (int32 Index = 0; Index < Data->PoolSize; ++Index)
		{
			FActorSpawnParameters SpawnParams;
			SpawnParams.SpawnCollisionHandlingOverride = ESpawnActorCollisionHandlingMethod::AlwaysSpawn;
	
			AActor* SpawnedActor = GetWorld()->SpawnActor<AActor>(Data->ActorClass, FVector::ZeroVector, FRotator::ZeroRotator, SpawnParams);
			if (IsValid(SpawnedActor) == false)
				continue;
	
			ActorPool.AllObjects.Emplace(SpawnedActor);
			ActorPool.AvailableObjects.Emplace(SpawnedActor);
		}
	}
}
```

최종적으로 객체를 초기화 해준다. 비동기로 얻어온 테이블을 이용해 각 행의 데이터를 따라 객체를 생성한 후 맵에 등록해주면 된다.

> `GetAllRows`의 첫 인자로 사용된 `__FUNCTION__`은 호출된 함수의 이름을 가져오는 매크로이다. 만약 타입 오류 등으로 인해 행을 가져오지 못한다면 로그 메시지로 함수 이름을 반환한다.
{: .prompt-info}

## 참고
{% linkpreview "https://github.com/GunnHB/ObjectPool" %}