---
layout: post
title: "Django REST Framework - Authentication"
date: 2018-03-19  00:00:00
img:
tags: [Django_REST]
---
>

---

### Authentication
- `request.user` 는 보통 `contrib.auth` 패키지의 User 클래스이다. 추가 인증 정보로도 사용한다. 예로 요청으로 할당 된 인증 토큰을 나타낼 때.
-  `request.auth` : 주어진 인증 정보들이 포함되어있다. (어떤 방식으로 인증했는가 이런거)

#### 인증 결정 방법
- 인증 스키마는 항상 클래스의 리스트 형태로 할당 되어진다. 그래서 리스트의 각 클래스가 인증되면 반환 값으로 `request.user` 그리고 `request.auth` 를 설정한다.

#### 인증 스키마 설정
- 순차적으로 진행

```py
# 전역 선정
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.BasicAuthentication',
        'rest_framework.authentication.SessionAuthentication',
    )
}

# APIView 클래스 이용해서 선언
# 특정 뷰에서 사용하고 싶을 때
from rest_framework.authentication import SessionAuthentication, BasicAuthentication
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from rest_framework.views import APIView

class ExampleView(APIView):
    authentication_classes = (SessionAuthentication, BasicAuthentication)
    permission_classes = (IsAuthenticated,)

    def get(self, request, format=None):
        content = {
            'user': unicode(request.user),  # `django.contrib.auth.User` instance.
            'auth': unicode(request.auth),  # None
        }
        return Response(content)

# 데코레이터로 선언
@api_view(['GET'])
@authentication_classes((SessionAuthentication,BasicAuthentication))
@permissions_classes((IsAuthenticated,))

def example_view(request, format=None):
  content = {
    'user':unicode(request.user)
    'auth':unicode(reuqest.auth)
  }

  return Response(content)

```

### 인증 권한이 없는 경우와 서버가 허용하지 않는 경우
- 인증 권한이 없는 요청은 크게 두가지 에러 코드로 표현된다.
  1. HTTP 401 Unauthorized : `WWW-Authenticate` 인증 방법을 제시하는 헤더를 항상 포함하고 있다.
  2. HTTP 403 Permission Denied : 헤더를 가지고 있지 않다.
- 여러 인증 스키마에서 사용 될 수 있지만, 뷰의 첫번째 클래스에 응답을 결정할 때는 오직 하나의 타입만 사용되어진다.
- 요청이 성공적으로 인증이 되어도, 권한 허가가 없다면, `403 Permission Denied`로 응답할 것이다.

---

## API Reference
### BasicAuthentication
- 이 인증 스키마는 테스트를 할때만 주로 사용하며 유저의 이름과 비밀번호를 확일 할때 `HTTP Basic Authentication`을 사용한다.
- 인증이 성공하면, `request.user`는 장고 `User`인스턴스가 할당 되고 `request.auth`는 `None`으로 할당된다.(auth에는 추가 정보가 없다)
- 인증에 실패하면 `WWW-Authenticate`헤더와 `HTTP 401 Unauthorized` 로 응답

```py
WWW-Authenticate: Basic realm="api"
```

**`BasicAuthentication`은 `http`에서만 사용할 수 있다. 그리고 항상 유저이름과 비밀번호로 로그인을 요청하고 영구저장장치에 이를 저장하지 말자**

### TokenAuthentication
- simple tocken-base Http 인증 스키마에 사용한다. 데스크탑과 모바일에서 사용하는데 유용<br>
  1 . `TokenAuthentication` 사용하려면 `INSTALLED_APPS`에 `rest_framework.authoken`추가한다. **설정을 변경한 후에 migrate를 하자.** <br>
  1.1 INSTALLED_APPS에 넣는 경우, 토큰을 위한 데이터베이스가 생성된다. <br>

  2 . 토큰 유저를 생성<br>

```py
from rest_framework.authtoken.models import Token
token = Token.objects.create(user=...)
print token.key
```

  3 . 클라이언트가 인증받으려면 토큰 키가 인증HTTP 헤더에 포함되어야 한다.<br>

```py
Authorization: Token 9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b
```

  4 . 인증에 성공하면 `request.user`에 장고 `User`인스턴스가 할당되고 `request.auth`에는 `rest_framework.authtoken.models.Token`인스턴스가 할당된다.<br>
  4-1. 반면 인증에 실패하는 경우엔 `WWW-Authenticate`헤더와 `HTTP 401 Unauthorized` 로 응답<br>

```py
WWW-Authenticate: Token
```

#### Generating Token
- 모든 사용자가 자동으로 생성된 토큰을 갖게 하기 위해선, `post_save` 시그널을 사용하자

```py
from django.conf import settings
from django.db.models.signals import post_save
from django.dispatch import receiver
from rest_framework.authtoken.models import Token

@receiver(post_save, sender=settings.AUTH_USER_MODEL)
def create_auth_token(sender, instance=None, created=False, **kwargs):
    if created:
        Token.objects.create(user=instance)
```
- 이미 생성된 유저가 있다면 아래와 같은 방법으로 토큰을 생성 할 수 있다.

```py
from django.contrib.auth.models import User
from rest_framework.authtoken.models import Token

for user in User.objects.all():
    Token.objects.get_or_create(user=user)
```

- `TokenAuthentication`은 사용자에게 유저이름과 패스워드가 있는 토큰을 제공한다. 이를 위해 `obtain_auth_token`뷰를 추가하자

```py
from rest_framework.authtoken import views
urlpatterns += [
    path('api-token-auth/', views.obtain_auth_token)
]
```

- 올바른 유저이름과 패스워드를 `POST`방식으로 요청하면 JSON형태로 데이터를 되돌려 준다.
- 관리자 인터페이스를 사용해서 토큰을 생성할 수 있다. 만약 유저가 많다면, 사용자 필드를 `raw_field`로 선언하여 `TokenAdmin` 클래스를 필요에 따라 커스터마이징 해서 사용하는 것을 추천한다.

```py
from rest_framework.authtoken.admin import TokenAdmin

TokenAdmin.raw_id_fields = ('user',)
```

#### Using Django manage.py command

```console
>> ./manage.py drf_create_token <username>
```
