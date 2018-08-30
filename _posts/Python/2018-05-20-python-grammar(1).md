---
title: Python 문법 기초 (1)
author: Leesoo
layout: post
---

# 문자열과 시퀀스

## 문자열

### 이스케이프 문자

 이스케이프 문자   |설명
---|---
\a | 비프음 발생
\t | 탭(tab)
\n | 줄바꿈
\\ | \입력
\' | ' 입력
\" | " 입력

### 인덱스 연산
```Python
>>> lux = '빛으로 강타해요!'
>>> lux[0]
'빛'
>>> lux[-1]
'!'
>>> lux[50]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: string index out of range
```

### 슬라이스 연산

```[start:end:step]``` 형식 사용

* ```[:]``` : 처음부터 끝까지
* ```[start:]``` : start부터 끝까지
* ```[:end]``` : 처음부터 end까지
* ```[start:end]``` : start부터 end까지
* ```[start:end:step]``` : start부터 end까지, step만큼 뛰어넘은 부분

### 문자열 나누기 : split

split 함수에 인자로 주어진 구분자를 기준으로 리스트 형태로 반환시켜준다.

```Python
>>> fruits = "apple, banana, cherry, durian"
>>> fruits.split(',')
['apple', 'banana', 'cherry', 'durian']
```
split 함수에 인자를 주지 않을 때, 공백문자를 구분자로 사용한다.

### 문자열 결합 : join

split 함수와 반대 역할 <br>
각 문자열을 결합해줄 구분자 문자열을 지정한 후 join 함수 사용

```Python
>>> fruits = "apple, banana, cherry, durian"
>>> fruits_list = fruits.split(',')
>>> fruits_str = ', '.join(fruits_list)
>>> print(fruits_str)
apple, banana, cherry, durian
```

### 대소문자 다루기

```Python
>>> lux = 'lux, the Lady of Luminosity'
>>> lux.capitalize()
'Lux, the lady of luminosity'
>>> lux.title()
'Lux, The Lady Of Luminosity'
>>> lux.upper()
'LUX, THE LADY OF LUMINOSITY'
>>> lux.lower()
'lux, the lady of luminosity'
>>> lux.swapcase()
'LUX, THE lADY OF lUMINOSITY'
```

### Format 함수

```string % data``` <br>
```{}.format(변수)``` <br>
```f'{변수 또는 표현식}'```

## sequence 타입

### List

리스트는 순차적인 데이터를 나타내는 데 유용, 문자열과는 달리 내부 항목을 변경할 수 있다.

```Python
>>> month_list = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
>>> list('League of legends')
['L', 'e', 'a', 'g', 'u', 'e', ' ', 'o', 'f', ' ', 'l', 'e', 'g', 'e', 'n', 'd', 's']
```

#### append 함수
```Python
>>> sample_list = ['a', 'b', 'c', 'd']
>>> sample_list.append('e')
>>> sample_list
['a', 'b', 'c', 'd', 'e']
```

#### extend 함수, +=
```Python
>>> fruits = ['apple', 'banana', 'melon']
>>> colors = ['red', 'green', 'blue']
>>> fruits.extend(colors)
>>> fruits
['apple', 'banana', 'melon', 'red', 'green', 'blue']
```
```Python
>>> fruits = ['apple', 'banana', 'melon']
>>> colors = ['red', 'green', 'blue']
>>> fruits += colors
>>> fruits
['apple', 'banana', 'melon', 'red', 'green', 'blue']
```

#### insert(offset) 함수
```Python
>>> fruits = ['apple', 'banana', 'melon']
>>> fruits.insert(1, 'mango')
>>> fruits
['apple', 'mango', 'banana', 'melon']
```

#### del 함수
```Python
>>> fruits = ['apple', 'banana', 'melon']
>>> del fruits[0]
>>> fruits
['banana', 'melon']
```

#### remove 함수
```Python
>>> fruits.remove('mango')
```

#### pop, index 함수
```Python
In [19]: fruits = ['apple', 'mango', 'banana', 'melon']

In [20]: fruits
Out[20]: ['apple', 'mango', 'banana', 'melon']

In [21]: fruits.pop()
Out[21]: 'melon'

In [22]: fruits
Out[22]: ['apple', 'mango', 'banana']

In [23]: fruits.pop()
Out[23]: 'banana'

In [24]: fruits
Out[24]: ['apple', 'mango']

In [25]: fruits.index('mango')
Out[25]: 1
```

#### count 함수
```python
>>> fruits = ['apple', 'mango']
>>> fruits.append('apple')
>>> fruits.append('apple')
>>> fruits.count('apple')
3
```

#### sort, sorted, copy 함수
* sort는 리스트를 정렬
* sorted는 리스트 정렬 복사본을 반환
* copy는 똑같은 리스트를 하나 더 생성

### Tuple
튜플은 정의 후 내부 항목의 삭제나 수정이 불가능

#### 튜플 생성
```python
>>> fruits_tuple = ('apple', 'banana', 'melon')
>>> fruits_one_tuple = ('apple',)
```

### Dictionary
key-value 형태로 항목을 가지는 자료구조

#### 딕셔너리 생성
```python
>>> empty_dict1 = {}
>>> empty_dict2 = dict()
>>> champion_dict = {
... 'Lux': 'the Lady of Luminosity',
... 'Ahri': 'the Nine-Tailed Fox',
... 'Ezreal': 'the Prodigal Explorer',
... 'Teemo': 'the Swift Scout',
... }
```

#### 형변환
dict 함수를 사용하여, 두 값의 시퀀스를 딕셔너리로 변환한다.
```Python
>>> sample = [[1,2], [3,4], [5,6]]
>>> dict(sample)
{1: 2, 3: 4, 5: 6}
```

#### 항목 찾기/추가/변경 [key]
```Python
>>> champion_dict['Lux']
'the Lady of Luminosity'
>>> champion_dict['Sona'] = 'Maven of the Strings'
>>> champion_dict['Lux'] = 'Demacia'
```

#### 결합 (update)
```python
>>> item_dict = {
... 'Doran\'s Ring': 400,
... 'Doran\'s Blade': 450,
... 'Doran\'s Shield': 450,
... }
>>> com_dict = {}
>>> com_dict.update(champion_dict)
>>> com_dict.update(item_dict)
>>> com_dict
```
서로 같은 키가 있을 경우,  update에 주어진 딕셔너리의 값이 할당된다.

#### 삭제 (del)
```python
>>> del com_dict['Doran\'s Blade']
>>> del com_dict['Doran\'s Ring']
>>> del com_dict['Doran\'s Shield']
```

### Set
셋은 키만 있는 딕셔너리와 같다. <br>
중복된 값이 존재할 수 없다.

#### 셋 생성
```Python
>>> empty_set = set()
>>> champions = {'lux', 'ahri', 'ezreal'}
```

#### 형변환
문자열, 리스트, 튜플, 딕셔너리를 셋으로 변환할 수 있으며, 중복된 값이 사라진다.

```Python
>>> set('ezreal')
{'e', 'z', 'a', 'l', 'r'}
>>> set(champion_dict)
{'Ahri', 'Lux', 'Ezreal', 'Sona', 'Teemo'}
```
