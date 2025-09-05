---
title: "[UnrealEngine] Subsystem을 활용한 ObjectPool_2"
description: 객체의 활성화와 비활성화
date: 2025-09-05 00:00:00 +09:00
categories: [GameDev, UnrealEngine]
tags:
  [
    UnrealEngine,
    C++,
    Subsystem
  ]
---

> 작업한 내용을 정리한 글입니다.

## PoolableInterface
풀에 추가할 객체는 `PoolableInterface`를 상속받도록 했다. 물론 인터페이스에 선언된 함수들을 반드시 구현해야 한다는 규칙이 있지만, 이를 통해 느슨한 결합, 타입 안정성 등 유지보수와 확장에 용이한 구조라 판단했다.

```c++
// This class does not need to be modified.
UINTERFACE(MinimalAPI)
class UPoolableInterface : public UInterface
{
	GENERATED_BODY()
};

/**
 * 
 */
class OBJECTPOOL_API IPoolableInterface
{
	GENERATED_BODY()

	// Add interface functions to this class. This is the class that will be inherited to implement this interface.
public:
	virtual void OnPoolActivate() = 0;
	virtual void OnPoolDeactivate() = 0;
};
```

이렇게 하면 객체의 타입을 하나하나 따질 필요 없이 인터페이스를 통해 활성화 / 비활성화할 수 있다.

##  Bullet
테스트를 위한 객체를 생성한다. 해당 프로젝트에서는 `Bullet`이라는 이름의 액터를 생성했고, 여기에 앞서 만든 PoolableInterface를 상속받도록 했다.

```c++
// ABullet.h
UCLASS()
class OBJECTPOOL_API ABullet : public AActor, public IPoolableInterface
{
	GENERATED_BODY()

public:
	ABullet();

	virtual void OnPoolActivate() override;
	virtual void OnPoolDeactivate() override;
};

// ABullet.cpp
ABullet::ABullet()
{
	bReplicates = true;
}

void ABullet::OnPoolActivate()
{
}

void ABullet::OnPoolDeactivate()
{
}
```

이렇게 만든 액터에 대한 파생 블루프린트를 만든 후 테이블에서 선택하면 객체를 미리 생성할 수 있게된다.

![](assets\img\post\2025-09-05-Subsystem을 활용한 ObjectPool_2_001.png)
_생성된 객체들_

그런데 한 가지 문제가 생겼다. 분명 풀에 들어가는 객체의 활성화 로직은 거의 같을 것이다. 즉, 객체들은 모두 같은 작업을 수행하는 셈인데, 객체 마다의 구현부에서 똑같은 코드를 일일이 작성하는 것은 굉장히 비효율적이다. 그렇기 때문에 반복 작업을 줄이고자 액터 컴포넌트를 추가하기로 했다.

## PoolableComponent
`PoolableComponent`는 객체 활성화의 실질적인 로직을 담당하는 컴포넌트이다. 다시 말해 외부에서 활성화 명령이 들어오면 실제 처리는 컴포넌트가 담당하는 것이다.

```c++
// PoolableComponent.h
UCLASS( ClassGroup=(Custom), meta=(BlueprintSpawnableComponent) )
class OBJECTPOOL_API UPoolableComponent : public UActorComponent
{
	GENERATED_BODY()

public:
	UPoolableComponent();
	
	void OnActivate();
	void OnDeactivate();

	bool GetIsActivate() const { return bIsActivate; }

private:
	bool bIsActivate = false;
};

// PoolableComponent.cpp
UPoolableComponent::UPoolableComponent()
{
	SetIsReplicatedByDefault(true);
}

void UPoolableComponent::OnActivate()
{
	AActor* Owner = GetOwner();
	if (IsValid(Owner) == false)
		return;

	bIsActivate = true;

	Owner->SetActorTickEnabled(true);
	Owner->SetActorHiddenInGame(false);
	Owner->SetActorEnableCollision(true);
}

void UPoolableComponent::OnDeactivate()
{
	AActor* Owner = GetOwner();
	if (IsValid(Owner) == false)
		return;

	bIsActivate = false;

	Owner->SetActorTickEnabled(false);
	Owner->SetActorHiddenInGame(true);
	Owner->SetActorEnableCollision(false);
}
```

`PoolableComponent`는 실제 객체의 활성화 / 비활성화 기능을 처리한다. 해당 컴포넌트를 가진 액터를 숨기거나 보여주고, 콜리전을 켜거나 끄게 된다.

그리고 해당 컴포넌트를 액터에 추가하고, 인터페이스 함수에 컴포넌트 함수를 호출해주면 된다.

``` c++
// ABullet.h
UCLASS()
class OBJECTPOOL_API ABullet : public AActor, public IPoolableInterface
{
	// ...
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly)
	TObjectPtr<UPoolableComponent> PoolableComponent = nullptr;
};

// ABullet.cpp
ABullet::ABullet()
{
	// ...
	PoolableComponent = CreateDefaultSubobject<UPoolableComponent>(TEXT("PoolableComponent"));
}

void ABullet::OnPoolActivate()
{
	if (IsValid(PoolableComponent))
		PoolableComponent->OnActivate();
}

void ABullet::OnPoolDeactivate()
{
	if (IsValid(PoolableComponent))
		PoolableComponent->OnDeactivate();
}
```

이렇게 되면 다른 객체라도 기본적인 동작은 동일하게 수행된다.

## 인터페이스를 통한 객체 초기화
초기화의 최종 단계이다. 인터페이스를 통해 생성한 객체를 비활성화시키는 로직을 추가한다.

```c++
// UObjectPoolSubsystem.cpp
void UObjectPoolSubsystem::InitPool(const UDataTable* InDataTable)
{
	// ...

	for (int32 Index = 0; Index < Data->PoolSize; ++Index)
	{
		// ...

		IPoolableInterface* Poolable = Cast<IPoolableInterface>(SpawnedActor);
		if (Poolable)
			Poolable->OnPoolDeactivate();
	}
}
```

이제 초기화 단계는 모두 끝났다. 다음 포스팅에서는 비활성화된 객체를 사용하는 방법에 대해 작성하겠다.

## 참고
{% linkpreview "https://github.com/GunnHB/ObjectPool" %}