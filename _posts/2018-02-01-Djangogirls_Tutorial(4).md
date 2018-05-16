---
layout: post
title: "Djangogirls_Tutorial(4)"
date: 2018-01-30 14:00:00
img: ./post_img/django.png
tags: [Django]
---

### 템플릿 확장하기
> 서로 다른 페이지에서 HTML의 입루를 재사용할 수 있다는 의미
> <BR>  템플릿을 확장하여 사용하면, 모든 파일마다 같은 내용을 반복해서 입력할 필요가 없으며, 유지보수가 편리하다.

### 기본 템플릿 생성하기
**templates/base.html** 파일 생성
```html
{% raw %}
{% load static %}
<!doctype html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <link rel="stylesheet" href="{% static 'bootstrap/css/bootstrap.css' %}">
    <link rel="stylesheet" href="{% static 'css/blog.css' %}">
</head>

<body>
<div class="header">
    <div class="container">
        <h1><a href="{% url 'post-list' %}">Django Girls Blog</a></h1>
    </div>
</div>
<div class="container">
    <!-- {% block content %}{% endblock %}은 HTML문서에 블럭을 지정  -->
    <!-- 비어있는 블록의 내용을 채우는 것은 '자식'템플릿이 할 일-->
    <h2>{% block title %}{% endblock %}</h2>
    {% block content %}
    {% endblock %}
</div>
</body>
</html>
{% endraw %}
```

### 기존 templates/blog/post_detail.html변경 및 base.html 연결
> 똑같은 방법으로 post_list.html 변경

```html
{% raw %}
<!-- base.html 연결 -->
{% extends 'base.html' %}
{% block content %}
<div class="container">
    <div class="post">
        <p class="post-date">created: {{ post.created_date }}</p>
        <h2 class="post-title"><a href="">{{ post.title }}</a></h2>
        <p class="post-content full">{{ post.content|linebreaksbr}}</p>
    </div>
</div>
{% endblock %}
{% endraw %}
```
