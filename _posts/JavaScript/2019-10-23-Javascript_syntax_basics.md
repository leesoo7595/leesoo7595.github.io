---
layout: post
title: "[JavaScript] Syntax Basics"
date: 2019-10-23
tags: [JavaScript]
comments: true
---

# 자바스크립트 기본 문법

## 변수

변수는 메모리 위치(주소)를 기억하는 저장소이다. 메모리 주소에 접근하기 위해 사람이 이해할 수 있는 언어로 지정한 식별자이다.

<img width="450" alt="Screen Shot 2019-10-24 at 8 06 26 PM" src="https://user-images.githubusercontent.com/39291812/67480079-41bfdc00-f69a-11e9-98a6-72d158044c97.png">

## 값

값은 프로그램에 의해 조작될 수 있는 대상을 말한다.

- `데이터 타입` : 프로그래밍 언어에서 사용할 수 있는 값의 종류
- `변수` : 값이 저장된 메모리 공간의 주소를 가리키는 식별자
- `리터럴` : 값을 구성하는 최소 단위, 소스 코드 안에서 직접 만들어낸 상수 값 자체

값은 다양한 방법으로 생성할 수 있다.
자바스크립트의 모든 값은 데이터 타입을 갖는다.

### 원시 타입

* number
* string
* boolean
* null
* undefined
* symbol

### 객체 타입

* object

자바스크립트는 변수를 선언할 때 데이터 타입을 미리 지정하지 않는다.

<img width="438" alt="Screen Shot 2019-10-24 at 8 18 29 PM" src="https://user-images.githubusercontent.com/39291812/67480779-7bddad80-f69b-11e9-82a6-0e2ad3b3b068.png">

변수에 할당된 값의 타입에 의해 동적으로 변수의 타입이 결정된다. 이를 동적 타이핑이라 한다.

## 연산자

산술, 할당, 비교, 논리, 타입 연산 등을 수행해 하나의 값으로 만들어주는 것이다.
연산의 대상은 피연산자라고 한다.

```javascript
// 산술
var area = 5 * 4; // 20

// 문자열 연결
var str = 'Hello ' + 'World'; // 'Hello World'

// 할당
var color = 'red'; // 'red'

// 비교
var foo = 3 > 5; // false
...
```

피연산자의 타입이 일치하지 않아도, 자바스크립트는 `암묵적으로 타입 강제 변환을 하여` 연산을 수행한다.

<img width="440" alt="Screen Shot 2019-10-24 at 8 29 02 PM" src="https://user-images.githubusercontent.com/39291812/67481445-ff4bce80-f69c-11e9-984b-9bcd11e874a9.png">

참고 : https://simsimjae.tistory.com/352

## 키워드

키워드는 수행할 동작을 규정한 것이다.

예를 들면 var, let, function, if 등등..

<img width="205" alt="Screen Shot 2019-10-24 at 8 32 03 PM" src="https://user-images.githubusercontent.com/39291812/67481711-5ce01b00-f69d-11e9-9439-4b5ccb3430a7.png">

## 주석

주석은 코드를 부연 설명하기 위해 있는 것이다.
하지만 주석을 안 달아도 되도록 코드를 짜는 것이 가장 최선이다.
이게 어렵지..

## 문

문(statement)은 스크립트가 실행될 때, 단계적으로 실행되는 각각의 명령을 의미한다.
문은 위에서 아래로 순서대로 실행된다. 

자바스크립트에서는 블록 유효범위를 생성하지 않고, 함수 단위의 유효범위가 생성된다.

참고 : http://tcpschool.com/javascript/js_function_functionScope

## 표현식

표현식은 하나의 값으로 평가되는 것을 말한다.
표현식은 결국 하나의 값이 된다.

### 문과 표현식의 비교

표현식은 문을 구성하는 요소이다. 그리고 표현식은 하나의 문이 될 수도 있따.

<img width="532" alt="Screen Shot 2019-10-25 at 12 37 07 AM" src="https://user-images.githubusercontent.com/39291812/67501636-9aee3680-f6bf-11e9-9556-a76336809481.png">

## 함수

함수는 어떤 작업을 수행하기 위해 필요한 문(statement)의 집합을 정의한 것이다.
함수는 함수의 이름과 매개변수를 가진다.
필요한 때에 호출하여 코드 블록에 담긴 문들을 실행할 수 있다.

동일한 작업을 반복적으로 수행해야 할 때 정의된 함수를 사용하는 것이 효율적이다.

## 객체

자바스크립트는 객체 기반의 스크립트 언어이다.
원시타입을 제외한 나머지 값들은 모두 객체이다.

자바스크립트 객체는 키와 값으로 구성된 프로퍼티의 집합이다.
자바스크립트의 함수는 일급 객체이므로 객체의 프로퍼티 값으로 함수를 포함한 모든 값을 사용할 수 있다.

일급 객체란, 다른 객체들에게 일반적으로 적용 가능한 연산을 모두 지원하는 객체를 말한다.

프로퍼티의 값이 함수인 경우, 일반 함수와 구분하기 위해 메소드라고 부른다.

<img width="887" alt="Screen Shot 2019-10-25 at 1 26 15 AM" src="https://user-images.githubusercontent.com/39291812/67505559-76e22380-f6c6-11e9-80d4-230111dabdb3.png">

`일급 객체` 참고: https://bestalign.github.io/2015/10/18/first-class-object/

자바스크립트는 자바스크립트 고유의 특징인 프로토타입이라고 불리는 객체의 프로퍼티와 메소드를 상속받을 수 있는 것이 존재한다.

`프로토타입` 참고: https://medium.com/@bluesh55/javascript-prototype-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-f8e67c286b67

## 배열

배열(array)은 1개의 변수에 여러 개의 값을 순차적으로 저장할 때 사용한다. 자바스크립트의 배열은 객체이며 여러 내장 메소드를 포함하고 있다.