---
layout: post
title: "[OS] Disk Scheduling"
date: 2020-01-01
tags: [OS]
comments: true
---

# 디스크 스케쥴링

<img width="577" alt="Screen Shot 2020-01-01 at 5 43 55 PM" src="https://user-images.githubusercontent.com/39291812/71639689-5cc45300-2cbe-11ea-9102-f7a9d0efe8b4.png">

위는 디스크 구조!! 헷갈릴 것 같아 넣어놓는다

* 디스크 접근 시간

`Seek time + rotational delay + transfer time`

<img width="571" alt="Screen Shot 2020-01-01 at 5 45 26 PM" src="https://user-images.githubusercontent.com/39291812/71639697-841b2000-2cbe-11ea-9ea8-0687bb7c1f27.png">

**탐색 시간(Seek time)이 가장 오래 걸린다.**

디스크 큐에는 하드디스크 서비스를 받으려는 많은 요청들이 쌓여있다. 이런 요청들을 어떤 방식으로 처리해야 효율적으로 탐색시간을 줄일 수 있을까? 해당 요청들은 디스크 안에 특정 트랙 및 블럭의 위치에 자리잡도록 되어있다.

## FCFS (First-Come First-Served)

<img width="554" alt="Screen Shot 2020-01-01 at 5 46 14 PM" src="https://user-images.githubusercontent.com/39291812/71639700-a9a82980-2cbe-11ea-864c-9c70b2b64b2f.png">

먼저 온 요청을 먼저 처리하도록 하는 것

* 간단하다.
* 공평하다.

디스크 헤더와 큐에 있는 요청 개수 및 위치에 따르는데, 헤당 요청온 파일 위치를 디스크 헤더가 먼저 온 요청부터 돌게 된다.

이렇게 될 경우, 먼저 온 프로세스 기준으로 디스크 헤더가 돌아다니기 때문에 디스크 헤더의 이동 거리가 길어져서 탐색시간은 비효율적으로 나올 수 밖에 없게 된다.

## SSTF (Shortest-Seek-Time-First)

<img width="522" alt="Screen Shot 2020-01-01 at 5 46 55 PM" src="https://user-images.githubusercontent.com/39291812/71639710-bd539000-2cbe-11ea-92ae-d77715c4729b.png">

디스크 헤더가 현재 있는 위치에서 움직일 수 있는 거리가 가장 짧은 파일부터 요청을 처리하도록 한다.

해당 디스크 스케쥴링 기법의 단점은 헤더에서 많이 위치가 떨어져있을 때, 해당 위치가 떨어져 있는 프로세스는 계속 기다리게 된다. 해당 프로세스는 기아 상태(Starvation)로 빠지게 된다.

SSTF는 최적화 기법이 아니다. 제일 효율적인 탐색 시간을 낼 수는 없다.

## SCAN (위에서부터 아래로 SCAN 한다.)

<img width="585" alt="Screen Shot 2020-01-01 at 5 47 42 PM" src="https://user-images.githubusercontent.com/39291812/71639719-03105880-2cbf-11ea-8d26-e093f64564c4.png">

디스크 헤더가 디스크 전체를 들어갔다 나왔다하는 방법이다.

즉, 디스크 헤더를 제일 깊은 곳까지 들어갔다 다시 나오는 방법이다. 디스크 헤더의 방향이 효율적인 탐색 시간을 내는데 중요하게 될 수 있다. 디스크 안에서 해당 파일의 위치를 디스크 헤더가 순차적으로 훑으면서 바로바로 처리하게 된다.

* SCAN은 여러 가지의 SCAN 기법들이 존재한다.

### C-SCAN

평균적으로는 해당 파일이 모든 디스크 안의 위치에서 골고루 존재하는 경우가 보편적일 것이다. 그렇게 되는 경우 디스크 헤더가 현재 있는 위치에서 안쪽 방향으로 갈지, 바깥쪽 방향으로 갈지 판단하는 법은 파일의 요청 개수가 더 적은 위치를 시작하여 해당 위치의 요청을 끝내면 디스크 헤더의 요청 처리 활동을 중단하고 반대 방향으로 이동해서 시작하여 요청을 처리하는 것이다.

요청 처리 순회를 중간에 중단하기 때문에, 속도가 더욱 빨라질 수 있다.

이를 Circular SCAN이라고 한다.

### LOOK

디스크 헤더가 안쪽 끝(0)까지 가는 것이 아니라, 제일 깊이 있는 요청까지만 가고 반대쪽으로 바로 돌아서 반대 방향의 요청을 처리하도록 하는 것이다.

### C-LOOK

<img width="553" alt="Screen Shot 2020-01-01 at 5 48 19 PM" src="https://user-images.githubusercontent.com/39291812/71639717-ee33c500-2cbe-11ea-8a1d-f4b2e801a444.png">

디스크 헤더가 요청의 끝까지만 갔다가, 요청을 받는 것을 중단하고 반대방향에서 다시 요청을 수행하는 것을 의미한다.

### Elevator Algorithm

SCAN 기법은 특정 방향으로 내려가면서 요청을 처리하고, 다시 올라오면서 요청을 처리하는데, 이를 엘레베이터와 비슷해보인다고 해서 엘레베이터 알고리즘이라고 한다.

* SCAN & C-SCAN
* LOOK & C-LOOK
