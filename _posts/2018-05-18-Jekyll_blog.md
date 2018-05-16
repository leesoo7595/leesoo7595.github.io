---
layout: post
title: "Jekyll 블로그 만들기"
date: 2018-05-16 12:00:00
img:  # Add image post (optional)
tags: [Jekyll]
---

> 본 문서는 패스트캠퍼스 'Web-Programming School' 수업 자료를 바탕으로 작성되었습니다.
> ubuntu 18.04 version

--------------

## 루비 설치
### 패키지 목록 업데이트
```
$sudo apt-get update
$sudo apt-get upgrade
```
온라인 저장소에서 최신 버전의 패키지들 업로드 -> 실제 설치 해주는 명령어는 `upgrade`

### Ruby에 필요한 의존성 패키지 설치
```
$sudo apt-get install autoconf bison build-essential libssl-dev libyaml-dev libreadline6-dev zlib1g-dev libncurses5-dev libffi-dev libgdbm-dev
```

### Ruby 버전관리 프로그램 설치
`git clone https://github.com/rbenv/rbenv.git ~/.rbenv`
<br>*.[파일명] : 숨긴 폴더*

### rbenv 실행 환경변수 설정(zsh 셸)
```
$echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.zshrc
$echo 'eval "$(rbenv init -)"' >> ~/.zshrc
$source ~/.zshrc
```

### Ruby - build 플러그인 설치
```
$git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
```

설치 후 `$rbenv` 명령어 입력, **install** 과 **uninstall** 생기는 것 확인

### Ruby 설치하기
```
$rbenv install -l
$rbenv install 2.5.1
$ruby -v
$rbenv global 2.5.1
```
설치 가능한 Ruby 버전확인 -> Ruby 2.5.1 버전 설치 -> Ruby 버전 확인 -> 시스템 전체 버전을 설정

---

## Jekyll블로그에 필요한 Gem 설치
### bundler 설치
```console
$gem install bundler
```

### github-pages 설치
```console
$gem install github-pages
```
git하고 연동하는데 필요한 패키지

### jekyll설치
```console
$gem install jekyll
```
----------------

## 로컬에서 Jekyll 블로그 시작하기
### 블로그 생성
```
$mkdir blog
$cd blog
$jekyll new 블로그이름
```
원하는 디렉토리를 만들고 그 곳으로 이동한 다음 `jekyll new 블로그이름` 명령어 실행

### Gemfile 수정 및 적용
```console
$ vim Gemfile

------
# gem "jekyll", "~> 3.7.0"  -> 커맨드 처리
gem "github-pages", group: :jekyll_plugins -> 커맨드 해제
```
저장 한 후 밑에 명령어 실행

```console
$bundle update
$bundle install
```

### 로컬 테스트 서버 실행하기
블로그가 있는 폴더에서 아래 명령을 실행 <br>
```console
$bundle exec jekyll server
```
-------------------

## Github에 블로그 업로드하기
### GIthub 에서 새로운 저장소 생성
**사용자명.github.io** 으로 설정

### 로컬 저장소 생성 및 연결
```
$git init
$git remote add origin [remote저장 주소]
```

### 로컬 저장소 파일 업로드
```
$git diff
$git add -A
$git commit -m 'commit메시지'
$git push origin master
```

`git diff`는  변경 사항이 무엇인지 알려주는 명령어<br>
`commit`하기 전에 확인하기!
그리고 자신의 블로그 주소에 접속하면 **완료!**
*만약 블로그에 들어갔는데, 변경 사항이 적용이 안됬을 경우, 몇 분 기달려보기*

------------

## Jekyll 테마 적용하기
### [Jekyll 테마 고르기](http://jekyllthemes.org/)

### git repository 생성 및 이름 변경
- **사용자명.github.io** 이름 설정하기
- 만약 이전에 블로그가 있다면 포크한 저장소를 `clone`하고 사용자명으로 된 저장소를 `remote add` 하는걸 추천한다. 왜냐하면 테마가 바로 적용되지 않는 경우가 있기 때문!

### _config.yml 수정
`url`과 `baseurl`은 빈칸으로 만들자. 설정이 되어있는 경우 localhost에서 접속 할때 뒤에 추가적인 주소가 필요하다.


-------

## 기타
- config.yml : 블로그 설정파일
- 서버를 하나 켜놓고, md파일을 수정하면서 확인 할것
- hosting.kr : 도메인사서 설정할 수 있는 페이지
- jekyll serve vs bundle exec jekyll : `jekyll serve` 는 개발 목적으로 사이트를 서비스 해주는 것이고, `bundle exec jekyll`은 현재 디렉토리에 있는 gemfile을 통해 Jekyll 루트 폴더를 실행시킨다.

----

## 참고 사이트
- [Che1's Blog](https://nachwon.github.io/)
- [이한영 강사님 블로그](https://lhy.kr/create-jekyll-blog-using-rbenv-and-github-pages)
- [오류 관련해서 참고 블로그](https://swifteyes.blogspot.kr/2016/12/jekyll-github.html)
- [전체적인 흐름 참고 블로그](https://webdesign.tutsplus.com/ko/tutorials/how-to-set-up-a-jekyll-theme--cms-26332)
