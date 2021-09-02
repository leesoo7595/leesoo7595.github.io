---
layout: post
title: Docker & Nginx & uWSGI 설정
tags: [AWS]
---

## Docker & Nginx & uWSGI 관계

1. Browser에서 HTTP 요청이 오면 Docker가 곧바로 받아서 포워드를 하여 Nginx에게 넘겨준다.
2. Nginx는 nginx conf를 이용해서 uwsgi로 보내는데, 여기서 nginx와 uwsgi는 unix socket이라는 것을 통해 연결되어 uwsgi로 넘겨지게 된다.
3. uWSGI는 자신의 환경설정을 통하여 Django로 넘겨지게 된다. 여기서는 WSGI module을 통해서 넘겨지게 된다.

**HTTP 프로토콜 VS UNIX 소켓** <br>
`HTTP 프로토콜`은 데이터를 주고 받을 수 있는 프로토콜이기 때문에 왓다갓다할 때 오버헤드가 많다. <br>
그래서 어떤 프로세스로 오고가는지 다 정해져있고, 데이터 자체가 엄청나게 길다. <br>

`UNIX 소켓`은 유닉스 기반 운영체제 안에서 프로세스간에 어떤 데이터를 교환할 때 `HTTP`에 비해 더 효율적으로 돌아간다. <br>
-> 어떤 파일을 소켓으로 지정하면(`.sock`) 바로 그것이 소켓 역할이 된다. 그렇게 `Nginx`에서 그 소켓으로 데이터를 전달하게 되면, 받은 데이터를 바로  `uWSGI`로 포워딩해주게 된다.

### Nginx

`Nginx`는 웹서버 프로그램이다. <br>
하나의 서버 내부에서 웹서버 프로그램이 가상 서버를 만들게 된다. -> `nginx_app.conf`

**nginx_app.conf** <br>
1. 80번 포트로부터 `request`를 받는다.
2. 도메인명이 어떤 경우에 해당하는지 `server_name 도메인`
3. 인코딩 방식 지정 `charset utf-8`
4. request/response의 최대 사이즈 지정 `client_max_body_size 128M`
5. '/' (모든 URL로의 연결에 대해) <br>
```
location / {
    uwsgi_pass            unix:///tmp/app.sock         
    include               uwsgi_params;
}
```
6. 이후 media파일이나 static파일을 연결시키고 싶을 땐,
```
location /static/ {
    alias           /srv/project/.static/;
}
```
이런식으로 추가!

**uwsgi.ini** <br>
1. 프로젝트 경로 설정 `chdir = /srv/project/app`
2. `export`모듈 설정 : `module = config.wsgi.${BUILD_MODE}:application`
3. 소켓 파일 설정 : `socket = /tmp/app.sock`
4. uWSGI 종료시 소켓 삭제 : `vacuum = true`
5. log 파일 설정 : `logto = /var/log/uwsgi.log`

```
[uwsgi]
chdir = /srv/project/app
module = config.wsgi.dev:application
socket = /tmp/app.sock
vacuum = true

logto = /var/log/uwsgi.log
```
