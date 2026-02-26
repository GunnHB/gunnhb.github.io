---
title: "[ELB] Application Load Balancer"
description: ALB 특징
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

## Application Load Balancer
**Application Load Balancer**<sup>ALB</sup>는 OSI 7계층(응용 계층)에서 작동하며 ***HTTP/HTTPS 트래픽의 라우팅과 로드 밸런싱을 담당**한다.

### 콘텐츠 기반 라우팅<sup>Content-based Routing</sup>
ALB는 패킷의 IP 주소나 포트 번호 뿐만 아니라 HTTP 헤더와 페이로드의 내용을 분석해여 트래픽을 정밀하게 분석한다.

- 경로 기반 라우팅<sup>Path-based Routing</sup>
  - URL의 경로에 따라 서로 다른 대상 그룹으로 트래픽을 전달한다.

- 호스트 기반 라우팅<sup>Host-based Routing</sup>
  - HTTP 요청의 Host 헤더를 기반으로 라우팅한다.

- 기타 라우팅 조건
  - HTTP 헤더, HTTP 메서드, 쿼리 문자열, 소스 IP 주소 등을 기반으로 상세한 규칙을 설정할 수 있다.

### 다양한 대상 그룹<sup>Target Group</sup> 지원
ALB는 단순한 가상 서버 인스턴스 외에도 최신 아키텍쳐 요구사항에 맞춘 다양한 컴퓨팅 리소스를 라우팅 대상으로 지원한다.

- EC2 인스턴스
  - Auto Scaling 그룹과 연동되어 트래픽 증감에 따라 **동적으로 변하는 서버 환경에 대응**한다.

- IP 주소
  - VPC 내부의 리소스뿐만 아니라 **온프레미스 서버**나 타 클라우드의 서버를 IP 기반으로 연결할 수 있다.

- 컨테이너
  - 마이크로서비스 아키텍쳐 환경에서 동적 포트 매핑을 통해 컨테이너화된 애플리케이션으로 트래픽을 분산한다.

- AWS Lambda
  - 서버리스 함수를 직접 호출하여 HTTP 요청을 처리할 수 있도록 지원한다.

### 보안 및 성능 최적화 기능
- SSL/TLS 오프로딩
  - ALB에서 SSL/TLS 인증서를 관리하고 트래픽의 암호화/복호화 작업을 대신 수행한다.
  - 백엔드 서버의 CPU 연산 부하를 크게 줄일 수 있다.

- AWS WAF 연동
  - ALB에 웹 애플리케이션 방화벽을 직접 연결하여 SQL 인젝션, XSS 등의 악의적인 웹 공격을 응용 계층 도달 전에 차단한다.

- 상태 검사
  - 네트워크 연결 여부 뿐만 아니라 HTTP/HTTPS 응답 코드를 기반으로 애플리케이션의 실제 응답 상태를 정밀하게 확인하고 비정상적인 대상으로는 라우팅을 중단한다.