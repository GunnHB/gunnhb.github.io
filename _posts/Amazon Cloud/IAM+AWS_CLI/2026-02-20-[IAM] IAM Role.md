---
title: "[IAM] IAM Role"
description: 
date: 2026-02-20 00:00:30 +09:00
categories: [Amazon, IAM]
tags:
  [
    Amazon,
    Cloud Computing,
    IAM,
  ]
---

![](/assets/img/post/Amazon%20Cloud/AWS.png){: width="500" height="350" }

## IAM Role
**IAM Role**은 임시 출입증과 유사하다. IAM User와 달리 비밀번호나 Access Key를 사용하지 않고도 서비스에 접근할 수 있게 된다.

IAM Role은 다음과 같은 상황에서 사용된다.

- AWS 서비스가 다른 AWS 서비스를 제어할 때
- 다른 AWS 계정에서 접근해야 할 때
- 외부 자격 증명 연동