---
layout: post
title: "원형 연결 리스트(Circular Linked List)"
date: 2018-07-30 00:00:00
img:
categories:
- Algorithm/DataStructure
tags: [알고리즘/자료구조]
---
> 파이썬과 함께하는 자료구조의 이해 책 정리

----
## 연결리스트
### 원형연결리스트
- 마지막 노드를 참조하는 **last**가 단순연결리스트의 **head**역할을 한다.
- 마지막 노드와 첫 노드를 **O(1)** 시간에 방문할 수 있는 장점이 있다.
- 리스트가 empty 가 아니면 어떤 노드도 None을 가지고 있지 않아서 프로그램에서 None 조건을 검사하지 않아도 된다는 장점이 있다.
- 원형연결리스트는 여러 사람이 차롇로 돌아가며 플레이하는 게임을 구현하는데 적합한 자료구조
- 많은 사용자들이 동시에 사용하는 컴퓨터에서 CPU시간을 분할하여 작업들에 할당하는 운영체제에도 사용

### 수행 시간
- t삽입이나 삭제 연산 각각 **O(1)**개의 레퍼런스를 갱신하므로 **O(1)** 시간에 수행
- last로부터 노드들을 순차적으로 탐색하므로 O(N)시간 요소

### 코드

```python
from list import EmptyError


class CList:
    class _Node:
        def __init__(self, item, link):
            self.item = item
            self.next = link

    def __init__(self):
        self.last = None
        self.size = 0

    def no_items(self):
        return self.size

    def is_empty(self):
        return self.size == 0

    def insert(self, item):
        n = self._Node(item, None)

        if self.is_empty():
            # 첫 노드를 삽입하는 경우,
            # 새 노드는 자기 자신을 참조, last가 새 노드 참조
            n.next = n
            self.last = n

        else:
            n.next = self.last.next
            self.last.next = n

        self.size += 1

    def first(self):
        if self.is_empty():
            raise EmptyError('UnderFlow')
        f = self.last.next
        return f.item

    def delete(self):
        if self.is_empty():
            raise EmptyError('UnderFlow')
        x = self.last.next
        if self.size == 1:
            # 노드가 한개인 경우는 last를 None
            self.last = None
        else:
            self.last.next = x.next

        self.size -= 1

        return x.item

    def print_list(self):
        if self.is_empty():
            print("리스트 비어있음")
        else:
            f = self.last.next
            p = f
            while p.next != f:
                # 첫 노드가 다시 방문되면 루프 중단
                print(p.item, '->', end='')
                p = p.next
            print(p.item)


if __name__ == '__main__':
    s = CList()
    s.insert('pear')
    s.insert('cheery')
    s.insert('orange')
    s.insert('apple')
    s.print_list()

    print('s의 길이= ', s.no_items())
    print('s의 첫 항목:', s.first())
    s.delete()
    print('첫 노드 삭제 후: ', end='')
    s.print_list()

    print('s의 길이= ', s.no_items())
    print('s의 첫 항목:', s.first())
    s.delete()
    print('첫 노드 삭제 후: ', end='')
    s.print_list()
```

### 코드 실행
```console
apple ->orange ->cheery ->pear
s의 길이=  4
s의 첫 항목: apple
첫 노드 삭제 후: orange ->cheery ->pear
s의 길이=  3
s의 첫 항목: orange
첫 노드 삭제 후: cheery ->pear
```
