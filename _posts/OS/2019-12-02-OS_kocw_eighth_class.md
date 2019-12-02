---
layout: post
title: "[OS] CPU Scheduling Algorithm - SJF, Priority, RR"
date: 2019-12-02
tags: [OS]
comments: true
---

# CPU 스케쥴링

## Shortest Job Fist

프로그램이 완료되기까지 수행시간이 가장 짧은 프로세스를 먼저 선택하여 수행한다.

* 대기시간을 줄이는 방법으로는 가장 최적화되어있는 방법이다.
* 현실성이 없다. (해당 프로세스가 CPU 사용 시간을 얼마나 쓸 것인지 알 길이 없다.)
* 해당 방법을 수행하기 위해서 prediction(예측)이 필요하다.
* 예측을 하기 위해서 해야하는 일이 오버헤드임
* Preemptive / NonPreemptive

선점 SJF의 경우, 이후에 늦게 들어온 프로그램이라도 수행시간이 짧을 경우 현재 CPU를 사용하는 프로세스를 큐로 내보내고 수행시간이 짧은 프로그램부터 먼저 CPU를 사용하도록 한다.

## Priority Scheduling

우선순위가 높은 프로그램을 먼저 서비스해준다.

* 우선순위를 정하는 기준
    - 내부적인 요소 : time limit(수행시간이 짧은), I/O & CPU burst(I/O나 CPU 시간을 사용하는 기준), memory requirement(메모리를 차지하는 정도)
    - 외부적인 요소
* Preemptive / Non-preemptive
* 문제점
    - 기아 : 어떤 프로그램은 수행시간이 너무 길어서(우선순위가 너무 낮아서) 자신의 차례가 오지않는 경우
    - 해결책 -> 노화 : 큐에서 오래 기다릴수록 우선순위를 높여주도록 OS가 봐준다.

## Round-Robin

빙빙 돌면서 프로그램을 수행한다. Time Sharing System

* 1초 동안 일정한 횟수로 프로그램을 돌린다.
* Preemptive Service
* time quantum의 크기에 따라 퍼포먼스가 달라진다.
* FCFS = time quantum

