---
title: Python 문법 기초 (2)
author: Leesoo
layout: post
---

## 제어문과 함수

### if, elif, else 조건문
```python
if 조건1:
	조건1이 참일 경우
elif 조건2:
	조건1은 거짓이나, 조건2가 참일 경우
else:
	조건1,2가 모두 거짓일 경우
```

### for문
시퀀스형 데이터를 순회하고자 할 때 사용
```python
>>> champion_list = ['lux', 'ahri', 'ezreal', 'zed']
>>> for champion in champion_list:
...   print(champion)
...
lux
ahri
ezreal
zed
```
딕셔너리에서 키나 값을 순회할 때, iterable한 객체의 위치에 dict.key()나 dict.values() 사용

#### zip : 여러 시퀀스 동시순회
```python
>>> fruits = ['apple', 'banana', 'melon']
>>> colors = ['red', 'yellow', 'green', 'purple']
>>> for fruit, color in zip(fruits, colors):
...   print('fruit:', fruit, ' color:', color)
...
fruit: apple  color: red
fruit: banana  color: yellow
fruit: melon  color: green
```

#### range : 특정 범위의 숫자 스트림 데이터 반환
```range(start, stop, step)```

```Python
>>> for x in range(0, 10):
...   print(x)
...
0
1
2
3
4
5
6
7
8
9
```

### while문
```
while 조건:
  조건이 참일경우 실행
  조건이 거짓이 될 경우까지 계속해서 반복
```

```Python
>>> count = 0
>>> while count < 10:
...   print(count)
...   count += 1
...
0
1
2
3
4
5
6
7
8
9
```

### Comprehension
```[표현식 for 항목 in iterable객체]```

example : [1,2,3,4,5]
```Python
>>> numbers = []
>>> for item in range(1, 6):
...   numbers.append(item)
...
>>> numbers
[1, 2, 3, 4, 5]
```

### 함수

#### parameter & argument 차이
```python
# 함수 정의때는 매개변수
def func(매개변수1, 매개변수2):
  ...

# 함수 호출시에는 인자
>>> func(인자1, 인자2)
```

#### Positional arguments
위치 인자 : 매개변수의 순서대로 인자를 전달하여 사용하는 경우
```python
>>> def student(name, age, gender):
...   return {'name': name, 'age': age, 'gender': gender}
...
>>> student('leesoo', 24, 'female')
{'name': 'leesoo', 'age': 24, 'gender': 'female'}
```

#### Keyword arguments
키워드 인자 : 매개변수의 이름을 지정하여 인자로 전달하여 사용하는 경우
```python
>>> student(age=24, name='leesoo', gender='female')
{'name': 'leesoo', 'age': 24, 'gender': 'female'}
```
***위치 인자와 키워드 인자를 동시에 쓴다면, 위치 인자가 먼저 와야 한다.***

#### default parameter 지정
인자가 제공되지 않을 경우. 기본 매개변수로 사용할 값을 지정할 수 있다.
```python
>>> def student(name, age, gender, cls='WPS'):
...   return {'name': name, 'age': age, 'gender': gender, 'class': cls}
...
>>> student('leesoo', 24, 'female')
{'name': 'leesoo', 'age': 24, 'gender': 'female', 'class': 'WPS'}
```

#### default parameter 정의 시점
기본 매개변수 값은 함수가 정의되는 시점에 계산되어 계속해서 사용한다.
```python
>>> def return_list(value, result=[]):
...   result.append(value)
...   return result
...
>>> return_list('apple')
['apple']
>>> return_list('banana')
['apple', 'banana']
```
함수가 실행되는 시점에 기본 매개변수값을 계산할 경우
```python
>>> def return_list(value, result=None):
...   if result is None:
...     result = []
...   result.append(value)
...   return result
...
>>> return_list('apple')
['apple']
>>> return_list('banana')
['banana']
```

#### arguments

```*args``` : 함수에 위치 인자로 주어진 변수들의 묶음
```**kwargs``` : 함수에 키워드 인자로 주어진 변수들의 묶음

#### docstring
함수를 정의한 문서 역할
```python
>>> def print_args(*args):
...   'Print positional arguments'
...   print(args)
...
>>> help(print_args)
```

### Scope
Global Scope : 전역 영역 <br>
Local Scope : 지역 영역 <br>
nonlocal : 자신의 영역보다 한 단계 위의 로컬 영역
```pythonfruit = 'apple'
def local1():
    fruit = 'banana'
    print('local1 locals() : {}'.format(locals()))

    def local2():
        nonlocal fruit
        fruit = 'melon'
        print('local2 locals() : {}'.format(locals()))
    local2()
    print('local1 locals() : {}'.format(locals()))

print('global locals() : {}'.format(locals()))
local1()
```

### lambda
```lambda <매개변수> : <표현식>```

익명함수 : 한 번 쓰고 버릴 함수, 한 줄 표현식으로 이루어진다.
정의, 호출이 나누어져있지않아 표현식이 바로 호출된다.

```python
import string
>>> for char in string.ascii_lowercase:
...   if char > 'i':
...     print(char.upper())
...   else:
...     print(char)
```
```python
>>> for char in string.ascii_lowercase:
...   print((lambda x : x.upper() if x > 'i' else x)(char))
```

### Closure
함수가 정의된 환경, 파일이 여러 개일때, 각 파일은 모듈 역할을 하고, 각 모듈은 독립적인 환경을 갖게 된다.

***closure/module_a.py***
```python
level = 100
def print_level():
    print(level)
```

***closure/module_b.py***
```python
import module_a
level = 50
def print_level():
    print(level)

module_a.print_level()
print_level()
```

module_b.py에서 module_a.py를 import하여 사용한다.

***내부함수의 closure***
```python
>>> level = 0
>>> def outer():
...   level = 50
...   def inner():
...     nonlocal level
...     level += 3
...     print(level)
...   return inner
...
>>> f = outer()
```
반복적으로 f함수를 실행하면 inner의 외부(outer)에 존재하는 level 변수 값을 증가시키고 print시키기 때문에, 값을 계속 증가한다.

### decorator
함수를 받아 다른 함수를 반환하는 함수 <br>
기존에 존재하던 함수를 바꾸지 않고 전달된 인자를 보기위한 디버깅이 가능하다. <br>

1. 기능을 추가할 함수를 인자로 받음
2. 데코레이터 자체에 추가할 기능을 함수로 정의
3. 인자로 받은 함수를 데코레이터 내부에서 적절히 호출
4. 위 2가지를 행하는 내부 함수를 반환

<br>
여러 함수에 대해 같은 기능을 추가할 경우, 데코레이터를 사용한다.
```python
def print_debug(f):
    def inner_function(*args, **kwargs):
        print('args :', args)
        print('kwargs :', kwargs)
        result = f(*args, **kwargs)
        return result
    return inner_function
```

```python
@print_debug
def any_function():
    pass
```

### generator

수많은 자료를 돌아다니면서 반복문을 돌려야할 때 모두 list에 올려놓는 대신 generator를 사용한다.

0 ~ 9까지의 3제곱을 출력하는 generator 함수
```python
def gen():
    for i in range(10):
        yield i ** 3

for x in gen():
    print(x)
```

#### yield
generator에서는 ```yield```라는 키워드를 사용한다.
```python
def gen():
    yield 'one'
    yield 'two'
    yield 'three'

g = gen()
print(next(g))  # one
print(next(g))  # two
print(next(g))  # three
print(next(g))  # raise StopIteration
```

yield는 함수 실행 중간에 빠져나올 수 있는 generator를 만들 때 사용한다.

```python
def gen():
    val = 111111
    while True:
        val = (yield val) * 111111

g = gen()
print(next(g))  # 111111
print(g.send(2))  # 222222
print(g.send(3))  # 333333
```
값을 내보낼 수 있는 것 뿐만 아니라, 넣어줄 수도 있다.

참고 블로그 : https://item4.github.io/2016-05-09/Generator-and-Yield-Keyword-in-Python/
						https://github.com/Fastcampus-WPS-8th/Python/blob/master/lecture/09.%20%ED%95%A8%EC%88%98.md
