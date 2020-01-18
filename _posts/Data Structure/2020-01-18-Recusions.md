---
layout: post
title: "Recusions"
date: 2020-01-18
tags: [Data Structure]
comments: true
---

# Recusions

## Repeating Problems

하나의 커다란 문제를 작게 만들어서 해결할 수 있도록 하는 것을 의미한다.

### Divide and Conquer

문제를 쪼개어서 해결해나가는 과정을 의미한다.

어떤 함수를 선언할 때, 내부적으로 더욱 작은 문제로 만들어 동일한 함수를 실행시키는 것이 로직이다.

ex) Russian Doll

* Factorial

`10! = 10 * 9 * 8 * 7 * ...`

Repeating problem을 가지고 있다.

**Fatorial(n) = n * Factorial(n - 1)**

* Greate Common Divisor 최대공약수

`GCD(32, 24) = 8`

**유클리드 알고리즘**

```
GCD(32, 24)         // size reducing
    = GCD(24, 8) 
    = GCD(8, 0)
```

* Commonality 

1. Repeating function calls
2. Reducing parameters
3. the mathematical induction (점화식)

## Recursions

function 속에서 function을 call한다.

* Pseudo Code

```javascript
function a(target) {
    ...
    a(target1) {    // reduce size
        ...
        if (escapeCondition) {
            return a;
        }
    }
}

a();
```

* Fibonacci(n)

```javascript
function fibonacci(n) {
    if (n === 0) return 0;  // 더이상 recursion하지 않고 탈출한다! -> EscapeCondition
    if (n === 1) return 1;
    return (fibonacci(n - 1) + fibonacci(n - 2));
}
```

### Recursions and Stackframe

* Recursions of functions

함수를 재귀적으로 호출한다. 함수를 재귀적으로 호출한다는 것은, `stackframe`에서 함수(items)를 증가시킨다.

`stackframe`이란, function 호출 이력을 stack으로써 저장해놓는 공간이다.

function이 호출되면(invoked) `stackframe`에 push된다. 그리고 function이 return되거나 끝나면 pop된다.

* 어떤 것을 저장해놓는가?

1. 지역변수와 function의 인자들을 저장해놓는다.

지역변수란 function 속에서만 접근이 가능한 변수들을 의미한다. 그리고 function의 인자란 function을 call할 때 들어가야하는 인자값을 의미한다.

## Merge Sort

Recursions을 이용한 Sorting Algorithm이다.

일단 array를 받으면, 함수 내부적으로 그 array 값을 쪼갤 수 없을 때까지 2등분하는 recursive를 한다.

그리고 쪼갤 수 없을 때까지 있으면, 다시 순서를 비교하여 해당 요소를 비교하여 크기에 맞게 다시 차례차례 합치는 과정이 merge sort가 될 것이다.

```javascript
function mergeSort(a) {
    if (a.length === 1) {
        return arr;
    }
    const halfLength = Math.ceil(a.length / 2);
    const leftSide = a.slice(0, halfLength);
    const rightSide = arr;
    
    return merge(mergeSort(leftSide), mergeSort(rightSide));
}

function merge(left, right) {
    const sortedArr = [];
    let leftIdx = 0;
    let rightIdx = 0;

    // left, right의 length가 leftIdx, rightIdx보다 작을 때까지 돈다
    while (leftIdx < left.length && rightIdx < right.length) {
        if (left[leftIdx] < right[rightIdx]) {
            sortedArr.push(left[leftIdx]);
            leftIdx++;
        } else {
            sortedArr.push(right[rightIdx]);
            rightIdx++;
        }
    }

    return [...sortedArr, ...left.slice(leftIndex), ...right.slice(rightIdx)]
}

const arr = [3, 8, 4, 2, 1, 6, 7, 5];
console.log(mergeSort(arr));
```

* function call이 많다.
* 불필요한 시간과 공간을 쓰게된다.
* 해결방법은 -> Dynamic Programming

한번 call하고, call된 결과를 기록하게 되는 것을 Dynamic Programming이라고 한다.
