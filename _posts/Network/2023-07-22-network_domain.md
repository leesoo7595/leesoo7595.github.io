---
layout: post
title: "네트워크 - DNS"
date: 2023-07-22
tags: [Network]
comments: true
---

## DNS

- 브라우저에 naver.com을 입력
- DNS resolver에 naver.com을 찾도록 함
- DNS resolver -> Root server -> TLS server -> 네임서버 (naver.com) -> 레코드에서 IP주소를 꺼내옴
- 해당 도메인이 반복되는 경우 캐시를 사용해서 꺼내옴

## 레코드

- 도메인 구입 (ex. naver.com)
- naver.com 에만 접근이 가능 (IP4: A 레코드, IP6: AAAA) 또는 앞에 특정 서브도메인 이름을 붙여서 할 수 있다. (react.naver.com 등)
- CNAME을 사용해서 www.naver.com으로도 접근이 가능토록함
- NS(name server) 공인된 IP로만 바꿔줘야 안전
- MX 메일 관련 기능을 추가하고 싶을때 이어주면 됨 (Mail Exchanger record) (ex. mail.naver.com)

위 레코드를 활용하여 한 도메인만으로 여러개의 서브 도메인을 이용할 수 있다.

### 리눅스 /etc/hosts 파일

리눅스에서 hosts 파일이 DNS 역할을 한다.

```$
$ cat /etc/hosts

127.0.0.1   localhost
::1         localhost
...
```

127.0.0.1과 localhost가 매핑이 되어있어서 브라우저에 localhost를 치면 IP가 127.0.0.1로 연결이 된다.

## 와이어샤크

네트워크를 통해서 인터넷으로 프레임이 왔다갔다함
이 전기신호가 왔다갔다하는 것을 와이어샤크가 캡쳐를 해줌

어떤 데이터가 왔다갔다하는지 편하게 보여줄 수 있도록 하는 툴

전기신호를 0, 1로 바꿔주면 너무 길어지기 때문에 16진법으로 많이 줄여서 물리계층에서 변환된다.
