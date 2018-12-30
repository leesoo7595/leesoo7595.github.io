---
layout: post
title: "[JavaScript] Destructuring - 디스트럭쳐링"
date: 2018-12-30
tags: [JavaScript]
comments: true
---

참고 : https://poiemaweb.com/es6-destructuring

디스트럭쳐링은 구조화된 배열 또는 객체를 Destructuring(비구조화, 파괴)하여 개별적인 변수에 할당하는 것이다. 배열 또는 객체 리터럴에서 필요한 값만을 추출하여 변수에 할당하거나 반환할 때 쓰인다.

## 배열 디스트럭쳐링

ES5

배열의 각 요소를 디스트럭쳐링하여 변수에 할당하기 위한 방법은 아래와 같다.

```javascript
// ES5
var arr = [1, 2, 3];

var one   = arr[0];
var two   = arr[1];
var three = arr[2];

console.log(one, two, three); // 1 2 3
```

ES6

ES6의 배열 디스트럭쳐링은 배열의 각 요소를 배열로부터 추출하여 변수 리스트에 할당한다. 이때 추출/할당 기준은 인덱스이다.

```javascript
// ES6 Destructuring
const arr = [1, 2, 3];

// 배열의 인덱스를 기준으로 배열로부터 요소를 추출하여 변수에 할당
const [one, two, three] = arr;
const [one, two, three] = [1, 2, 3];


console.log(one, two, three); // 1 2 3
```

ES6의 배열 디스트럭처링은 배열에서 필요한 요소만 추출하여 변수에 할당하고 싶은 경우에 유용하다.

```javascript
const today = new Date();
const formattedDate = today.toISOString().substring(0, 10);
const [year, month, day] = formattedDate.split('-');
console.log([year, month, day]); // [ '2018', '05', '05' ]
```

## 객체 디스트럭처링 (Object destructuring)

ES5에서 객체의 각 프로퍼티를 객체로부터 디스트럭처링하여 변수에 할당하기 위해서는 프로퍼티 이름을 사용해야한다.

```javascript
// ES5
var obj = { firstName: 'Leesoo', lastName: 'Tsun' };

var firstName = obj.firstName;
var lastName  = obj.lastName;

console.log(firstName, lastName); // leesoo Tsun
```

ES6의 객체 디스트럭처링은 객체의 각 프로퍼티를 객체로부터 추출하여 변수 리스트에 할당한다. 이때 할당 기준은 **프로퍼티 이름** 이다.

```javascript
// ES6 Destructuring
const obj = { firstName: 'Leesoo', lastName: 'Tsun' };

const { firstName, lastName } = obj;

console.log(firstName, lastName); // Leesoo Tsun
```

객체 디스트럭처링을 위해서는 할당 연산자 왼쪽에 객체 형태의 변수 리스트가 필요하다.

```javascript
const { prop1: p1, prop2: p2 } = { prop1: 'a', prop2: 'b' };
console.log(p1, p2); // 'a' 'b'
console.log({ prop1: p1, prop2: p2 }); // { prop1: 'a', prop2: 'b' }

// 아래는 위의 축약형이다
const { prop1, prop2 } = { prop1: 'a', prop2: 'b' };
console.log({ prop1, prop2 }); // { prop1: 'a', prop2: 'b' }

// default value
const { prop1, prop2, prop3 = 'c' } = { prop1: 'a', prop2: 'b' };
console.log({ prop1, prop2, prop3 }); // { prop1: 'a', prop2: 'b', prop3: 'c' }
```
