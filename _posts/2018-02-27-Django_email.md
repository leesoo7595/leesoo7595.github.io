---
layout: post
title: "Django-email보내기"
date: 2018-02-27 00:00:00
img:
tags: [Django]
---
참고 자료
- http://devgyugyu.tistory.com/14
- https://www.christiaan.com/thoughts/django-email-form-and-smtp-settings-tutorial/

### 장고 패키지 찾을 때 유용한 사이트
https://djangopackages.org/

### SMTP(Simple Mail Transfer Protocol)란 무엇인가?
> 간이 전자 우편 전송 프로토콜(Simple Mail Transfer Protocol, SMTP)은 인터넷에서 이메일을 보내기 위해 이용되는 프로토콜이다. 사용하는 TCP 포트번호는 25번이다. 상대 서버를 지시하기 위해서 DNS의 MX레코드가 사용된다. RFC2821에 따라 규정되어 있다. 메일 서버간의 송수신뿐만 아니라, 메일 클라이언트에서 메일 서버로 메일을 보낼 때에도x 사용되는 경우가 많다. _위키백과_

### G-mail 설정

### G-mail 앱 비빌번호
http://www.itworld.co.kr/news/89055

### Django에서 민감한 정보를 처리하는 법
http://fearofcode.github.io/blog/2013/01/15/how-to-scrub-sensitive-information-from-django-settings-dot-py-files/


### 만들면서 겪언던 오류
- 이메일 보내는이가 꼭 튜플이나, 리스트로 들어가야 한다.
- flack
