---
layout: post
title: "[OS] History of Operating System"
date: 2019-11-23
tags: [OS]
comments: true
---

# 운영체제 역사 (컴퓨터)

## 1940년대 후반 ~

2차 대전 중에 만들어짐!

* 하드웨어 기술 발전 -> 운영체제 기술도 함께 발전
* 일반인들은 컴퓨터를 사용하지 못하였다.
* 컴퓨터를 운영하는 Operator가 있었다.

1. 종이에 프로그램을 짬 
2. 종이의 프로그램에 맞춰서 구멍을 뚫는다
3. 그 구멍 위치에 따라 컴퓨터가 읽어서 메모리에 적재시킴
4. 컴파일러에 올림
5. 번역하는 기계어가(컴파일러) 실행됨
6. 결과는 프린터에 찍힌다 (모니터가 없었다)

**No Operating System**

### 최초의 OS : Batch Processing System

Batch: 꾸러미 -> 일괄처리한다.

* 옛날에는 Operator가 다 했던 일련의 과정들
* 이런 일련의 과정들을 메모리에 저장하여 처음부터 하게 하도록 하자
* 이런 일들을 해주는 애들 -> `Resident monitor` 
* 프로그램을 실행시킬 수 있도록 처리하는 일련의 과정들을 처리하는 애들!
* 최조의 운영체제!

### Multiprogramming System (다중프로그래밍)

하드웨어의 발전이 이루어지면서 운영체제에도 커다란 변화가 오게되었다.

#### 메모리

* OS program
* user program

user program에는 하나의 프로그램만 올라갔다.

여기서 하나의 프로그램이 돌아갈 때,

- CPU(Processor)를 사용한다. (연산장치)
- IO(입출력장치)를 사용한다. (print 함수 등)

CPU와 IO가 반복되어 돌아가면서 실행되기 때문에, CPU가 멈추고 IO가 돌아갈 때에는 CPU가 할 일이 없는 것이다... CPU가 놀게됨 (CPU idle)

이를 해결하려고 메모리에 user program을 여러 개 돌리기로 한다.

ex) user program 1이 CPU에서 실행되다가 IO를 하러 내려가면, 그 다음 user program 2를 실행시키도록 한다. -> CPU는 계속 일한다!

그래서 메모리 위에 여러 프로그램을 올린 것을 다중프로그램(`multiprogramming system`)이라한다. 

* 메모리 관리, CPU 스케쥴링, 보호 등 고려할 문제가 생긴다.

## 1960년대 후반 ~

* 키보드, 모니터가 생긴다.
* 인터렉티브(컴퓨터와 대화하는 것)가 가능해짐

대화하는 컴퓨터를 지원하기 위해, 커다란 본체와 그와 연결된 여러 대의 모니터와 키보드(단말기)가 생겨 사용하였다.

본체 안에는 `OS`와 메모리가 있고, 메모리 안에는 여러 `user program`들이 있다.

### Time Sharing System (시공간 공유)

똑같은 짧은 시간동안 user program을 계속 바꾸면서 실행한다.(강제전환)

CPU는 빠르기 때문에 혼자서 사용하는 것처럼 느껴진다.

- 하나의 컴퓨터를 여러 명이 사용하게 되어 users들끼리 통신이 가능해진다.
- 여러 프로그램이 동시에 있어서 순서를 정할 필요가 있다. 이것을 동기화라고 한다.
- 한계 : 여러 사용자가 생기면 메모리 부족 현상이 발생한다.

Time Sharing System에 기반하여 생김

**Unix**가 생김! -> **Linux**

#### 가상 메모리 (virtual memory)

하드디스크의 일부를 메인 메모리처럼 사용한다. 메인 메모리가 부족할 때 사용할 수 있다.

`Time Sharing System`은 **단일 CPU를 사용한 가장 최신 컴퓨터이다**

## OS 기술 변천사

* 컴퓨터 규모별 분류

- 1980년 대까지 : SuperComputer > MainFrame > Mini > Micro
- 지금 : SuperComputer > Server > WorkStation > PC > HandHeld > Embedded

* 고성능 컴퓨터의 OS 기술이 Handheld/Embedded까지 들어가있다.
    - Batch Processing
    - MultiProgramming
    - Time Sharing

* 고등 컴퓨터 구조