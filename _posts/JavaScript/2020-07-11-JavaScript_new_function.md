---
layout: post
title: "Javascript - New Function"
date: 2020-07-11
tags: [JavaScript]
categories:
- Javascript
comments: true
---

## New Function 문법

함수를 만드는 방법은 함수 표현시과 함수 선언문이 있는데, 그 외에도 `New Function` 문법을 사용하여 함수를 만들 수가 있다.

```javascript
// 인수가 있는 함수
let sum = new Function('a', 'b', 'return a + b');

alert( sum(1, 2) ); // 3

let sayHi = new Function('alert("Hello")');

// 인수가 없는 함수
sayHi(); // Hello
```

`new Function` 문법을 사용하여 함수를 만들면, 런타임시에 받은 문자열을 사용하여 함수를 만들 수 있다는 것이다. 함수 표현식과 함수 선언문은 개발자가 직접 스크립트를 짜야만 만들 수 있다는 차이점이 있다.

그래서 위 문법은 서버에서 값을 받아 함수를 동적으로 만들어 동작시켜야할 때 사용할 수 있다.

함수는 `[[Environment]]`에 저장된 정보를 이용하여 함수 자신이 만들어진 곳을 기억하게 되는데, `new Function` 문법을 사용하여 만들어진 함수의 `[[Environment]]`는 현재 렉시컬 환경을 참조하지않고, 전역 렉시컬 환경을 참조하게 된다.

**그래서 `new Function` 문법을 사용한 함수는 외부 변수에 접근할 수 없고, 전역 변수에만 접근이 가능하다.**

```javascript
function getFunc() {
  let value = "test";

  let func = new Function('alert(value)');

  return func;
}

getFunc()(); // ReferenceError: value is not defined

function getFunc() {
  let value = "test";

  let func = function() { alert(value); };

  return func;
}

getFunc()(); // getFunc의 렉시컬 환경에 있는 값 "test"가 출력됩니다.
```

## Reference

- [모던자스 - New Function 문법](https://ko.javascript.info/new-function)