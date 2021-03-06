---
layout: post
title: "[ES6] Symbol"
date: 2020-02-08
tags: [JavaScript]
categories:
- Javascript
comments: true
---

ES6에서 새롭게 추가된 `Symbol`은, 자바스크립트의 원시 타입 중 7번째 타입으로 변경이 불가능하다. Symbol은 주로 유일한 객체의 프로퍼티 키를 만들기 위해 사용한다.

즉, 객체의 경우 mutable한 객체인데, 객체지만 변경 불가능한 값으로 사용하고 싶을 때 Symbol을 쓴다.

## create Symbol

```javascript
let mySymbol = Symbol();

console.log(mySymbol);          // Symbol()
console.log(typeof mySymbol);   // symbol

let symbolWithDesc = Symbol('leesoo');

console.log(symbolWithDesc);    // Symbol(leesoo)
console.log(symbolWithDesc === Symbol('leesoo')); // false
```

`Symbol`은 `Symbol()` 함수로 생성한다. 이 함수는 호출될 때마다 `Symbol` 값을 생성하고, 이 때, `Symbol`은 객체가 아니라 변경 불가능한 원시 타입의 값이다.

**`Symbol()` 함수는 `String`, `Number`, `Boolean` 같은 객체를 생성하는 생성자 함수와는 다르게 `new` 연산자를 사용하지 않는다.** 사용할 경우, 심볼은 생성자가 아니라는 타입 에러를 뱉는다.

`Symbol()` 함수는 문자열을 인자로 전달할 수 있는데, 이 문자열은 `Symbol` 생성과는 영향이 없으며 디버깅할 때, description 정도로만 사용된다.

## use Symbol

객체의 경우, 모든 문자열로 프로퍼티의 키를 만들 수 있다. 그리고 객체의 프로퍼티 키로 `Symbol` 값도 사용할 수 있는데, 여기서 `Symbol`의 값은 유일한 값이므로 **해당 객체의 프로퍼티 키로 `Symbol` 값을 가질 경우, 다른 어떤 프로퍼티 키와도 충돌하지 않는다.**

```javascript
const obj = {};

obj.prop = 'myProp';
obj[123] = 123; // 123은 문자열로 변환된다.
// obj.123 = 123;  // SyntaxError: Unexpected number
obj['prop' + 123] = false;

console.log(obj); // { '123': 123, prop: 'myProp', prop123: false }
const mySymbol = Symbol('mySymbol');
obj[mySymbol] = 123;

console.log(obj); // { [Symbol(mySymbol)]: 123 }
console.log(obj[mySymbol]); // 123
```

## Symbol 객체

`Symbol()` 함수로 `Symbol` 값을 생성할 수 있는 의미는 `Symbol`은 함수 객체라는 뜻이다! 

<img width="703" alt="Screen Shot 2020-02-08 at 7 36 18 PM" src="https://user-images.githubusercontent.com/39291812/74083679-5b9af880-4aaa-11ea-85bc-0f9a58e6db73.png">

### Well-Known Symbols

`Well-Known Symbols`이란, ecmascript에서 사용하는 알고리즘에서 명시적으로 참조하는 `Symbol`의 내부적인 값들이다. 이 메소드들은, 모든 영역에서 공유된다.

이 `Well-Known Symbols`들은, `@@name`이라는 형태로 언급된다. 얘네들은 자바스크립트 엔진에서 상수로 존재하고, 이 아이들을 참조하여 일정한 처리를 한다. 이중에 `@@iterator`이라는 메소드는, `for-of`로 호출될 수 있는 이터레이터로 반환하는 메소드이다.

특정 객체가 `Symbol.interator`를 프로퍼티 key로 가지고 있으면, 자바스크립트 엔진이 이 객체는 **이터레이션 프로토콜**을 따르는 것으로 간주하고 이터레이터로 동작하도록 한다.

### Symbol.iterator

`Symbol.iterator`가 프로퍼티 key로 가지고 있는 객체는 **이터레이션 프로토콜**을 따른다. 이는 이터레이터를 반환한다.

`Symbol.iterator`를 내부적으로 프로퍼티 key로 사용하고 있는 `built-in` 객체는 `Array`, `String`, `Map`, `Set`, `Dom data structures`, `arguments` 가 있다. 이 객체들은 모두 `prototype` 체인으로 `Symbol.iterator`를 가지고 있다.

### Symbol.for

`Symbol.for` 메소드는 인자로 문자열을 전달받을 수 있다. 그 문자열을 키로 사용하여 `Symbol` 값들이 저장되어있는 전역 `Symbol` 레지스트리에서 해당 키와 일치하는 저장된 `Symbol` 값을 검색하는 일을 한다.

검색에 성공하면 검색된 `Symbol` 값을 반환하고, 실패할 경우 새로운 `Symbol` 값을 생성하여 전역에 저장한 후 Symbol 값을 반환한다.(`Symbol()` 함수를 사용하여 생성한 것과 같다.)

```javascript
// 전역 Symbol 레지스트리에 foo라는 키로 저장된 Symbol이 없으면 새로운 Symbol 생성
const s1 = Symbol.for('foo');
// 전역 Symbol 레지스트리에 foo라는 키로 저장된 Symbol이 있으면 해당 Symbol을 반환
const s2 = Symbol.for('foo');

console.log(s1 === s2); // true
```

`Symbol()` 함수는 인자에 같은 문자열을 넣든 상관없이 매번 다른 `Symbol` 값을 생성하지만, `Symbol.for` 메소드는 인자의 문자열에 따라 한번 전역 `Symbol`을 검색하여 같은 `Symbol` 값을 공유할 수 있도록 한다.

또한 `Symbol.for` 메소드를 통해서 생성한 `Symbol` 값은 반드시 키를 가지지만, `Symbol()` 함수를 통해서 생성된 `Symbol`은 키 값이 없다.

```javascript
const shareSymbol = Symbol.for('myKey');
const key1 = Symbol.keyFor(shareSymbol);
console.log(key1); // myKey

const unsharedSymbol = Symbol('myKey');
const key2 = Symbol.keyFor(unsharedSymbol);
console.log(key2); // undefined
```

## 참고

- [Symbol - poiemaweb](https://poiemaweb.com/es6-symbol)
- [ecma262](https://tc39.es/ecma262/)