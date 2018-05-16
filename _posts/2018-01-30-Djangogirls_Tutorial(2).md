---
layout: post
title: "Djangogirls_Tutorial(2)"
date: 2018-01-30 09:00:00
img: ./post_img/django.png
tags: [Django]
---
>본 문서는 패스트캠퍼스 'Web-Programming School' 수업 자료를 바탕으로 작성되었습니다.
><br> [장고걸스](https://tutorial.djangogirls.org/ko/) 페이지를 참고하였습니다.
><Br>**Ubuntu 16.04 환경**
><br>**Django 2.0 환경**

---

### URL이란?
> URL(Uniform Resource Locator, 문화어: 파일식별자, 유일자원지시기)은 네트워크 상에서 자원이 어디 있는지를 알려주기 위한 규약이다. 흔히 웹 사이트 주소로 알고 있지만, URL은 웹 사이트 주소뿐만 아니라 컴퓨터 네트워크상의 자원을 모두 나타낼 수 있다  <Br> - 위키백과 참고 -

### 장고URL 작동
**config/urls.py** 파일

```py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [

    path('admin/', admin.site.urls),
    # 새로운 앱들의 urls 추가하는 곳
    # "localhost:8000/"
    path('', include('blog.urls')),

    # 장고 2.0 버전 이전  
    # url(r'^admin/', admin.site.urls)
    # 장고 2.0 버전
    # path('articles/<int:year>/', views.year_archive)

]
```
장고2.0이 나오면서 URL부분이 바뀌었다고 한다. <br>
`url()` 에서 `path()`로 새로운 URL syntax로 바뀌었으며,<br>
정규표현식으로 URL 패턴을 만들었으나, 장고2.0 부터는 복잡한 <br>
정규표현식이 아닌 URL파라미터의 타입을 통해 패턴을 만들 수 있다.<br>
**Path Converters** 관련한 [공식문서](https://docs.djangoproject.com/en/2.0/topics/http/urls/)<br>

### blog/urls.py 만들기
```Py
from django.urls import path, re_path
# .은 현재 패키지 (상대경로) from blog import views (절대경로)
from . import views

urlpatterns = [
    # url - localhost:8000/list -> views.post_list 실행
    # name은 url에 이름을 붙인 것으로 뷰를 식별
    path('list', views.post_list, name = 'post-list')
]
```
이를 추가하고 서버를 실행하면 에러가 발생한다. <BR>
AttributeError: module 'blog.views' has no attribute 'post_list' <br>
blog/views에 post_list를 발견하지 않았다는 오류.

---

###  장고 뷰 만들기
뷰는 앱의 "로직"이 들어가는 곳이다. 뷰는 `모델`에서 필요한 정보를 받아와서 `템플릿`에 전달하는 역할을 한다. <br>

**작동 순서**
1. 브라우저에서 요청 (localhost:8000 입력)/ 0.0.0.0:8000 외부에서 접근 가능
2. 요청 runserver로 실행중인 서버에 도착
3. runserver 요청을 django code로 전달
4. Django code 중 config.urls 모듈이 해당 요청을 받음
5. config.urls 모듈은 ''(/admin/를 제외한 모든 요청)을 blog.urls 모듈로 전달
6. blog.urls모듈은 받은 요청의 URL과 일치하는 패턴이 있는지 검사
7. 있다면 일치하는 패턴과 연결된 함수 (view)를 실행
   - 7.1 settings 모듈의 TEMPLATES속성 내의 DIRS목록에서 blog/post_list.html 파일의 내용을 가져옴
   - 7.2 가져온 내용을 적절히 처리(렌더링, render()함수))하여 리턴
8. 함수의 실행 결과를 브러우저로 다시 전달


**blog/views.py** 파일
```py
from django.shortcuts import render

def post_list(request):
    # HTTP 프로토콜로 텍스트 데이터 응답
    # return HttpResponse('<html>~')

    # 'blog/post_list.html'템플릿 파일을 이용해 http 프로토콜로 응답
    return render(request, 'blog/post_list.html', {})
```
이렇게 하면 위의 작동 순서에서 6번까지 만든 것이다. <br>
그러나 `blog/post_list.html`을 만들지 않았기 때문에, 서버를 실행시켜도 오류가남.<br>
이제 html 템플릿을 만들자!<br>

---
### 템플릿 만들기
Django/templates/blog 디렉토리 만들기<Br>
그리고 이 안에 **post_list.html** 새 파일 만들기

```command
├── blog
│   ├── admin.py
│   ├── apps.py
│   ├── __init__.py
│   ├── migrations
│   │   ├── 0001_initial.py
│   │   ├── __init__.py
│   │   └── __pycache__
│   │       ├── 0001_initial.cpython-36.pyc
│   │       └── __init__.cpython-36.pyc
│   ├── models.py
│   ├── __pycache__
│   │   ├── admin.cpython-36.pyc
│   │   ├── __init__.cpython-36.pyc
│   │   ├── models.cpython-36.pyc
│   │   ├── urls.cpython-36.pyc
│   │   └── views.cpython-36.pyc
│   ├── tests.py
│   ├── urls.py
│   └── views.py
├── config
│   ├── __init__.py
│   ├── __pycache__
│   │   ├── __init__.cpython-36.pyc
│   │   ├── settings.cpython-36.pyc
│   │   ├── urls.cpython-36.pyc
│   │   └── wsgi.cpython-36.pyc
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── db.sqlite3
├── manage.py
└── templates
    └── blog
```
**post_list.html** 을 등록하고 서버를 실행시키면, 장고걸스 튜토리얼에는 그냥 하얀 페이지가 띈다고 한다. 그러나 나는 오류가 떠서 당황....<BR>
알고보니 templates 을 찾게끔 만들었어야 했는데 그 부분을 빼먹었다.<br>
templates 디렉토리를 추가했다면, **config/settings.py** 에 몇가지를 추가하자

```py

# Build paths inside the project like this: os.path.join(BASE_DIR, ...)
# 템플릿을 찾을 수 있는 경로를 추가해주면 된다.
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
TEMPLATES_DIR = os.path.join(BASE_DIR, 'templates')

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        # Django에서 템플릿을 검색할 경로 목록
        'DIRS': [
            TEMPLATES_DIR,
        ],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

템플릿 경로를 추가해주고, `localhost:8000`을 들어가면 빈 페이지를 볼 수 있다!<br>

### 맞춤형 템플린 만들기
**blog/template/blog/post_list.html** 파일
```html
<html>
    <head>
        <title>Django Girls blog</title>
    </head>
    <body>
        <div>
            <h1><a href="">Django Girls Blog</a></h1>
        </div>

        <div>
            <p>published: 14.06.2014, 12:14</p>
            <h2><a href="">My first post</a></h2>
            <p>Aenean eu leo quam. Pellentesque ornare sem lacinia quam venenatis vestibulum. Donec id elit non mi porta gravida at eget metus. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa justo sit amet risus.</p>
        </div>

        <div>
            <p>published: 14.06.2014, 12:14</p>
            <h2><a href="">My second post</a></h2>
            <p>Aenean eu leo quam. Pellentesque ornare sem lacinia quam venenatis vestibulum. Donec id elit non mi porta gravida at eget metus. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut f.</p>
        </div>
    </body>
</html>
```
**post_list.html** 작성 완료 후, 서버에 들어갔을 때 보이는 화면
<img src = "{{ site.url }}/assets/post_img/django_post_list.png/">
