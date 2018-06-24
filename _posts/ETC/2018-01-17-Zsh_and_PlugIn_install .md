---
layout: post
title: "Zsh 및 플러그인 설치"
date: 2018-01-17 09:00:00
tags: [Zsh]
---
# Zsh 및 플러그인 설치
>Web-Programming School 1월 17일 수업<br>
본 문서는 패스트캠퍼스 'Web-Programming School' 수업 자료를 바탕으로 작성되었습니다.

기본으로 제공되는 Bash Shell보다 좀 더 효율적인 zsh <br>
zsh 플러그인 설치로 좀 더 나은 터미널 환경 세팅<br>

**Ubuntu 16.04 환경**

---------

## Zsh 설치
```
sudo apt-get install zsh
curl -L http://install.ohmyz.sh | sh
chsh -s `which zsh`
```

or

```
curl -OL https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh
bash install.sh
```
<br>

-------

## zsh 플러그인 설치
### zsh-autosuggestions 설치
터미널에서 명령어에 색을 입혀주는 플러그인
```
$ git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
$ vim ~/.zshrc
```
`~/.zshrc`에서 **plugins** 부분에  **zsh - autosugeestions** 추가

<br>

### zsh-syntax-highlighting 설치
터니멀에서 명령어 입력 전, 먼저 보여줌<br>
오타가 많이 나는 나에겐 정말 필요한 플러그인<br>
[zsh - autosugeestions 설치 관련 git](https://github.com/zsh-users/zsh-syntax-highlighting/blob/master/INSTALL.md)

```
$git clone https://github.com/zsh-users/zsh-syntax-highlighting.git [ 경로 : ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting]
```

`~/.zshrc`에서 **plugins** 부분에  **zsh - syntax-highlighting** 추가
(plugins 부분에 추가 )

<br>
### alias
자주 쓰는 명령어를 미리 설정해서, 터미널에서 편리하게 사용 가능

```
$vim ~/.zshrc
```
`~/.zshrc`에서 **alias** 부분에 추가

```
alias [사용자설정명령어] = "[원래 명령어]"
alias glo ="git log --oneline --all --graph"
```
