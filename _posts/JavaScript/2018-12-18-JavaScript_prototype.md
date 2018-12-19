---
layout: post
title: "[JavaScript] Prototype"
date: 2018-12-18
tags: [JavaScript]
comments: true
---

자바스크립트는 클래스 기반 객체지향 프로그래밍인 java, c++과는 달리 프로토타입 기반 객체지향 프로그래밍 언어이다.

자바스크립트의 모든 객체는 자신의 부모 역할을 담당하는 객체와 연결되어 있다. 그리고 이는 부모 객체의 프로퍼티 또는 메소드를 상속받아 사용할 수 있다. 이러한 부모 객체를 Prototype이라고 한다.

```JavaScript
var student = {
  name: 'Lee',
  score: 90
};

// student에는 hasOwnProperty 메소드가 없지만 아래 구문은 동작한다.
console.log(student.hasOwnProperty('name')); // true

console.dir(student);
```

자바스크립트의 모든 객체는 자신의 프로토타입을 가리키는 prototype이라는 숨겨진 프로퍼티를 가진다.

```JavaScript
var student = {
  name: 'Lee',
  score: 90
}
console.log(student.__proto__ === Object.prototype); // true
```

객체를 생성할 때 프로토타입은 결정된다. 결정된 프로토타입 객체는 다른 임의의 객체로 변경할 수 있다. 이것은 부모 객체인 프로토타입을 동적으로 변경할 수 있다는 것을 의미하며 이를 활용하여 객체의 상속을 구현할 수 있다.

## [[Prototype]] 프로퍼티 vs prototype 프로퍼티

[[Prototype]] 프로퍼티는 자바스크립트의 모든 객체가 가지고 있는 자신의 프로토타입, 즉 숨겨진 프로퍼티이다. `__proto__와 [[prototype]]은 같은 개념`이다.

[[prototype]] 프로퍼티는 자신의 프로토타입 객체를 가리키는 숨겨진 프로퍼티이다. 함수도 객체이므로 [[prototype]] 프로퍼티를 갖는데, 함수 객체는 일반 객체와는 달리 prototype 프로퍼티도 갖는다.

* [[prototype]] 프로퍼티

- 함수를 포함한 모든 객체가 가지고 있는 프로퍼티이다.
- 객체의 입장에서 자신의 부모 역할을 하는 프로토타입 객체를 가리키며 함수 객체의 경우 `Function.prototype`를 가리킨다.

```JavaScript
console.log(Person.__proto__ === Function.prototype);
```

* prototype 프로퍼티

- 함수 객체만 가지고 있는 프로퍼티이다.
- 함수 객체가 생성자로 사용될 때 이 함수를 통해 생성될 객체의 부모 역할을 하는 객체(프로토타입 객체)를 카리킨다.

## constructor 프로퍼티

프토로타입 객체는 constructor 프로퍼티를 갖는다. 이것은 객체의 입장에서 자신을 생성한 객체를 가리킨다.

이건 아직 무슨 소리인지 잘 모르겠당 constructor가 뭔지만 아는정도 ㅎㅎ

## Prototype Chain

자바스크립트는 특정 객체의 프로퍼티나 메소드에 접근하려고 할 때 해당 객체에 접근하려는 프로퍼티 또는 메소드가 없다면 [[Prototype]] 프로퍼티가 가리키는 링크를 따라 자신이 부모 역할을 하는 프로토타입 객체의 프로퍼티나 메소드를 검색한다. 이를 프로토타입 체인이라 한다.

```javascript
var student = {
  name: 'Lee',
  score: 90
}
console.dir(student);
console.log(student.hasOwnProperty('name')); // true
console.log(student.__proto__ === Object.prototype); // true
console.log(Object.prototype.hasOwnProperty('hasOwnProperty')); // true
```
