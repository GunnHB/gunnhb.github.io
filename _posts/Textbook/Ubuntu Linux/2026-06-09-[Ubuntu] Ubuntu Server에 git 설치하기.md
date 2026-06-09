---
title: "[Ubuntu] Ubuntu Server에 git 설치하기"
description: git 설치 후 프로젝트 업로드
date: 2026-06-09 00:00:00 +09:00
categories: [Linux]
tags:
  [
    ComputerScience,
    Linux
  ]
---

![](/assets/img/post/Textbook/Ubuntu%20Linux/linux.png){: width="500" height="350" }

## git 설치와 로컬 저장소 업로드
### git 설치하기

```bash
# apt 업데이트
sudo apt update
# git 설치
sudo apt install -y git
# git 설치확인 (버전 확인)
git -v
```

![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_version.png)
_설치된 git_

### 내 정보 입력하기

```bash
git config --global user.name "사용자명"
git config --global user.email "사용자 이메일"
```

![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_account.png)
_명령 입력 후에 아무것도 나오지 않는 것이 정상_

### git 초기화

```bash
# 프로젝트 폴더로 이동
cd ~/tbs
# git 초기화
git init
```

![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_init.png)
_깃 초기화 완료_

### gitignore 추가

```bash
# 불필요한 파일 업로드 방지를 위해 ignore 파일 생성
echo "*/__pycache__/" > .gitignore
```

### 로컬 저장소에 프로젝트 업로드

```bash
# 모든 파일 추가
git add .
# 로컬에 프로젝트 업로드
git commit -m "커밋 메시지"
```

![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_commit.png)
_프로젝트 업로드_

### 로그 확인하기

```bash
# 로그 확인
git log
```

![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_commit_log.png)
_커밋 로그 확인_

## 원격 저장소 연결

> github에 새로운 레파지토리 추가하는 과정은 생략😎
{: .prompt-info}

### 브랜치 변경과 원격 주소 설정

```bash
# 기본 브랜치 이름을 main으로 변경
git branch -M main

# 생성한 레파지토리의 원격 주소로 설정
git remote add origin "원격 주소"
```

![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_remote_url.png)
_HTTPS 주소로 입력_

### Personal Access Token 발급받기

1. 프로필 - `Settings`

    ![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_pat_1.png){: width="200" }

2. 왼쪽 메뉴 맨 아래 `Developer setting` 이동

    ![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_pat_2.png)

3. 왼쪽 메뉴 `Personal access tokens` - `Tokens (classic)` 이동 후 `Generate new token` - `Generate new token (classic)` 선택

    ![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_pat_3.png)

4. `Note` 이름 입력 - `Expiration` 설정 - `repo` 체크박스 선택 후 토큰 생성

    ![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_pat_4.png)

    ![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_pat_5.png)
    _생성된 토큰_

### 호스트에서 가상머신으로 원격 접속하기

> 호스트에서 복사한 내용을 가상머신으로 붙여넣을 수 없어 원격을 통해 토큰을 입력하기 위함😎
{: .prompt-tip}

1. `ssh` 포트 열기

    ```bash
    # 22번 포트 열기
    sudo ufw allow 22/tcp
    ```

2. `openssh-server` 패키지 설치

    ```bash
    # 원격 접속 관리를 위한 패키지 설치
    sudo apt install -y openssh-server
    # 서비스 활성화 확인
    sudo systemctl status ssh
    ```

    ![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_ssh.png)
    _활성화 중인 서비스_

3. 호스트의 파워셸에서 원격 접속

    ```bash
    # 가상머신 IP로 원격 접속
    ssh '우분투계정명'@'가상머신IP'
    ```

### 원격에 커밋 푸시하기

```bash
# 자격 증명 저장 기능 켜기
git config --global credential.helper store
# 커밋 푸시
git push -u origin main
```

> 최초 푸시에서 아이디와 패스워드(토큰)을 한 번만 입력
{: .prompt-tip}

![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_upload.png)
_프로젝트 업로드 완료_