---
layout: post
title: "[JavaScript] Data Type"
date: 2018-12-23
tags: [JavaScript]
comments: true
---

참고도서 : 러닝 자바스크립트

## 원시타입과 객체

* 숫자
* 문자열
* 불리언
* null
* 심볼
* undefined

```JavaScript
let str = "hello"
str = "world"
```

위 식은 str은 불변의 값인 "hello"로 초기화했고, 다시 새로운 불변값 "world"를 할당받았다. 여기서 "hello"와 "world"는 다른 문자열이다. 바뀐 것은 str의 저장하는 값 뿐이다.

위의 여섯 가지 원시타입 외엔 객체가 있다. 원시 값과 달리 객체는 여러 가지 형태와 값을 가질 수 있다. 객체의 유연한 성질 때문에 커스텀 데이터 타입을 만들 때 객체를 많이 사용한다. 자바스크립트에는 다음과 같이 몇 가지 내장된 객체 타입이 있다.

* Array
* Date
* RegExp
* Map & WeakMap
* Set & WeakSet

그리고 원시 타입 중 숫자와 문자열, 불리언에는 각각 대응하는 객체 타입인 Number, Boolean, String이 있다. 이들은 대응하는 원시 값에 기능을 제공하는 역할을 한다.

## 숫자

자바스크립트에는 숫자형 데이터 타입이 하나밖에 없다. 이는 자바스크립트를 단순한 언어로 만들었다는 장점이 있지만, 고성능 정수 연산이나 정밀한 소수점 연산이 필요한 애플리케이션에서 쓸 수 없게 만든 선택이기도 하다.

숫자에 대응하는 Number 객체에는 중요한 숫자형 값에 해당하는 프로퍼티가 존재한다.

```JavaScript
const small = Number.EPSILON; // 1에 더했을 때 1과 구분되는 결과를 만들 수 있는 가장 작은 값
const bigInt = Number.MAX_SAFE_INTEGER; // 표현할 수 있는 가장 큰 정수
const max = Number.MAX_VALUE; // 표현할 수 있는 가장 큰 숫자
...
```

## 심볼

심볼은 유일한 토큰을 나타내기 위해 `ES6`에서 도입한 새 데이터 타입이다. 심볼은 항상 유일하다. 다른 어떤 심볼과도 일치하지 않는다. 이런면에서 객체와 유사하다.

```javascript
const RED = Symbol("The color of a sunset");
const ORANGE = Symbol("The color of a sunset");

RED === ORANGE // false : 심볼은 모두 서로 다르다.
```

우연히 다른 식별자와 혼동해서는 안 되는 고유한 식별자가 필요하다면 심볼을 사용한다.

## null & undefined

```javascript
let currentTemp;          // undefined
const targetTemp = null;  // null = 아직 모르는 값
currentTemp = 22.5;       // 이제 값이 있음
```

## 객체

객체의 본질은 컨테이너이다. 원시 타입은 단 하나의 값만 나타낼 수 있고 불변이지만, 객체는 여러 가지 값이나 복잡한 값을 나타낼 수 있고, 변할 수도 있다.

컨테이너의 내용물은 지간이 지나면서 바뀔 수 있지만 컨테이너가 바뀌는 것은 아니다. 즉, 같은 객체이다.

`const obj = {};` : 빈 객체

객체의 콘텐츠는 프로퍼티나 멤버라고 부른다. 프로퍼티는 키 값으로 구성된다. 이의 이름은 문자열 또는 심볼이어야 하고, 값은 그 어떤 것이든 상관 없다.

`obj.color = "yellow";` : 빈 객체에 color 프로퍼티 추가

프로퍼티 이름에 유효한 식별자를 쓸 땐 멤버 접근 연산자를 사용할 수 있다. 프로퍼티 이름에 유효한 식별자가 아닌 이름을 쓴다면 계산된 멤버 접근 연산자를 써야 한다.

```javascript
obj["not an identifier"] = 3;
obj["not an identifier"];  // 3 : 유효한 식별자가 아닌 문자열
obj["color"];              // "yellow" : 유효한 식별자 문자열

const SIZE = Symbol();
obj[SIZE] = 8;
obj[SIZE];            // 8 : 심볼
```

위와 같은 세 가지 프로퍼티가 있다. obj는 빈 객체로 만들었지만, 객체 리터럴 문법에서는 객체를 만드는 동시에 프로퍼티를 만들 수 있다.

```JavaScript
const sam1 = {
  name: 'Sam',
  age: 4,
};

const sam2 = {
  name: 'Sam',
  age: 4,
  classification: {
    family: 'Felidae',
  },
};
```

`sam2`에서 `family`에 접근하는 방법은 여러가지이다.

```javascript
sam2.classification.family;
sam2["classification"].family;
sam2["classification"]["family"];
sam2.classification["family"];
```

객체에 함수를 담을 수도 있다. 함수를 추가할 때는 다음과 같이 한다.

```JavaScript
sam2.speak = function() {return "Hello";};
sam2.speak(); // "Hello"
```

객체에 프로퍼티를 제거할 때는 delete 연산자를 사용한다.

```javascript
delete sam2.classification; // classification 트리 전체 삭제
delete sam2.speak;  // speak 함수 삭제
```

## Number, String, Boolean 객체

숫자와 문자, 불리언에는 각각 대응하는 객체 타입이 있다. 이들 객체가 있는 이유는 첫 번째로, `Number.INFINITY` 같은 특별한 값을 저장하는 것이고, 두 번째로 함수 형태로 기능을 제공하는 것이다.

```javascript
const s = "hello";
s.toUpperCase();    // "HELLO"
```

s는 객체처럼 함수 프로퍼티를 가진 걸로 보인다. 하지만 s는 분명 원시 문자열 타입이다. 이는 자바스크립트가 일시적으로 String 객체를 만든 것이다. 이 임시 객체 안에 toUpperCase 함수가 존재한다. 자바스크립트는 이 함수를 호출하는 즉시 임시 객체를 파괴한다.

```javascript
const s = "hello";
s.rating = 3;
s.rating;     // undefined
```

이 예제는 문자열 s에 프로퍼티를 할당하는 것처럼 보이지만, 일시적인 String 객체에 프로퍼티를 할당한 것이므로 즉시 파괴되어 undefined라고 뜨는 것이다.

## 배열

자바스크립트 배열에는 다음과 같은 특징이 있다.

* 배열 크기는 고정되지 않는다. 언제는 요소를 추가/제거 할 수 있다.
* 요소의 데이터 타입을 가리키지 않는다. 즉, 문자열만 쓸 수 있는 배열이라는 개념 자체가 없다.
* 배열 요소는 0으로 시작한다.

```javascript
const arr = ['a', 'b', 'c'];
arr.length; // 3
arr[0];     // 'a'
arr[arr.length - 1];  // 'c'

arr[2] = 'a'; // arr = ['a', 'b', 'a']
```

## 날짜

자바스크립트의 Date 객체는 원래 자바에서 가져온 것이다.

```javascript
// 현재 날짜와 시간을 나타내는 객체를 만들 때
const now = new Date();
now;  // Sun Dec 23 2018 16:20:20 GMT +0900 (KST)

// 특정 날짜에 해당하는 객체를 만들 때
const myBirth = new Date(1995, 03, 21);
```

## 맵과 셋

`ES6`에서 새로운 데이터 타입인 Map, Set을 도입하였다. 맵은 객체와 마찬가지로 키와 값을 연결하지만, 특정 상황에서 객체보다 유리한 부분이 있다. 셋은 배열과 비슷하지만 중복이 허용되지 않는다. 위크맵과 위크셋은 특정 상황에서 성능이 더 높아지도록 일부 기능을 제거한 버전이다.

## 데이터 타입 변환

```javascript
// str -> num
const numStr = "33.3";
const num = Number(numStr);

// " volts"는 무시된다. 10진수 16
const a = parseInt("16 volts", 10);

// 16 진수 3a를 10진수로 바꾼다. -> 58
const b = parseInt("3a", 16);

// " kph" 무시된다. parseFloat는 항상 기수가 10이라고 가정한다.
const c = parseFloat("15.5 kph");

// 현재 날짜
const d = new Date();
const ts = d.valueOf(); // UTC 1970년 1월 1일부터 몇 밀리초가 지났는지 숫자

// 불리언 값을 1이나 0으로 바꿔야할 때
const b = true;
const n = b ? 1 : 0;

// num -> str
const n = 33.5;
const s = n.toString();

const arr = [1, true, "hello"];
arr.toString(); // "1,true,hello"

// boolean으로 변환
const n = 0;            // 거짓 같은 값
const b1 = !!n;         // false
const b2 = Boolean(n);  // false
```
