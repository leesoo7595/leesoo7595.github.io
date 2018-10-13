---
layout: post
title: "[DB] ODBC란?"
date: 2018-10-13
tags: [DB]
comments: true
---

# ODBC

`Open Database Connectivity`는 DBMS와 통신을 할 때 흔히 사용하는 **데이터베이스에 접근하기 위한 소프트웨어의 표준 규격이다.** <br>
<br>

## ODBC가 있는 목적

### 소켓 통신

응용프로그램은 DBMS와 데이터를 주고 받는 통신을 한다. <br>
즉, 응용프로그램 상에서 실행 중인 프로세스와, DBMS에서 실행 중인 프로세스가 서로 통신을 한다는 뜻. <br>
<br>

소켓 통신 방법은 소켓을 생성해서 연결한 다음, 어떤 형식으로 데이터를 주고 받을 것인지 `프로토콜`에 따라 주고 받는다. <br>
<br>

**문제는 DBMS사마다 프로토콜이 다르다.** <br>
**그리고 DBMS사가 프로토콜을 공개하지 않는다.** <br>

### Vender API 통신

그래서 DBMS사는 프로토콜을 공개하지 않으면서 응용프로그램과 통신하는 방법으로 API를 제공한다. <br>
하지만 DBMS마다 API 사용법이 다 다르다. <br>
<br>

결국 DBMS마다 각각 다른 프로그램을 개발해야한다는 문제점이 여전히 존재한다.

### ODBC API 통해 통신

그래서 MS에서 ODBC API를 내놓았다. <br>
ODBC는 데이터베이스에 접근하기 위한 표준 규격으로, 모든 DBMS에 접근하는 방법을 통일시켰다. <br>
<br>

## ODBC의 특징

1. DBMS 비종속적이다.
2. 속도가 느리다.

참고 : http://wanzargen.tistory.com/18
