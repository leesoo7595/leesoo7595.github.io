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

`Symbol.iterator`가 객체에 특수 내장 메소드로 존재하지 않으면, `for...of`가 실행되지 않는다. 즉, `for...of`가 시작되자마자 `Symbol.iterator`를 호출한다. 또한 `Symbol.iterator`는 이터레이터를 반환해야한다. 여기서 이터레이터란 내부적으로 `next()` 메소드를 가지고 있는 애들이다. 그리고 `for...of`로 동작하는 아이들은 모두 이터레이터들이다.

`for...of`는 이터레이터의 `next()` 메서드를 호출한다. 그리고 `next()`의 반환값은 `{done: Boolean, value: any}`와 같은 형태이다. `done=true`인 경우, 반복이 종료되었음을 의미한다. 반대로는 value 값에 다음 값이 저장된다.

정리하면, 배열, 문자열은 대표적인 이터러블이다. 이터러블은 리스트를 일반화시킨 객체이다. `for...of`가 시작되면, 다음 값이 필요할 때, `next()` 메소드를 이용하여 돌아간다. 여기서 `next()`의 반환값은 키밸류 형태의 객체여야 한다. 정확히는 `{done: Boolean, value: any}` 이런 형태가 이루어지는데, 여기서 key값이 `done=false` 일때는 value에 다음 값(any)이 저장되도록 한다. (순회가 계속 되도록 한다.)

## Iterable와 Array-like

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

## Iterator

이터러블 프로토콜에는 `Iterable`과 `Iterator` 두 가지 형태가 존재한다. 우선 `Iterable`은 위에서 언급했듯이 객체 내부를 순회할 수 있는 객체라는 의미이다. 객체 내부적으로 `Symbol.iterator`를 확인하여 `for...of`를 이용해서 순회한다.

```javascript
const iterable = new Object();

iterable[Symbol.iterator] = function* () {
  yield 1;
  yield 2;
  yield 3;
};

console.log([...iterable]); // 1 2 3
for(var value of iterable) {
    console.log(value); // 1 2 3
}
```

`Iterable`과 `Iterator`의 정확한 구분을 하자면, Iterable 객체는 내부적으로 `@@iterator` 프로퍼티를 포함하고 있다. 그리고 이 프로퍼티가 `Iterator`를 말하는 것으로, 이 프로퍼티는 `Iterator` 객체를 반환한다.. 그리고 Iterator라는 인터페이스는 내부적으로 `next`라는 프로퍼티를 포함하고 있다. `next`라는 아이는 `IteratorResult`라는 객체를 반환하는 함수이다.

`next()`라는 프로퍼티는 정확히 `IteratorResult` 인터페이스를 따르는 객체를 리턴한다. 만약 이전 `next` 메소드가 호출됐다면 `IteratorResult` 객체가 반환되는데, 그 값이 위에서 말한 `{done: Boolean, value: any}` 이런 식으로 되어있는 객체이다. done=true 일때까지 순회한다.
next() 메소드를 가지고 있고, 해당 역할까지 수행하는 인터페이스가 `Iterator`으로, `Iterable`만으로는 확연히 차이가 있다고 말할 수 있다.

그리고 결국, 이 `Iterator`를 가지고 있는 인터페이스가 `Iterable`이 된다. 다시말하면, `[[Iterator]]`를 가지고 있는 객체일 것이다.

## 참고

- [poiemaweb - iterator, for...of](https://poiemaweb.com/js-array)
- [iterable](https://ko.javascript.info/iterable)
- [Javascript와 Iterator](https://medium.com/@pks2974/javascript%EC%99%80-iterator-cdee90b11c0f)
- [ecma262 - iterator](https://tc39.es/ecma262/)