---
layout: post
title: "네트워크 - HTTP"
date: 2023-07-23
tags: [Network]
comments: true
---

# 네트워크탭

- RFC 7231 or 9110

## 주소 구성 체계 (URL, URI, origin)

**URL : uniform resource locator**
**URI : uniform resource identifier**

옛날엔 실제 자원을 가리키는 url로 사용하는 경우가 다수였지만(끝에 파일확장자가 붙는), 현재는 가상자원인 uri을 사용하는 경우가 대다수다.

**origin**

```
┌────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                              href                                              │
├──────────┬──┬─────────────────────┬────────────────────────┬───────────────────────────┬───────┤
│ protocol │  │        auth         │          host          │           path            │ hash  │
│          │  │                     ├─────────────────┬──────┼──────────┬────────────────┤       │
│          │  │                     │    hostname     │ port │ pathname │     search     │       │
│          │  │                     │                 │      │          ├─┬──────────────┤       │
│          │  │                     │                 │      │          │ │    query     │       │
"  https:   //    user   :   pass   @ sub.example.com : 8080   /p/a/t/h  ?  query=string   #hash "
│          │  │          │          │    hostname     │ port │          │                │       │
│          │  │          │          ├─────────────────┴──────┤          │                │       │
│ protocol │  │ username │ password │          host          │          │                │       │
├──────────┴──┼──────────┴──────────┼────────────────────────┤          │                │       │
│   origin    │                     │         origin         │ pathname │     search     │ hash  │
├─────────────┴─────────────────────┴────────────────────────┴──────────┴────────────────┴───────┤
│                                              href                                              │
└────────────────────────────────────────────────────────────────────────────────────────────────┘
(All spaces in the "" line should be ignored. They are purely for formatting.)
```

프로토콜 포함해서 전체적인 주소(`https://user:pass@sub.example.com:8080`)가 origin이라고 부른다.

모든 주소가 포함된게 href(uri)이라고 한다.

## 헤더(headers)

- `GET /book/2 HTTP/1.1` 자원의 위치만 표기
- `Accept`: content negotiation 서버에게 나는 이런 형식의 데이터를 원한다고 알려주는 것 (`Content-Type`)
- `cache-Control`: 데이터를 주고받을때 비용을 아낄 수 있는 방법 (잘 활용해야함)
- `Connection: keep-alive` 옛날부터 올라온 아이..(HTTP 1.0)
- `Cookie`: HTTP의 무상태를 보완하기 위함 (특정 문자열을 서버로 보내면 서버에서 저장되어있는 것과 비교 -> 어떤 사용자의 쿠키인지 알아냄. HTTP의 무상태함을 서버에서 이런식으로 비교해서 판단)
- `HOST`
- `User-Agent`: 요청을 보내는 내 컴퓨터의 브라우저가 무엇인지 (근데 정확하지 않음.. 중요하지 않다 유명무실 -> 아예 없앤다는 얘기도 나오는 중. 참고: [브라우저의 사용자 에이전트는 왜 이렇게 복잡하게 생겼을까?](https://wormwlrm.github.io/2021/10/11/Why-User-Agent-string-is-so-complex.html))
- `Content-Encoding`: gzip(용량 압축), base64(용량 늘어남)

### General 탭

General으로 따로 빼둔 아이들은 좀 중요한 아이들이라서 따로 빼놓은 것 (개발자 도구의 편의)

### Request Method

- GET
- HEAD
- POST
- PUT
- PATCH
- DELETE
- CONNECT
- OPTIONS
- TRACE

**REST API**

`METHOD /{dbtable}/{id}` 형식을 지키면 대충 REST API

**약속만 잘 되어있으면 된다.**

`HEAD` 응답에 바디가 없는 메소드

-> 이 라우터가 제대로 동작하는지 체크할때 주로 사용
-> 응답 바디 없이 응답 헤더만 오기 때문에 주고받는 비용이 적다.

`OPTIONS` CORS 요청할 때

-> 다른 도메인에서 요청을 보낼때 사용
-> Preflight

`TRACE` 중간 서버를 거치면서 헤더가 바뀌었는지 추적할 때 사용

-> 하지만 추적하는 것 자체도 위험할 수 있음
-> 거의 사용할 일이 없다

**GET 메소드를 사용하면 POST보다 보안이 안좋은가?**

요즘은 틀린 얘기이다. HTTPS를 쓰면 둘다 보안성은 높다. 요즘은 HTTPS가 필수로 쓰이기 때문에 상관없는 이야기. 그리고 HTTPS를 사용하지 않으면 둘다 안전하지 않다.

### 안전한 메서드, 멱등성 메서드

**안전한 메서드** : 서버의 상태를 바꾸지 않는 메서드 (GET, HEAD, OPTIONS, TRACE)

서버의 데이터를 변경하지 않는 메서드를 안전한 메서드라고 불린다. (조회)

**멱등성 메서드** : GET, HEAD, OPTIONS, TRACE, PUT, DELETE

같은 메서드을 여러번 호출해도 최종적으로는 서버에서 영향이 다르지 않은 것을 말함

조회 메서드의 경우 항상 서버의 상태가 같기 때문에 멱등성 메서드이다. PUT, PATCH, DELETE도 또한 한번 바꾸나, 여러번 바꾸나 서버에서는 한번 바뀐 것과 같음 그래서 멱등성을 가졌다.

POST, PATCH의 경우, 만번 호출될 경우 만번 생성하거나 바꾸기 때문에, 서버를 횟수대로 바꾸기 때문에 멱등성을 지녔다. 멱등성을 지닌 메서드(GET, HEAD)는 캐시될 수 있다. 즉 브라우저가 임시적으로 데이터를 저장할 수 있다.

PUT, PATCH를 구분해서 사용할 때, 멱등성을 가지는지 유무에 따라 설계하면 될 것!
