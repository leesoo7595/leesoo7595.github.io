---
layout: post
title: "Project 카카오톡 챗봇(세종대 도서 검색)-3"
date: 2018-06-05 00:00:00
img:
categories:
- Project
tags: [Project]
---
## 프로젝트 이야기
프로젝트를 진행하면서 발생한 오류 및 여러가지 일들 정리

---
## 0604
`Toy 프로젝트 이야기` <br>
어제 새벽 카톡으로 서버에 500에러가 발생했다고 연락이 왔다. 그래서 eb_ssh로 nginx/access.log에서 500에러를 확인은 했지만 어떤 요청을 했는지는 알 수 없었다. 믿었던 Sentry에는 아무런 메시지가 오지 않았다. 결국 Sentry 연결을 다시 확인했고 키워드 생성 시간을 추가했다. 앞으로 발생되는 오류에 대해선 내가 반드시 볼 수 있도록 설정하는게 얼마나 중요한지 깨닫게 된... 사건 그리고 사용자는 내가 원하는데로 요청하지 않는다는 걸 절실히 느끼고 있다.<br>

### 추가사항
sentry에 500에러에 대한 내용이 이메일로 오지 않았기 때문에 eb_ssh에서 **nginx/access.log** 를 확인해봤다. 그러다 발견한 `/friend/` 404 에러. 이부분은 친구 추가/삭제 할때 카카오톡 api에서 발생되는 부분이었다. 그래서 이러한 404 에러를 바꾸기 위해 url과 views에 이 부분을 추가했다. 로그를 확인하면서 print로 실행되는 문구들은 **tmp/uwsgi.log** 에서 볼 수 있다는 것도 알게 되었다.

```
/* 오류가 발생했던 로그 */
- [04/Jun/2018:13:49:54 +0000] "GET /keyboard HTTP/1.1" 301 0 "-" "KakaoTalk/Bot 2.0"
- [04/Jun/2018:13:49:54 +0000] "GET /keyboard/ HTTP/1.1" 200 51 "-" "KakaoTalk/Bot 2.0"
- [04/Jun/2018:13:50:06 +0000] "POST /friend HTTP/1.1" 404 80 "-" "KakaoTalk/Bot 2.0"
- [04/Jun/2018:13:50:15 +0000] "POST /message HTTP/1.1" 200 829 "-" "KakaoTalk/Bot 2.0"
- [04/Jun/2018:13:50:28 +0000] "POST /message HTTP/1.1" 500 27 "-" "KakaoTalk/Bot 2.0"

/* 수정 후 로그 */
- [07/Jun/2018:03:08:19 +0000] "DELETE /friend/{user_key} HTTP/1.1" 301 0 "-" "KakaoTalk/Bot 2.0"
- [07/Jun/2018:03:08:19 +0000] "DELETE /friend/{user_key} HTTP/1.1" 200 2 "-" "KakaoTalk/Bot 2.0"
- [07/Jun/2018:03:08:21 +0000] "POST /friend HTTP/1.1" 200 2 "-" "KakaoTalk/Bot 2.0"
```

### 코드
```python

@csrf_exempt
def plus_friend(request):
    if request.method == 'POST':
        request.JSON = json.loads(request.body.decode('utf-8'))
        user_key = request.JSON.get('user_key', '')
        if user_key:
            user, _ = User.objects.get_or_create(
                username=user_key
            )
        return JsonResponse({})

@csrf_exempt
def delete_friend(request, user_key):
    if request.method == 'DELETE':
        print(f'{user_key}님이 친구삭제를 하셨습니다.')
        return JsonResponse({})
```

----
## 0605
`Toy 프로젝트 오류` <br>
브랜치를 나눠서 작업하지 않았던게 문제였다. `master` 내용을 `dev`에 merge하지 않으면서 초반에 생성했던 `img_profile`이란 필드가 데이터베이스가 생긴 상태에서 다시 merge를 했고, 그 결과 merge한 파일은 삭제되었으나 실질적으로 DB에 이 칼럼은 새로 생긴 것 !!! 그러나 `local` 환경에서는 오류가 없었지만 `dev/production` 환경에서는 계속해서 500에러가 발생했다.

### 해결 과정
```
IntegrityError
null value in column "img_profile" violates not-null constraint
DETAIL:  Failing row contains (59, , null, f, kaheetest_local, , , , f, t, 2018-06-05 06:05:45.112751+00, null).
```
- 서버 500에러를 보기 위해 했던 방법들
    - eb ssh /tmp/uwsgi.log
    - eb ssh /var/log/nginx/access.log
    - sentry 에러 내용(이유는 알 수 없으나 에러 메일이 조금 늦게 온다)

```console
toy_study_room=> \d+ members_user
toy_study_room=> \d+ members_user
toy_study_room=> alter table members_user drop column img_profile;
```
- DB에 생긴 'img_profile'을 지우는 방법
    - migrate 파일을 이용해서 -> 나의 경우 이 파일이 삭제 되었기 때문에 아래 방법을 사용(git commit단위를 통해 img_profile migrate 파일이 삭제된 것 확인)
    - DB에서 직접 'img_profile' 칼럼을 삭제

### 변경해야 할것들
- `local` 환경의 경우 DB를 Django 기본 설정으로 되어있는 sqlite를 사용했다. 이렇게 하는 경우 `dev/production` 환경과 다르기때문에 오류잡기가 어렵다. 그래서 로컬 환경 역시 postgresql로 변경하자
