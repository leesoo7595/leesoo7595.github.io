---
layout: post
title: "[JavaScript] JavaScript란?"
date: 2018-12-18
tags: [JavaScript]
comments: true
---

자바스크립트는 명령형, 함수형, 프로토타입 기반 객체지향 프로그래밍을 지원하는 멀티-패러다임 프로그래밍 언어이다.

## script 태그의 async/defer 어트리뷰트

async : 웹페이지 파싱과 외부 스크립트 파일의 다운로드가 동시에 진행된다. 스크립트는 다운로드 완료 직후 실행된다.

defer : 웹페이지 파싱과 외부 스크립트 파일의 다운로드가 동시에 진행된다. 스크립트는 웹페이지 파싱 완료 후 실행된다.

동적 타이핑 : 변수에 할당된 값의 타입에 의해 동적으로 변수의 타입이 결정되는 것

암묵적 타입 강제 변환 -> 피연산자의 타입은 반드시 일치할 필요가 없다

프로그램(스크립트)은 컴퓨터(Client-side JavaScript의 경우, 웹 브라우저)에 의해 단계별로 수행될 명령들의 집합이다. 각각의 명령을 문이라 하며 문이 실행되면 무슨 일인가가 일어나게 된다.

문 -> 리터럴, 연산자, 표현식, 키워드 등으로 구성되며 세미콜론으로 끝나야한다.

## 객체

자바스크립트는 객체(object) 기반의 스크립트 언어, 자바스크립트를 이루고 있는 거의 모든 것이 객체이다. 원시 타입을 제외한 나머지 값들(함수, 배열, 정규표현식 등)은 모두 객체이다.

자바스크립트 객체는 키와 값으로 구성된 프로퍼티의 집합이다. 자바스크립트의 함수는 일급 객체이므로 값으로 취급할 수 있다. 프로퍼티 값으로 함수를 사용할 수 있고, 이를 일반함수와 구분하기 위해 메소드라 부른다.

```JavaScript
var person = {
  name: 'Lee',
  gender: 'male',
  sayHello: function () {
    console.log('Hi! My name is ' + this.name);
  }
};

console.log(typeof person); // object
console.log(person); // { name: 'Lee', gender: 'male', sayHello: [Function: sayHello] }

person.sayHello(); // Hi! My name is Lee
```

## 변수 호이스팅

호이스팅이란 var 선언문이나 function 선언문 등 모든 선언문이 해당 스코프의 선두로 옮겨진 것처럼 동작하는 특성 -> 자스는 모든 선언문이 선언되기 전에 참조 가능

var 키워드 -> 중복선언 가능
변수가 아직 선언되지 않았ㅇ을 경우 -> undefined

자바스크립트는 모든 선언문이 선언되기 이전에 참조 가능

선언 단계 / 초기화 단계 / 할당 단계

선언 단계 : 변수 객체에 변수 등록 -> 스코프 참조 대상
초기화 단계 : 변수 객체에 등록된 변수를 메모리에 할당 -> undefined로 초기화
할당 단계 : undefined로 초기화된 변수에 실제값 할당

자스의 변수는 **블록 레벨 스코프** 를 가지지 않고 **함수 레벨 스코프** 를 갖는다. 단, let, const 키워드를 사용하면 블록 레벨 스코프를 사용할 수 있다.

함수 헤벨 스코프 : 함수 내에서 선언된 변수는 함수 내에서만 유효, 함수 외부에서는 참조할 수 없다. 즉 함수 내부에서 선언한 변수는 지역 변수이며 함수 외부에서 선언한 변수는 모두 전역 변수이다.

블록 레벨 스코프 : 코드 블록 내에서 선언된 변수는 코드 블록 내에서 유효하며, 코드 블록 외부에서는 참조할 수 없다.

## var 키워드로 선언된 변수의 문제점

1. 함수 레벨 스코프
  * 전역 변수의 남발
  * for loop 초기화식에서 사용한 변수를 for loop 외부 또는 전역에서 참조할 수 있다.

2. var 키워드 생략 허용
  * 의도하지 않은 변수의 전역화

3. 중복 선언 허용
  * 의도하지 않은 변수값 변경

4. 변수 호이스팅
  * 변수를 선언하기 전에 참조가 가능하다.

-> 변수의 유효 범위는 좁을수록 좁다.
-> let, const 키워드 도입

## typeof 연산자

함수의 경우 function을 반환하고, null값은 object를 반환한다. 선언하지 않은 식별자는 undefined를 반환한다.

## 타입 변환

개발자에 의해 의도적으로 값의 타입을 변환하는 것을 **명시적 타입 변환 또는 타입 캐스팅** 이라한다.

```JavaScript
var x = 10;

// 명시적 타입 변환
var str = x.toString(); // 숫자를 문자열로 타입 캐스팅한다.
console.log(typeof str); // string
```

동적 타입 언어인 자스는 개발자의 의도와는 상관없이 자바스크립트 엔진에 의해 암묵적으로 타입이 자동 변환되기도 한다. 이를 **암묵적 타입 변환 또는 타입 강제 변환** 이라한다.

```JavaScript
var x = 10;

// 암묵적 타입 변환
// 숫자 타입 x의 값을 바탕으로 새로운 문자열 타입의 값을 생성해 표현식을 평가한다.
var str = x + '';

console.log(typeof str, str); // string 10

// 변수 x의 값이 변경된 것은 아니다.
console.log(x); // 10
```

## 문자열 타입으로 변환

```
console.log(String(1));
console.log((1).toString());
console.log(1 + '');
```

## 숫자 타입으로 변환

```
console.log(Number('0'));
console.log(parseInt('0'));
console.log(+'0');
```

## 단축 평가

  * 객체가 null인지 확인하고 프로퍼티를 참조할 때

```JavaScript
var elem = null;

console.log(elem.value);  // TypeError: Cannot read property 'value of null'
console.log(elem && elem.value);  // null
```

  * 함수의 인수(arguments)를 초기화할 때

  ```JavaScript
  // 단축 평가를 사용한 매개변수의 기본값 설정
  function getStringLength(str) {
    str = str || '';
    return str.length;
  }

  getStringLength();     // 0
  getStringLength('hi'); // 2

  // ES6의 매개변수의 기본값 설정
  function getStringLength(str = '') {
    return str.length;
  }

  getStringLength();     // 0
  getStringLength('hi'); // 2
  ```

  함수를 호출할 때 인수를 전달하지 않으면 매개변수는 undefined를 갖는다. 이때 단축 평가를 사용하여 매개변수의 기본값을 설정하면 undefined로 인해 발생할 수 있는 에러를 방지할 수 있다.

## 객체

자바스크립트는 객체 기반의 스크립트 언어이다. 또한 프로토타입 기반 객체 지향 언어로서 클래스라는 개념이 없고 별도의 객체 생성 방법이 존재한다.

**ECMA6에서 클래스 도입, 프로토타입 기반 프로그래밍은 클래스가 존재하지 않는 객체지향 프로그래밍 스타일이다. 클래스 없이 프로토타입 체인과 클로저 등으로 객체 지향 언어의 상속, 캡슐화(정보 은닉) 등의 개념을 구현한다. 이를 ECMA6의 클래스는 기존 프로토타입 기반 객체지향 프로그래밍보다 클래스 기반 언어에 익숙한 프로그래머가 보다 빠르게 학습할 수 있는 단순하고 깨끗한 새로운 문법을 제시하고 있다. ES6의 클래스가 새로운 객체지향 모델을 제공하는 것이 아니며 클래스도 사실 함수이고 기존 프로토타입 기반 패턴의 문법적 설탕이다.**

## 생성자 함수

```JavaScript
// 생성자 함수
function Person(name, gender) {
  this.name = name;
  this.gender = gender;
  this.sayHello = function(){
    console.log('Hi! My name is ' + this.name);
  };
}

// 인스턴스의 생성
var person1 = new Person('Lee', 'male');
var person2 = new Person('Kim', 'female');

console.log('person1: ', typeof person1);
console.log('person2: ', typeof person2);
console.log('person1: ', person1);
console.log('person2: ', person2);

person1.sayHello();
person2.sayHello();
```

* 생성자 함수 이름은 일반적으로 대문자로 시작한다. 생성자 함수임을 인식하도록 도움을 준다.
* 프로퍼티 또는 메소드명 앞에 기술한 this는 생성자 함수가 생성할 인스턴스를 가리킨다.
* this에 연결(바인딩)되어 있는 프로퍼티와 메소드는 public하다.
* 생성자 함수 내에서 선언된 일반 변수는 private하다. 즉, 생성자 함수 내부에서는 자유롭게 접근 가능하나, 외부에서 접근할 수 없다.

## for-in 문

```JavaScript
var person = {
  'first-name': 'Ung-mo',
  'last-name': 'Lee',
  gender: 'male'
};

// prop에 객체의 프로퍼티 이름이 반환된다. 단, 순서는 보장되지 않는다.
for (var prop in person) {
  console.log(prop + ': ' + person[prop]);
}

/*
first-name: Ung-mo
last-name: Lee
gender: male
*/

var array = ['one', 'two'];

// index에 배열의 경우 인덱스가 반환된다
for (var index in array) {
  console.log(index + ': ' + array[index]);
}

/*
0: one
1: two
*/
```

for-in 문은 객체의 문자열 키를 순회하기 위한 문법이다. 배열에는 사용하지 않는 것이 좋다.

* 객체의 경우, 프로퍼티의 순서가 보장되지 않는다. 원래 객체의 프로퍼티에는 순서가 없기 때문이다. 배열은 순서를 보장하는 데이터 구조이지만 순서를 보장하지 않는다.
* 배열 요소들만을 순회하지 않는다.

```JavaScript
// 배열 요소들만을 순회하지 않는다.
var array = ['one', 'two'];
array.name = 'my array';

for (var index in array) {
  console.log(index + ': ' + array[index]);
}

/*
0: one
1: two
name: my array
*/
```

-> 그래서 for-of 문이 추가되었다.

```JavaScript
const array = [1, 2, 3];
array.name = 'my array';

for (const value of array) {
  console.log(value);
}

/*
1
2
3
*/

for (const [index, value] of array.entries()) {
  console.log(index, value);
}

/*
0 1
1 2
2 3
*/
```

## pass-by-reference

object type을 객체 타입 또는 참조 타입이라 한다. 참조 타입이란 객체의 모든 연산이 실제값이 아닌 참조값으로 처리됨을 의미한다. 원시 타입ㄷ은 값이 한 번 정해지면 변경할 수 없지만`immutable`, 객체는 프로퍼티를 변경, 추가, 삭제가 가능하므로 변경 가능한 값`mutable`이라 할 수 있다.

따라서 객체 타입은 동적으로 변화할 수 있으므로 어느 정도의 메모리 공간을 확보해야 하는지 예측할 수 없기 때문에 런타임에 메모리 공간을 확보하고 메모리의 힙 영역에 저장된다.

이에 반해 원시 타입은 값으로 전달된다. 즉, 복사되어 전달된다. 이를 `pass-by-value`라 한다.

```JavaScript
// Pass-by-reference
var foo = {
  val: 10
}

var bar = foo;
console.log(foo.val, bar.val); // 10 10
console.log(foo === bar);      // true

bar.val = 20;
console.log(foo.val, bar.val); // 20 20
console.log(foo === bar);      // true
```

위에서 foo 객체는 리터럴 방식으로 생성하였다. 즉 변수 foo는 객체 자체를 저장하고 있는 것이 아니라 생성된 객체의 참조값을 저장하고 있다.

그리고 변수 bar에 변수 foo 값을 할당하였다. foo값은 생성된 객체를 가리키는 참조값이므로 변수 bar에도 같은 참조값이 저장된다. 즉, foo와 bar 모두 동일한 객체를 참조하고 있다.

```JavaScript
var foo = { val: 10 };
var bar = { val: 10 };

console.log(foo.val, bar.val); // 10 10
console.log(foo === bar);      // false

var baz = bar;

console.log(baz.val, bar.val); // 10 10
console.log(baz === bar);      // true
```

변수 foo와 bar은 비록 내용은 같지만 별개의 객체를 생성하여 참조값을 할당하였다. 따라서 변수 foo와 변수 bar의 참조값 즉 주소는 동일하지 않다.

변수 baz에는 bar의 값을 할당하여 동일한 객체를 가리키는 참조값을 저장하고 있어 둘의 참조값은 같다.

## 객체의 분류

* Built-in Object(내장 객체) : 웹페이지 등을 표현하기 위한 공통의 기능을 제공. 웹페이지가 브라우저에 의해 로드되자마자 별다른 행위없이 바로 사용이 가능하다.
  - Standard Built-in object
  - BOM (Browser Object Model)
  - DOM (Document Object Model)

BOM, DOM은 Native Object라고 분류하기도 한다. 또한 사용자가 생성한 객체를 Host Object라고 한다.

* Host Object(사용자 정의 객체)
