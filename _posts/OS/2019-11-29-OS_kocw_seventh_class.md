---
layout: post
title: "[OS] CPU Scheduling Algorithm"
date: 2019-11-29
tags: [OS]
comments: true
---

### Context Switching (문맥 전환)

현재 프로세스가 끝나서 다음 프로세스로 넘어갈 때를 말한다.

현재 프로세스의 정보를 PCB에 저장해놔야한다. 그리고 다음 프로세스를 받아올 때 다음 프로세스의 저장된 값을 복원해줘야 다음 프로세스로 넘어가서 CPU가 돌릴 수 있게 된다.

이 일을 OS 내부의 Dispatcher가 해주게된다. 여기서 Overhead가 발생하게 되는데, 이 상황을 `Context Switching Overhead`라고 한다.

## CPU Scheduling

메인메모리 안에 있는 프로세스 중에 어느 프로세스를 선택해서 CPU가 돌아가게 해줄 것인지를 결정하는 방법을 의미한다.

### Preemptive / Non-preemptive

- 선점 / 비선점

#### Preemptive

한 프로세스가 실행중에 있을 경우, 그 프로세스가 끝날 때까지 기다리지 않고 강제 종료시키고 다른 프로세스를 들여오는 것을 의미한다.

ex ) 병원에서 응급환자가 발생했을 때

#### Non-preemptive

한 프로세스가 실행중에 있으면, 그 프로세스가 끝나고 난 후 다른 프로세스를 실행시키는 것을 의미한다.

ex ) 병원에 환자를 진료하는 것, 은행 서비스

### Scheduling criteria

* CPU Utilization (CPU 이용률) : CPU가 노는 일이 없게끔!(%)
* Throughput: 단위 시간당 몇 개의 작업을 끝냈는지 (처리율)
* Turnaround time: 작업이 메인메모리에 들어가고부터 끝내고 나오기까지 걸리는 시간(반환시간)
* Waiting time: CPU service를 받기 위해서 얼마나 기다렸는지(대기시간)
* Response time: 응답시간, 컴퓨터와 대화하는 시간 (interactive system)

### First-Come, First-Served (FCFS)

먼저 온 프로그램을 먼저 실행한다.

* Simple & Fair
* Average Waiting Time: 대기시간 구하기
* Gantt Chart
* Convoy Effect (호위효과): CPU 실행시간을 많이 필요로 하는 프로세스가 먼저 실행하게 되었을 때, 그 뒤의 프로그램들이 기다리는 상황
* Non preemptive scheduling(비선점 스케쥴링)
* 들어온 순서대로 해준다고 해서 최대의 성과가 나는 것은 아니다.
* 화장실 / 은행 창구
