---
layout: post
title: "이중 연결 리스트(Doubly Linked List)"
date: 2018-07-30 00:00:00
img:
categories:
- Algorithm/DataStructure
tags: [알고리즘/자료구조]
---
> 파이썬과 함께하는 자료구조의 이해 책 정리

----
## 연결리스트
### 이중연결리스트
- 각 노드가 두개의 레퍼런스를 가지고 각각 이전 노드와 다음 노드를 가리키는 연결리스트
- 단순연결리스트는 삽입, 삭제를 할때 반드시 이전 노드를 가리키는 레퍼런스를 추가로 알아야 하고, 역방향으 노드들을 탐색 할 수 없다.
- 이중연결리스트는 이러한 단점을 보완했으나, 각 노드마다 1개의 레퍼런스를 추가로 더 저장해야 한다는 단점이 있다.


### 수행시간
- 단순연결리스트의 삽입, 삭제연산보다 복잡하지만 결과적으로 각각 **O(1)**개의 레퍼런스만 갱신하므로 **O(1)**시간에 수행된다.
- 탐색 연사의 경우 단순연결리스트와 마찬가지로 head 또는 tail로부터 순차적으로 탐색해야 하므로 **O(N)** 이다.

### 코드
```python
class DList:
    class Node:
        def __init__(self, item, prev, link):
            self.item = item
            self.prev = prev
            self.next = link

    def __init__(self):
        # 2개의 dummy 노드를 생성, 실제 항목은 저장하지 않음
        self.head = self.Node(None, None, None)
        self.tail = self.Node(None, self.head, None)
        self.head.next = self.tail
        self.size = 0

    def size(self):
        return self.size

    def is_empty(self):
        return self.size == 0

    def insert_before(self, p, item):
        t = p.prev
        n = self.Node(item, t, p)
        p.prev = n
        t.next = n
        self.size += 1

    def insert_after(self, p, item):
        t = p.next
        n = self.Node(item, p, t)
        t.prev = n
        p.next = n
        self.size += 1

    def delete(self, x):
        f = x.prev
        r = x.next
        f.next = r
        r.prev = f
        self.size -= 1
        return x.item

    def print_list(self):
        if self.is_empty():
            print('리스트 비어있음')
        else:
            p = self.head.next
            while p != self.tail:
                if p.next != self.tail:
                    print(p.item, ' <=> ', end='')
                else:
                    print(p.item)

                p = p.next


if __name__ == '__main__':
    s = DList()
    s.insert_after(s.head, 'apple')
    s.insert_before(s.tail, 'orange')
    s.insert_before(s.tail, 'cherry')
    s.insert_after(s.head.next, 'pear')
    s.print_list()

    print('마지막 노드 삭제후 :', end='')
    s.delete(s.tail.prev)
    s.print_list()

    print('첫 노드 삭제 후 :', end='')
    s.delete(s.head.next)
    s.print_list()

    print('첫 노드 삭제 후 :', end='')
    s.delete(s.head.next)
    s.print_list()

    print('첫 노드 삭제 후 :', end='')
    s.delete(s.head.next)
    s.print_list()

```

### 코드 실행

```console
apple  <=> pear  <=> orange  <=> cherry
마지막 노드 삭제후 :apple  <=> pear  <=> orange
첫 노드 삭제 후 :pear  <=> orange
첫 노드 삭제 후 :orange
첫 노드 삭제 후 :리스트 비어있음
```
