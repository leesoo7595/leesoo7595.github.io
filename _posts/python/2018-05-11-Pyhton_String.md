---
layout: post
title: "Python_문자열"
date: 2018-05-11 00:00:00
img:
tags: [Python]
---
> 본 문서는 Introduction Python 책을 공부하면서 정리한 내용입니다.

---
### 문자열
다른 언어와 다르게 파이썬은 문자열이 **불변**이다. 그래서 `stript()`와 같은 함수들을 사용해서 변경하는 경우 문자열을 바꾸는 것이 아니라, 함수 수행후 새로운 문자열로 결과를 반환하는 것이다.

#### 2.3.1 인용 부호로 문자열 생성
`'''`을 사용하면 여러줄의 문자열을 입력할 때 사용한다. 그리고 이 경우 라인 끝 문자가 보존되고 양쪽 끝 공백이 있는 경우에도 보존된다.

#### 2.3.2 데이터 타입 변환: str()
문자열이 아닌 객체를 `print()`로 호출할 때도 내부적으로 `str()`함수를 사용한다.

#### 2.3.3 이스케이프 문자

#### 2.3.4.결합
리터럴 문자열 또는 문자열 변수를 결합 할 수 있다.

```python
# 공백을 자동으로 붙이지 않는다.
In [3]: "My word! " "A gentleman caller!"
Out[3]: 'My word! A gentleman caller!'

# print() 는 각 인자 사이에 공백과 마지막에 줄바꿈 문자열을 붙###
In [4]: a = "Duck"
In [5]: b = a
In [6]: c = 'Grey Duck'
In [7]: print(a,b,c)
Duck Duck Grey Duck
```

#### 2.3.5 복제하기: *
```python
In [8]: start = 'Na ' *4 +'\n'
In [9]: middle = 'Hey ' *3 + '\n'
In [10]: end = 'GoodBye.'
In [11]: print(start+start+middle+end)
Na Na Na Na
Na Na Na Na
Hey Hey Hey
GoodBye.
```

#### 2.3.6 문자 추출:[]
#### 2.3.7 슬라이스:[start:end:step]
- [:] = [0:] : 처음부터 끝까지 전체 시퀀스 추출
#### 2.3.8 문자열 길이: len()
#### 2.3.9 문자열 나누기: split()
구분자를 지정하지 않으면  `split()`는 문자열에 등장하는 공백문자(줄바꿤,스페이스,탭)을 사용

```python
In [12]: todos = 'get gloves,get mask,give cat vitamins,call ambulance'
In [13]: todos.split()
Out[13]: ['get', 'gloves,get', 'mask,give', 'cat', 'vitamins,call', 'ambulance']
```

#### 2.3.10 문자열로 결합하기: join()
```python
In [14]: crypto_list = ['Yeti','Bigfoot','Loch Ness Monster']
In [16]: crypto_string = ', '.join(crypto_list)
In [17]: crypto_string
Out[17]: 'Yeti, Bigfoot, Loch Ness Monster'
```

#### 2.3.11 문자열 다루기
- len
- startswith : 특정단어로 시작되는지 체크
- endswith: 특정단어로 끝나는지 체크
- find: 특정단어가 첫버재 단어가 나오는 경우
- rfind: 특정단어가 마지막으로 나오는 경우
- count: 특정단어 횟수 체크
- isalnum : 글자와 숫자로만 이루어져있는가 체크

#### 2.3.12 대소문자와 배치
- capitalizer : 첫번째 단어 대소문
- title : 첫글자만 대문자
- upper : 대소문자
- lower: 소문자
- swapcase() : 소->대 대->소
- ljust/rjust : 왼쪽 , 오론쪽 정렬
- replace : 바꾸는 것
