---
layout: post
title: "Django REST Framework -  Tutorial"
date: 2018-03-19  00:00:00
img:
tags: [Django_REST]
---
> Django Rest Framework 공식문서 참고 <br>
> [raccoony's cave님 블로그](http://raccoonyy.github.io/drf3-tutorial-1/) 참고

---

## 1장
### 직렬화란 무엇일까?


---

## 2장 요청과 응답
### 요청 객체
- request.POST : 폼 데이터만 다루며, 'POST' 메서드에서만 사용 가능
- request.data : 아무 데이터에서 사용이 가능하며, 'POST','PUT','PATCH' 메서드에서도 사용 가능
### 응답 객체
- `Response(data)`: 클라이언트가 요청한 형태로 콘텐트를 렌더링
### 상태 코드

### Parser
- REST Framework에서는 여러가지 유형의 요청을 받을 수 있다.
  - JSONParser/FormParser/MultiPartParser/Custom parsers/Third party packages(YAML, XML,MessagePack,CamelCase JSON)
  - 튜토리얼에서는 JSONParser을 사용한다 `form-data`경우 트리 구조를 갖는 데이터를 가져오는데 어려움이 있기 때문에

---

## 3장
### format_suffix_patterns
- url 패턴에 추가된 형식을 url 패턴 목록을 반환
- 함수에 `format`인자 없이 url에 query parameter 형태로 보내줘도 된다.

```py
# settings.py
# xml 추가
REST_FRAMEWORK = {
    'DEFAULT_PARSER_CLASSES': (
        'rest_framework.parsers.JSONParser',  'rest_framework.renderers.BrowsableAPIParser',
        'rest_framework_xml.parsers.XMLParser',
    ),
    'DEFAULT_RENDERER_CLASSES': (
        'rest_framework.renderers.JSONRenderer',
        'rest_framework.renderers.BrowsableAPIRenderer',
        'rest_framework_xml.renderers.XMLRenderer',
    ),
}
```

```console
url : http://localhost:8000/snippets/1.xml/
    : http://localhost:8000/snippets/1/?format=xml

<?xml version="1.0" encoding="utf-8"?>
<root>
    <list-item>
        <id>1</id>
        <title></title>
        <code>hello</code>
        <linenos>False</linenos>
        <language>python</language>
        <style>friendly</style>
    </list-item>
</root>

```

### url 모듈 패키지화
**오류가 난다면, 서버를 껐다가 다시 켜서 해보자**

### mixins/generic
[generic](http://www.django-rest-framework.org/api-guide/generic-views/#get_querysetself)


---

## 4장
### migrate 오류

```console
admin.0001_initial is applied before its dependency members.0001_initial on database 'default'.
```
- db를 삭제하고 다시 migrage를 했음

### ReadOnlyField
- 직렬화에 사용되었을 땐, 읽기 전용이므로 인스턴스를 업데이트 할 때 사용 불가

```py
owner = serializers.ReadOnlyField(source='owner.username')
# 같은 의미
CharField(read_only=True)
```

### HTTP 인증 - basic
[Basic 인증](http://devidea.tistory.com/entry/HTTP-%EC%9D%B8%EC%A6%9D1-Basic)
- 유저 이름과 패스워드로 인증을 한다.
- 유저이름과 패스워드를 Authorization헤더는 `유저이름:패스워드`로 구성되어있고 base64 인코딩한 문자열이 된다.
- https가 아니면 평문과 같기 때문에 사용하면 안된다. 그냥 테스트를 위해 사용하는 것
- Session_id : 장고에서 관리하는 값

### TokenAuthentication
request -> SessionMiddleware(django_sessions) -> request.user -> views


1. 클라이언트가 HTTP 'Authorization' Header에 Token <token vlaue>를 담아서 보내면, 해당 Token을 분석해서 request.user를 할당
2. 특정 유저가 Token을 요청 -> 서버는 해당 User의 authentication에 성공하면 Token을 생성, 반환
<br> -> 유저는 받은 토큰을 자신의 저장소에 보관하고 매 request마다 해당 토큰을 Header에 담아서 보냄
<br> -> 서버는 받은 request에 'Token'과 관련된 Header가 있는지 검사 후, `request.user` 할당
<br>

**토큰의 장점** : 특정 권한만 줄 수 있다. 그리고 만약 인증시스템에 있는 토큰이 유출됐을 경우, 토큰만 폐기하고 다시 발급하면 된다. 또한 세션의 경우 브라우저 환경에서만 사용되지만, 토큰은 앱과 브라우저 모두 사용가능하다 <br>

### permissions.SAFE_METHODS
- GET,HEAD, OPTIONS 요청은 항상 허용함
  - head : header 만 요청하는 것
  - option : 어떤 옵션을 사용할 수 있는지에 대해 알려줌

----

## 5장
### reverse
- `urlname`을 통해 `url`을 반환해준다.

```py

class APIRoot(APIView):
    def get(self, request, format=None):
        data = {
            'users': reverse('user-list', request=request, format=format),
            'snippets': reverse('snippet-list', request=request, format=format)
        }
        return Response(data)

# result
{
    "users": "http://localhost:8000/users/",
    "snippets": "http://localhost:8000/snippets/generic/"
}

```
### renderers.StaticHTMLRenderes
- html 형식으로 렌더링해주는 함수

```py
class SnippetHighlight(generics.GenericAPIView):
    queryset = Snippet.objects.all()
    renderer_classes = (
        renderers.StaticHTMLRenderer,
    )

    def get(self, request, *args, **kwargs):
        snippet = self.get_object()
        return Response(snippet.highlighted)
```

### Serializer relations

```py
# example models
class Album(models.Model):
    album_name = models.CharField(max_length=100)
    artist = models.CharField(max_length=100)

class Track(models.Model):
    album = models.ForeignKey(Album, related_name='tracks', on_delete=models.CASCADE)
    order = models.IntegerField()
    title = models.CharField(max_length=100)
    duration = models.IntegerField()

    class Meta:
        unique_together = ('album', 'order')
        ordering = ['order']

    def __unicode__(self):
        return '%d: %s' % (self.order, self.title)
```

- PrimaryKeyRelatedField : 기본키를 사용해 관계의 대상을 나타낸다.

```py
class AlbumSerializer(serializers.ModelSerializer):
    tracks = serializers.PrimaryKeyRelatedField(many=True, read_only=True)

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'tracks')
```

- HyperlinkedRelatedField : 하이퍼링크를 사용하여 다른 인스턴스와의 관계를 나타내는데 사용한다.
- HyperlinkedIdentityField : 하이퍼링크를 사용하여 대상을 나타낸다. 이때 현재 객체 자체의 하이퍼링크이다.

[참고 자료](https://stackoverflow.com/questions/37737553/django-rest-framework-hyperlinked-fields-understanding)

```py
class AlbumSerializer(serializers.ModelSerializer):
    tracks = serializers.HyperlinkedRelatedField(
        many=True,
        read_only=True,
        # 관계의 대상을 나타내는데 사용되는 뷰 이름
        view_name='track-detail'
    )

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'tracks')

# result
{
    'album_name': 'Graceland',
    'artist': 'Paul Simon',
    'tracks': [
        'http://www.example.com/api/tracks/45/',
        'http://www.example.com/api/tracks/46/',
        'http://www.example.com/api/tracks/47/',
        ...
    ]
}
```

### pagination
- 페이지 갯수 설정
- 설정 후, `results`에 설정한 갯수만큼만 데이터가 전달된다.

```py
# http://localhost:8000/snippets/generic/?page=1&page_size=2
# page_size의 경우, max_page_size까지.
{
    "count": 7,
    "next": "http://localhost:8000/snippets/generic/?page=2",
    "previous": null,
    "results": [
        {
            "id": 2,
            "title": "",
            "code": "from django.apps import AppConfig\r\n\r\n\r\nclass MembersConfig(AppConfig):\r\n    name = 'members'",
            "linenos": false,
            "language": "python",
            "style": "friendly",
            "owner": "kahee",
            "highlight": "http://localhost:8000/snippets/generic/2/highlight/"
        },
        {
            "id": 3,
            "title": "",
            "code": "from django.contrib.auth.models import AbstractUser\r\nfrom django.db import models\r\n\r\n# Create your models here.\r\nclass User(AbstractUser):\r\n    pass",
            "linenos": false,
            "language": "python",
            "style": "friendly",
            "owner": "kahee",
            "highlight": "http://localhost:8000/snippets/generic/3/highlight/"
        }
    ]
}
```

----

## 6장
### dispatch
- ViewSet의 경우, 모든 함수들을 다 가지고 있다 .그래서 `get` 요청이 들어왔을 때, 이게 List인지 Retrieve인지 어떻게 구분할까? 바로 `dispatch()`함수를 이용하여 요청을 처리한다.
