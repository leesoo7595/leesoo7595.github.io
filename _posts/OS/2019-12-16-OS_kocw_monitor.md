---
layout: post
title: "[OS] Monitor"
date: 2019-12-16
tags: [OS]
comments: true
---

## 모니터

* 세마포어 이후 프로세스 동기화 도구
* 세마포어보다 고수준의 개념

### 구조

* 공유자원 + 공유자원 접근함수
* 2개의 큐를 가지고 있다. (배타동기 + 조건동기)
* 공유자원 접근함수에는 최대 1개의 스레드만 진입할 수 있다.

모니터에는 common variable(공통변수 = 공유자원)이 있고, 그 곳에는 공통변수에 접근할 수 있는 함수들이 있다. 큐는 2개가 양쪽에 존재하는데, 앞쪽의 큐는 상호배제 역할을 하는 큐인데, 예를 들어 공통변수 공간에 스레드가 돌고 있으면, (공유자원을 사용하는 프로세스가 있으면) 앞쪽에 있는 큐가 다른 스레드가 접근할 수 없도록 막고, 그 큐 안에서 스레드가 대기하고 있어야하는 것이다. 

공유자원 부분에서는 최대 1개의 함수만(스레드) 들어갈 수 있다. 진입 스레드가 조건동기 큐로 인해서 wait()이 실행된다면, (block)들어왔던 스레드는 조건동기 큐에 블록이된다. 그러면 새 스레드가 진입이 가능하다. 반대로 notify()를 호출하면 조건동기 큐에서 블록되어있던 스레드는 현재 들어가있는 스레드가 나가게되면 다시 들어가게된다.

세마포어에 비해 어려운 것 같아보이지만, 세마포어보다 더 사용하기 수월하다.

### 자바 모니터

자바의 모든 객체는 모니터가 될 수 있다.

배타동기는 공통변수 공간에 접근할 수 있는 스레드는 오직 하나만 가능해서, 그 다음 스레드가 접근할 수 없도록 하는 것이 배타동기이다.

* 배타동기 : synchronized 키워드 사용하여 지정할 수 있다. 공통변수를 업데이트하는 함수가 여러 개 있어서, 한 공통변수가 업데이트되는 함수가 실행되면, 다른 함수는 해당 공통변수를 업데이트하지 못하는 것이다.

* 조건동기 : wait(), notify(), notifyAll() 메소드를 사용한다. 스레드가 해당 공통변수 공간에 들어오고 난 후에도 중간에 wait()을 실행시켜 잠시 블록 시켜놓는 상태를 말한다.(스레드는 프로세스이다!!! 해당 돌아가는 함수를 실행시키는 아이)

1. Mutual Exclusion 상호배타
2. Ordering 순서

모니터를 사용하면,(synchronized) 입금이 먼저 일어나고, 출금이 나중에 일어나면, 디파짓의 끝에는 notify()를 넣어주고, 위드로에는 wait()을 넣어준다. 그렇게되면, 입금이 일어난 후에 출금하는 함수를 깨워주고, 출금하는 함수가 실행된 후에는 wait()을 통해 다시 조건동기 큐에 들어가서 블록이 된다. 그러면 입금 함수가 다시 실행되는 식이다.

3. Producer and Consumer Problem

버퍼가 꽉 찼을 때, Producer는 wait()하고, 버퍼가 비어있으면 Consumer가 wait()한다. 그러다가 Consumer가 버퍼를 사용해서 버퍼가 비어지기 시작하면 Consumer가 Producer를 notify()한다. 반대도 마찬가지이다.

4. Readers Writers Problem
5. Dining Philosopher Problem

