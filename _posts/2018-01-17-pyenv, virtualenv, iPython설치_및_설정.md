---
layout: post
title: "pyenv, virtualenv, iPython 설치 및 설정"
date: 2018-01-18 09:00:00
img:  # Add image post (optional)
tags: [pyenv]
---
>Web-Programming School 1월 18일 수업 <br>
>본 문서는 패스트캠퍼스 'Web-Programming School' 수업 자료를 바탕으로 작성되었습니다.

**Ubuntu 16.04 환경**

----------------
### pyenv
**pyenv** 는 프로젝트별로 파이썬 버전을 관리 할 수 있도록 도와주는 라이브러리<br>

### virtualenv
**virtualenv** 는 파이썬 개발환경을 프로젝트별로 분리해서 관리할 수 있게 해주는 라이브러리

### pyenv-virtualenv
pyenv-virtualenv은 virtualenv의 pyenv 확장 플러그인

### pyenv와 virtualenv 차이
*pyenv 는 파이썬 버전 관리, virtualenv는 파이썬 패키지 설치 환경 관리*

------

### pyenv 설치 및 설정

```
# 설치
curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash
# 설정
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

설정 후, 터미널을 재시작 하거나 `source ~/.zshrc` 또는 `source ~/.zsh_profile`을 실행
[참고 사이트](https://github.com/yyuu/pyenv-installer)

----------

### 파이썬 설치 전 필요 패키지 설치

```console
$sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \ xz-utils tk-dev

```

[참고페이지](https://github.com/pyenv/pyenv/wiki/Common-build-problems)

---------

### pyenv를 이용하여 python 설치
```
$pyenv install 3.6.4
$pyenv global 3.6.4
$pyenv --version
$python --version
#파이썬 버전이 설치버전과 다르면 터미널 끄고 다시 켜서 확인

$pip list
# 명령어 실행 후
DEPRECATION: The default format will switch to columns in the future.
You can use --format=(legacy|columns) (or define a format=(legacy|columns)
in your pip.conf under the [list] section) to disable this warning.
**pip (9.0.1)**
**setuptools (28.8.0)**
```
3.6.4 버전 설치 시, 두개의 패키지만 있으면 됨

------------------

### 가상환경 생성
```
$pyenv virtualenv [python버전] [가상환경이름]
$pyenv local [가상환경이름]
```
터미널이 *(가상환경이름) ➜  python*  이런식으로 바뀜

-----

### jupyter 사용
파이썬 IDLE를 사용했다가 이번 교육을 통해서 주피터를 사용하게 되었다.
코드를 정리하기 편하다는 장점
아직까진 jupyter의 많은 기능들을 알지 못했다.
```
$pip install ipython jupyter
$jupyter notebook
```
설정 디렉토리의 가상환경에만 **jupyter** 설치 됨

`alt + enter` : 새로운 명령창 <br>
`ctrl + enter` : 명령어 실행
