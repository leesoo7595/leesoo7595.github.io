---
layout: post
title: "[OS] CPU Scheduling Algorithm (3)"
date: 2019-12-03
tags: [OS]
comments: true
---

# CPU Scheduling Algorithm

## Multilevel Queue Scheduling

* Process groups
    - System processes (Priority)
    - Interactive processes (RR)
    - Interactive editing processes
    - Batch processes
    - Student processes (FCFS)

    각각의 프로세스 종류에 따른 Queue를 두는 것을 말한다. 기준은 우선순위를 기준으로 배분

* Single ready queue -> Several separate queues
    - 각각의 Queue에 절대적 우선순위 존재
    - 또는 CPU time을 각 Queue에 자동배분
    - 각 Queue는 독립된 scheduling 정책을 따른다.

## Multilevel Feedback Queue Scheduling

* 복수 개의 Queue
* 다른 Queue로의 점진적 이동
    - 모든 프로세스는 하나의 입구로 진입
    - 너무 많은 CPU time 사용시 다른 Queue로
    - 기아 상태 우려 시 우선순위 높은 Queue로 

운영체제는 다양한 종류의 큐를 두고, 다양한 CPU 스케쥴링을 가지고 상황에 맞게 다방면으로 사용한다.

# 프로세스의 생성과 종료

## Process Creation

프로세스는 프로세스에 의해 만들어진다.

* 부모 프로세스
* 자식 프로세스

OS가 만들어내는 첫 번째 프로세스 : init

첫 번째 프로세스가 다른 프로세스를 만들어낸다. (트리형 구조: 프로세스 트리)

프로세스 identify (PID) : 첫 번째 프로세스는 0이라는 번호가 붙여진다. 이 번호는 절대 중복될 수 없다.
    - integer number

프로세스 생성
    - fork() system call - 부모 프로세스 복사
    - exec() 실행파일을 메모리로 가져오기

프로세스 종료
    - exit()

* Ubuntu Linux 프로세스 나열

`ps` 명령어 : process state

## Thread

프로그램 내부의 흐름, 맥이다.

항상 하나의 프로그램은 하나 이상의 스레드가 돌아갈 수 있다. 하나의 프로그램에 여러 스레드가 돌아는 경우를 다중 스레드라고 한다.

### Multi-Threads

* 한 프로그램에 두 개 이상의 스레드가 돌아가는 경우
* 스레드가 빠른 시간 간격으로 스위칭 된다. => 여러 스레드가 동시에 실행되는 것처럼 보인다.
* concurrent / simultaneous
* ex) 웹브라우저 - 화면 출력 스레드 + 데이터 읽기 스레드
* ex) 워드프로세서 - 화면 출력 스레드 + 키보드 입력 스레드 + 철자 / 문법 올류 확인 스레드
* 요즘 운영체제는 다중 스레드를 지원한다.
* 특정 프로세스엔 기본적으로 다중 스레드가 존재한다. => 스레드가 빠른 시간 간격으로 스위칭된다.

컨텍스트 스위칭되는 단위가 요즘은 스레드 단위로 실행된다.

#### 스레드 구조

* 프로세스의 메모리를 공유한다.(프로세스 안에 다중 스레드 존재)
* 스택은 공유하지 않는다. => PC, SP, register도 마찬가지
