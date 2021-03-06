---
layout: post
title: "[HTTP] URL Resource"
date: 2018-11-19
tags: [HTTP]
comments: true
---

참고 도서 : HTTP 완벽 가이드

# URL과 리소스

학습 목표

- URL 문법, 여러 URL 컴포넌트가 어떤 의미를 가지고 무엇을 수행하는지
- 웹 클라이언트가 지원하는 상대 URL과 단축 URL에 대해서
- URL의 인코딩과 문자 규칙
- 여러 인터넷 정보 시스템에 적용되는 공통 URL 스키마
- 기존 이름은 유지하면서 객체들은 다른 장소로 옮기는 것을 가능하게 해주는 URN을 포함한 URL의 미래


## 인터넷의 리소스 탐색하기

`URL`은 브라우저가 정보를 찾는데 필요한 리소스의 위치를 가리킨다. `URL`을 통해 사람이 `HTTP` 및 다른 프로토콜을 통해 접근할 수 있다.

`URL`은 통합 자원 식별자(`Uniform Resource Identifier`) 혹은 `URI`라고 불리는 일반화된 부류의 부분집합이다. `URI`는 두 가지 주요 부분집합인 `URL`과 `URN`으로 구성된 종합적인 개념이다.

`URN`은 현재 그 리소스가 어디에 존재하든 상관없이 그 이름만으로 리소스를 식별하지만, `URL`은 리소스가 어디 있는지 설명해서 리소스를 식별한다.

`HTTP` 명세에서는 `URI`를 더 일반화된 개념의 `URL`로 사용한다. 대부분 `URL`을 가리키는 것으로 보면 될 것이다.

- `URL`의 첫 부분인 `http`는 `URL`의 스키마다. 스키마는 웹 클라이언트가 리소스에 어떻게 접근하는지 알려준다. `URL`이 `HTTP` 프로토콜을 사용한다는 의미이다.
- `URL`의 두 번째 부분은 서버의 위치를 말한다. 이는 웹 클라이언트가 리소스가 어디에 호스팅 되어 있는지 알려준다.
- `URL`의 세 번째 부분은 리소스의 경로이다. 경로는 서버에 존재하는 로컬 리소스들 중에서 요청받은 리소스가 무엇인지 알려준다.


`FTP (File Transfer Protocol)` : 서버에 올라가 있는 파일

대부분의 `URL`은 동일하게 `스키마://서버위치/경로` 구조로 이루어져 있다.

### URL이 있기 전

`URL`이 있기 전엔 애플리케이션마다 달리 가지고 있는 분류 방식을 사용하였다.

## URL 문법

`URL` 문법을 스키마에 따라서 달라진다. 대부분의 URL은 일반 `URL` 문법을 따르며, 서로 다른 `URL` 스키마도 형태와 문법 면에서 유사하다.

스키마의 예 : `HTTP`, `FTP`, `SMTP` 등

`스키마://사용자:비밀번호@호스트:포트/경로;파라미터?질의#프래그먼트`

- 스키마 : 리소스를 가져올 때 어떤 프로토콜을 사용하여 서버에 접근해야 하는지
- 사용자 이름 : 몇몇 스키마는 리소스에 접근하기 위해 필요하다
- 비밀번호 : 사용자의 비밀번호
- 호스트 : 서버 호스트 명이나 IP주소
- 포트 : 리소스를 호스팅하는 서버가 열어놓은 포트번호
- 경로 : 서버 내 리소스가 서버 어디에 있는지
- 파라미터 : 이름/값을 쌍으로 가진다. 다른 파라미터와는 ;로 구분
- 질의 : 스키마에서 애플리케이션에 파라미터를 전달할 때 쓰인다
- 프래그먼트 : 리소스의 조각이나 일부분을 가리키는 이름이다.

## 단축 URL

### 상대 URL

주로 봐왔던 것이 절대 `URL`이다. 절대 `URL`은 리소스에 접근하는데 필요한 모든 정보를 가지고 있다.

상대 `URL`은 모든 정보를 담고 있지는 앟ㄴ다. 상대 `URL`로 리소스에 접근하는데 필요한 정보를 얻기 위해서는 기저(base)라고 하는 다른 `URL`을 사용해야 한다.

`./example.html`

### URL 확장

어떤 브라우저는 `URL`을 입력한 다음이나 입력하고 있는 동안에 자동으로 `URL`을 확장한다. 자동으로 `URL`이 확장되기 때문에 `URL` 전체를 입력하지 않아도 된다.

* 호스트 명 확장 : 단순한 휴리스틱만을 사용해서 입력한 호스트 명을 전체 호스트 명으로 확장할 수 있다.
* 히스토리 확장 : 과거에 사용자가 방문했던 `URL`의 기록을 저장해 놓는 것

## 안전하지 않은 문자

안전한 전송이란, 정보가 유실될 위험 없이 `URL`을 전송할 수 있다는 것을 의미한다.

### 인코딩

## 스키마

* http : 일반 URL 포맷을 지키는 스키마, 포트값 생략시 기본값 80
* https : http 스키마와 거의 같다. 다른 점은 http 커넥션의 양 끝단에서 암호화하기 위해 보안 소켓 계층(`Secure Sockets Layer, SSL`)을 사용한다. 기본 포트값은 443
* mailto : 이메일 주소를 가리키고, 다른 스키마와는 다르게 동작하기 때문에 mailto URL은 표준 URL과는 다른 포맷을 가진다.
* ftp : 파일 전송 프로토콜 URL은 일반 URL 포맷이다.
* rtsp, rtspu : 실시간 스트리밍 프로토콜 rtspu는 UDP 프로토콜이 사용된다는 뜻
* file : 주어진 로컬에서 바로 접근할 수 있는 파일
* news : 특정 문서나 뉴스 그룹에 접근시 사용
* telnet : 대화형 서비스에 접근하는 데에 사용
