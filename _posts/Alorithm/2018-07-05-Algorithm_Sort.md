---
layout: post
title: "Algorithm - 정렬 "
date: 2018-07-05 00:00:00
img:
categories:
- Algorithm
tags: [알고리즘]  
---

## 정렬


### 선택정렬
선택 정렬(選擇整列, selection sort)은 제자리 정렬 알고리즘의 하나로, 다음과 같은 순서로 이루어진다.
- 주어진 리스트 중에 최솟값을 찾는다.
- 그 값을 맨 앞에 위치한 값과 교체한다(패스(pass)).
- 맨 처음 위치를 뺀 나머지 리스트를 같은 방법으로 교체한다.
- 비교하는 것이 상수 시간에 이루어진다는 가정 아래, n개의 주어진 리스트를 이와 같은 방법으로 정렬하는 데에는 Θ(n2) 만큼의 시간이 걸린다.

```python

def selection(list):
  length = len(list)

  for i in range(length-1):
    min_index = i
    for j in  range(i+1, length):
      if list[min_index] > list[j]:
        min_index = j
      if i != min_index:
        list[min_index], list[i] = list[i], list[min_index]
    return list

```

### 버블정렬
거품 정렬(Bubble sort)은 두 인접한 원소를 검사하여 정렬하는 방법이다. 시간 복잡도가 {\displaystyle O(n^{2})} O(n^{2})로 상당히 느리지만, 코드가 단순하기 때문에 자주 사용된다.

```python
def bubble(list):
  length = len(list)

  for i in range(length,0,-1):
    for j in range(i):
      if list[j]>list[j+1]:
        list[j],list[j+1] = list[j+1], list[j]

  return list
```

### 삽입정렬
삽입 정렬(揷入整列, insertion sort)은 자료 배열의 모든 요소를 앞에서부터 차례대로 이미 정렬된 배열 부분과 비교하여, 자신의 위치를 찾아 삽입함으로써 정렬을 완성하는 알고리즘이다.


```python
def insertion(list):
  length = len(list)

  for i in range(1.length):
    curr_value = list[i]
    index = i

    while index > 0 and list[index-1] > curr_value:
      list[index] = list[index-1]
      index -=1

    list[index] = curr_value

  return list
```

## 재귀함수

### 피보나치 수열

```python

def fibonacci(index):
  if index < 2:
    return index
  return fibonacci(index-1) + fibonacci(index-2)

```

### 피보나치 수열 - for문

```python
def fibonacci_con_recuresive(index):
  if index <2:
    return index
  value = 1
  prev1 = 1
  prev2 = 1

  for i in range(index-2):
    value = prev1+prev2
    prev2 = prev1
    prev1 = value

  return value
```

### 피보나치 수열 -다이나믹 프로그래밍

```python
memory = [0]*101

def dynamic(index):
  if index < 2:
    return index

  if memory[index]:
    return memory[index]

  else:
    memory[index] =  dynamic(index-1) + dynamic(index-2)
    return memory[index]

```
