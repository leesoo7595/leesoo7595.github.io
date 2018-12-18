---
layout: post
title: "[JavaScript] function"
date: 2018-12-18
tags: [JavaScript]
comments: true
---

참고 : https://poiemaweb.com/js-function

함수의 일반적 기능은 어떤 작업을 수행하는 statement의 집합을 정의하여 코드의 재사용에 목적이 있다. 이러한 일반적 기능 이외에 객체 생성, 객체의 행위 정의, 정보, 은닉, 클로저, 모듈화 등의 기능을 수행할 수 있다.

자바스크립트의 함수는 객체(일급 객체)이다. 함수도 객체이므로 다른 값처럼 사용할 수 있다.

## 함수의 정의

* 함수 선언문
* 함수 표현식
* Function 생성자 함수

### 함수 선언문

```JavaScript
// 함수 선언문
function square(number) {
  return number * number;
}
```

### 함수 표현식

```JavaScript
// 함수 표현식
var square = function(number) {
  return number * number;
};
```

함수는 일급객체이기 때문에 변수에 할당할 수 있다. 이 변수는 함수명이 아니라 할당된 함수를 가리키는 참조값을 저장하게 된다. 그래서 함수 호출시 함수명이 아니라 함수를 가리키는 변수명을 사용하여야한다. 그래서 함수표현식에서는 함수명을 생략하는 것이 일반적이다.

### Function 생성자 함수

함수 선언문과 함수 표현식은 모두 함수 리터럴 방식으로 함수를 정의하는데 이것은 결국 내장 함수 Function 생성자 함수로 함수를 생성하는 것을 단순화시킨 short-hand(축약법)이다.

```JavaScript
new Function(arg1, arg2, ... argN, functionBody)

var square = new Function('number', 'return number * number');
console.log(square(10)); // 100
```

## 함수 호이스팅

`함수 선언문`으로 함수가 정의되기 전에 함수 호출이 가능하다. 즉, 함수 선언의 위치와는 상관없이 어디서든지 호출이 가능한데 이것을 함수 호이스팅이라한다.

```JavaScript
var res = square(5);

function square(number) {
  return number * number;
}
```

자바스크립트는 모든 선언`var, let, const, function, class`을 호이스팅 한다.

호이스팅이란, 모든 선언문이 해당 scope의 선두로 옮겨진 것처럼 동작하는 특성을 말한다. 즉 선언되기 이전에 참조 가능하다.

`함수 선언문`으로 정의된 함수는 함수 선언, 초기화, 할당이 한번에 이루어진다.

`함수 선언문`과 달리 `함수 표현식`으로 함수를 정의한 경우, 변수 호이스팅이 발생한다.

변수 호이스팅은 변수 생성 및 초기화와 할당이 분리되어 진행된다. 즉, 호이스팅된 변수는 undefined로 초기화되고 실제값의 할당은 할당문에서 이루어진다.

```JavaScript
var res = square(5); // TypeError: square is not a function

var square = function(number) {
  return number * number;
}
```

## First-class object 일급객체

일급 객체란 생성, 대입, 연산, 인자 또는 반환값으로 전달 등의 프로그래밍 언어의 기본적 조작을 제한없이 사용할 수 있는 대상을 말한다.

1. 무명의 리터럴로 표현이 가능하다.
2. 변수나 자료 구조(객체, 배열…)에 저장할 수 있다.
3. 함수의 파라미터로 전달할 수 있다.
4. 반환값(return value)으로 사용할 수 있다.
