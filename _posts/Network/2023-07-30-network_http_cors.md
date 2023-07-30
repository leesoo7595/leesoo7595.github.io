---
layout: post
title: "네트워크 - HTTP cors"
date: 2023-07-30
tags: [Network]
comments: true
---

# CORS

**Access-Control-\* 관련 헤더들에 관한 이야기**

CORS 정책 때문에 다른 도메인간의 요청이 막히는 에러이다. 오리진 도메인이 다른 경우는 서로 요청을 못보내는 것이 맞음.

`access-control-allow-origin` 다른 사이트에도 요청을 보낼 수 잇도록 하는 헤더이다.

서버쪽에서 해당 헤더를 넣어주는 것이 규칙, 클라이언트 도메인을 서버에서 추가해주면, 클라이언트에서 서버에 요청을 보낼 수 있게 되는 것이다.

서버에서 해당 헤더를 설정해주지 않은 클라이언트에서 요청하게되면 CORS에러가 브라우저에서 발생하게 된다. 서버 간의 요청/응답은 서로 받을 수 있다. CORS 에러가 나지 않음

CORS에러를 브라우저에서 띄우는 이유는 의도치 않은 타도메인 요청으로 개인정보가 빠져나가지 않도록 자체적으로 방어하는 것이다.

타도메인에 대한 요청에서 Access-Control 관련 헤더가 없는데 CORS 에러가 나지 않을 수도 있다. 이런 경우는 기준에 따라 다름.

## Same-Origin-Policy

**포트나 프로토콜, 서브도메인이 달라져도 모두 다른 도메인임.**

### `simple requests`

- `Access-Control-Allow-Origin` 헤더를 넣어주면 CORS 에러를 해결할 수 있음

### `preflight requests`

- Access-Control 관련 더 다양한 헤더를 사용해서 추가적인 정보를 입력해줘야 CORS 에러를 해결할 수 있음
- `Access-Control-Allow-Credentials: true` 크로스 오리진일 때 CORS를 해결하고, 추가로 쿠키 등의 정보를 공유하고 싶다면 사용하는 헤더

요청을 허용할 url을 적어주는 것

네트워크탭에서 Domain 칸을 보면 확인할 수 있다.

CORS 에러는 브라우저에서만 발생한다. 브라우저가 다른 origin 서버에 요청을 보내면 CORS 에러가 남. 같은 오리진 서버에서 다른 오리진 서버에 요청보내는 것도 CORS 에러가 나지 않음

브라우저에서 같은 오리진 서버로 보내고, 서버에서 다른 오리진 서버로 요청을 보내면 에러가 나지 않는다. 이것을 프록시 서버를 사용한다고 말한다. `webpack-dev-server`에서 간단하게 제공해줌

`**.com(브라우저) -> **.com(서버) or 간단한 프록시 서버 -> facebook.com(서버)`
