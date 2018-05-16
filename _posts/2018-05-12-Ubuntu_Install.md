---
layout: post
title: "Ubuntu Install 18.04(듀얼부팅)"
date: 2018-05-10 00:00:00
img:
tags: [UBUNTU]
---
> 본 문서는 패스트캠퍼스 'Web-Programming School' 수업 자료를 바탕으로 작성되었습니다.

---

### Ubuntu 18.04 듀얼 부팅 설치하기
처음 프로그래밍을 배울 때 편하디 편한(?) 윈도우를 버리고 바로 리눅스로 갈아타기가 정말 힘들었다. 그래서 듀얼 부팅을 통해 수업시간에는 리눅스를 사용하고 그 외에는 윈도우를 사용했었다. 물론 수업이 끝난 이후엔 윈도우를 쓰는 일이 극히 드물다보니 현재 리눅스만 사용한다. (그러나 PPT 그외 문서 작업을 할 때는 윈도우가 필요하다ㅜㅜ)

#### 1. Ubuntu 18.04 LTS 다운로드
[ubuntu 18.04 공식 사이트](https://www.ubuntu.com/download/desktop)
<BR>
우선 우분투를 설치하기 위해선 공식사이트에서 `.ios`파일을 다운로드 받아야 한다. (설치에 필요한 포맷된 USB 경로에 다운로드 받는 것을 추천!)

#### 2. Ubuntu 18.04 USB 만들기
[우분투 리눅스 설치용 USB 만드는 법](http://jimnong.tistory.com/674?category=575588)<BR>
직접 정리하기엔 이미 너무 잘 정리된 블로그들이 많기에 참고했던 블로그 주소를 공유한다.

#### 3. Ubuntu 18.04 설치
[우분투 설치 참고 블로그](http://jimnong.tistory.com/676)<BR>
위에 분 블로그를 통해서 듀얼 부팅으로 설치를 했으나 조금 다르게 한 부분만 정리를 하려고 한다.

- **부팅순서에 어떻게 들어가나요?** 라고 묻는 분들이 계셨었는데 노트북 제조사마다 다르지만 왠만해선 `F2`를 부팅시에 빠르게 누르면 부트메뉴에 들어갈 수 있다.
- 키보드배치에서 `한국어`를 선택하는 경우 **한국어/한국어-한국어(101/104키 호환)**이 있는데 `한국어` 옵션을 선택하자
- 파티션 할당단계에서 swap영역을 구분하지 않고 바로 루트 파티션을 생성하고 설치를 진행했다.
- 설치가 완료된 후 `지금 다시 시작`을 누룬 후, F2로 부트 모드를 다시 들어간다. 그리고 나서 부트 순서를 `Ubuntu - Windows`이런 순서로 바꿔준 후 usb를 제거하자.

----

### 기본 셋팅하기
#### **한글 키보드가 제대로 작동되지 않는 경우**
<img src = "{{ site.url }}/assets/post_img/setting_korean.png">
우분투 18.04는 기본적으로 `IBus`로 설정되어있고 한국어를 선택하면 기본적으로 입력소스에 두개의 한국어 옵션이 들어가있다.여기서 `한국어(Hangul)` 옵션만 남겨두고 삭제하자.

#### **마우스 패드 오른쪽 버튼 클릭이 되지 않는 경우**
<img src = "{{ site.url }}/assets/post_img/setting_mouse.png">
```console
>> sudo apt install gnome-tweaks
```
위 명령어로 **tweaks**(한국어설정의 경우 기능 개선)를 설치했다면 앱으로 들어가자
키보드와 마우스 설정에서 *마우스 클릭 흉내*에서 영역을 선택하고 컴퓨터를 다시 재부팅하면 완료<br>
[참고 블로그(영문)](https://itsfoss.com/fix-right-click-touchpad-ubuntu/)

#### **git 설치**
```console
>> sudo apt-get update
>> sudo apt-get upgrade
>> sudo apt-get install git
```
#### [**pyenv/python 설치**](https://kahee.github.io/2018/01/18/pyenv,-virtualenv,-iPython%EC%84%A4%EC%B9%98_%EB%B0%8F_%EC%84%A4%EC%A0%95.html)
#### [**zshell 설치**](https://kahee.github.io/2018/01/17/Zsh_and_PlugIn_install.html)

----

### 그 외 프로그램
- 18.04로 업데이트가 되면서 **Ubuntu Software**에서 `Slack`과 `Atom`을 설치 할 수 있다. 그러나 이 경로로 다운 받는 경우 최신버전이 아니거나 한글 입력이 안된다는 문제가 발생한다. 그렇기 때문에 공식 사이트에서 리눅스용 프로그램을 설치 받자.
