---
layout: post
title: "스택(Stack)"
date: 2018-08-01 00:00:00
img:
categories:
- Algorithm/DataStructure
tags: [알고리즘/자료구조]
---
> 파이썬과 함께하는 자료구조의 이해 책 정리

----

## 스택
<img src="{{ site.url }}/assets/post_img/stack1.jpeg">
- 일상에서 쉽게 접할 수 있는 자료 구조
    - 스택 : 이전 웹 페이지로 돌아가는 기능
- 한 쪽 끝에서만 항목(item)을 삭제하거나 새로운 항목을 저장하는 자료구조
- push: 새로운 항목을 저장하는 연산
- pop:항목을 삭제하는 연산

----

## 리스트로 구현한 스택
<img src="{{ site.url }}/assets/post_img/stack2.jpeg">

### 수행시간
- `push()`와 `pop()` 연산은 각각 **O(1)**시간이 소요 된다.
- 파이썬의 리스트는 크기가 동적으로 확대 또는 축소되기 때문에, 동적 크기 조절은 스택(리스트)의 모든 항목들을 새 리스트로 복사하므로 **O(N)** 시간이 소요된다.

### 코드

```python
def push(item):
    stack.append(item)


def peek():
    # top 항목 접근
    if len(stack) != 0:
        return stack[-1]


def pop():
    # 삭제 연산
    if len(stack) != 0:
        # 리스트의 맨 뒤에 있는 항목 제거
        item = stack.pop(-1)
        return item


stack = []
push('apple')
push('orange')
push('cherry')
print(stack)

print(peek())
push('pear')
print(stack)

pop()
push('grape')
print(stack)
```

### 코드 실행
```console
['apple', 'orange', 'cherry']
cherry
['apple', 'orange', 'cherry', 'pear']
['apple', 'orange', 'cherry', 'grape']
```

----

## 단순연결리스로 구현된 스택
### 수행 시간
- `push()`와 `pop()`은 각각 **O(1)** 시간이 걸린다. 이는 연결리스트의 맨 앞 부분에서 노드를 삽입하거나 삭제하기 때문

### 코드
```python
class Node:
    def __init__(self, item, link):
        self.item = item
        self.next = link


def push(item):
    global top
    global size
    top = Node(item, top)
    size += 1


def peek():
    if size != 0:
        return top.item


def pop():
    global top
    global size
    if size != 0:
        top_item = top.item
        top = top.next
        size -=1
        return top_item


def print_stack():
    print('top->', end='')
    p = top
    while p:
        if p.next != None:
            print(p.item, '->', end='')
        else:
            print(p.item, end='')
        p = p.next
    print()


# 초기화
top = None
size = 0

push('apple')
push('orange')
push('cherry')
print_stack()

print('top항목:', end='')
print(peek())
push('pear')
print_stack()
pop()
push('grape')
print_stack()
```
- [전역변수와 지역변수 참고](https://python.bakyeono.net/chapter-3-4.html)

----

## 스택 응용
### 괄호 짝 맞추기 문제
> pop()된 왼쪽 괄호와 바로 읽었던 오른쪽 괄호가 다른 종류이면 에러 처리, 같은 종류이면 다음 괄호를 읽는다. 모든 괄호를 읽은 뒤, 에러가 없고 스택이 empty 이면 괄호들이 정상적으로 사용된 것이고, 만일 모든 괄호를 처리한 후 스택이 empty가 아니면 짝이 맞지 않는 괄호가 스택에 남아있으므로 에러 처리

```python
def push(item):
    stack.append(item)


def peek():
    # top 항목 접근
    if len(stack) != 0:
        return stack[-1]


def pop():
    # 삭제 연산
    if len(stack) != 0:
        # 리스트의 맨 뒤에 있는 항목 제거
        item = stack.pop(-1)
        return item


def check_bracket(list):
    """
    괄호문 체크 함수
    :param list:
    :return:
    """
    left_bracket = ['{', '(', '[']
    right_bracket = ['}', ')', ']']

    # 매개변수로 받은 list의 문자 순회
    for letter in list:

        #  왼쪽 괄호 인 경우 Push()
        if letter in left_bracket:
            push(letter)

        # 오른쪽 괄호 인 경우 pop()
        if letter in right_bracket:

            # 오른쪽 괄호가 먼저 오는 경우가 있기 때문에
            # 체크를 해줘야 한다.
            if len(stack) != 0:
                item = pop()

                # pop한 왼쪽 괄호와 오른쪽 괄호가 같은 짝인지 체크
                # 만약 아니라면 다시 push()
                if left_bracket.index(item) != right_bracket.index(letter):
                    push(item)

    if len(stack) != 0:
        return print('괄호문 체크 실패')
    else:
        return print('괄호문 체크 완료')

stack = []
string = 'abc+[(x+y)+[z+e+y]+d]}'

check_bracket(string)
```

### 회문 검사하기(Palindrome)
> 주어진 문자열의 앞부분 반을 차례대로 읽어 스택에 push한다. 문자열 길이가 짝수이면 뒷부분의 문자 1개를 읽을 때마다 pop하여 읽어들인 문자ㅘ pop문자를 비교하는 과정을 반복 수행한다. 만약 문자열 길이가 홀수인 경우, 주어진 문자열 앞부분은 앞에서 동일하게 스택에 push 하고 중간문자를 읽고 버린다. 이후 짝수 경우와 동일하게 비교를 수행한다. 마지막 비교까지 두 문자가 동일하고 스택이 empty가 되면 문자열은 회문이다.

```python
def push(item):
    stack.append(item)


def pop():
    # 삭제 연산
    if len(stack) != 0:
        # 리스트의 맨 뒤에 있는 항목 제거
        item = stack.pop(-1)
        return item


def check_palindrome(string):
    """
    회문 검사 함수
    :param string:
    :return:
    """
    length = len(string)

    # 매개변수로 받은 문자열이 짝수 인 경우
    if length % 2 == 0:
        # 앞에 반절은 스택에 push
        for letter in string[:length // 2]:
            push(letter)

        # 스택에 있는 문자와 뒤에 있는 문자 비교
        for letter in string[length // 2:]:
            item = pop()
            # 같지 않은 경우 push
            if item != letter:
                push(item)

    # 매개변수로 받은 문자열이 홀수 인 경우
    else:
        for letter in string[:length // 2]:
            push(letter)

        # 중간 문자를 버리고 비교
        for letter in string[length // 2 + 1:]:
            item = pop()
            print(length//2+1)
            print(letter)

            if item != letter:
                push(item)


    if len(stack) != 0:
        return print('회문이 아닙니다.')

    else:
        return print('회문입니다.')


stack = []
letters = "raceDar"
check_palindrome(letters)
```
