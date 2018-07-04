---
layout: post
title: "Project 카카오톡 챗봇(세종대 도서 검색)-5"
date: 2018-07-03 00:00:00
img:
categories:
- Project
tags: [Project]
---

지난글에선 챗봇의 문제점이었던 **속도 개선** 을 위해 Celery와 Redis에 대해 정리해봤다. 이번글에선 배포환경에 이를 적용하기위해 진행했던 과정들을 정리하려 한다.

---
## Docker 오류

```console
OperationalError
could not connect to server: Connection refused
	Is the server running on host "postgres" (172.20.0.2) and accepting
	TCP/IP connections on port 5432?
```
프로젝트 배포 당시 **local환경** 에서 사용했던 Dockerfile을 build하고 run하는 순간 위와 같은 메시지를 보았다. local 환경도 dev/production환경과 같은 **postgreSQL** 로 변경해서 발생된 문제였다. 이를 해결하기 위해 처음엔 psql을 설치하는 명령어를 Dockerfile에 넣어서 다시 build하려고 했으나 좀더 편하게 관리 할 수 있는 방법을 알게 되었다.

## Docker Compose
 <img src="{{ site.url }}/assets/post_img/docker_compose.png">
[이미지출처](http://raccoonyy.github.io/docker-usages-for-dev-environment-setup/)<br>
Docker Compose란 컨테이너 여러개를 띄우는 도커 애플리케이션을 정의하고 실행하는 도구다. 즉 두개의 도커 이미지를 실행시키고 django 프로젝트 도커 컨테이너를 psql 컨테이너로 연결해서 사용하는 것이다.
### Docker Compose 사용하기
[참고:raccoony님 블로그](http://raccoonyy.github.io/docker-usages-for-dev-environment-setup/)<Br>
영문으로도 검색해보고 이리저리 찾아봤지만 위 참고 블로그가 설명도 잘 되어있고, 따라하기도 쉽다.

## Docker 배포
배포 환경의 경우, AWS S3를 사용하기 때문에 Docker compose를 사용하지 않아도 됐다. 대신 nginx가 supervisor를 사용하여 백그라운드에서 실행되는 것처럼 Redis와 Celery 역시 백그라운드에서 실행되게하여 한개의 도커 컨테이너에서 모든 것이 실행되면 된다.

```
<!-- supervisor.conf -->
[program:redis]
command=redis-server
autostart=true
stdout_logfile=/var/log/redis/%(program_name)s.log
stderr_logfile=/var/log/redis/%(program_name)s_err.log

[program:celeryd]
directory=/srv/project/app/
command=celery -A config worker -l info
autostart=true
autorestart=true
startsecs=10
stopwaitsecs=600
stdout_logfile=/var/log/celery/%(program_name)s.log
stderr_logfile=/var/log/celery/%(program_name)s_err.log
```

----

## ElastiCache
> Redis용 Amazon ElastiCache 사용 설명서를 시작합니다. ElastiCache는 클라우드에서 분산된 인 메모리 데이터 스토어 또는 캐시 환경을 손쉽게 설정, 관리 및 확장할 수 있는 웹 서비스입니다. 이 서비스는 분산된 캐시 환경의 배포 및 관리와 관련된 복잡성을 제거하면서 고성능, 확장 가능 및 비용 효율적인 캐싱 솔루션을 제공합니다.

### 근데 왜 써야 하죠?
배포 환경에서 로컬 redis가 잘 작동하는데 왜 ElastiCache를 사용해야 하는가에 대한 의문이 있었다. 물어본 결과 AWS EB가 배포 될때마다, 이전 redis 로그가 사라지기 때문에 문제가 될 수 있다고 한다. 그리고 ElastiCache는 로컬 환경에서는 테스트가 힘들기 때문에 EB나 EC2환경에서 ElastiCache의 Redis가 잘 작동하는지 확인하고 프로젝트에 redis 서버 주소를 적용시켜야 한다.

## ElastiCache 생성 및 작동 확인
### 주의 할 점
<img src="{{ site.url }}/assets/post_img/redis-server.png">
<img src="{{ site.url }}/assets/post_img/redis-server2.png">
<br>
- 기본 설정 되어있는 노드 유형 `cache.r4.large`를 `cache.r4.large`으로 변경해야만 초과 요금이 없다. 나의 경우 이부분을 잘 보지 않아 요금을 내야 하는 상황이다ㅜㅜ
- 작은 프로젝트의 경우 복사본을 없음으로 지정해도 괜찮다. 복사본이 있을 경우 상당수 프리티어가 제공하는 시간을 초과하여 추가 요금이 청구되는 경우가 많다.

### 동작 확인
EB나 EC2 인스턴스에 접속하여 아래와 같이 접속후 `MONITOR`를 입력하면 ElastiCache 가 잘 동작하는지 알 수 있다.

```console
[ec2-user@ip-172-31-27-153 ~]$ telnet {ElastiCache엔드포인트} 6379
Trying 172.31.19.49...
Connected to redis-project.2bfocc.0001.apn2.cache.amazonaws.com.
Escape character is '^]'.
MONITOR
+OK
```

### ElastiCache 적용하기
로컬환경에서 Redis와 Celery가 잘 동작하는지 확인후엔 ElastiCache의 Redis를 붙이면 된다. 이때 `CELERY_BROKER_URL`과 `CELERY_BROKER_BACKEND`만 ElastiCache 엔드포인트로 지정하고 배포하면 끝!
<br>
[참고:darkblank's blog](https://darkblank.github.io/development/Elasticache/)
<br>
[참고:야생강아지 WILDPUP](http://wildpup.cafe24.com/archives/1053)

```python
# config/settings/production,py

# local 환경에서 사용
# CELERY_BROKER_URL = 'redis://localhost:6379/0'
# CELERY_RESULT_BACKEND = 'redis://localhost:6379/0'

CELERY_BROKER_URL = 'redis://' + AWS_ELASTIC_CACHE
CELERY_RESULT_BACKEND = 'redis://' + AWS_ELASTIC_CACHE

# celery.py
import os
from celery import Celery

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "config.settings.production")
app = Celery('config')
# namespace 지정 CELERY_시작할때만 Settings에서 가져서와서 사용하도록 지정
app.config_from_object('django.conf:settings', namespace='CELERY')
app.autodiscover_tasks()

@app.task(bind=True)
def debug_task(self):
    # Calls repr() on the argument first
    print('Request: {0!r}'.format(self.request))
```

-----

## 결과
**수정전** 에는 도서 검색 키워드가 입력되면 모든 크롤링 함수가 진행된 후에 검색 결과를 볼 수 있었다. 그렇기 때문에 검색 결과가 많은 경우 속도가 느렸다. **수정후** 에는 검색 결과에 필요한 정보들은 바로 출력하고 이 외 DB저장 실행은 Celery를 이용하기 때문에 속도가 확실히 빨라졌다.
<br>수정전 코드의 경우 크롤링하는 과정에서 에러가 생기면 (Data가 max_length가 오버하는 경우) 모든 동작이 멈추기 때문에 **서버 에러**가 발생되었다. 그러나 수정후 위와 같은에러가 발생해도 해당 책 정보 저장만 문제가 있을 뿐 전체 코드에는 영향을 주지 않는다는 점이 가장 좋다.

### 수정전 진행

```console
도서 검색
    > 상세 검색: search_book_detail()
    > 제목 검색: search_book_title()
                            > 도서 검색 리스트: get_book_lists()           - requests요청
                            - 1)도서 상세 정보: get_book_detail(book_id)   - requests요청/DB 저장
                            - 2)도서 위치 정보: get_book_location(book인스턴스)  - requests요청/DB 저장
                                                                                                > 검색 결과
```

### 수정후 진행

```console
도서 검색
    > 상세 검색: search_book_detail()
    > 제목 검색: search_book_title()
                            > 도서 검색 리스트: get_book_lists()     - requests요청
                              - 0)도서 인스턴스 생성 book(book_id)                                   > 도서 검색 결과
                          ======================================================================================
                            > celery
                              - 1)도서 상세 정보: get_book_detail(book_id) - requests요청/DB 저장
                              - 2)도서 위치 정보: get_book_location(book_id,그외 상세) - requests요청/DB 저장
```

### 실행시간 차이

```console
1. Test
입력 키워드 : 호주
수정전 실행시간 : 0:00:46.297935
수정후 실행시간 : 0:00:01.061140

2. Test
입력 키워드 : 컴퓨터 구조론
수정전 실행시간 : 0:00:03.994295
수정후 실행시간 : 0:00:01.063694

3. Tset
입력 키워드 : 파이썬
수정전 실행시간 : 0:00:03.985461
수정후 실행시간 : 0:00:00.921255

4. Test
입력 키워드 : 토지
수정전 실행시간 : 0:00:07.549409
수정후 실행시간 : 0:00:02.050128
```
