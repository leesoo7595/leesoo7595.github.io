---
layout: post
title: "[ES6] Generator"
date: 2020-02-16
tags: [JavaScript]
categories:
- Javascript
comments: true
---

`Generator`란, ES6에 도입된 함수의 한 종류로 이터러블을 생성하는 함수이다. `Generator` 함수 객체는 `GeneratorFunction` 생성자를 통해서 생성된다. 그래서 제너레이터 함수는 `GeneratorFunction` 객체이다. `GeneratorFunction`의 인스턴스는 `GeneratorFunction.prototype`로부터 프로퍼티와 메소드를 가지고 온다.

```javascript
var gen = function* () {
    console.log('generator');
    yield 1;
}

gen().next();
// generator
// {value: 1, done: false}

console.log(gen.prototype)  // Generator {}

Object.getPrototypeOf(gen).constructor  // GeneratorFunction

var g = new GeneratorFunction("a", "yield a * 2");
var iterator = g(10);
console.log(iterator.next().value); // 20
```

이는 비동기 처리에 유용하게 사용된다. 제너레이터 함수를 사용하게되면 이터레이션 프로토콜을 준수하여 이터러블을 생성하는 방식보다 훨신 간편하게 이터러블을 구현할 수 있다.

그 이유는 제너레이터 함수는 코드 블록의 실행을 일시 중지하거나 특정 시점에서 재시작할 수 있는 동작을 한다. 제너레이터 함수를 호출하면, 제너레이터를 반환한다. 제너레이터 함수는 `Symbol.iterator`를 소유한 이터러블이다. 그래서 제너레이터는 `next` 메소드를 소유하고 있다.

```javascript
function* counter() {
  console.log('첫번째 호출');
  yield 1;                  // 첫번째 호출 시에 이 지점까지 실행된다.
  console.log('두번째 호출');
  yield 2;                  // 두번째 호출 시에 이 지점까지 실행된다.
  console.log('세번째 호출');  // 세번째 호출 시에 이 지점까지 실행된다.
}

const generatorObj = counter();

console.log(generatorObj.next()); // 첫번째 호출 {value: 1, done: false}
console.log(generatorObj.next()); // 두번째 호출 {value: 2, done: false}
console.log(generatorObj.next()); // 세번째 호출 {value: undefined, done: true}
```

## 정의

제너레이터 함수는 `function*` 키워드로 선언하고, `yield` 문을 가지고 있다.

```javascript
// 함수 선언문
function* generatorFunc() {
    yield 1;
}

// 함수 표현식
const generatorFunc2 = function* () {
    yield 1;
};

// 제너레이터 메소드
const obj = {
    * genObjMethod() {
        yield 1;
    }
}

class Person {
    * genPersonMethod() {
        yield 1;
    }
}

// 호출방식은 보통 함수와 같다.
```

## 호출

제너레이터 함수를 호출하면, 제너레이터 객체를 반환한다. 제너레이터 객체는 이터러블이다. 그래서 다른 이터러블 객체처럼 `next` 메소드를 별도로 구현할 필요없이 바로 사용할 수 있다.

즉, 좀 더 간편하게 이터러블을 구현하여 편하게 사용되기 위한 함수이다.

```javascript
const genFunc = function* () {
  console.log('Point 1');
  yield 1;                // 첫번째 next 메소드 호출 시 여기까지 실행된다.
  console.log('Point 2');
  yield 2;                // 두번째 next 메소드 호출 시 여기까지 실행된다.
  console.log('Point 3');
  yield 3;                // 세번째 next 메소드 호출 시 여기까지 실행된다.
  console.log('Point 4'); // 네번째 next 메소드 호출 시 여기까지 실행된다.
};

console.log(generatorObj.next());
// Point 1
// {value: 1, done: false}
// 첫번째 next 메소드 호출: 첫번째 yield 문까지 실행되고 일시 중단된다.

console.log(generatorObj.next());
// Point 2
// {value: 2, done: false}
// 두번째 next 메소드 호출: 두번째 yield 문까지 실행되고 일시 중단된다.
//...
```

제너레이터 함수를 만들어서 next 메소드를 통해 호출하면, 처음 만나는 `yield` 문까지만 실행되고 중단(suspend)된다. 그리고 그 후 next 메소드를 호출하면 마찬가지로 그 다음 `yield`까지 실행되고 중단된다. 그리고 반환 값을 제너레이터 객체로 `value`, `done` 프로퍼티를 갖는 객체를 가진다. 마지막 `yield` 문까지 실행되면 `done` 값은 `true`가 된다.

## 활용

이터레이터의 next 메소드는 인수를 전달할 수 없지만, 제너레이터 객체의 next 메소드에는 인수를 전달하는 것이 가능하다. 그래서 이를 통해 제너레이터 객체에 데이터를 전달한다.

```javascript
function* gen(n) {
  let res;
  res = yield n;    // n: 0 ⟸ gen 함수에 전달한 인수

  console.log(res); // res: 1 ⟸ 두번째 next 호출 시 전달한 데이터
  res = yield res;

  console.log(res); // res: 2 ⟸ 세번째 next 호출 시 전달한 데이터
  res = yield res;

  console.log(res); // res: 3 ⟸ 네번째 next 호출 시 전달한 데이터
  return res;
}
const generatorObj = gen(0);

console.log(generatorObj.next());  // 제너레이터 함수 시작
console.log(generatorObj.next(1)); // 제너레이터 객체에 1 전달
console.log(generatorObj.next(2)); // 제너레이터 객체에 2 전달
console.log(generatorObj.next(3)); // 제너레이터 객체에 3 전달
/*
{ value: 0, done: false }
{ value: 1, done: false }
{ value: 2, done: false }
{ value: 3, done: true }
*/
```

난 이 코드가 잘 이해가 안가지만.....이를 통해 제너레이터 객체에 데이터를 넣어 next 메소드에 인수 전달이 가능하고, 이를 통해 동시성 프로그래밍이 가능하게 한다고 한다... 동시성 프로그래밍은 한번 호출할 때 하나의 실행만을 하는 것이 아닌 여러 개의 일을 수행할 수 있는 것을 말한다.

### 비동기 처리

제너레이터를 활용하여 비동기 처리를 동기 처리처럼 만들 수 있는데, 이것이 나중에 ES7에서 나오는 async/await이다.

```javascript
function getUser(genObj, username) {
  fetch(`https://api.github.com/users/${username}`)
    .then(res => res.json())
    // ① 제너레이터 객체에 비동기 처리 결과를 전달한다.
    .then(user => genObj.next(user.name));
}

// 제너레이터 객체 생성
const g = (function* () {
  let user;
  // ② 비동기 처리 함수가 결과를 반환한다.
  // 비동기 처리의 순서가 보장된다.
  user = yield getUser(g, 'jeresig');
  console.log(user); // John Resig

  user = yield getUser(g, 'ahejlsberg');
  console.log(user); // Anders Hejlsberg

  user = yield getUser(g, 'ungmo2');
  console.log(user); // Ungmo Lee
}());

// 제너레이터 함수 시작
g.next();
```

<img width="505" alt="Screen Shot 2020-02-16 at 6 14 12 PM" src="https://user-images.githubusercontent.com/39291812/74602058-2f096100-50e8-11ea-9243-6da9dd6bedd6.png">

## 참고

- [Generator: Async/await - Poiemaweb](https://poiemaweb.com/es6-generator)
- [GeneratorFunction - mdn](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/GeneratorFunction)
