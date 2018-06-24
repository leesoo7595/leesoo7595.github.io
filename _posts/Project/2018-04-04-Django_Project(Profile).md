---
layout: post
title: "Django_Project(profile)"
date: 2018-04-04 00:00:00
img:
tags: [Django_Project]
---

### 구현하고 싶은 기능
1. 토큰을 가진 회원의 정보를 넘겨줌
2. 회원정보 변경 (비밀번호, 프로필 이미지)
3. 핸드폰 번호 인증 후, 핸드폰 번호 변경

---

## 회원 프로필
### 회원 정보 보내기
0. members/info/ - GET 방식
1. 토큰이 맞는지 유효성 체크
2. return 회원 정보

### 회원 비밀번호 변경
0. memebers/info/ - PATCH 방식
1. 토큰이 맞는지 유효성 체크
2. 비밀번호 유효성 체크
3. 비밀번호 변경
4. 비밀번호 변경 완료 status 넘겨주기

- UserPasswordSerializer()작성

### 회원 프로필 변경
0. members/info/img_change/ - update 방식
1. 토큰 맞는지 확인
2. 이미지 저장

### 핸드폰 번호 변경
0. members/info/phone_change/ - PUT 방식
1. 토큰 맞는지 확인
2. 핸드폰 번호 put 으로 전달
3. 인증번호 만들고 해당번호로 문자 보내기
4. 인증번호가 맞으면 다시 put방식으로 핸드폰 번호 변경후 저장

----

## 회원 로그아웃
- 토큰을 삭제한다.
