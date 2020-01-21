---
layout: post
title: "Dynamic Programming"
date: 2020-01-18
tags: [Data Structure]
comments: true
---

# Dynamic Programming

Recursion이 불필요한 function call을 막아낼 수 있도록 하는 방법이다. 다른 말로 하면, overlapping sub-instances의 recursion 문제를 해결하는 알고리즘이다.

**sub-instances가 오버래핑된다.**라는 의미는, 쓸데없이 반복되어 실행되는 같은 함수값이 많다는 것을 의미한다.

특정함수의 결과값을 저장해놓고 다시 call될 때, 저장해놓은 값을 가져오는 로직을 의미한다. 이는 Memoization이라고 한다.

## Memoization

recursion이 될 때, 이전에 실행되었던 function의 결과값을 저장해놓고 있다가 재활용되는 경우 바로 해당값을 가져오는 것을 말한다.

큰 값을 구하기 위해 작은 값부터 저장해놓고 올라가는 것을 의미하고, 이는 Bottom-up이라고도 부른다.

반대로 Recursion은 Top-down으로 내려간다고 한다. stackframe에 해당 작은 함수들을 지속적으로 쌓아나가는 의미이다.

### Fibonacco Sequence in DP

```javascript

// memoization을 이용한 fibonacci 구현

function fibonacci(n) {
    const fiboObj = {};
    fiboObj[0] = 0;
    fiboObj[1] = 1;
    for (let i = 0; i < n; i++) {
        fiboObj[i] = fiboObj[i - 1] + fiboObj[i - 2];
    }
    return fiboObj[n];
}
```

Dynamic Programming을 이용하면, for문이 하나만 필요하게 되는 경우가 많다. object를 생성함으로써 속도가 빨라질 수 있게 되기 때문이다.

### Assembly Line Scheduling

생산공정 같은 곳에서 많이 쓰이는 알고리즘이다.

여러 라인에서 각 station 당에 걸리는 시간이 정해져 있다고 하자. 특정 라인 station이 고장 등이 나는 경우를 대비해서 중간 과정에서 해당 과정을 대리수행할 수 있는 통로도 만들어 놓게될 때, 어떤 과정으로 왔다갔다해야 효율적으로 수행하게 되는지가 Dynamic Programming이다.

생산이 시작되면, 첫 번째 station에서 도착할 때까지 걸리는 최단 시간을 계산하고, 그 상태에서 그 다음 station에서 최단 시간을 구하게 될 때, 최단 시간에서 갈 수 있는 방법을 또 한 번 따져서 계산한다. 

재귀적으로 이를 수행하게 될 경우, station 단위로 쪼개서 갈 수록 계산을 줄여서 할 수 있게끔하는데, 재귀적으로 하면 반복되는 실행 함수가 많아지므로 메모이제이션을 해서 반복 실행 함수가 없게 하도록 하는 것이다.
