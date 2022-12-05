# GitHub Multiple Accounts

### Environment

- macOS

## Introduction

협업 혹은 기록을 하다보면 Git이라는 형상관리 프로그램을 사용하게된다.  
그리고 그 저장소인 GitHub을 사용하게 되는데,  
여러 계정을 사용하고 싶을 경우 SSH Key 생성 및 gitconfig 추가적인 설정이 필요하다.

[//]: # (TODO: SSH 공부필요)

## How To Use

### 1. SSH Key 생성

GitHub의 계정마다 서로 접근할 수 있는 Repository가 다르기 때문에 SSH키를 계정 별로 생성하여 접근하도록 한다.  
ssh key 는 `~/.ssh` 디렉토리 상에 저장하게 되며 키 생성은 아래와 같으며 이메일은 `GitHub 계정`을 입력하도록 한다.

```shell
ssh-keygen -t rsa -C "personal@email.com" -f "id_rsa_personal"
ssh-keygen -t rsa -C "work@eamil.com" -f "id_rsa_work" 
```

실행하면 저장 위치와 패스워드를 입력 받으며 공식적으로는 패스워드 설정을 권장하고 있다.  
그러면 공개키(id_rsa_*.pub)와 개인키(id_rsa_*)를 저장 되어 있는 것을 확인할 수 있다.  
그리고 생성한 키를 등록한다.

```shell
ssh-add id_rsa_personal
ssh-add id_rsa_work
```

그 후 ssh config 설정에 추가해주도록 한다.

```shell
# Personal GitHub Account
Host github.com-personal
HostName github.com
IdentityFile ~/.ssh/id_rsa_personal
User git

# Work GitHub Account
Host github.com-work
HostName github.com
IdentityFile ~/.ssh/id_rsa_work
User git
```

### 2. SSH Key 등록

앞서 생성 된 공개키를 GitHub에서 등록 해주면 된다.(`Settings-Access-SSH and GPG keys - New SSH Key`)

```shell
Title: 구분할 수 있는 이름
Key type: Authenication Key
Key: {공개 키}
```

등록 후 아래 테스트를 실행하여 정상적으로 등록되어있는지 확인한다.

```shell
> ssh -T github.com-personal
Hi hyoguoo! You've successfully authenticated, but GitHub does not provide shell access.
```

### 3. gitconfig 변경

commit은 gitconfig에 설정된 email과 name으로 commit이 찍히게 되는데  
이를 디렉토리별로 남겨지는 github 계정을 설정해준다.  
보통 사용자라면 `.gitconfg`가 아래와 같이 되어있는데,

```shell
[user]
  name = personal
  email = personal@email.com
```

새로 두 개의 config를 생성하고 기존 gitconfig는 경로별로 해당 config를 가져오도록 설정한다.

- ~/.gitconfig-personal

```shell
[user]
  name = personal
  email = personal@email.com
```

- ~/.gitconfig-work

```shell
[user]
  name = work
  email = work@email.com
```

- ~/.gitconfig

```shell
[includeIf "gitdir:~/dev/personal-repo/"]
	path = ~/.gitconfig-personal

[includeIf "gitdir:~/dev/work-repo/"]
	path = ~/.gitconfig-work
```

### 4. 설정된 SSH를 이용하여 GitHub Repo에 접근하는 방법

Git Clone 및 Remote Add할 때 HTTPS 방식이 아닌 SSH를 통하여 가져온다.  
현재 레포지토리에서 SSH를 통한 Clone URL을 가져올 때 아래와 같이 생성되는데  
그대로 사용하는 것이 아닌 위에서 설정한 Host로 변경하여 가져오도록 한다.

```shell
git@github.com:hyoguoo/DOCS.git
git@github.com-personal:hyoguoo/DOCS.git # -personal 추가
```