---
title: "[EC2] 절전 모드"
description: EC2 인스턴스의 절전 모드
date: 2026-02-24 00:00:00 +09:00
categories: [Amazon, EC2]
tags:
  [
    Amazon,
    Cloud Computing,
    EC2,
  ]
---

![](/assets/img/post/Amazon%20Cloud/AWS.png){: width="500" height="350" }

## 절전 모드
EC2 인스턴스는 노트북의 화면을 덮는 것처럼 **절전 모드<sup>Hibernate</sup>**가 가능하다.

## 동작 방식
- 절전 모드 실행
  - EC2 인스턴스의 메모리<sup>RAM</sup>에 올라가 있던 모든 데이터를 서버의 가상 하드디스크인 ***루트 EBS 볼륨***에 복사

- 서버 중지
  - 복사가 끝나면 서버의 전원을 차단
  - EC2 인스턴스의 실행 요금은 발생하지 않는 대신 ***EBS 스토리지 요금이 발생***

- 다시 시작
  - 서버를 켜면 OS를 처음부터 시작하는 것이 아닌 EBS에 저장해두었던 메모리 상태를 RAM으로 다시 불러 ***빠르게 부팅***

## 주의사항
RAM의 데이터를 하드디스크에 백업하는 방식이기 때문에 루트 EBS 볼륨에 RAM 크기 이상의 여유 공간이 필요하며 ***루트 EBS 볼륨을 반드시 암호화***해야 한다.

또한 절전 모드 기능은 인스턴스를 ***최초로 생성할 때만 활성화***할 수 있으며 생성 이후에는 변경할 수 없다.

## 실습
새로운 인스턴스를 추가할 때 절전 모드를 활성화한다.

![](/assets/img/post/Amazon%20Cloud/EC2/Hibernate/Hibernate_behavior.png)
_절전 모드 활성화_

`Enable`을 선택하면 보안을 위해 EBS 볼륨을 암호화하라는 안내글이 나온다. 이를 위해 `Storage` 영역에서 볼륨에 대한 암호화를 활성화한다.

![](/assets/img/post/Amazon%20Cloud/EC2/Hibernate/Hibernate_storage_settings.png)
_KMS Key는 default로 세팅_

그 후 `Connect`-`EC2 Instance Connect`를 통해 터미널을 띄운다.

![](/assets/img/post/Amazon%20Cloud/EC2/Hibernate/Hibernate_connect.png)
_인스턴스를 선택 후 Connect 클릭_

터미널에서 `uptime` 명령을 통해 서버가 얼마나 오래 실행 중인지 확인한다.

![](/assets/img/post/Amazon%20Cloud/EC2/Hibernate/Hibernate_uptime_1.png)
_서버가 1분째 실행 중_

그런 다음 인스턴스를 절전 모드를 활성화한다.

![](/assets/img/post/Amazon%20Cloud/EC2/Hibernate/Hibernate_active_hibernate.png)
_절전 모드 활성화_

중지된 인스턴스를 다시 시작한 뒤 다시 실행 지속 시간을 확인하면 초기화되지 않았음을 확인할 수 있다. 이는 서버가 완전히 중지된 것이 아닌 ***일시 정지한 상태였음을 의미한다.***

![](/assets/img/post/Amazon%20Cloud/EC2/Hibernate/Hibernate_uptime_2.png)
_서버가 3분째 실행 중_