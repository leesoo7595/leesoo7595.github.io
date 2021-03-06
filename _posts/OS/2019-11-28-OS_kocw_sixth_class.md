---
layout: post
title: "[OS] Process Management"
date: 2019-11-28
tags: [OS]
comments: true
---

# 프로세스 관리

## 프로세스

실행 중에 있는 프로그램을 의미한다.

프로그램과 프로세스의 차이는 하드디스크 내부에 있는 것은 프로그램으로, 동작하지 못한다. 프로세스는 현제 진행되고 있는 프로그램이다.

process = task = job

프로세스는 여러가지 상태를 갖게 된다.

1. 하드디스크 안에 프로그램이 있을 땐, 아무 동작도 하지 못한다.
2. 하드디스크에서 메모리에 올라올 때, `new`라고 실행된다. (초기화를 실행한다.)
3. 초기화를 끝낸 후를 `ready`라고 부른다
4. CPU가 메모리에 올라온 프로그램을 실행할 때,(실제로 실행하는 프로그램 상태) `running`라고 부른다.
5. CPU가 중간에 다른 프로그램 작업으로 넘어갈 때, 하던 일은 `waiting` 상태로 바꾼다.
6. 실행되고 있는 프로그램이 끝났을 때,  `terminated` 상태라고 한다.

## PCB (Process Control Block)

하나의 프로세스에 대해서 한 개의 PCB가 할당된다.

PCB 안에는 프로세스에 대한 모든 정보가 들어있다.

- 프로세스의 상태가 현재 어떤 상태인지
- 프로그램 카운터 (현재는 몇번째인가)
- CPU를 얼마나 사용하고 있는지(조절을 하기 위해)
- Process ID(PID) 기록
- list of open files(이 프로세스가 어떤 파일을 사용하고 있는지)
- 그 외 등등

## Queue

모든 프로그램은 하드디스크 안에 들어있는데, 이 프로그램들을 실행시키려고 메인 메모리에 올릴 때, `Queue`를 거친다. 이를 Job Queue라고 한다. (작업줄)

줄을 서서 기다리다가 `CPU Service`를 받는다. 하나의 프로세스가 어떤 작업을 수행할 때, 항상 Queue를 통해 대기 상태를 가졌다가 수행하게 된다. 하드웨어에서 메인 메모리로 이동하거나, 메인 메모리에서 CPU로 이동할 때 등등.... 하지만, 실행이 끝났을 땐 곧 바로 사라진다.

대기줄에 있는 프로그램들 중에 어떤 기준으로 프로그램을 골라서 올릴 것인지 결정하는 것이 `작업 스케쥴링`이라고 한다.

메모리에 올라와있는 프로그램들이 줄 서서 CPU가 수행해주길 기다리는데, 어떤 프로그램을 먼저 CPU에 올려줄지 정하는 것이 `CPU 스케쥴러`이다.

그 등등....

* 정리

컴퓨터의 OS 안에는 여러 줄(Queue)들이 있다. 그 줄에 서있는 프로그램들을 어떤 기준으로 선택하여 올릴 것인지 결정하는 것을 스케쥴러라고 한다.

- Job Queue
    - Job scheduler
    - Long-term scheduler
    - 메모리에 어느 프로세스를 올릴 것인가, 메모리가 꽉 차있으면 필요없는 스케쥴러 즉, 자주 일어나지 않는다.
- Ready Queue
    - CPU scheduler
    - Short-term scheduler
    - 메인메모리에 프로세스가 있는데, 여러 경우의 수로 프로세스가 다음 프로그램으로 넘어갈 때 쓰이는 것
    - 자주자주 일어나는 일(1초에도 수십, 수백번...)
- Device Queue
    - Device scheduler

## 멀티프로그래밍

메인 메모리에, 하나의 프로세스를 올리는 것이 아닌 여러 개의 프로그램을 올리는 것을 말한다.

* `i/o bound process` : 주로 i/o(입출력) 프로세스만 진행하는 것
* `cpu bound process` : 주로 계산 프로세스만 진행하는 것(일기예보)

잡 스케쥴러가 CPU와 I/O가 적절히 일할 수 있도록 위 두 가지의 프로세스를 적절히 분배한다.

* medium-term scheduler 

`time-sharing system` 대화형 시스템을 의미한다. 시간을 쪼개서 각 프로세스에게 균등하게 주어지는 것을 말한다.

메모리에 올라와 있는 자원이 일을 하고 있지 않을 때, 운영체제가 이를 확인하고 하드디스크에 쫓아내는 것을 `Swap out`이라고 한다. 프로세스를 쫓아낼 때, 해당 하드디스크를 `swap device`라고 한다. 그러다가 쫓겨난 자원이 다시 일을 시작하여 프로세스 위로 올라올 때는 `swap in`이라고 한다. (비어있는 곳에 들어가게 된다.)

OS가 어떤 프로세스를 몰아낼지, `short-term` 보다는 자주 일어나고, `long-term` 보다는 덜 일어나는 것을 `medium-term scheduler`라고 한다.