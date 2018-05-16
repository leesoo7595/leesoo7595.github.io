---
layout: post
title: "Django- Facebook_login"
date: 2018-03-24 00:00:00
img:
tags: [Django]
---
>본 문서는 패스트캠퍼스 'Web-Programming School' 수업 자료를 바탕으로 작성되었습니다.

---

### OAuth
[OAuth 참고 사이트](http://d2.naver.com/helloworld/24942)<br>
> OAuth는 인증을 위한 오픈 스탠더드 프로토콜로, 사용자가 Facebook이나 트위터 같은 인터넷 서비스의 기능을 다른 애플리케이션(데스크톱, 웹, 모바일 등)에서도 사용할 수 있게 한 것이다.

#### OAuth 와 로그인
OAuth와 로그인 OAuth와 로그인은 반드시 분리해서 이해해야 한다. 일상 생활을 예로 들어 OAuth와 로그인의 차이를 설명해 보겠다.<br>
사원증을 이용해 출입할 수 있는 회사를 생각해 보자. 그런데 외부 손님이 그 회사에 방문할 일이 있다. 회사 사원이 건물에 출입하는 것이 로그인이라면 OAuth는 방문증을 수령한 후 회사에 출입하는 것에 비유할 수 있다.<br>

다음과 같은 절차를 생각해 보자.

```console
1. 나방문씨(외부 손님)가 안내 데스크에서 업무적인 목적으로 김목적씨(회사 사원)를 만나러 왔다고 말한다.
2. 안내 데스크에서는 김목적씨에게 나방문씨가 방문했다고 연락한다.
3. 김목적씨가 안내 데스크로 찾아와 나방문씨의 신원을 확인해 준다.
4. 김목적씨는 업무 목적과 인적 사항을 안내 데스크에서 기록한다.
5. 안내 데스크에서 나방문 씨에게 방문증을 발급해 준다.
6. 김목적씨와 나방문씨는 정해진 장소로 이동해 업무를 진행한다.
```
위 과정은 방문증 발급과 사용에 빗대어 OAuth 발급 과정과 권한을 이해할 수 있도록 한 것이다. 방문증이란 사전에 정해진 곳만 다닐 수 있도록 하는 것이니, '방문증'을 가진 사람이 출입할 수 있는 곳과 '사원증'을 가진 사람이 출입할 수 있는 곳은 다르다. 역시 직접 서비스에 로그인한 사용자와 OAuth를 이용해 권한을 인증받은 사용자는 할 수 있는 일이 다르다.<br>
**OAuth에서 'Auth'는 'Authentication'(인증)뿐만 아니라 'Authorization'(허가) 또한 포함** 하고 있는 것이다. 그렇기 때문에 OAuth 인증을 진행할 때 해당 서비스 제공자는 '제 3자가 어떤 정보나 서비스에 사용자의 권한으로 접근하려 하는데 허용하겠느냐'라는 안내 메시지를 보여 주는 것이다.

---

### Django facebook login
<img src="{{ site.url }}/assets/post_img/OAuth2.jpg">
1. 사이트에서 '페이스북 로그인'버튼 클릭
2. 페이스북 사이트로 이동해서 사용자 인증 및 권한 인가 완료 (클라이언트 쪽)
3. 토큰이 사이트에 get매캐변수로 전달됨 (페이스북 -> 사이트 서버로 전달) // 프론트엔드
4. 사이트의 View에서 받은 토큰을 사용해서 페이스북과 인증과정 거침 (사용자 개입 없음)
5. 인증이 완료되면 사이트에서 회원가입 및 로그인 유지를 위한 처리

---

### 로그인 플로 직접 빌드
#### 로그인 대화 상자 호출 및 리디렉션 URL 설정
- login template 생성

```html
<!doctype html>
<html lang="ko-kr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Facebook 로그인</title>
</head>
<body>

<h1>페이스북 로그인</h1>
<a href="https://www.facebook.com/v2.12/dialog/oauth?client_id=423094694780567">페이스북 로그인</a>
</body>
</html>
```

- view, url 생성
- template url 클릭하면 아래와 같은 오류가 발생한다.
<img src="{{ site.url }}/assets/post_img/facebook_login1.png">
사용자가 다시 로그인하도록 리디렉션할 URL을 적어주자.

```html
......

<h1>페이스북 로그인</h1>
<a href="https://www.facebook.com/v2.12/dialog/oauth?client_id=423094694780567&redirect_uri=http://localhost:8000/members/login/">페이스북 로그인</a>

```

- 오류가 발생함 설정에서 앱 도메인에  `localhost` 등록
<img src="{{ site.url }}/assets/post_img/facebook_login2.png">
<img src="{{ site.url }}/assets/post_img/facebook_login3.png">
*https 요청하는 부분 배워야 한다..*

### secret 파일 분리
- FACEBOOK_SECRET_CODE는 `.secret`으로 분리해서 관리하자

---

### code로부터 access_token얻기
```py
# members/vies.py
import requests
from django.contrib.auth import get_user_model
from django.shortcuts import render
from config import settings

User = get_user_model()


# Create your views here.
def facebook_login(request):
    # code로부터 AccessToken 가져오기
    client_id = settings.FACEBOOK_APP_ID
    client_secret = settings.FACEBOOK_SECRET_CODE
    # 페이스북 로그인 버튼을 누른 후, 사용자가 승인하면 redirect_url에 GET parameter로 'code'가 전송됨
    # 이 값과 client_id, secret을 사용해서 Facebook서버에서 access_token을 받아와야 함
    code = request.GET.get('code')
    # 이전에 페이스북 로그인 버튼을 눌렀을 때, 'code'를 다시 전달받은 redirect_url값으로 그대로 사용
    redirect_uri = 'http://localhost:8000/members/login/'

    # 아래 엔드포인트에 get요청을 보냄
    url = 'https://graph.facebook.com/v2.12/oauth/access_token'
    params = {
        'client_id': client_id,
        'redirect_uri': redirect_uri,
        'client_secret': client_secret,
        'code': code,
    }
    response = requests.get(url, params)

    # 전송받은 json 결과값을 python 객체로 반환
    response_dict = response.json()

    for key, value in response_dict.items():
        print(f'{key}:{value}')

    return render(request, 'login.html')

```

### access_token으로 사용자 정보 가져오기
[그래프API](https://developers.facebook.com/docs/graph-api/reference)
[그래프API탐색기](https://developers.facebook.com/tools/explorer/) : 여기서 한번 실행해보고 코드에 사용하는게 편리하다.
- 사용자 액세스 토큰 받기 클릭 후, 제출하면 아래와 같은 이미지를 볼 수 있다. 이는 기본적으로 access_token을 통해 얻을 수 있는 유저 정보이다. 얻을 수 있는 정보는 권한 설정에 따라 요청이 달라질 수 있다.
<img src="{{ site.url }}/assets/post_img/facebook_login4.png">


### 엑세스 토큰 검사
- 앱 엑세스 토큰
- 사용자 엑세스 토큰
- `input_token` : 사용자 엑세스 토큰을 의미

### GraphAPI에 GET요청 보내기

```py
# members/views.py
.....위의 코드

    # GraphAPI의 me 엔드포인트에 Get요청 보내기

    url = "https://graph.facebook.com/v2.12/{user_id}"
    params = {
        'access_token': response_dict['access_token'],
        'fields': ','.join([
            'id',
            'name',
            'picture.width(2500)',
        ])
    }

    response = requests.get(url, params)
    response_dict = response.json()
    print(response_dict)
    return render(request, 'login.html')
```

### Get요청으로 받은 정보로 User 생성

```py

    # Get요청으로 받은 정보
    # 애플리케이션 별로 사용자에게 주어지는 id
    facebook_id = response_dict['id']
    name = response_dict['name']
    url_picture = response_dict['picture']['data']['url']

    # login 기능 추가필요함
    # Facebook_id가 username인 User가 존재할 경우
    if User.objects.filter(username=facebook_id):
        user = User.objects.get(username=facebook_id)

    # 존재하지 않는 경우
    else:
        user = User.objects.create_user(
            username=facebook_id,
        )

    return render(request, 'login.html')

```


----


### facebook_id로 유저 인증을 위한 backends
- 커스텀 백엔드를 사용하기 위해 `settings.py`에 등록하자

```py
# config/settings.py

AUTHENTICATION_BACKENDS = [
    'django.contrib.auth.backends.ModelBackend',
    'members.backends.FacebookBackend',
]

```
- `get_user` 함수는 항상 들어가야한다. (무슨 역할 하는지 찾아볼것)

```py

# memebers/backends.py
import requests
from django.contrib.auth import get_user_model
from config import settings

User = get_user_model()


class FacebookBackend:
    CLIENT_ID = settings.FACEBOOK_APP_ID
    CLIENT_SECRET = settings.FACEBOOK_SECRET_CODE
    URL_ACCESS_TOKEN = 'https://graph.facebook.com/v2.12/oauth/access_token'
    URL_ME = "https://graph.facebook.com/v2.12/me"

    def authenticate(self, request, code):
        def get_access_token(auth_code):
            """
            유저가 페이스북에서 우리 애플리케이션의 사용에 대해 '승인'한 경우,
            페이스북에서 우리 애플리케이션의 주소(redirect_uri)에 'code'라는 GET parameter로 전해주는
            인증 코드 (auth_code)를 사용해서
            페이스북 GraphAPI에 access_token요청, 결과를 가져와 리턴
            :param auth_code: 유저가 페이스북에 로그인/앱 승인한 결과로 돌아오는 'code' GET parameter
            :return:
            """
            redirect_uri = 'http://localhost:8000/members/login/'
            # 아래 엔드포인트에 GET요청을 보냄
            params_access_token = {
                'client_id': self.CLIENT_ID,
                'redirect_uri': redirect_uri,
                'client_secret': self.CLIENT_SECRET,
                'code': auth_code,
            }
            response = requests.get(self.URL_ACCESS_TOKEN, params_access_token)
            # 전송받은 결과는 JSON형식의 텍스트. requests가 제공하는 JSON 디코더를 사용해서
            # JSON텍스트를 Python dict로 변환해준다
            response_dict = response.json()
            return response_dict['access_token']

        def get_user_info(user_access_token):
            """
            User access token을 사용해서
            GraphAPI의 'User'항목을 리턴
                (엔드포인트 'me'를 사용해서 access_token에 해당하는 사용자의 정보를 가져옴)
            :param user_access_token: 정보를 가져올 Facebook User access token
            :return: User정보 (dict)
            """
            params = {
                'access_token': user_access_token,
                'fields': ','.join([
                    'id',
                    'name',
                    'picture.width(2500)',
                    'first_name',
                    'last_name',
                ])
            }
            response = requests.get(self.URL_ME, params)
            response_dict = response.json()
            return response_dict

            # try:

        access_token = get_access_token(code)
        user_info = get_user_info(access_token)

        facebook_id = user_info['id']
        name = user_info['name']
        url_picture = user_info['picture']['data']['url']

        try:
            user = User.objects.get(username=facebook_id)
        except User.DoesNotExist:
            user = User.objects.create_user(
                username=facebook_id,
            )
        return user
        # except Exception:
        #     return None

    def get_user(self, user_id):
        try:
            return User.objects.get(pk=user_id)
        except User.DoesNotExist:
            return None

```

### facebook login 뷰 수정
- 앞서 만든  `backends.py`를 통해 code를 넘겨주면 access_token을 받고 이를 통해 유저의 정보도 얻을 수 있는 함수를 구현하였다. 그렇기 때문에 이전에 작성했던 `facebook_login` 함수를 수정하자

```py
# memebers/views.py

def facebook_login(request):
    # GET paramter가 왔을 것으로 가정
    code = request.GET.get('code')
    #  우리가 만든 backends가 실행됨
    user = authenticate(request, code=code)
    return redirect('index')

```
