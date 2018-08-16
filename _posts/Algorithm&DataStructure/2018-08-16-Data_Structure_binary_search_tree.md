---
layout: post
title: "탐색트리 - 이진탐색트리(Binary Search Tree) "
date: 2018-08-16 00:00:00
img:
categories:
- Algorithm/DataStructure
tags: [알고리즘/자료구조]
---
> 파이썬과 함께하는 자료구조의 이해 책 정리

----

## 이진탐색트리
- 이진탐색 개념을 트리 형태의 구조에 접목시킨 자료구조
- 정렬된 데이터의 중간에 위치한 항목을 기준으로 데이터를 두 부분으로 나누어 가며 특정 항목을 찾는 탐색방법
- 단순연결리스트에서는 각 노드가 다음 노드만을 가리키므로 이진탐색이 쉽지 않다. 그러나 이진탐색을 수행하기 위해 변형한 단순 연결리스트는 중간 노드를 중심으로 좌측 노드들은 이전 노드를 가리키도록 하고 같은 방법으로 각각 좌.우측 노드들에 재구적으로 적용하면 이진탐색트리를 구현할 수 있다.
- 이진탐색트리는 중위순회(왼쪽 - N - 오른쪽) 을 수행하면서 정렬된 출력을 얻는다.
- 이진탐색트리는 가장 기본적인 트리 형의 자료구조이며, 균형 이진탐색트리, B-트리, 다방향 탐색트리에 기반이 되는 자료구조다 또한 데이터베이스 등의 대용량 데이터 저장 기본 개념으로도 화룡한다.
- 이진탐색트리 조건 : 각 노드의 n의 키값이 n의 왼쪽서브트리에 있는 노드들의 키값들보다 크고, n의 오른쪽 키값들보다 작다.

### 수행시간
- 이진탐색트에서 탐색,삽입,삭제 연산은 공통적으로 루트에서 탐색을 시작하여 최악의 경우 이파리까지 내려가고, 삽입과 삭제 연산은 다시 루트까지 거슬러 올라가야 함
- 트리를 한 층내려갈때마다 재귀호출이 발생하고, 한층을 올라갈때는 부모와 자식노드를 연결할때는 각각 **O(1)** 시간밖에 걸리지 않는다. 다라서 **수행시간은 각각 이진탐색 트리의 높이에 비례** 따라서 각 연산의 최악경우 수행시간은 **O(h)**
- N개의 노드가 있는 이진탐색트리의 높이가 가장 낮은 경우 완전이진트리 형태이고 가장 높은 경우는 편향이진트리이다. 

### 코드
```python
class Node:
    def __init__(self, key, value, left=None, right=None):
        self.key = key
        self.value = value
        self.left = left
        self.right = right


class BST:

    def __init__(self):
        self.root = None

    def preorder(self, n):
        # 전위순회
        if n is not None:
            # 맨 먼저 노드 방문
            print(str(n.key), '', end='')
            # 왼쪽 서브트리 방문 후, 오른쪽 서브트리 방문
            if n.left:
                self.preorder(n.left)
            if n.right:
                self.preorder(n.right)

    def inorder(self, n):
        # 중위 순회
        if n is not None:
            if n.left:
                self.inorder(n.left)
            print(str(n.key), '', end='')

            if n.right:
                self.inorder(n.right)

    def postorder(self, n):
        # 후위 순회
        if n is not None:
            if n.left:
                self.postorder(n.left)
            if n.right:
                self.postorder(n.right)
            print(str(n.item), '', end='')

    def get(self, key):
        # 탐색연산
        # 탐색은 루트에서 시작
        return self.get_item(self.root, key)

    def get_item(self, n, k):
        if n is None:
            # 탐색 실패
            return None
        if n.key > k:
            # k가 노드의 key보다 작으면 왼쪽 서브트리 탐색
            return self.get_item(n.left, k)
        elif n.key < k:
            # k가 노드의 key보다 크면 오른쪽 서브트리 탐색
            return self.get_item(n.right, k)
        else:
            # 탐색 성공
            return n.value

    def put(self, key, value):
        # 삽입연산
        # 탐색연산과 유사하데, 마지막 None을 반환하는 대신, 삽입하고자 하는 key, value 를 갖는 새로운 노드를 생성
        # 그리고 새노드를 부모노드와 연결
        self.root = self.put_item(self.root, key, value)

    def put_item(self, n, key, value):
        if n is None:
            return Node(key, value)
        if n.key > key:
            n.left = self.put_item(n.left, key, value)
        elif n.key < key:
            n.right = self.put_item(n.right, key, value)
        else:
            # 이미 존재하는 key의 경우 value만 갱신
            n.value = value
        # 부모노드와 연결하기 위해 노드 n을 리턴
        return n

    def min(self):
        # 최솟값 가진 노드 찾기
        # 루트로부터 왼쪽자식을 따라 내려가며, None을 만났을 때 None의 부모노드가 가진 Key가 최솟값
        if self.root is None:
            return None
        return self.minmum(self.root)

    def minimum(self, n):
        if n.left is None:
            return n
        return self.minimum(n.left)

    def delete_min(self):
        # 최소값 삭제
        # 최솟값을 가진 노드 n을 찾아낸 뒤, n의 부모노드 p와 n의 오른쪽자식 c를 연결
        if self.root is None:
            return None
        self.root = self.del_min(self.root)

    def del_min(self, n):
        if n.left is None:
            return n.right
        n.left = self.del_min(n.left)
        return n

    def delete(self, key):
        # 임의의 키를 가진 노드를 삭제하는 연산
        # get()탐색 과정과 같이 삭제할 노드 찾은 후, 이진탐색트리 조건을 만족하도록 삭제된 노드를 부모노드와 자식노드들 연결해야함
        # 2. 자식이 하나인 경우
        # 3. 자식이 둘인 경우
        self.root = self.del_node(self.root, key)

    def del_node(self, n, k):
        # 삭제되는 노드가 자식이 없는 경
        if n is None:
            return None
        if n.key > k:
            n.left = self.del_node(n.left, k)

        elif n.key < k:
            n.right = self.del_node(n.right, k)
        else:
            # 자식이 하나인 경우
            if n.right is None:
                return n.left
            if n.left is None:
                return n.right
            # 자식이 둘인 경우
            # 1. 왼쪽에서 가장 큰 노드 찾기
            # 2. 오른쪽에서 가장 작은 값찾기 <- 이 방법 사용

            target = n
            n = self.minimum(target.right)
            n.right = self.del_min(target.right)
            n.left = target.left

        return n


if __name__ == '__main__':
    t = BST()
    t.put(500, 'apple')
    t.put(600, 'banana')
    t.put(200, 'melon')
    t.put(100, 'orange')
    t.put(400, 'lime')
    t.put(250, 'kiwi')
    t.put(150, 'grape')
    t.put(800, 'peach')

    print('전위순회: ', end='')
    t.preorder(t.root)
    print()
    print('중위순회: ', end='')
    t.inorder(t.root)

    print('\n250 ', t.get(250))

    t.delete(200)

    print('200 삭제 후: ')

    print('전위순회: ', end='')
    t.preorder(t.root)
    print()
    print('중위순회: ', end='')
    t.inorder(t.root)


```

### 코드 실행
```console
전위순회: 500 200 100 150 400 250 600 800
중위순회: 100 150 200 250 400 500 600 800
250  kiwi
200 삭제 후:
전위순회: 500 250 100 150 400 600 800
중위순회: 100 150 250 400 500 600 800
```
