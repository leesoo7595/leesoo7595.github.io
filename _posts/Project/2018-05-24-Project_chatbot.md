---
layout: post
title: "Project 카카오톡 챗봇(세종대 도서 검색)"
date: 2018-05-23 00:00:00
img:
categories:
- Project
tags: [Project]
---

## 카카오톡 챗봇 '세종대 도서 검색'
학교 다니면서 가장 마음에 안들었던 것은 '학교 홈페이지'다. 반응형 웹도 아닌데다가 안되는게 너무너무 많은 학교 사이트. <br>
원래 하고 싶었던 것은 스터디룸 예약관련 프로젝트였지만, selenium을 이용해서 로그인 자체가 안되서 포기ㅜㅜ<br>
그래서 만들게 된 `세종대 도서 검색 챗봇`!! 카톡을 통해 도서명을 입력하면 대출여부와 도서 위치를 알려주는 시스템이다.

----

## 카카오톡 플러스 친구 만들기
카카오톡 챗봇을 만들기 위해선 `카카오톡 플러스 친구`를 만들어야 한다. 카카오톡 아이디로 가입하면 금방되는 것 같다. 그리고 플러스 친구를 생성하면 끝!

----

## 카카오톡 챗봇 - 스마트 채팅
- 환경
    - AWS EC2
    - Django
    - Python

### Django 새 프로젝트 시작 및 앱 생성
기본 배포했던 프로젝트를 이용해서 진행했기때문에 서버 배포 관련 글을 읽고 진행하시길 바란다.<br>
예) `app`이란 새 프로젝트 안에 `chatbot`이란 앱을 생성
```console
.
└── app
    ├── chatbot
    │   ├── admin.py
    │   ├── apps.py
    │   ├── __init__.py
    │   ├── migrations
    │   │   └── __init__.py
    │   ├── models.py
    │   ├── tests.py
    │   └── views.py
    ├── config
    │   ├── __init__.py
    │   ├── __pycache__
    │   │   ├── __init__.cpython-36.pyc
    │   │   └── settings.cpython-36.pyc
    │   ├── settings.py
    │   ├── urls.py
    │   └── wsgi.py
    └── manage.py
```

### Django 환경 설정
`confing/settings.py`에 카카오톡 서버와 자신의 도메인주소 혹은 IP를 등록해주자.

```python

ALLOWED_HOSTS = [
    '127.0.0.1',
    'localhost',
    #  카카오톡 proxy 서버
    '110.76.143.234',
    '110.76.143.235',
    '110.76.143.236',
    '172.31.27.153',
    # 자신의 도메인주소 혹은 IP 등록
]
```

### Django url.py/views.py 설정
카카오톡 챗봇에 자신의 URL 을 등록하는 경우 `API테스트`를 실행한다. 그렇기 때문에 먼저 `views.py` 를 설정하고 url과 연결하자<br>

```python
# chatbot/views.py
def keyboard(request):
    return JsonResponse({
        'type': 'text',
        'buttons': ['사용법',]
    })

# chatbot/urls.py
from django.urls import path
from chatbot import views

urlpatterns = [
    path('keyboard/', views.keyboard, name='keyboard'),
    ]

```
- **type** : `text` 와 `button`형식이 있는데, 이 프로젝트의 경우 사용자가 직접 도서명을 검색해야 하기 때문에 `text`로 설정했다.

### 카카오톡 스마트 채팅
- [카카오톡 API 문서](https://github.com/plusfriend/auto_reply)
- 스마트 채팅 >> API형 선택
<img src="{{ site.url }}/assets/post_img/chatbot1.png">

- API형에서 자신의 앱 URL을 등록하고 API테스트를 실행하자. 만약 제대로 연결이 되었다면 아래 사진과 같이 **OK** 를 볼 수 있다. <BR> 내가 했던 실수중 하나는 앱 URL에 `http://myhomepage.com/keyword` 이렇게 작성했었는데 **Fail**이 떴다.  앱 URL 에는 `http://myhomepage.com/` 이런식으로 작성해야 한다.

<img src="{{ site.url }}/assets/post_img/chatbot2.png">

### 카카오톡에서 확인하기
웰컴 메시지와 사진은 메시지 메뉴에서 기본 설정이 가능하다.
<img src="{{ site.url }}/assets/post_img/chatbot3.jpg" width='400px'>

### 메시지 수신 및 자동응답 API
<img src="{{ site.url }}/assets/post_img/chatbot4.png">
사용자가 선택하거나 입력한 명령어를 서버로 전달해주는 API이다. 예로 *사용법* 버튼을 누르면 사용법관련 내용을 자동으로 응답해준다.  API 사용을 위해 `views.py`와 `urls.py`를 셋팅하자.<br>

```python
# chatbot/vies.py
@csrf_exempt
def message(request):
    message = ((request.body).decode('utf-8'))
    return_json_str = json.loads(message)
    content = return_json_str['content']

    if content == '사용법':
        return JsonResponse({
            'message': {
                'text': HELP_TEXT,
            },
        })

# chatbot/urls.py
from django.urls import path
from chatbot import views

urlpatterns = [
    path('keyboard/', views.keyboard, name='keyboard'),
    path('message', views.message, name='message'),
]
```
- **message API** 를 연결하는 부분에서 꾀 많은 시간을 보냈다. 왜냐하면 기존 장고 프로젝트에서 url을 등록할때 항상 `/`이걸 붙여줬는데.... 카카오톡 **message API**의 경우 뒤에 없어야했다. (API마다 이러한 설정이 다르기 때문에 문서 읽어볼때 꼼꼼히 읽어봐야한다는 생각이..)
- POST요청이기 때문에 `@csrt_exempt`를 넣어놨다.
- return 값의 경우 무조건 문자열로만 보낼 수 있는 것 같다.

### 카카오톡에서 확인하기
<img src="{{ site.url }}/assets/post_img/chatbot5.jpg" width='400px'>
