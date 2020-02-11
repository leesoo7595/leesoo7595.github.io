---
layout: post
title: "[ES6] Iterator"
date: 2020-02-11
tags: [JavaScript]
categories:
- Javascript
comments: true
---

ES6에서 `Symbol`이 나오면서 도입된 이터레이션 프로토콜은, 데이터 컬렉션을 순회하기 위한 프로토콜이다. 이터레이션 프로토콜을 준수한 객체는 `for..of` 문으로 순회가 가능하고, Spread 문법의 피연산자가 될 수 있다. 이터레이션 프로토콜을 준수한 객체를 `Iterable`이라고 한다. `Iterable`은 내부적으로 `Symbol.iterator` 메소드를 가지고 있는 객체이다. 예를 들면 배열이 바로 `Iterable`이다.

```javascript
const array = [1, 2, 3];

// 배열은 Symbol.iterator 메소드를 소유한다.
// 따라서 배열은 이터러블 프로토콜을 준수한 이터러블이다.
console.log(Symbol.iterator in array); // true

// 이터러블 프로토콜을 준수한 배열은 for...of 문에서 순회 가능하다.
for (const item of array) {
  console.log(item);
}

const obj = { a: 1, b: 2 };

// 일반 객체는 Symbol.iterator 메소드를 소유하지 않는다.
// 따라서 일반 객체는 이터러블 프로토콜을 준수한 이터러블이 아니다.
console.log(Symbol.iterator in obj); // false

// 이터러블이 아닌 일반 객체는 for...of 문에서 순회할 수 없다.
// TypeError: obj is not iterable
for (const p of obj) {
  console.log(p);
}
```

배열, 문자열은 대표적인 이터러블이다. 이터러블은 리스트를 일반화시킨 객체이다. `for...of`가 시작되면, 다음 값이 필요할 때, `next()` 메소드를 이용하여 돌아간다. 여기서 `next()`의 반환값은 키밸류 형태의 객체여야 한다. 정확히는 `{done: Boolean, value: any}` 이런 형태가 이루어지는데, 여기서 key값이 `done=false` 일때는 value에 다음 값(any)이 저장되도록 한다. (순회가 계속 되도록 한다.)

## Iterator와 Array-like

ES6부터 이터러블이라는 것이 나오면서 확실히 순회가능한 종류들을 `Symbol.iterator` 메소드로 구분하지만, 사실 ES5 까지 계속 사용됐었던 유사 배열과 아주 비슷해보인다. 하지만 이 둘은 다르다.

* 이터러블은 `Symbol.iterator` 메소드가 존재하는 객체이다.
* `Array-like`는 `index`, `length` 프로퍼티만 존재하여 배열처럼 보이는 객체이다.

유사배열, 그리고 이터러블 기능을 모두 가지고 있는 객체도 존재한다. 이 말은 `for...of`를 사용할 수 있고, `index`, `length` 프로퍼티 또한 가지고 있는 객체인데, 이는 문자열이 대표적인 예이다.

```
Array-like !== Iterable
```

## Array.from

여기서 주의할 것은, 이터러블과 유사배열 객체는 `Array`가 아니라서, `push`, `pop` 등의 메소드를 지원하지 않는다. 이럴 때 사용하는 Array 메소드가 `Array.from`!!

`Array.from`은 범용 메소드로 이터러블, 유사배열 객체 둘다 파라미터로 받아서 리얼 Array로 만들어준다. 이렇게 한번 변환시켜주고 나서 배열 메소드를 사용할 수 있게된다.

```javascript
let arrayLike = {
  0: "Hello",
  1: "World",
  length: 2
};

let arr = Array.from(arrayLike); // (*)
alert(arr.pop()); // World (메서드가 제대로 동작합니다.)
```

## 참고

- [poiemaweb - iterator, for...of](https://poiemaweb.com/js-array)
- [iterable](https://ko.javascript.info/iterable)