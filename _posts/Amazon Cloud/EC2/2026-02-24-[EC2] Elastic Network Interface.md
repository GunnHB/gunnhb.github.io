---
title: "[EC2] Elastic Network Interface"
description: 인스턴스에 고정 IPv4 할당하기
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

## Elastic Network Interface
**ENI<sup>Elastic Network Interface</sup>**는 **<u>AWS의 가상 랜 카드</u>**이다. EC2 인스턴스의 외부 서버 통신을 위한 소프트웨어이며 인스턴스에 고정적인 IPv4 주소를 할당할 수 있다.

예를 들어 웹 서버 `A`에 장애가 발생했을 때 IP 주소가 서버 본체에 완전히 종속되어 있다면 기존 IP로의 접속이 불가능해진다. 하지만 `A`에 연결해두었던 ***보조 ENI<sup>Secondary ENI</sup>***를 예비 서버 `B`로 옮겨서 연결한다면 사용자는 IP 변경 없이 서비스를 이용할 수 있게 된다.

단, 인스턴스 생성 시 기본으로 할당되는 ***기본 ENI<sup>Primary ENI</sup>***는 분리가 불가능하다.

> 각 인스턴스로의 탈부착이 쉽기 때문에 **탄력적<sup>Elastic</sup>**이라 불린다.
{: .prompt-tip}

## ENI가 가지는 정보
- `Private IP`
  - AWS 내부망에서 서로 통신하기 위한 주소

- `Public IP` / `Elastic IP`
  - 외부 통신을 위한 주소

- `MAC`
  - 네트워크 장비(하드웨어)의 고유 식별 번호

- `Security Group`
  - 트래픽을 구분해서 받기 위한 ENI 전용 방화벽

## 주의사항
특정 AZ의 인스턴스에 연결한 ENI는 다른 AZ의 인스턴스에 연결할 수 없다.
