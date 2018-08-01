---
layout: post
title: "큐(Queue)"
date: 2018-08-01 00:00:00
img:
categories:
- Algorithm/DataStructure
tags: [알고리즘/자료구조]
---
> 파이썬과 함께하는 자료구조의 이해 책 정리

----

## 큐(Queue)
<img src="{{ site.url }}/assets/post_img/queue.png">
- 일상에서 쉽게 접할 수 있는 자료 구조
    - 큐 : 대기열을 의미하며, 번호표를 받아 자신이ㅡ 순서를 기다리는 경우에 사용
- 삽입과 삭제가 양끝에서 각각 수행되는 자료구조로 선입선출(First-In, First-Out, FIFO) 원칙하에 동작한다.
- 큐의 가장 앞의 항목을 가리키는 변수 **front** 와 가장 뒤에 있는 항목을 가리키는 변수 **rear**가 필요하다.
- 큐는 cpu으 태스크 스케줄링, 네트워크 프린터, 실시간 시스템의 인터럽트 처리등 다양한 이벤트 구동방식 등에 사용된다.

----

## 리스트로 구현한 큐
### 수행 시간
- 리스트로 구현한 큐의 `add()`와 `remove()`연산은 각각 **O(1)** 시간이 소요된다. 그러나 리스트 크기 확대 또는 추소의 경우 모든 항목들을 새 리스트로 복사해야 하므로 **O(N)** 시간이 필요하다.

### 코드

```python
def add(item):
    q.append(item)


def remove():
    # 맨 앞의 항목 삭제
    if len(q) != 0:
        item = q.pop(0)
        return item


def print_q():
    print('front ->', end='')
    for i in range(len(q)):
        # :<8 문자열을 왼쪽으로 정렬하고 문자열의 총 자릿수를 8로 맞출 수 있다.
        print('{!s:<8}'.format(q[i]), end='')

    print('<- rear')


q = []
add('apple')
add('orange')
add('cherry')
add('pear')
print_q()
remove()
print_q()
remove()
print_q()
add('grape')
print_q()
```

### 코드 실행

```console
front ->apple   orange  cherry  pear    <- rear
front ->orange  cherry  pear    <- rear
front ->cherry  pear    <- rear
front ->cherry  pear    grape   <- rear

```

## 단순연결리스트로 구현한 큐
<img src="{{ site.url }}/assets/post_img/queue2.jpg">

### 수행시간
- 단순연결리스트로 구현한 큐의 `add()`와 `remove()`연산은 각각 **O(1)** 시간이 걸리는데, 삽입 삭제 연산이 rear와 front로 인하여 다른 노드를 순차적으로 방문하지 않아도 각 연산이 수행되기 때문이다.

### 코드
```python
class Node:
    def __init__(self, item, n):
        self.item = item
        self.next = n


def add(item):
    global size
    global front
    global rear

    new_node = Node(item, None)

    if size == 0:
        front = new_node
    else:
        rear.next = new_node

    rear = new_node
    size += 1


def remove():
    global size
    global front
    global rear

    if size != 0:
        fitem = front.item
        front = front.next
        size -= 1
        if size == 0:
            rear = None

        return fitem


def print_q():
    p = front
    print('front: ', end='')

    while p:

        if p.next != None:
            print(p.item, '-> ', end='')

        else:
            print(p.item, end='')
        p = p.next

    print(': rear')


front = None
rear = None
size = 0

add('apple')
add('orange')
add('cherry')
add('pear')
print_q()
remove()
print_q()
remove()
print_q()
add('grape')
print_q()
```

### 코드 실행
```console
front: apple -> orange -> cherry -> pear: rear
front: orange -> cherry -> pear: rear
front: cherry -> pear: rear
front: cherry -> pear -> grape: rear
```

---

## 데크(Deque)
- 데크는 양쪽에서 삽입과 삭제를 허용하는 자료구조
- 예로 스크롤, 문서 편집기 등의 undo 연산 dp tkdydehlsek.
- 파이썬의 리스트나, 이중연결리스트로 구현 가능하다. 단 데크를 구현하는 경우 단순연결시트보단 이중연결리스트가 더 낫다. 왜냐하면 rear가 가르키는 노드의 이전 노드의 레퍼런스를 알아야 ㅎ삭제가 가능하기 때문이다.
- 수행시간의 경우, 스택과 큐의 각 연산수행시간과 같다.(파이썬 리스트 O(N) | 연결리스트 O(1))

### 파이썬의 데크 사용한 코드

```python
from collections import deque

dq = deque('data')
for elem in dq:
    print(elem.upper(), end='')
print()

# 맨뒤와 맨앞에 항목 삽입
dq.append('r')
dq.appendleft('k')

print(dq)

# 맨뒤와 맨앞 항목 삭제
dq.pop()
dq.popleft()

# 마지막 항목 출력
print(dq[-1])
print('x' in dq)

dq.extend('structure')
dq.extendleft(reversed('python'))
print(dq)
```

### 코드 실행
```console
DATA
deque(['k', 'd', 'a', 't', 'a', 'r'])
a
False
deque(['p', 'y', 't', 'h', 'o', 'n', 'd', 'a', 't', 'a', 's', 't', 'r', 'u', 'c', 't', 'u', 'r', 'e'])
```
