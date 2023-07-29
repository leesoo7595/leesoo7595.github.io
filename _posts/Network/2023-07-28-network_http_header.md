---
layout: post
title: "네트워크 - HTTP 헤더"
date: 2023-07-28
tags: [Network]
comments: true
---

# HTTP - Hyper Text Transfer Protocol

HTTP는 헤더와 바디로 구성된다. 바디는 (본문, content, payload) 등으로 불린다.

## 헤더 특징

**view source 눌러서 원본보기**

### 헤더 key 중 Accept, Content (MIME Type)

- `key: value`의 형태로 이루어짐.
- 한글 안됨 -> 되게하려면 `encodeURIComponent` 사용!
- Accept: 서버한테 프론트가 어떤 형식을 원하는지 알려주는 것
  - MIME Type 대분류/확장자 (image/png, image/jpeg, application/json...)
  - q: q를 기준으로 앞쪽이 더 선호하는 우선순위 MIME Type. 1이 가장 높음. 0이 가장 낮음 (제일 선호하지 않는 MIME Type)
- Accept-Encoding: gzip, deflate, br 서버에게 이런 압축된 형식의 아이들을 받을 수 있다고 알려주는 것 (선택은 서버에게 달려있다 - 서버주도협상)
- Accept-Charset: utf-8, ascii, euc-kr 문자열을 어떤 방식으로 인코딩해서 줄지
- Content-Type: (서버)응답에서 온 타입을 알려주는 것 (Accept, Accept-Charset)
- Content-Encoding: 응답에서 온 타입을 알려주는 것 (Accept-Encoding)
- Content-Language: 응답에서 온 타입을 알려주는 것 (Accept-Language)
- Content-Length: 응답한 길이 (1000으로 나누면 byte 단위)

### Connection

`Connection: keep-alive`: 서버와 클라이언트 사이에서 요청/응답을 보낼때마다 매번 커넥션을 새로 시도하게되면 비용이 크므로 keep-alive 옵션을 통해 연결을 유지하도록 한다. 요즘은 디폴트 값 -> 한번만 커넥션을 맺으면 그 후부터는 재사용하여 요청응답만을 바로 할 수 있게하는 것

`Keep-Alive: timeout=5`: 5초 안에 요청응답이 오지 않으면 커넥션이 끊키는 것

`Date`: 서버의 시간을 의미 (클라이언트측과 시간 차이를 알 수 있음) -> 메세지가 생성된 시간

`Transfer-Encoding: chunked`: 바디를 쪼개서 보낸다는 뜻 (인코딩된 문자열을 어떠한 규칙으로 잘라서 보냄, 얼마나 잘려서 오는지 순서를 같이 알려줘서 다음 문자열을 받아올 수 있도록 기다림)

### Authorization

Basic, Bearer, Digest -> 서버쪽에서 어떻게 보내야하는지 형식이 정해져있는데 그 형식을 정해주는 값. 그 뒤에는 jwt 토큰을 받아와서 인증을 한다.

### 기타 헤더

어떠한 요청을 했을때 특정 statusCode 응답과 헤더의 어떤 값과 연결지어서 볼 수 있다.
그래서 mdn 문서로 한번 읽어보면 도움이 될 듯.

헤더 값 중 끝에 policy가 붙는 아이들은 정책을 의미하는데, 현재 사이트에 대한 정보 및 개인정보를 가져가는 경우 때문에 지속적으로 추가가 되는 중이다.

### 커스텀 헤더

mdn 문서에 없는 헤더가 있을 수도 있다. 이런 경우는 커스텀 헤더일 가능성이 높음. 그런 경우는 몰라도된다~

X-시리즈가 보통 커스텀 헤더로 많이 앞에 붙여서 쓰는 듯 하다.

### 쿠키

HTTP는 상태가 없다. 즉, 이번 요청을 보내고 다음 요청을 보낼때 아무런 기억을 하지 못한다.

그래서 헤더 중에 하나를 사용해서 이 무상태인 HTTP를 상태를 가질 수 있도록 꼼수를 쓴다.
쿠키를 설정하면 `Set-Cookie` 헤더를 추가해서 보내면 브라우저에 쿠키가 저장된다. 네트워크탭에 Cookies 탭이 추가된다. 그러면 Cookies 탭에서 어떤 Key=value 형태의 쿠키가 저장되었는지 확인을 할 수 있다.

**mdn에서 Set-Cookie 탭을 보기! (옵션 체크)**

쿠키는 어플리케이션 탭에서 저장이 되는 것을 확인할 수 있다.

- `Expires/Max-age`: (Max-age의 경우 '초' 단위를 의미, 요즘 훨신 더 많이 쓰인다. 왜냐면 Expires의 경우 서버 시간 기준으로 되기 때문에 정확하지 않을 수도 있다.) 쿠키가 만료되는 날짜를 알려준다.(서버에서 전송한 직후부터 시간을 계산함) 만료된 후에는 해당 쿠키정보가 사라짐 (날짜가 써있거나, Session이 써있음 - Session은 브라우저를 끄면 사라진다.)
- `Domain` : 기재되어있는 도메인 주소와 서브도메인에서 해당 쿠키를 다 조회할 수 있다.
- `Path` : 정해진 도메인(쿠키에 적혀있는 Domain)에서는 쿠키가 다 전달이 되기 때문에, Path를 통해 클라이언트에서 어떤 요청에 쿠키를 안보내게끔할지 설정할 수 있음
- `HttpOnly` : 자바스크립트에서 가져갈 수 있는지 유무, 브라우저 콘솔에 `document.cookie`를 쳤을 때 쿠키 값을 노출당할 수 있다. 이 옵션을 주는 경우 브라우저에서 접근이 불가능
- `Secure` : Https일 때만 쿠키를 사용할 수 있다는 의미
- `SameSite`: 쿠키의 Domain이 같다는 뜻.
  - `SameSite=Strict`: 같은 도메인에서만 쿠키 공유 가능
  - `SameSite=Lax`: 기본값 - 크로스 사이트 쿠키를 쓸 수 없다. 어떤 링크를 클릭할 때만 예외적으로 가능
  - `SameSite=None; Secure`: 브라우저가 크로스 사이트 및 동일 사이트 요청과 함께 쿠키를 전송한다는 의미로 이 값을 설정할 경우 보안 속성도 포함하여야함. 보안이 누락되면 오류가 기록된다.
