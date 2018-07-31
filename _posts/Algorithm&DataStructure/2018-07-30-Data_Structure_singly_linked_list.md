---
layout: post
title: "단순 연결 리스트(Singly Linked List)"
date: 2018-07-30 00:00:00
img:
categories:
- Algorithm/DataStructure
tags: [알고리즘/자료구조]
---
> 파이썬과 함께하는 자료구조의 이해 책 정리

----

## 연결리스트
### 단순연결리스트
- 동적메모리 할당을 이용해 노드들을 한 방향으로 연결하여 리스트를 구현하는 자료구조
- 삽입, 삭제 시 항목들 이동이 필요없다.
- 항목 탐색시, 첫노드부터 원하는 노드를 찾을 때까지 차례로 방문하는 **순차 탐색** 을 해야한다.
- 스택, 큐 및 해싱의 체이닝에서 응용하여 사용됨

### 수행시간
- `search()`의 경우 첫 노드부터 순차적으로 방문해야 하기때문에 **O(N)**, N은 노드 개수
- 삽입, 삭제 연산의 경우 **O(1)** / 하나씩만 갱신하고 삭제하는 방식이기때문에
- `insert_after()`, `delete_after()`의 경우 특정 노드 p의 정보가 주어지지 않으면 head로부터 p를 찾기 위해 search()를 수행 해야 하기 때문에 **O(N)** 시간 소요


### 코드
```python

class EmptyError(Exception):
    pass

class SList:
    class Node:
        def __init__(self, item, link):
            self.item = item
            self.next = link

    def __init__(self):
        self.head = None
        self.size = 0

    def size(self):
        return self.size

    def is_empty(self):
        return self.size == 0

    def insert_front(self, item):
        if self.is_empty():
            self.head = self.Node(item, None)

        else:

            self.head = self.Node(item, self.head)

        self.size += 1

    def insert_after(self, item, p):
        # p = 오렌지를 선택
        # p.next 오렌지 다음의 아이템을 연결(포도)
        # P가 결과적으로 오렌지니까 사과랑 연결이 됨
        p.next = self.Node(item, p.next)

        self.size += 1

    def delete_front(self):
        """
        첫 노드를 삭제 하는 경우
        :return:
        """
        if self.is_empty():
            raise EmptyError('Underflow')
        else:
            self.head = self.head.next
            self.size -= 1

    def delete_after(self, p):
        """
        p가 가리키는 노드의 다음 노드를 삭제
        :param p:
        :return:
        """
        if self.is_empty():
            raise EmptyError('Underflow')
        t = p.next
        p.next = t.next
        self.size -= 1

    def search(self, target):
        # head부터 순차적으로 검색
        p = self.head

        for k in range(self.size):
            # 탐색 성공
            if target == p.item:
                return k
            p = p.next
        # 탐색 실패
        return None

    def print_list(self):
        p = self.head

        while p:
            if p.next != None:
                print(p.item, '->', end='')
            else:
                print(p.item)

            p = p.next


if __name__ == '__main__':
    s = SList()
    s.insert_front('grape')
    s.insert_front('orange')
    s.insert_front('cherry')
    s.insert_after('apple', s.head.next)
    s.print_list()
    print('cherry는 %d 번째' % s.search('cherry'))
    print('kiwi는 ', s.search('kiwi'))
    # end = 줄바꿈 방지
    # print 문 실행 시 항상 문자열 마지막에 \n 문자가 출력되어 줄바꿈이 일어나게 된다. 이렇게 마지막 문자인 \n을 생략할 수 있는 방법
    print('pear 다음 노드 삭세후:', end='')
    s.delete_after(s.head)
    s.print_list()
    print('첫 노드 삭세 후:', end= '')
    s.delete_front()
    s.print_list()
    print('첫 노드로 망고, 딸기 삽입 후:', end='')
    s.insert_front('mango')
    s.insert_front('strawberry')
    s.print_list()
```

### 코드 실행
```console
cherry ->orange ->apple ->grape
cherry는 0 번째
kiwi는  None
pear 다음 노드 삭세후:cherry ->apple ->grape
첫 노드 삭세 후:apple ->grape
첫 노드로 망고, 딸기 삽입 후:strawberry ->mango ->apple ->grape
```

### 문제
- 2개의 단순연결리스트를 하나의 정렬된 단순연결리스트로 만드는 함수 정의하시오

```python
def add_list(self, list):
    p = self.head
    while p:
        if p.next == None:
            p.next = list.head
            break
        p = p.next
```

- 위 문제를 해결하면서 무한루트를 보게 되었다. 대체 왜그런가에 대해 고민하다가.. 기본적인 python 개념을 놓쳤다는것을 알게되었다.
- [얕은 copy, 깉은 copy 참고 자료](https://wikidocs.net/16038)

```python
import copy
s = SList()
s.insert_front('grape')
s.insert_front('orange')
s.insert_front('cherry')

s.print_list()
# s2 = s
# 얕은 copy
# s2 = copy.copy(s)
# 깊은 copy
# s2 = copy.deepcopy(s)

s2 = SList()
s2.insert_front('키위')
s2.print_list()
```
- 만들어진 리스트를 활용하여 또다른 단순연결리스트를 만들었는데 같은 메모리 주소를 바라보기때문에 무한루프에 빠졌던 것, `copy.deepcopy()`를 이용하거나 새로운 `SList` 클래스 인스턴스를 생성해줘야 한다.
