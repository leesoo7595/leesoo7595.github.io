---
layout: post
title: "Algorithm - 정렬 "
date: 2018-07-05 00:00:00
img:
categories:
- Algorithm/DataStructure
tags: [알고리즘/자료구조]
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

  for i in range(1,length):
    curr_value = list[i]
    index = i

    while index > 0 and list[index-1] > curr_value:
      list[index] = list[index-1]
      index -=1

    list[index] = curr_value

  return list
```

-------

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

-----

## 퀵정렬
### quick_sort_easy
- 파이썬을 이용한 퀵정렬
- 매개변수로 받은 list를 사용하는 것이 아니라 새로운 list를 생성해서 메모리 문제가 있다.

```python
def quick_sort_pythonic(list):
    # 재귀 함수의 종료 조건을 항상 먼저 쓴다.
    if len(list)<=1:
        return list
    else:
    # 피봇은 맨 앞 숫자 (*임의로 정의)
        pivot = list[0]
    # 피봇보다 작은 숫자들 모음
        less = [i for i in list[1:] if i <= pivot]
    # 피봇보다 큰 숫자들 모임
        greater = [i for i in list[1:] if i > pivot]
    # 재귀호출
        return quick_sort_pythonic(less) + [pivot] + quick_sort_pythonic(greater)

```

### quick_sort_classic

```python
# partition() : 피봇을 정하고, 피봇에 따라 작은 수, 큰수로 분류 후, 피봇의 원래 자리를 리턴
def partition(list, start,end):
#     피봇값 설정
    pivot = list[start]
#     앞에서부터 뒤로 가면서 pivot보다 큰 값을 찾아갈 index
    left = start + 1
#     뒤에서부터 앞으로 오면서 pivot 보다 작은 값을 찾아갈 index
    right = end
#     pivot기준으로 분류가 다 되었는지 확인하는지 flag
#     분류가 어떻게 다 되었는지 아는가? right과 left가 교차할때
    done = False

    while not done :
#         left는 큰 값의 인덱스를 찾는다.
        while left <= right and list[left]<=pivot:
#         작은 값을 찾으면 그다음으로 넘어간다.
            left += 1
        while left <= right and list[right] > pivot:
            right-=1
        if left > right:
            done = True
        else:
#             분류가 안끝났으면  swap
            list[right], list[left]= list[left], list[right]
# 분류 끝, 피봇을 제자리로 !!!!! 피봇은 작은 값들의 인덱스 중에서 가장 큰 값과 바꾸면 된다.
        list[start], list[right] = list[right], list[start]
#     파티션의 마지막 임무 - 피봇 인덱스 넘기기
        return right


# quick_sort_classic()
def quick_sort_classic(list,start,end):
#  재귀함수 종료조건
    if start > end:
        return list
    else:
#       제자리를 찾은 피봇의 인덱스를 partition에게 받는다.
        pivot = partition(list, start, end)
#         피봇보다 작은 것
        quick_sort_classic(list,start, pivot -1)
#         피봇보다 큰 것
        quick_sort_classic(list,pivot+1, end)
#     모든 정렬 마치고 리턴
    return list

# quick_sort_classic(numbaser, 0, len(numbers-1))
# 파티션이 얼마나 둘로 잘 쪼개느나냐가 bast케이스 worst의 경우 한쪽만 움직이는 것 - 완전히 역순이거나, 정렬이되어있거나 n^2
# 시간복잡도 앤로그앤
# 캐시메모리쪽에서 사용하기 때문에 빠르다.
```

----

## 병합정렬(merge sort)
- 실행시간 nlon(n)
- 별도의 리스트를 만들어 사용하기 때문에 공간복잡도 O(n)
- Timsort: 삽입 + 병합 정렬 -> python sort()사용하는 알고리즘
    - 적당히 작은 단위로 나눔 (적당히 = 캐시메모리 안에서 정리 될 수 있는 만큼)
    - 작은 단위의 리스트에서는 삽입정렬을 진행

```python

def merge_sort(list):
  length = len(list)


# divide 상황
  if length > 1:
    # // 소수점 아랫자리는 버리는 연산자
    # 리스트 반으로 자르기
    mid = length//2
    left_list = list[:mid]
    right_list = list[mid:]

    #length == 1이 될때까지 재귀적으로 나눈다.
    merge_sort(left_list)
    merge_osrt(right_list)

    # divide 가 끝난 뒤, 정렬(conquer)
    # i = lest_list에 사용할 인덱스
    # j = right_list에 사용할 인덱스
    # k = list에서 사용할 인덱스
    i = 0
    j = 0
    k = 0

    # 나뉘어진 리스트들의 len
    length_left = len(left_list)
    length_right = len(right_list)

    # 정렬
    while i < length_left and j < length_right:
        if left_list[i] < right_list[j]:
            list[k] = left_list[i]
            i +=1
            k +=1
        else:
            list[k] = right_list[j]
            j +=1
            k +=1

    # 남은 숫자들 정리
    while i < length_left:
        list[k] = left_list[i]
        i +=1
        k +=1
    while j < length_right:
        list[k] = right_list[j]
        j +=1
        k +=1

    return list
```
