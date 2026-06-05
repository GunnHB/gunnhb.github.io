---
title: "[LinuxStructure][Ch01] 리눅스 개요"
description: 리눅스와 커널, 프로세스
date: 2026-06-05 00:00:00 +09:00
categories: [Linux]
tags:
  [
    ComputerScience,
    Linux
  ]
---

![](/assets/img/post/Textbook/Ubuntu%20Linux/linux.png){: width="500" height="350" }

## 프로그램과 프로세스
- **프로그램**<sup>program</sup>
  - 컴퓨터에서 동작하는 관련된 명령 및 데이터의 묶음
  - 컴파일러<sup>compiler</sup>형 언어라면 소스 코드를 빌드해 만들어진 **실행 파일**
  - 스크립트<sup>script</sup>형 언어라면 **소스 코드 자체**

- **프로세스**<sup>process</sup>
  - 실행되어 동작 중인 프로그램

### 프로세스 확인하기
- 터미널에서 멈춰있는 프로그램을 5분간 백그라운드로 실행

```bash
sleep 300 &
```

- `sleep`의 위치와 크기를 확인

```bash
ls -l /bin/sleep
```

- 메모리 상에서 돌고 있는 프로세스의 상태와 PID를 확인

```bash
ps -ef | grep sleep
```

- 해당 PID의 상세 정보(신원, 실행 정보, 상태 등)를 파일 형태로 출력

```bash
ls -l /proc/[확인한 PID]
```

![](/assets/img/post/Textbook/Ubuntu%20Linux/linux_structure_ps_list.png)
_프로세스의 속성 리스트_

> **상세 정보가 파일 형태로 나오는 이유?** <br>
> 리눅스에서는 '모든 것은 파일'이라는 철학 아래에 시스템을 쉽게 제어할 수 있도록 설계되었기 때문
{: .prompt-tip}

## 커널
- **커널**<sup>kernel</sup>
  - 하드웨어와 소프트웨어의 간접적 연결을 위한 **프로그램**
  - 특징
    - **동시성 제어**<sup>concurrency control</sup>
      - 여러 프로그램이 동시에(병렬적으로) 저장 장치에 접근해 같은 데이터를 조작한다면 손상이 발생할 수 있음
      - 커널은 명령 실행 순서를 제어하여 이러한 문제를 해결

- **모드**<sup>mode</sup>
  - CPU가 명령을 실행할 때 가지는 권한의 수준

  - **사용자 모드**<sup>user mode</sup>
    - 일반적인 응용 프로그램(프로세스)이 동작하는 기본 상태
    - 하드웨어에 직접 접근할 수 있는 권한이 **없음**
    - **사용자 공간**<sup>userland</sup>에서 프로세스를 실행

  - **커널 모드**<sup>kernel mode</sup>
    - 운영체제의 핵심인 **커널 프로그램**이 실행되는 상태
    - 시스템의 **모든 권한**을 가짐
    - 특권 명령어를 통해 하드웨어 직접 제어 가능