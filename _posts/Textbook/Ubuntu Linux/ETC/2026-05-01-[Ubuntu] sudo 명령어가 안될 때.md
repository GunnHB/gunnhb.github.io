---
title: "[Ubuntu] sudo 명령어가 안될 때"
description: sudo 명령이 안될 때 해결법
date: 2026-05-01 00:00:00 +09:00
categories: [Linux]
tags:
  [
    ComputerScience,
    Linux
  ]
---

![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu.png){: width="350" height="250" }

## 문제
관리자 권한을 사용하고자 sudo 명령 입력 시 다음과 같은 에러가 발생하는 경우가 있다.

```bash
(username) is not in the sudoers file.
```

이는 관리자 권한이 없기 때문인데, `sudoers` 파일을 약간 수정하면 된다.

## 해결
우선 `su` 명령을 통해 root 계정으로 변경한다.

```bash
su
```

그런 다음 `/etc/sudoers` 파일을 수정하면 되는데, `sudoers` 파일은 **읽기 전용**이기 때문에 `visudo` 명령으로 파일을 수정해야 한다.

```bash
visudo /etc/sudoers
```

파일을 확인하면 다음과 같은 라인이 있다.

```bash
# User privilege specification
root	ALL=(ALL:ALL) ALL
```

바로 아랫 줄에 다음과 같이 추가하면 된다.

```bash
(username)	ALL=(ALL:ALL) ALL
```

> (username) 자리에는 사용자에 맞는 유저를 추가하면 된다.
{: .prompt-tip}