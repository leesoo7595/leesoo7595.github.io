---
layout: post
title: "[OS] macOS에 개발환경 구축하기"
date: 2018-10-06
tags: [OS]
comments: true
---

참고 :
- https://subicura.com/2017/11/22/mac-os-development-environment-setup.html
- https://younlab.github.io/2018-09-11/pyenv-&-pipenv-install
시스템 설정은 패스하고 설치 프로그램만 알려주겠다.
<br>
<br>

## Xcode 설치

`xcode-select --install`

### 확인

```
# gcc test
$ gcc
clang: error: no input files
```

## Homebrew 설치

`/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

### 확인

```
# brew test
$ brew doctor
Your system is ready to brew.
```

## Git 설치

`brew install git git-lfs`

### 설정

```
git config --global user.name "Your Name"
git config --global user.email "you@your-domain.com"
git config --global core.precomposeunicode true
git config --global core.quotepath false
```

## 터미널 설정

### iTerm2 설치

`brew cask install iterm2`


### zsh with oh-my-zsh 설치

`brew install zsh zsh-completions`

`sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`

#### 플러그인 설치

```
# zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# zsh-autosuggestions
git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```

#### `~/.zshrc`에 플러그인 설정 추가하기

```
plugins=(
  git
  zsh-syntax-highlighting
  zsh-autosuggestions
)
```

### 테마 변경

```
brew install nodejs # nodejs가 설치되어 있다면 skip
npm install --global pure-prompt
```

`~/.zshrc`에 다음 설정 추가

```
autoload -U promptinit; promptinit
prompt pure
```

## pyenv 설치

`brew install pyenv`

`~/.zshrc`에 다음 설정 추가

```
export PYENV_ROOT=/usr/local/var/pyenv
if which pyenv > /dev/null; then eval "$(pyenv init -)"; fi
```

### pyenv로 python 설치 전 필요 패키지

`brew install readline xz`

`pyenv install 3.6.x`

## pipenv 설치

`brew install pipenv`

끄읏끄읏
