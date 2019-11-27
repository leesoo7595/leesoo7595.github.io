---
layout: post
title: "[OS] 운영체제 서비스"
date: 2019-11-27
tags: [OS]
comments: true
---

## CPU 보호

* 한 사용자가 실수나 고의로 CPU 독점

ex) while (n=1) ...

* 해결법

인터럽트 서비스 루틴에서 Timer를 두어 일정 시간 경과시 강제로 다음 user program에 넘어가도록 한다. (타이머 인터럽트)

인터럽트 > 운영체제 > 다른 프로그램으로 강제 전환

## 운영체제 서비스

* 우리나라 정부가 하는 일과 비슷하다.

컴퓨터에 해당된 자원을 운영체제가 효율적으로 조절하여 애플리케이션에 분배한다. 관리하는 부서가 여러 개 있다.

### Process Management

* 프로세스

메인 메모리에서 실행 중인 프로그램

* 주요기능

- 프로세스의 생성, 소멸
- 프로세스 활동 일지 중지, 활동 재개
- 프로세스간 통신
- 프로세스간 동기화
- 교착상태 처리(데드락 핸들링)

### Main Memory Management(주기억장치 관리)

* 주요기능

- 프로세스에게 메모리 공간 할당 (allocation)
- 메모리의 어느 부분이 어느 프로세스에 할당되었는가 추석 및 감시
- 프로세스 종료 시 메모리 회수 (deallocation)
- 메모리의 효과적 사용
- 가상 메모리 : 물리적 실제 메모리보다 큰 용량을 갖도록

### File Management (파일 관리)

* Track/sector로 구성된 디스크를 파일이라는 논리적 관점으로 볼 수 있도록 한다.

* 주요기능

- 파일 생성과 삭제
- 디렉토리 생성과 삭제
- open, close, read, write, create, delete
- Track/sector - file간의 매핑
- 백업

### Secondary Storage Management (보조기억장치 관리)

* 하드 디스크, 플래시 메모리 등

* 주요기능

- 빈 공간 관리
- 저장 공간 할당
- 디스크 스케쥴링

### I/O Device Management (입출력 장치 관리)

* 주요기능

- 장치 드라이브
- 입출력 장치의 성능향상: buffering, caching, spooling

## 시스템 콜

* System calls

운영체제 서비스를 받기 위한 호출 (요청)

* 주요 시스템 콜

- Process: end(종료), abort(강제 종료), load, execute, create, terminate, get/set, attributes, wait event, signal event
- Memory: allocate, free
- File: create, delete, open, close, read, write, get/set attributes
- Device: request, release, read, write, get/set attributes, attach/detache device
- Information: get/set time, system, data
- Communication: socket, send, receive

* MS-DOS

하나의 파일 만들기!

메모리 어딘가(100)에 파일 이름을 넣는다.

mov CX, 0          // 파일의 어트리뷰트를 넣는다.
mov DX, 100        // 메모리 100번지에 넣는다.
mov AH, 3C         // file을 생성한다.
int 21             // 소프트웨어 인터럽트를 일으킨다.

* Linux

하나의 파일 만들기

mov EAX, 8
mov ECX, 0
mov EBX, name
int 80h