---
layout: post
title: "[JavaScript] immutability"
date: 2018-12-18
tags: [JavaScript]
comments: true
---

참고 : https://poiemaweb.com/js-immutability

# 객체와 변경 불가성

`immutability (변경불가성)`는 객체가 생성된 이후 그 상태를 변경할 수 없는 디자인 패턴을 의미한다. 이는 함수형 프로그래밍의 핵심 원리이다.

객체는 참조 형태로 전달을 주고받는다. 객체가 참조를 통해 공유되어 있다면 그 상태가 언제든지 변경될 수 있기 때문에 문제가 될 가능성도 커지게 된다.

의도하지 않은 객체의 변경이 발생하는 원인의 대다수는 **레퍼런스를 참조한 다른 객체에서 객체를 변경** 하기 때문이다. 이에 대한 해결 방법은 객체를 `불변객체`로 만들어 프로퍼티의 변경을 방지하고, 객체의 변경이 필요한 경우에 참조가 아닌 객체의 `방어적 복사`를 통해 새로운 객체를 생성한 후 변경하거나, `Observer 패턴`으로 객체의 변경에 대처할 수도 있다.

불변객체를 사용하면 복제나 비교를 위한 조작을 단순화 할 수 있고, 성능 개선에도 도움이 된다. ES6에서는 `불변 데이터 패턴(immutable data pattern)`을 쉽게 구현할 수 있는 새로운 기능이 추가되었다.

## immutable value & mutable value

Javascript의 원시 타입 `primitive data type`은 변경 불가능한 값이다.

`Boolean, null, undefined, Number, String, Symbol`

원시 타입 이외의 모든 값은 객체 타입이며, 이는 변경 불가능한 값이다. 즉 새로운 값을 다시 만들 필요없이 직접 변경이 가능하다는 것이다.

C언어와는 다르게 Javascript의 문자열은 변경 불가능한 값이다. 이는 메모리 영역에서 변경이 불가능하다는 뜻으로, 재할당은 가능하다.

```JavaScript
var statement = 'I am an immutable value'; // string은 immutable value

var otherStr = statement.slice(8, 17);

console.log(otherStr);   // 'immutable'
console.log(statement);  // 'I am an immutable value'
```

String 객체의 slice() 메소드는 statement 변수에 저장된 문자열을 변경하는 것이 아니라 새로운 문자열을 생성하여 반환한다. 문자열은 변경할 수 없는 immutable value 이기 때문.

```JavaScript
var arr = [];
console.log(arr.length); // 0

var v2 = arr.push(2);    // arr.push()는 메소드 실행 후 arr의 length를 반환
console.log(arr.length); // 1
```

배열의 메소드인 push()는 `직접 대상 배열을 변경`한다. 배열은 객체이고 객체는 mutable value이기 때문.

```JavaScript
var user1 = {
  name: 'Lee',
  address: {
    city: 'Seoul'
  }
};

var user2 = user1; // 변수 user2는 객체 타입이다.

user2.name = 'Kim';

console.log(user1.name); // Kim
console.log(user2.name); // Kim
```

위의 경우 객체 user2의 name 프로퍼티에 새로운 값을 할당하면 객체는 mutable value이기 때문에 객체 user2가 변경되지만, user1도 동시에 변경된다. 이는 user1과 user2가 같은 어드레스를 참조하고 있기 때문.

## immutable data pattern

의도하지 않은 객체의 변경을 방지하기 위해 객체를 불변객체로 만들어 프로퍼티의 변경을 방지한다. 단, 객체의 변경이 필요할 경우엔 참조가 아닌 객체의 방어적 복사를 통해 새로운 객체를 생성한 후 변경한다.

* 객체의 방어적 복사(defensive copy)
`Object.assign`
* 불변객체화를 통한 객체 변경 방지
`Object.freeze`

### Object.assign

`Object.assign`은 타깃 객체로 소스 객체의 프로퍼티를 복사한다. 이때 소스 객체의 프로퍼티와 동일한 프로퍼티를 가진 타겟 객체의 프로퍼티들은 소스 객체의 프로퍼티로 덮어쓰기 된다. 리턴값으로 타겟 객체를 반환한다.

```JavaScript
// Copy
const obj = { a: 1 };
const copy = Object.assign({}, obj);
console.log(copy); // { a: 1 }
console.log(obj == copy); // false

// Merge
const o1 = { a: 1 };
const o2 = { b: 2 };
const o3 = { c: 3 };

const merge1 = Object.assign(o1, o2, o3);

console.log(merge1); // { a: 1, b: 2, c: 3 }
console.log(o1);     // { a: 1, b: 2, c: 3 }, 타겟 객체가 변경된다!

// Merge
const o4 = { a: 1 };
const o5 = { b: 2 };
const o6 = { c: 3 };

const merge2 = Object.assign({}, o4, o5, o6);

console.log(merge2); // { a: 1, b: 2, c: 3 }
console.log(o4);     // { a: 1 }
```

위처럼 `Object.assign`을 사용하여 기존 객체를 변경하지 않고 객체를 복사하여 사용할 수 있다.

```JavaScript
const user1 = {
  name: 'Lee',
  address: {
    city: 'Seoul'
  }
};

// Shallow copy
const user2 = Object.assign({}, user1); // user1을 {}에 Copy

user2.name = 'Kim';

// 상기 2행은 아래와 동치이다.
// {name: 'Kim'}은 user1에 병합되는 것이 아니라 첫번째 인자인 {}에 병합된다.
// const user2 = Object.assign({}, user1, {name: 'Kim'});

console.log(user1.name); // Lee
console.log(user2.name); // Kim
```

이렇게 user1와 user2는 주소값을 공유하지 않으므로 한 객체를 변경하여도 다른 객체에 아무런 영향을 주지 않는다. 하지만 객체의 프로퍼티는 보호되지 않는다. 즉 객체의 내용은 변경할 수 있다.

### Object.freeze

`Object.freeze`를 사용하여 불변 객체로 만들 수 있다.

```JavaScript
const user1 = {
  name: 'Lee',
  address: {
    city: 'Seoul'
  }
};

// Shallow copy
const user2 = Object.assign({}, user1, {name: 'Kim'});

console.log(user1.name); // Lee
console.log(user2.name); // Kim

Object.freeze(user1);

user1.name = 'Kim'; // 무시된다!

console.log(user1); // { name: 'Lee', address: { city: 'Seoul' } }

console.log(Object.isFrozen(user1)); // true
```

하지만 객체 내부의 객체는 변경 가능하다.
내부 객체까지 변경 불가능하게 하려면 Deep freeze를 하여야한다.

```JavaScript
function deepFreeze(obj) {
  const props = Object.getOwnPropertyNames(obj);

  props.forEach((name) => {
    const prop = obj[name];
    if(typeof prop === 'object' && prop !== null) {
      deepFreeze(prop);
    }
  });
  return Object.freeze(obj);
}

const user = {
  name: 'Lee',
  address: {
    city: 'Seoul'
  }
};

deepFreeze(user);

user.name = 'Kim';           // 무시된다
user.address.city = 'Busan'; // 무시된다

console.log(user); // { name: 'Lee', address: { city: 'Seoul' } }
```

## Immutable.js

`Object.assign`, `Object.freeze` 를 사용하여 불변객체를 만드는 방법은 성능상 이슈가 있어 큰 객체에는 사용하지 않는 것이 좋다.

그래서 Facebook이 제공하는 Immutable.js가 있다.

이는 `List, Stack, Map, OrderedMap, Set, OrderedSet, Record`와 같은 영구 불변 데이터 구조를 제공한다.

사용방법

`npm install immutable`


```JavaScript
const { Map } = require('immutable')
const map1 = Map({ a: 1, b: 2, c: 3 })
const map2 = map1.set('b', 50)
map1.get('b') // 2
map2.get('b') // 50
```

map1.set('b', 50)의 실행에도 불구하고 map1이 불변하였다. -> map1.set()은 결과를 반영한 새로운 객체를 반환한다.
