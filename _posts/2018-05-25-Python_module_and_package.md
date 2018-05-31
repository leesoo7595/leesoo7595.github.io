---
layout: post
title: "Python_모듈,패키지,프로그램"
date: 2018-05-25 00:00:00
img:
tags: [Python]
categories:
- Python
---

## 5.5 파이썬 표준 라이브러리
### 누락된 키 처리하기 : setdefault(), defaultdict()
- setdefault() : 딕셔너리에 키가 없는 경우 새 값이 사용된다. 존재하는 키에 다른 기본값을 할당하려 하면 키에 원래 값이 반환되고 아무것도 변환되지 않는다.
```python
In [3]: periodic_table = {'Hydrogen':1, 'Heliun':2}
In [4]: carbon = periodic_table.setdefault('Carbon',12)
In [5]: carbon
Out[5]: 12
In [6]: periodic_table
Out[6]: {'Hydrogen': 1, 'Heliun': 2, 'Carbon': 12}
#  존재하는 키 기본값 변경하기
In [11]: helium = periodic_table.setdefault('Heliun',947)
In [12]: helium
Out[12]: 2
In [13]: periodic_table
Out[13]: {'Hydrogen': 1, 'Heliun': 2, 'Carbon': 12}
```
- defaultdict() : 딕셔너리를 생성할 때 모든 새 키에 대한 기본값을 먼저 지정한다. <br>
빈 기본값을 반환하려면 `ing(),정수0` , `list()함수, 빈리스트[]`, `dict()함수, 빈 딕셔너리{}`,그리고 인자가 입력되지 않았으면 초깃값은 `None`이다.

```python
In [14]: from collections import defaultdict

# 기본값 0으로 설정됨
In [15]: periodic_table = defaultdict(int)
In [16]: periodic_table['Hydrogen']=1
In [17]: periodic_table['Lead']
Out[17]: 0
In [18]: periodic_table
Out[18]: defaultdict(int, {'Hydrogen': 1, 'Lead': 0})

# 기본값을 함수로 지정
In [5]: def no_idea():
   ...:     return 'HUh?'

In [6]: bestiary = defaultdict(no_idea)
In [7]: bestiary['a'] ='apple'
In [9]: bestiary['b']='banana'
In [10]: bestiary['c']
Out[10]: 'HUh?'

# 람다를 이용하여 기본값 만드는 함수 정의
In [12]: bestiary = defaultdict(lambda:'Huh?')
In [13]: bestiary['E']
Out[13]: 'Huh?'
```
