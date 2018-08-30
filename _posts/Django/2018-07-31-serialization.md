---
layout: post
title: "[Django] 직렬화 - serialization"
img:
tags: [Django]
---

# serialization (직렬화)

객체 자체는 메모리상에 존재한다. 이를 저장할 수 있는가? <br>
이는 직렬화 과정을 통해 저장이 가능하다. 예를 들면 어떤 클래스 자체의 인스턴스를 데이터베이스에 저장하는 것이 아니라, 메모리상에서 바로 저장할 수 있도록 하는 것이다. <br>
데이터 구조나 오브젝트 상태를 동일하거나 다른 컴퓨터 환경에 파일이나 메모리 버퍼 상으로 저장, 또는 나중에 재구성할 수 있는 포맷으로 변환하는 과정이다.

파일의 데이터는 연속적인 데이터인 것에 반해, 메모리상의 데이터는 연속적인 데이터가 아니다. <br>
이 메모리상의 데이터를 연속적으로(파일상의 데이터로) 만드는 것이 `직렬화`라고 한다. 그리고 반대로 메모리상의 객체는 `역직렬화`를 통해서 불러오게 된다. <br>

`python objects`를 `serialization`하는 데에는 `pickle`이라는 것으로 하게 된다.
```python
In [1]: class User:
   ...:     def __init__(self, name):
   ...:         self.name = name
   ...:     def __repr__(self):
   ...:         return self.name
   ...:     

In [2]: u = User(name='leesoo')

In [3]: u
Out[3]: leesoo

In [4]: type(u)
Out[4]: __main__.User

In [5]: u.age = 24

In [6]: u
Out[6]: leesoo

In [7]: import pickle

In [8]: with open('user.pickle', 'wb') as f:
   ...:     pickle.dump(u, f)
   ...:     

In [9]: exit
```
ipython으로 위 코드를 입력 후 바로 나가주고, 해당 경로로 들어가게되면 `user.pickle`이라는 `binary` 파일이 생기게 된다. 이것이 `serialization` `직렬화`이다. <br>

반대로 `역직렬화`는 다음과 같이 이루어진다.

```python
In [1]: import pickle

In [2]: with open('user.pickle', 'rb') as f:
   ...:     user = pickle.load(f)
   ...:     
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-2-fc2037caa02e> in <module>()
      1 with open('user.pickle', 'rb') as f:
----> 2     user = pickle.load(f)
      3

AttributeError: Can't get attribute 'User' on <module '__main__'>

In [3]: class User:
   ...:     def __init__(self, name):
   ...:         self.name = name
   ...:     def __repr__(self):
   ...:         return self.name
   ...:     

In [4]: with open('user.pickle', 'rb') as f:
   ...:     user = pickle.load(f)
   ...:     

In [5]: user
Out[5]: leesoo
```
사실상 `python pickle`은 임시적으로 쓰는 것 외엔 불안정할 수 밖에 없다. <br>
직렬화를 할 떄, 다시 역직렬화시키기도 쉽고, 데이터 전송하기도 좋고, 공통적인 규격을 많이쓴다.
그것이 `json`방식인데, `json` <-> `python`으로 직렬화, 역직렬화를 할 것이다. <br>

단순한 배열, 즉 array, dictionary 등의 객체는 `json`과 1대1 매칭이 된다. 하지만 여기서 다시 `python`으로 자동으로 바꿔주지 않는데,
요새는 객체를(`python`) `json` 방식으로 데이터를 직렬화시킨다. 그리고 그 `json`을 다시 `python` 클래스 객체로 역직렬화시킨다. <br>
이 과정을 `serializer`가 하는 것이다.

### serializer 정의
`serializer`란, 어떤 모델 클래스가 있을 때, 이 클래스 인스턴스가 어떻게 `json` 형태로 바뀌는지, 그리고 `json` 데이터는 어떻게 다시 클래스 인스턴스로 바뀌는지 정의를 하는 것을 말한다.

- form : 필드를 적어놓은것 request post 받아서 처리, html 폼을 렌더링받아서 처리
1. serializer class 만들기
    - `json` 데이터를 받아서 처리해줄 수 있다.
    - `json` 데이터를 보냈을 때, `serialize`를 통해 데이터를 받으면, 그 데이터로 isvalid 검사를 할 수 있고, 이를 이용해서 객체를 만들어주는 역할까지 한다.

2. serializer 사용하기
<br>

```python
>>> from rest_framework.renderers import JSONRenderer
>>> from rest_framework.parsers import JSONParser

>>> s = <ModelClassName>(ModelVar='test_test')
>>> s.save()
>>> serializer = <ModelClassNameSerializer>(s)
>>> serializer.data
>>> {'pk': 1, 'title': '', ...} # JSON 형식으로 바꾸기위해 python dict로 바꿈

>>> content = JSONRenderer().render(serializer.data) # JSON 형식의 데이터 직렬화
>>> content
>>> b'{"pk": 1, "title": "", ...}' # byte String의 데이터: 문자열 형태로 기록할 수 있다.

>>> from django.utils.six import BytesIO
>>> stream = BytesIO(content)
>>> stream
>>> <_io.BytesIO at 0x1123b0fc0>
>>> data = JSONParser().parse(stream)
>>> data
>>> {'pk': 1, 'title': '', ...} # json을 python dict로 변환! 역직렬화
```

`직렬화`와 `역직렬화`를 다시 정리해보자면, <br>

```python
s = <ModelClassName>.objects.last() # 모델클래스의 객체 마지막꺼 불러오기
serializer = <ModelClassNameSerializer>(s) # serializer클래스에 s 대입
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser
content = JSONRenderer().render(serializer.data) # content값 : 객체를 직렬화한 json

from django.utils.six import BytesIO
# 다시 Python 객체로 만들기위해 Parse를 쓴다.
stream = BytesIO(content) # stream 데이터로 만든다! (읽을 수 있는 연속적인 데이터 전달)
data = JSONParser().parse(stream) # json 데이터를 다시 python 인스턴스로 역직렬화
```

## ModelSerializers 사용하기

SnippetSerializer 클래스는 Snippet 모델의 정보들을 그대로 베낀다. 이런 특징을 이용해서 코드를 좀 더 간단하게 줄일 수 있다. <br>

Django에서 Form 클래스와 ModelForm 클래스를 제공하듯이, REST 프레임워크에서도 Serializer 클래스와 ModelSerializers 클래스를 제공한다. <br>

`example/serializers.py` <br>
```python
class ExampleSerializer(serializers.ModelSerializer):
  class Meta:
    model = Example
    fields = (
      'id',
      'title',
      ...
    )
```

`serializers.py`에 위와 같이 설정해놓기만 해도 모든 필드를 확인할 수 있다.

```python
>>> from example.serializers import ExampleSerializer
>>> serializer = ExampleSerializer()
>>> print(repr(serializer))
ExampleSerializer():
    id = IntegerField(label='ID', read_only=True)
    title = CharField(allow_blank=True, max_length=100, required=False)
    ...
```

`ModelSerializer` 클래스의 특징
1. 필드를 자동으로 인식한다.
2. create() 메서드와 update() 메서드가 이미 구현되어 있다.
