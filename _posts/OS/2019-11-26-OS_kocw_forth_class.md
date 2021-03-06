---
layout: post
title: "[OS] 이중모드, 하드웨어 보호"
date: 2019-11-26
tags: [OS]
comments: true
---

# 이중모드

한 사람이 여러 개의 프로그램을 동시에 사용하는 환경 또는 한 컴퓨터를 여러 사람이 동시에 사용하는 환경

- 한 사람이 고의/실수로 프로그램 전체에 영향을 줄 수 있다.
- 일반 사람들이 프로그램 전체에 영향을 줄 수 없도록 관리자 모드, 일반 모드 두 가지로 나누어진다.
- `privileged instructions`: 특권 명령
- 관리자 모드 = 시스템 모드 = 모니터 모드 = 특권 모드
- Supervisor, system, monitor, privileged mode
- 해당 명령들을 Operating System만 내릴 수 있고 일반적으로는 명령을 내리지 못하게 한다.

## 이중 모드의 원리

- 시스템 비트를 하나 더 추가한다.
- 시스템 비트 자리가 1 인지 0 인지에 따라서 관리자 모드인지 아닌지 구별한다.
- OS가 작동할 때 시스템 비트가 관리자 모드로 돌아가지만, 일반 사용자가 어떤 프로그램을 실행할 때에는 실행하는 순간에 OS가 시스템 비트를 일반 모드로 다시 바꾼다.
- 하드디스크에 프로그램 관련 데이터를 저장하는 경우, software interrupt가 일어나서 user program이 OS에 부탁을 하게 된다. 그러면 OS가 시스템 모드로 바꾸고, 인터럽트 서비스 루틴이 활성화되고, 해당 데이터를 하드웨어에 저장시켜주는 일을 실행한다. 그리고 다시 일반모드로 바꾸고 (user) 프로그램 실행으로 돌아간다.
- 그런식으로 하나의 프로그램이 실행되고 인터럽트가 일어날때마다 일반 모드와 시스템 모드를 넘나든다.

**모든 CPU는 이중 모드를 지원한다.(모니터를 나타내는 비트가 따로 존재한다)**

1. 일반 모드가 시스템 모드에서 실행할 수 있는 동작을 실행하라고 명령을 내리면
2. CPU는 모니터 비트를 확인해서 시스템 모드가 아닌 것을 확인하고
3. 이 명령은 잘못된 명령이다라고 인지하여 내부 인터럽트가 발생한다. 
4. 그리고 잘못된 명령에 관련된 인터럽트 서비스 루틴을 실행하여  메모리에 없어지도록 강제종료를 하게 한다.

## 하드웨어 보호

### 입출력 장치 보호

1. 사용자의 잘못된 입출력 명령
    - 시스템, 즉 OS만이 입출력 명령을 내릴 수 있도록 한다.
    - 유저 측에서 사용할 때, OS측에 요청을 내린다.
    - OS에서 요청을 받아 시스템 모드로 명령을 수행한다.
    - 요청을 마치고 다시 유저 프로그램으로 돌아온다.
2. 사용자가 입출력 명령을 내리면 CPU가 내부 인터럽트를 일으켜서 강제종료 시킨다.(위 과정)

3. 하드웨어와 어플리케이션 계층 사이에는 OS 계층이 자리잡고 있다. 그레서 어플리케이션에서 하드웨어 관련된 요청을 할 때 OS에게 요청을 내리고 OS가 처리해준다는 것!

### 메모리 보호

메인 메모리에는 OS가 항상 상주하고 있고, 나머지는 user program이 돌고 있다.

ex) 특정 user program이 다른 user program 영역의 메모리에 접근하려 하거나, OS 메모리 영역에 접근하려하는 경우

ex) 운영체제 해킹!

#### 해결방안

CPU에서 메모리로 `메모리 주소`를(`address bus`) 통해 메모리를 읽는다. 현재 CPU가 돌고 있는 user program에 해당된 메모리 주소와 CPU가 요청하는 메모리 주소 범위가 같을 적에만 허용하도록 문지기를 두는 법이 있다.

자신의 범위를 넘어서는 요청을 하게 되면, 문지기 측에서 CPU에 인터럽트가 일어나도록 한다. 그리고 OS 측에서 강제종료 시키도록 한다.(`Segment Violation`)

그 문지기를 `MMU`(`Management Unit`)라고 한다. MMU 안에 있는 베이스 범위는 OS가 정한다.

## 결론

* 일반 유저는 절대 직접적으로 명령을 내릴 수 없다.
* 관리자 모드로 명령을 내릴 수 있다.
* 관리자 모드는 OS가 항상 가지고 있다.
* 일반 유저가 특정 명령을 내릴 때, OS에 요청을 하여 처리하도록 한다.
* 정상적인 접근이 아닐 경우 CPU에서 내부 인터럽트가 발생하여 OS 측에서 강제 종료 시키도록 한다.
* 메모리 범위 접근 제한의 경우, MMU라는 CPU와 메모리 사이의 문지기가 체크한다.