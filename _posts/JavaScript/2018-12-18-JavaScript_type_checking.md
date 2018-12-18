---
layout: post
title: "[JavaScript] Type Checking"
date: 2018-12-18
tags: [JavaScript]
comments: true
---

자바스크립트는 동적 타입 언어이므로 변수에 어떤 타입의 값을 받아 반환하려는 건지 미리 지정해두지 않으면 혼란이 올 수 있다. 그러므로 자바스크립트는 타입 체크가 필요하다.

## typeof

```JavaScript
typeof '';              // string
typeof 1;               // number
typeof NaN;             // number
typeof true;            // boolean
typeof [];              // object
typeof {};              // object
typeof new String();    // object
typeof new Date();      // object
typeof /test/gi;        // object
typeof function () {};  // function
typeof undefined;       // undefined
typeof null;            // object (설계적 결함)
typeof undeclared;      // undefined (설계적 결함)
```

`typeof` 연산자의 경우 null을 제외한 원시 타입을 체크하는 데는 문제가 없다. 하지만 객체의 종류까지 구분하여 체크할 땐 사용할 수 없다.

## Object.prototype.toString

`Object.prototype.toString` 메소드는 객체를 나타내는 문자열을 반환한다.

Function.prototype.call 메소드를 사용하면 모든 타입의 값을 알아낼 수 있다.

```JavaScript
function getType(target) {
  return Object.prototype.toString.call(target).slice(8, -1);
}
```

## instanceof

위의 `Object.prototype.toString`를 사용하여 객체의 종류까지 식별할 수 있지만, 객체의 상속 관계까지 체크할 수는 없다.

```JavaScript
function Person() {}
const person = new Person();

console.log(person instanceof Person); // true
console.log(person instanceof Object); // true
```

`instanceof` 연산자는 피연산자인 객체가 우항에 명시한 타입(`constructor`)의 인스턴스인지 여부를 알려준다.

## 유사 배열 객체

`Array.isArray` 메소드는 배열인지 체크할 수 있다.

```JavaScript
console.log(Array.isArray([]));    // true
console.log(Array.isArray({}));    // false
console.log(Array.isArray('123')); // false
```

유사 배열 객체란 length 프로퍼티를 갖는 객체로 문자열, arguments, HTMLCollection, NodeList 등을 말한다. 이는 length 프로퍼티를 가지고 있어 순회할 수가 있고 call, apply 함수를 사용하여 배열의 메소드를 사용할 수도 있다.

그러므로 어떤 객체가 유사 배열인지 체크하려면 length 프로퍼티를 갖는지, 갖는다면 length프로퍼티의 값이 정상적인 값인지 체크한다.
