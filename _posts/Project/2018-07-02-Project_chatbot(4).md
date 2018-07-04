---
layout: post
title: "Project 카카오톡 챗봇(세종대 도서 검색)-4"
date: 2018-07-02 00:00:00
img:
categories:
- Project
tags: [Project]
---

## 카카오톡 챗봇 문제점
베타서비스를 통해 약 40명의 사용자들이 챗봇을 사용했다. 물론 친구추가 초반에만 검색을 요청할 뿐, 하루에 한두건 정도만 검색 요청이 있다. 시간이 지나 서비스의 문제점들이 보이기 시작했다. 그중 가장 신경쓰이는 `검색 결과 속도`였다. 크롤링 결과가 많을 수록 카카오톡 챗봇 답장 속도 역시 느려졌다. 이를 해결하기 위해 '어떤 방법이 있는가'에 대해 조언을 구한 결과 **비동기 프로그래밍** 이라는 답을 얻을 수 있었다.

----

## 비동기, 동기 그게 뭔데?
 <img src="{{ site.url }}/assets/post_img/Synchronous_asynchronous.jpg">
[이미지 출처](http://www.haninsociety.com/index.php?mid=blogO&page=612&document_srl=322098)<br>
학부생시절 정확하게 어떤식으로 동작하는지 이해하지 못했다. 이번 기회에 이 두가지를 정확하게 알아보자!
- 동기식 처리 : 요청이 들어오면 순차적으로 작업을 끝내고 다음 작업을 진행
- 동기식 처리 장단점 : 설계가 간단하고 직관적이며, 결과가 있을 때까지 대기
- 비동기식 처리 : 순차적으로 작업을 진행하지 않고, 각자 맡은 일에 대해서 작업을 진행하여 요청과 동시에 결과가 동시에 일어나지 않음
- 비동기식 처리 장단점 : 설계가 동기식보다 복잡하고 결과가 주어지는데 시간이 걸려도 동시에 다른 작업을 진행할 수 있어서 자원을 효율적으로 사용 할 수 있다.
<br>
[출처:공부해서 남 주자](http://private.tistory.com/24)
## Celery 그리고 Redis
<img src="{{ site.url }}/assets/post_img/django_celery.png">
[이미지 출처](http://whatisthenext.tistory.com/127)<br>
**celery** 는 메시지 브로커(message broker)와 python 작업 프로세스를 연결해서 비동기 작업을 수행할 수 있는 시스템을 제공한다. 메시지 큐 시스템에 작업을 전달해주는 시스템인 메시지 브로커가 필요하다. 이번 프로젝트에선 **Redis** 를 이용했다.
### 메시지 브로커 필요성
비동기 작업이 필수적인 곳이 있는데, 바로 은행 전산 시스템이다. 전국의 은행 지점에서 돈이 오고 가는 것이 전산적으로 어딘가의 중앙 시스템에 기록이 된다. 동시다발적으로 거래가 일어날 텐데, 전산 작업 중 일부가 충돌해서 문제가 발생한다면 은행 입장에서는 재앙과 같은 일이 된다. 이런 위험을 해소해주는 것이 메시지 브로커를 이용한 비동기 작업 큐를 구축하는 것이다.
<br>
[출처:개발새발 블로그](http://dgkim5360.tistory.com/entry/python-celery-asynchronous-system-with-redis)

### Celery, Redis 따라해보기
프로젝트 코드에 바로 적용하기 전에, celery 튜토리얼을 통해서 어떤식으로 동작하는지 파악했다. 물론 이때까지만 해도 그냥 코드를 따라 치는 수준이었고 비동기로 이루어지는 것을 완벽하게 이해하기엔 많은 시간이 필요했다. ㅋㅋㅋ
- [Celery-Django 튜토리얼 문서](http://docs.celeryproject.org/en/latest/django/first-steps-with-django.html)
- [Redis install 참고 자료](http://dgkim5360.tistory.com/entry/install-redis-for-linux-or-windows)

### Celery 오류
튜토리얼을 바탕으로 `celery-docker`라는 프로젝트를 하나 만들었다. 그리고 현재 챗봇 프로젝트 모델과 크롤링 코드를 가져와서 celery 를 적용했다. 그러한 과정에서 발생했던 오류

#### import 문제
아래와 같이 import하는 경우, celery worker에 내가 만들었던 task 함수가 나오지 않았다. 사실 이부분은 완벽하게 왜 문제가 발생했는지 이해는 못했지만, import 순서 문제였던걸로 판단된다.(혹시라도 이 부분에 대해 명확한 답을 주실 수 있는 분은 댓글 부탁드립니다!)

```python
# books/tasks.py
import time

from .models import Book, BookLocation
from .utils import get_book_detail
from config.celery import app

__all__ = (
    'book_detail_save',
)

@app.task()
def book_detail_save(book_id):
    get_book_detail(book_id)
    return f'{book_id}가 저장되었습니다.'

# books/utils/crawling.py
from books.tasks import book_location_save
book_detail_save.delay(book_id)
```

```console
>> celery -A config worker -l info
-------------- celery@kahee v4.2.0 (windowlicker)
---- **** -----
--- * ***  * -- Linux-4.15.0-23-generic-x86_64-with-debian-buster-sid 2018-07-04 01:11:18
-- * - **** ---
- ** ---------- [config]
- ** ---------- .> app:         config:0x7f616764e1d0
- ** ---------- .> transport:   redis://localhost:6379/0
- ** ---------- .> results:     redis://localhost:6379/0
- *** --- * --- .> concurrency: 8 (prefork)
-- ******* ---- .> task events: OFF (enable -E to monitor tasks in this worker)
--- ***** -----
-------------- [queues]
               .> celery           exchange=celery(direct) key=celery
[tasks]
 . config.celery.debug_task
```

#### import 문제 해결
아래와 같이 import 한 경우, Celery worker에 내가 만든 task 함수들이 잘 나오고 동작했다.

```python
from books import tasks
tasks.book_detail_save.delay(book_id)

# import locally
from books.tasks import book_detail_save
book_detail_save.delay(book_id)
```

```console
>> celery -A config worker -l info
-------------- celery@kahee v4.2.0 (windowlicker)
---- **** -----
--- * ***  * -- Linux-4.15.0-23-generic-x86_64-with-debian-buster-sid 2018-07-04 02:14:10
-- * - **** ---
- ** ---------- [config]
- ** ---------- .> app:         config:0x7f015366e1d0
- ** ---------- .> transport:   redis://localhost:6379/0
- ** ---------- .> results:     redis://localhost:6379/0
- *** --- * --- .> concurrency: 8 (prefork)
-- ******* ---- .> task events: OFF (enable -E to monitor tasks in this worker)
--- ***** -----
-------------- [queues]
               .> celery           exchange=celery(direct) key=celery

[tasks]
 . books.tasks.book_detail_save
 . books.tasks.book_location_save
 . config.celery.debug_task
```

----

다음 편에서 계속....
