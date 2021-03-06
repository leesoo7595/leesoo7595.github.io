---
layout: post
title: "[OS] Main Memory Allocation"
date: 2019-12-18
tags: [OS]
comments: true
---

# 연속 메모리 할당

* 다중 프로그래밍 환경
* 메모리 단편화(Memory fragmentation)

메모리가 사용되다가, 어떤 프로세스가 사용되고 종료되는 그런 일이 반복되면, 메모리 중간중간 위치에는 비어있는 상태가 됨. 이를 메모리가 쪼개졌다해서 메모리 단편화라고 한다.

메모리가 쪼개져있는 상태가 되면, 특정 쪼개진 위치의 크기가 다 다를 것이기 때문에 새로운 프로그램을 넣으려할 때 쪼개져있는 크기가 완전히 맞지 않아 들어갈 수 없는 경우가 생긴다. 이런 경우를 **외부 단편화**라고 한다. (해당 비어있는 곳을 모두 합하면 메인메모리에 프로그램을 올릴 수 있지만, 중간중간 비어있어서 넣지 못하는 경우이다.)

* first fit

메모리를 처음부터 서칭하여 첫 번째 비어있는 곳에(크기가 맞는 가장 첫번째) 프로그램을 넣는 것을 최초적합(first fit)이라고 한다.

* best fit

메모리를 서칭했을 때, 들어갈 프로그램의 크기와 제일 맞는 곳에 넣는 것을 최적적합(best fit)이라고 한다.

* worst fit

메모리를 서칭했을 때, 들어갈 프로그램의 크기와 제일 안맞는 곳(차이가 큰)에 넣는 것을 최악적합(worst fit)이라고 한다.

### 할당 방식 성능 비교

* 속도 : first fit
* 이용률 : first fit, best fit

**하지만, 이런 경우를 잘 활용하더라도, 외부 단편화로 인한 메모리 낭비는 1/3 수준이 사용 불가 상태가 된다.**

해결방법은?

* compaction : OS가 지켜보다가, 비어있는 곳(holes)들을 다 한 군데로 모으도록 한다.

이는 최적 알고리즘이 쓰인다.(존재X) 이를 최적화시키도록하는 방법이 너무 복잡하고 부담이 높다.

* 다른 방법이 있을까

## Paging 

일정한 크기만큼 프로세스를 쪼개서(page) 메모리의 비어있는 곳에 나누어 넣도록 한다. 그리고 MMU 안에 relocation register를 여러개 두어 해당 프로세스가 쪼개진 여러 흩어진 위치를 가져와서 CPU에게 연속적으로 프로세스를 주는 것처럼 동작하게 하는 것이다. 
그래서 CPU는 프로세스가 연속적으로 되어있다고 착각한다.


프로세스를 쪼개는 것과는 반대로 메모리를 일정 크기로 자른 것을 frame이라고 한다. 반대로 프로세스를 paging한다는 것은 프로세스 자체를 일정크기로 잘라서 메모리의 알맞은 holes에 올리는 것을 의미한다.

* 페이지를 프레임에 할당하도록 한다.

* MMU -> page table이라고 부르기도 한다.

relocation register가 여러 개 존재하게되기 때문에, table 형식으로 되어있다해서 지어진 이름이다.

### 주소 변환

*  논리주소
    - CPU가 내는 주소는 2진수로 표현한다.
    - 프로세스의 일정 크기(페이지)마다 CPU에게 정해지는 논리주소 자리수가 다르다.

