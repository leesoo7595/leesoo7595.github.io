---
layout: post
title: "[JavaScript] function (2)"
date: 2018-12-23
tags: [JavaScript]
comments: true
---

참고도서 : 러닝 자바스크립트

## 호출과 참조

자바스크립트에서는 함수도 객체이다. 함수 식별자 뒤에 괄호를 쓰면 자바스크립트는 함수를 호출하려 하는 것이라 이해하고 이를 실행하려 한다. 그리고 함수를 호출한 표현식은 반환값 `return`이 된다.

함수 뒤에 괄호를 쓰지 않으면 함수를 호출하는 것이 아니라 참조하는 것이다. 그러므로 그 함수는 실행되지 않는다. 이는 자바스크립트를 유용한 언어로 만들며, 함수를 변수에 할당하면 다른 이름으로 함수를 호출할 수도 있다. 그리고 함수를 객체 프로퍼티에 할당할 수도 있고, 배열 요소로 할당할 수도 있다.

## 함수와 매개변수

매개변수는 함수가 호출되기 전에는 존재하지 않는다는 것을 제외하면 일반적인 변수와 같다. 매개변수는 변수와 매우 비슷하지만 함수 바디 안에서만 존재한다.

```javascript
const a = 5, b = 10;
avg(a, b);
```

첫 행의 변수 a, b는 avg 매개변수인 a, b와 같은 이름이지만, 다른 변수이다. 함수를 호출하면 함수 매개변수는 변수 자체가 아니라 변수의 값을 전달받는다.

즉, 함수 안에서 a, b값이 바뀌더라고 바깥의 변수 a, b엔 아무 영향도 없다는 것이다. 이름은 같지만, 다른 객체이다. 하지만 함수 안에서 객체 자체를 변경하면 그 객체는 함수 밖에서도 반영된다.

```javascript
function f(o) {
  o.message = `f 안에서 수정함 (이전 값: '${o.message}')`; // o.message의 참조값이 바뀜
  o = {
    message: "새로운 객체";  // function 내부에 o.message 재할당
  };
  console.log(`f 내부: o.message="${o.message}" (할당 후)`); // 새로운 객체
}
let o = {
  message: "초기 값"
};
console.log(`f를 호출하기 전: o.message="${o.message}"`); // 초기 값
f(o); // 함수 호출 -> 새로운 객체
console.log(`f를 호출한 다음: o.message="${o.message}"`); // f안에서 수정함
```

함수 f 안에서 객체 o를 수정하였다. 이렇게 바꾼 내용은 함수 바깥에서도 o에 그대로 반영되어 있다. 이는 원시 값과 객체의 핵심적인 차이이다. 원시 값은 불변이므로 수정할 수 없고, 원시값을 담은 변수는 수정할 수 있지만 원시 값 자체는 바뀌지 않는다. 반면 객체는 바뀔 수 있다.

위 예제는 함수 내에서 객체 값을 수정한 것이다. 원시 타입은 해당 값을 가져올 때 그 값을 복사하여 가지고 오지만, 객체는 그 값을 참조하여 가져온다. 그런 차이로 f 함수를 호출하면 f 내부에서 o에 할당한 객체는 전혀 다른 객체이고, 다른 둘은 같은 객체를 가리키므로 함수 바깥의 o는 여전히 원래 객체를 가리키고 있다.

-> ..리얼 어렵

## 매개변수가 함수를 결정하는가?

자바스크립트에서는 함수 f가 있다면 호출할 때 매개변수를 다르게 전달해도 같은 함수를 호출한다. 정해진 매캐변수에 값을 제공하지 않으면 암시적으로 `undefined`가 할당된다.

```javascript
function f(x) {
  return `in f: x=${x}`;
}
f();    // "in f: x=undefined"
```

## 매개변수 해체

매개변수도 해체할 수 있다.

```javascript
function getSentence({ subject, verb, object }) {
  return `${subject} ${verb} ${object}`;
}

const o = {
  subject: "I",
  verb: "love",
  object: "JavaScript"
}

getSentence(o);   // "I love JavaScript"
```

해체 할당과 마찬가지로 프로퍼티 이름은 유효한 식별자여야 하고, 들어오는 객체에 해당 프로퍼티가 없는 변수는 undefined를 할당받는다. 배열도 마찬가지로 해체할 수 있다. 함수를 선언할 때 확산 연산자는 마지막 매개변수여야 한다.

## 매개변수 기본값

`ES6`에서 매개변수에 기본값을 지정하는 기능도 추가되었다. 일반적으로 매개변수에 값을 제공하지 않으면 undefined가 값으로 할당된다. `ES6`에서는 기본값을 지정할 수 있다.

```javascript
function f(a, b = "default", c = 3) {
  return `${a} - ${b} - ${c}`;
}

f(5, 6, 7); // "5 - 6 - 7"
f(5, 6); // "5 - 6 - 3"
f(5); // "5 - default - 3"
f(); // "undefined - default - 3"
```

## 객체의 프로퍼티인 함수

객체의 프로퍼티인 함수를 메서드라고 불러서 일반적인 함수와 구별한다. 함수를 다른 객체에 추가할 수 있고, 객체 리터럴에서도 메서드를 추가할 수 있다.

```javascript
const o = {
  name: 'Wallace',      // 원시 값 프로퍼티
  bark: function() {    // 함수 프로퍼티 (메서드)
    return 'Woof';
  },
}

// ES6에서 간편하게 메서드를 추가할 수 있는 문법이 새로 생김
const o = {
  name: 'Wallace',            // 원시 값 프로퍼티
  bark() { return 'Woof'; },  // 함수 프로퍼티 (메서드)
}
```

## this 키워드

함수 바디 안에는 특별한 읽기 전용 값인 `this`가 있다. `this`는 일반적으로 객체지향 프로그래밍 개념에 밀접한 연관이 있다.

메서드를 호출하면 `this`는 호출한 메서드를 소유하는 객체가 된다.

```javascript
const o = {
  name: 'Wallace',            // 원시 값 프로퍼티
  bark() { return 'Woof'; },  // 함수 프로퍼티 (메서드)
}

// o.bark() 호출하면 this는 o에 묶이게된다.
o.bark();                     // 'Woof'
```

`this`는 함수를 어떻게 선언했느냐가 아니라, 어떻게 호출헀느냐에 따라 달라진다. this가 위의 식에서 o에 묶인 이유는 bark가 o의 프로퍼티여서가 아니라, o에서 bark를 호출했기 때문이다......

```javascript
const bark = o.bark;
bark === o.bark;          // true; 두 변수는 같은 함수를 가리킨다.
bark();                   // 'Undefined'
```

같은 함수를 변수에 할당한 것이다. 위처럼 함수를 호출할 경우 자바스크립트는 이 함수가 어디에 속하는지 알 수 없으므로 this는 undefined에 묶인다.

## 함수 표현식과 익명 함수

함수를 선언하면 함수에 바디와 식별자가 모두 주어진다. 자바스크립트는 익명 함수도 지원하는데, 익명 함수에서는 함수에 식별자가 주어지지 않는다.

함수 표현식은 함수를 선언하는 한 가지 방법이다. 그리고 그 함수가 익명이 될 수도 있다. 함수 표현식은 식별자에 할당할 수도 있고, 즉시 호출할 수도 있다.

```javascript
const f = function() {
  // ....
};

const g = function f() {
  // ...
}
```

이런 식으로 함수를 만들면 이름 g에 우선순위가 있다. 그리고 함수 바깥에서 함수에 접근할 때는 g를 써야 하고, f로 접근하려하면 변수가 정의되지 않았다는 에러가 생긴다.

나중에 호출할 생각으로 함수를 만든다면 함수 선언을 사용하고, 다른 곳에 할당하거나 다른 함수에 넘길 목적으로 함수를 만든다면 함수 표현식을 사용한다.

그리고 난 이 말이 무슨 말인지 모르겟다;; 그래서 따로 알아봄

함수 선언식과 표현식의 차이점

- 함수 선언식은 호이스팅에 영향을 받지만, 표현식은 영향을 받지 않는다.
- 함수 표현식은 클로저, 콜백으로 사용된다.(다른 함수의 인자로 넘길 수 있다.)

## 화살표 표기법

* function을 생략한다.
* 함수에 매개변수가 단 하나 뿐이라면 괄호(())도 생략한다.
* 함수 바디가 표현식 하나라면 중괄호와 return문도 생략할 수 있다.

화살표 함수는 항상 익명이다. 변수에 할당할 수는 있지만 function 키워드처럼 이름 붙은 함수를 만들 수 없다.

```javascript
const f1 = function() { return "hello!"; }
const f1 = () => "hello!";

const f2 = function(name) { return `Hello, ${name}!`; }
const f2 = name => `Hello, ${name}!`;

const f3 = function(a, b) { return a + b; }
const f3 = (a, b) => a + b;
```

화살표 함수는 일반 함수와 달리 `this`가 다른 변수와 마찬가지로 정적으로 묶인다. 화살표 함수를 사용하면 내부 함수 안에서 `this`를 사용할 수 있다.

화살표 함수는 객체 생성자로 사용할 수 없고, arguments 변수도 사용할 수 없다.

## call과 apply, bind

`this`를 일반적인 방법 외에도 함수를 어디서 어떻게 호출했느냐와 관계없이 지정할 수 있다.
`call` 메서드는 모든 함수에서 사용할 수 있고, `this`를 특정 값으로 지정할 수 있다.

```javascript
const bruce = { name: "Bruce" };
const madeline = { name: "Madeline" };

// 이 함수는 어떤 객체에도 연결되지 않았지만 this를 사용한다.
function greet() {
  return `Hello, I'm ${this.name}.`;
}

greet();              // "Hello, I'm undefined." - this는 어디에도 묶이지 않았다.
greet.call(bruce);    // "Hello, I'm Bruce." - this는 bruce이다.
greet.call(madeline); // "Hello, I'm Madeline." - this는 madeline이다.
```

함수를 호출하면서 call을 사용하고 this로 사용할 객체를 넘기면 해당 함수가 주어진 객체의 메서드인 것처럼 사용할 수 있다.

`apply`는 매개변수를 배열로 받는다. 이 외에는 `call`과 같다.

```javascript
function update(birthYear, occupation) {
  this.birthYear = birthYear;
  this.occupation = occupation;
}

update.call(bruce, 1949, 'singer');
// bruce는 이제 { name: "Bruce", birthYear: 1949, occupation: "singer"}

update.apply(bruce, [1995, "actor"]);
// bruce는 이제 { name: "Bruce", birthYear: 1995, occupation: "actor"}

const updateBruce = update.bind(bruce);

updateBruce(1904, "actor");
// bruce는 이제 { name: "Bruce", birthYear: 1904, occupation: "actor"}

updateBruce(madeline, 1274, "king");
// bruce는 이제 { name: "Bruce", birthYear: 1274, occupation: "king"}
// madeline은 변하지 않았다.
```

그리고 `this` 값을 바꿀 수 있는 마지막 함수는 `bind`이다. `bind`를 사용하면 함수의 `this` 값을 영구히 바꿀 수 있다. `update`메서드를 이리저리 옮기면서도 호출할 때 `this`값을 항상 `bruce`가 되게끔하려면 `bind`를 사용한다.

`bind`는 함수의 동작을 영구적으로 바꾸므로 `bind`를 사용한 함수는 `call`, `apply`, 다른 `bind`와 함께 사용할 수 없다. 함수의 `this`가 어디에 묶이는지 정확히 파악하고 사용해야 한다.

어렵당어렵당어렵다...
