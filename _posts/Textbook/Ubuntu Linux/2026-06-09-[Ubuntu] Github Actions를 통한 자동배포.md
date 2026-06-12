---
title: "[Ubuntu] Github Actions를 통한 자동배포"
description: Github Actions를 이용한 CI/CD
date: 2026-06-09 00:00:10 +09:00
categories: [Linux]
tags:
  [
    ComputerScience,
    Linux
  ]
---

![](/assets/img/post/Textbook/Ubuntu%20Linux/linux.png){: width="500" height="350" }

## 자동 배포
게임이나 웹 프로젝트를 업데이트할 때 코딩한 내용을 기반으로 빌드 후 배포해야 변경사항이 적용된다. 코딩 작업만 해도 머리가 아픈데 빌드와 배포까지 해야하니 수고가 더 드는데, 이를 자동화하는 것이 **CI**<sup>continuous integration</sup>/**CD**<sup>continuous deployment</sup>이다.

## Github Actions를 이용한 CI/CD
Github에 올라간 코드는 특정 서버를 통해 자동으로 테스트와 빌드가 진행된다. 여기서 사용되는 서버는 클라우드 제공 방식인 `Github-Hosted Runner`와 내부 호스팅 방식인 `Self-Hosted Runner`가 있다. 내 환경은 온프레미스<sup>On-Premise</sup>이므로 `Self-Hosted Runner`를 통해 자동화를 구축하려 한다. 이는 내부에서 외부로 접속해 코드를 최신화하여 빌드를 생성하게 된다.

### Self-Hosted Runner 발급
1. 깃허브에 접속하여 상단 메뉴 `Settings`를 선택해 왼쪽 메뉴 `Actions` - `Runners` 선택

    ![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_actions_1.png)

2. 우측 상단의 `New self-hosted runner` 클릭 후 OS는 `Linux`, 아키텍쳐는 `x64`를 선택

    ![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_actions_2.png)
    ![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_actions_3.png)

3. 선택 후 나오는 코드를 띄운 채로 가상머신으로 이동

### 가상머신에 Runner 설치
위 작업을 통해 얻은 명령들을 가상머신에서 실행시켜주면 된다. 나의 경우 호스트 환경에서 가상머신에 원격 접속하여 명령들을 실행했다.

1. 작업 중인 디렉토리로 이동하여 폴더를 생성하고 파일을 다운로드

    ```bash
    # 작업 디렉토리 이동
    cd ~/tbs/
    # actions-runner 디렉토리 생성 후 이동
    mkdir actions-runner && cd actions-runner
    # 압축 파일 다운로드
    curl -o actions-runner-linux-x64-...
    # 압축 해제
    tar xzf ./actions-runner-linux-x64-...
    ```

2. Runner 등록 및 세팅

    ```bash
    # 이름이나 작업 디렉토리를 묻는 프롬프트에서는 바로 엔터키 입력
    ./config.sh --url https://github.com/...
    ```

3. 백그라운드 서비스로 실행

    ```bash
    # 서버가 켜져있는 한 뒤에서 계속 돌아가도록 서비스를 등록
    sudo ./svc.sh install
    sudo ./svc.sh start
    ```

    ![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_actions_4.png)
    _활성화된 서비스_

### YAML 파일 생성
이제 자동 배포를 위한 지시서 파일을 만들어준다.

```bash
# 작업 디렉토리 안에 자동배포를 위한 디렉토리 생성
mkdir -p .github/workflows
# deploy.yml 파일 생성
vi .github/workflows/deploy.yml
```

`deploy.yml` 파일을 다음과 같이 작성한다.

```bash
# 작업의 이름표
name: Deploy to Local VM (Self-Hosted)

# main 브랜치에 커밋됐을 때 작업 시작
on:
  push:
    branches: [ "main" ]

jobs:
  deploy:
    # 내 가상머신에서 직접 실행
    runs-on: self-hosted 

    # 외부 도구 모음
    steps:
      # 수정된 내용을 가져옴 (pull)
    - name: Checkout Code
      uses: actions/checkout@v3

    - name: Restart Docker Compose
      # 핵심 실행
      run: |
        sudo docker compose down
        sudo docker compose up -d --build
```

### 자동화 테스트
이제 자동화를 테스트 해야한다. 지시서에 `main`브랜치에 커밋이 `push`된다면 내용을 실행하도록 했으니 지금까지의 수정 내용을 깃에 올려주면 간단히 확인할 수 있을 것이다.

![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_actions_5.png)

그런데 에러가 떴다. 이는 git에서 **대용량 파일 업로드를 차단**하는 것으로 보안 정책 위반으로 인해 생긴 에러이다. 이미 불필요한 디렉토리를 커밋한 상태이니 이를 삭제하도록 한다.

```bash
git rm -r --cached actions-runner
```

이 명령은 스테이지에 추가된 파일(디렉토리)을 삭제시킨다. `--cached` 옵션으로 작업 공간의 파일은 삭제하지 않고 오로지 **스테이지의 파일들만 삭제**시킨다.

> 스테이지의 파일들은 마지막 커밋을 기준으로 존재한다.
{: .prompt-tip}

그리고 다음 번에도 똑같이 스테이지에 올라가지 않도록 .gitignore 파일에 해당 디렉토리를 추가해준다.

```bash
echo "actions-runner/" >> .gitignore
```

그런 다음 이전 커밋의 내용을 수정한다. 커밋 메시지는 수정하지 않는다.

```bash
git commit --amend --no-edit
```

> 이 명령은 개인 프로젝트라면 사용해도 상관없지만 협업 프로젝트에서는 **절대 사용하면 안된다.**
{: .prompt-tip}

이렇게 수정된 커밋을 다시 원격 저장소에 push 해준다.

![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_actions_6.png)

그런데 또 에러가 떴다. 찾아보니 보안을 위해 `workflow`용 토큰을 또 하나 발급받아야된다고 한다.

![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_actions_7.png)
_repo와 workflow 체크박스를 체크_

이후 다시 push 명령을 실행한 뒤 계정명과 비밀번호(토큰)를 입력해주면 된다. 만약 입력 프롬프트가 나오지 않는다면 이전 캐시를 지우는 명령을 먼저 실행한 다음 push 해주면 된다.

```bash
# 캐시 삭제
git config --global --unset credential.helper
# 원격 저장소에 push
git push -u origin main
```

![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_actions_8.png)

푸시는 됐지만 이번엔 명령 실행에서 에러가 발생했다. 알고보니 `sudo` 명령은 관리자를 대행하는 것이므로 실행 시 비밀번호를 입력해야 한다. 하지만 runner는 비밀번호를 입력할 수 있는 능력이 없을 뿐더러 비밀번호를 알지도 못한다.

그렇기에 runner는 비밀번호를 몰라도 명령을 실행할 수 있어야한다. 즉, **`sudo` 없이 명령을 실행**해야 한다.

이를 위해 `docker` 그룹에 사용자 계정을 등록한다. `docker` 그룹은 `sudo` 없이 `docker`를 관리하는데, 이 그룹에 계정을 등록하여 `runner`가 해당 계정을 통해 docker를 관리할 수 있게 만드는 것이다.

```bash
sudo usermod -aG docker $USER
```

그런 다음 runner 서비스를 재시작해준다.

```bash
cd ~/tbs/actions-runner
sudo ./svc.sh stop
sudo ./svc.sh start
```

그리고 `deploy.yml` 파일 내용 중 `sudo`를 빼준다.

```bash
cd ~/tbs
vi .github/workdlow/deploy.yml
```

```bash
# ..
- name: Restart Docker Compose
      run: |
        docker compose down
        docker compose up -d --build
```

마지막으로 수정된 내용을 다시 커밋, push 해주고 결과를 확인하면 된다.

![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_actions_9.png)
_😨_

이번에는 자동화 이전에 실행했던 docker로 인해 중복 실행되어 실패했다. 이를 위해 수동으로 실행했던 docker를 종료했다.

```bash
docker compose down
```

> 권한 문제로 인해 명령이 실패했다면 앞에 sudo를 붙여야한다.
{: .prompt-tip}

이제 우측 상단의 `Re-run jobs`를 통해 다시 runner를 동작해 결과를 확인했다.

![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu_git_actions_10.png)
_마참내!_

정상적으로 동작하는 것을 확인했다😎