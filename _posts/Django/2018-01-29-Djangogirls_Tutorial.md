---
layout: post
title: "Djangogirls_Tutorial(1)"
date: 2018-01-29 09:00:00
img: ./post_img/django.png
tags: [Django]
---
>본 문서는 패스트캠퍼스 'Web-Programming School' 수업 자료를 바탕으로 작성되었습니다.
><br> [장고걸스](https://tutorial.djangogirls.org/ko/) 페이지를 참고하였습니다.
><Br>**Ubuntu 16.04 환경**
><br>**Django 2.0 환경**

---

## 가상환경 설정
```
$pyenv virtualenv 3.6.4 fc-djangogirls
$pyenv logcal fc-djangogirls
$git init
$vim .gitignore
// gitignore.io 사이트에서 환경설정에 지정해서 복붙
```
pycharm - setting - project interpreter - fc-dajangogirls로 설정<br>
`git status`로 .idea/ 폴더 있는지 확인 후, 연결 된 Git Repository에 commit!

## 장고 설치
```
$pip install django
Collecting django
  Using cached Django-2.0.1-py3-none-any.whl
Collecting pytz (from django)
  Using cached pytz-2017.3-py2.py3-none-any.whl
Installing collected packages: pytz, django
Successfully installed django-2.0.1 pytz-2017.3
```

---
## Django란?
> Django는 파이썬으로 만들어진 무료 오픈소스 웹 애플리케이션 프레임워크<br>

## 서버와 웹페이지 동작 방식
1. 브라우저에서  HTTP 요청 -> DNS -> IP -> request -> 서버컴퓨터에 도착
2. 서버컴퓨터의 웹 서버 프로그램이 해당 요청을 받음
3. 웹 서버 프로그램은 웹 애플리케이션에 요청에 대한 응답의 제작을 요청
4. 웹 애플리케이션은 요청에 대한 응답을 동적으로 생성해서 웹 서버에게 돌려줌
5. 웹 서버는 자신이 요청하고 받은 응답을 다시 사용자에게 전송
6. 브라우저에 전송

---
## 나의 첫 번째 장고 프로젝트!
> Note 이 장의 일부는 Geek Girls Carrots (http://django.carrots.pl/)의 튜토리얼을 기초로 작성되었습니다.<br>
> Note 이 장의 일부는 Creative Commons Attribution-ShareAlike 4.0 International License에 준수하여 django-marcador 튜토리얼를 기초로 작성되었습니다. django-marcador 튜토리얼 저작권은 Markus Zapke-Gründemann et al이 소유하고 있습니다.

### 프로젝트 만들기
Django project 를 구성하는 코드를 자동 생성하며 이 과정에서 데이터베이스 설정, 옵션, 어플리케이션을 위한 설정과 관련된 Django인스턴스를 구성하기 때문에 **초기 설정 주의**

```
$django-admin startproject mysite
```

현재 디렉토리에서 mysite라는 디렉토리 생성

```
└── mysite
    ├── manage.py
    └── mysite
        ├── __init__.py
        ├── settings.py
        ├── urls.py
        └── wsgi.py
```

`tree`로 확인한 디렉토리 구조
mysite - mystie 디렉토리의 경우, setting 관련한 디렉토리이므로 **config** 로 디렉토리명 변경 <br>
mysite 디렉토리 이름도 알기 쉽게 **Django** 로 변경

```
Django
    ├── config
    │   ├── __init__.py
    │   ├── settings.py
    │   ├── urls.py
    │   └── wsgi.py
    └── manage.py
```
Django 디렉토리 **Source root** 로 설정 (Pycharm에서 디렉토리가 파란색으로 변함)
- manage.py : Django 프로젝트와 다양한 방법으로 상호작용 하는 커맨드라인의 유틸리티
- settings.py : project 환경/구성 저장
- urls.py : 현재 Django project의 url선언 저장.

### 설정 변경
**config/seetings.py** 변경
```
<!-- config/seetings.py -->
# Internationalization
# https://docs.djangoproject.com/en/2.0/topics/i18n/

LANGUAGE_CODE = 'Ko-kr'

TIME_ZONE = 'Asia/Seoul'
```
**TIME_ZONE** 은 국제표준 UTC기준을 따름<BR>
서버에 저장하는 시간과 클라이언트에 따라 보여주는 것을 다르게 하기 위해 설정

### 웹 서버 시작
프로젝트 디렉토리에 **manage.py** 파일이 있어야 하며, 아래 명령어를 통해 웹 서버 실행<br>
사용하는 브라우저에서 `localhost:8000`/`127.0.0.1:8000`으로 확인 <br>
```
<!-- manage.py가 있는 command-line -->
$python manage.py runserver
```
<img src = "{{ site.url }}/assets/img/post_img/djangorunserver.png">


### 장고 모델
장고 안의 모델은 객체의 특별한 종류<br>
이 모델을 저장하면 그 내용이 **데이터베이스** 에 저장되는 것이 특별하다.
이번 튜토리얼에서는  SQLite 데이터베이스사용

### requirements.txt 생성
개발환경이 바뀔때마다 환경에 맞는 패키지를 설치하는 것은 굉장히 번거롭다.
<br> 이를 위해, **requirements.txt** 파일을 생성하여, 한번에 패키지를 받을 수 있도록 하자.
> pip freeze란?
Output installed packages in requirements format.<br>
packages are listed in a case-insensitive sorted order.

```
<!-- requirements.txt 생성 -->
$pip freeze
Django==2.0.1
pytz==2017.3
$pip freeze > requirements.txt

<!-- 이후 패키지를 설치하고 싶다면.. -->
$pip install -r requirements.txt
```

### 어플리케이션 만들기
프로젝트 내부에 별도의 애플리케이션 생성
```commend
<!-- commend-line -->
$python manage.py startapp blog

<!-- tree로 확인한 디텍토리와 파일들  -->
.
├── Django
│   ├── blog
│   │   ├── admin.py
│   │   ├── apps.py
│   │   ├── __init__.py
│   │   ├── migrations
│   │   │   └── __init__.py
│   │   ├── models.py
│   │   ├── tests.py
│   │   └── views.py
│   ├── config
│   │   ├── __init__.py
│   │   ├── __pycache__
│   │   │   ├── __init__.cpython-36.pyc
│   │   │   ├── settings.cpython-36.pyc
│   │   │   ├── urls.cpython-36.pyc
│   │   │   └── wsgi.cpython-36.pyc
│   │   ├── settings.py
│   │   ├── urls.py
│   │   └── wsgi.py
│   ├── db.sqlite3
│   └── manage.py
└── requirements.txt
```

애플리케이션 생성 후, 장고에 사용한다는 것을 알리기 위해 <br>
**config/setting.py** 에 `blog`추가 및 주석으로 분리

```
# Application definition
INSTALLED_APPS = [
    // 기본앱
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    //써드파티앱

    //커스텀앱
    'blog',
]
```

### 블로그 글 모델 만들기
**blog/models.py** 내용 추가

```py
# blog/models.py
from datetime import timezone
from django.db import models

# Create your models here.
# models.Model 데이터베이스 형태로 바꿔줌
class Post(models.Model):
    # 테이블 컬럼
    author = models.ForeignKey(
        'auth.User',
        on_delete=models.CASCADE,
    )
    title = models.CharField(max_length=200)
    content = models.TextField(blank=True)
    created_date = models.DateTimeField(
          default=timezone.now
          )
    published_date = models.DateTimeField(
          blank=True, null=True
          )


    def publish(self):
        self.published_date = timezone.now()
                # 데이터베이스에 기록을 하는 것 save()
        self.save()

    def __str__(self):
        return self.title
```

### 데이터베이스에 모델을 위한 테이블 만들기
```
<!-- commend-line -->

$./manage.py makemigrations

Migrations for 'blog':
  blog/migrations/0001_initial.py
    - Create model Post

$./manage.py migrate  
    - db.sqlite3 파일에 테이블이 생성됨
```
migrate 적용 후에, Sqlector에서 테이블 생성된 것을 확인 할 수 있음.  

### migration의 생성과 적용
**1. 맨 처음 Model구성시**<br>
1-1. 클래스 구성<br>
class Post(models.Model):<br>
    title = models.CharField(max_length=200)<br>
1-2. migration생성<br>
manage.py makemigrations<br>
    0001_initial.py <- 테이블을 생성하기 위한 로직
1-3. migration을 적용<br>
manage.py migrate<br>
    db.sqlite3 <- 이 파일에 테이블이 생성됨<br>

**2. 이미 있던 Model을 수정시**<br>
2-1. 클래스의 속성 변경<br>
class Post(models.Model):<br>
    ....
    created_date = models.DateTimeField(auto_now_add=True)<br>
2-2. migration생성<br>
manage.py makemigrations<br>
    0002_add_field_created_date.py <- 테이블에 새 column을 추가하는 로직<br>
2-3. migration을 적용<br>
manage.py migrate<br>
    db.sqlite <- blog_post테이블에 새 column 'created_date'가 생성
<br>

### admin 페이지에서 blog확인하기
**admin.py** 페이지 수정
```
<!-- admin.py -->
from django.contrib import admin
from blog.models import Post

admin.site.register(Post)
```
만들어진 blog 앱을 확인하기 위해서, `localhost:8000/admin`으로 들어간다.<br> 로그인을 해야만 확인이 가능하므로, admin 계정을 생성

```
$./manage.py createsuperuser

Username (leave blank to use 'zoe'): zoe
Email address: // 굳이 안해도 됨.
Password:
Password (again):
Superuser created successfully.
```
 `localhost:8000/admin` 에서 글을 추가 할 수 있다!
 테스트로 5개 정도 업로드

 <!-- 장고 admin 페이지 이미지 넣기  -->
