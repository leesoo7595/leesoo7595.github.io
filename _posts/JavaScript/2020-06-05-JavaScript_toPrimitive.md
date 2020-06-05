---
layout: post
title: "Javascript - toPrimitive"
date: 2020-06-05
tags: [JavaScript]
categories:
- Javascript
comments: true
---

## 객체를 원시형으로

객체를 제외한 원시형 자료가 문자, 숫자, boolean형으로 변환되는지 전에 공부하였다. 하지만 메소드와 심볼의 지식을 갖춘다면, 객체를 원시형으로 어떻게 변환을 할 수 있는지를 공부해볼 수 있다.

그 전에 짚고 갈 기본적인 상식!

* 객체는 boolean 평가시 true를 반환한다. 즉, 객체는 숫자, 문자형으로 형 변환이 일어날 수 있다.

* 객체에서 숫자형으로 형 변환을 할 땐, 객체끼리 빼는 연산을 하거나 수학 관련 함수를 적용할 때 일어난다.

ex) Date 함수

* 문자형으로 객체를 변환시킬 땐, 객체를 출력하려할 때 일어난다.

### ToPrimitive

