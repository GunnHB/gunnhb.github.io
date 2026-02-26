---
title: "[ELB] Gateway Load Balancer"
description: GWLB 특징
date: 2026-02-26 00:00:00 +09:00
categories: [Amazon, ELB]
tags:
  [
    Amazon,
    Cloud Computing,
    ELB,
  ]
---

![](/assets/img/post/Amazon%20Cloud/AWS.png){: width="500" height="350" }

## Gateway Load Balancer
**Gateway Load Balancer**<sup>GWLB</sup>는 OSI 3계층에서 작동하며 주로 타사의 보안 솔루션을 클라우드에 쉽게 배포하고 가용성을 유지하며 유연하게 확장하기 위해 만들어진 특수 목적의 로드 밸런서이다.

GWLB의 가장 큰 특징은 트래픽의 출발지와 목적지 사이에 완전히 ***투명하게 동작***한다는 점이다.

- L3 라우팅
  - 트래픽의 내용이나 포트 번호를 떠나 라우팅 테이블을 통해 들어오는 전체 IP 패킷을 가로채어 보안 검사 장비로 보낸다.

- 원본 유지
  - 클라이언트와 서버는 중간에 GWLB가 개입해서 패킷을 검사한다는 사실을 알지 못한다. 
  - 네트워크 구조나 IP 주소 체계를 복잡하게 변경할 필요 없이 기존 아키텍쳐 중간에 검사 구간만 끼워넣을 수 있다.

### GENEVE 캡슐화
GWLB는 들어온 트래픽을 대상 그룹으로 라우팅할 때 **GENEVE**<sup>Generic Network Virtualiztion Encapsulation</sup> 프로토콜을 사용한다. (UDP 포트 6081)

원본 패킷을 전혀 변형하지 않고 그대로 둔 채 겉에 새로운 헤더를 씌워 보안 장비로 보낸다. 보안 장비가 안전 검사를 마치고 안전하다 판단해 패킷을 돌려보내면 GWLB는 캡슐을 벗기고 원래 가려던 목적지로 트래픽을 안전하게 전달한다.

### GWLB의 특수한 대상 그룹
웹 서버나 애플리케이션 서버로 트래픽을 보내는 ALB나 NLB와 달리 GWLB는 ***보안 및 네트워크 분석 장비***로 트래픽을 보낸다.

- 서드파티 가상 얼라이언스
  - Palo Alto, Fortinet, Check Point 등 외부 전문 보안 업체의 차세대 방화벽

- 침입 탐지 및 방지 시스템
  - 네트워크를 통과하는 패킷의 악의적인 활동이나 정책 위반을 모니터링하고 실시간으로 차단하는 시스템

- 패킷 심층 검사<sup>Deep Packet Inspection</sup>
  - 패킷의 내부 구조까지 뜯어보며 사내 보안 정책을 강제하는 장비