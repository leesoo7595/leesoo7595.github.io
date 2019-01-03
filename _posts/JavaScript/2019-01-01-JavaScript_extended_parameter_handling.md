---
layout: post
title: "[JavaScript] Extended Parameter Handling"
date: 2019-01-01
tags: [JavaScript]
comments: true
---

참고 : https://poiemaweb.com/es6-extended-parameter-handling

## 파라미터 기본값

```JavaScript
// ES5
function plus(x, y) {
  x = x || 0; // 매개변수 x에 인수를 할당하지 않은 경우, 기본값 0을 할당한다.
  y = y || 0; // 매개변수 y에 인수를 할당하지 않은 경우, 기본값 0을 할당한다.

  return x + y;
}

console.log(plus());     // 0
console.log(plus(1, 2)); // 3
```

ES5에서는 파라미터에 기본값을 설정할 수 없다. 그래서 적절한 인수가 전달되었는지 확인하여야한다.

ES6에서는 파라미터에 기본값을 설정하여 함수 내에서 수행하던 파라미터 체크 및 초기화를 간소화할 수 있다.

```javascript
// ES6
function plus(x = 0, y = 0) {
  // 파라미터 x, y에 인수를 할당하지 않은 경우, 기본값 0을 할당한다.
  return x + y;
}

console.log(plus());     // 0
console.log(plus(1, 2)); // 3
```

## REST 파라미터

Rest 파라미터란 Spread 연산자 `...`를 사용하여 파라미터를 정의한 것을 의미한다. 인수의 리스트를 함수 내부에서 배열로 전달받을 수 있다. 단, Rest 파라미터는 반드시 마지막 파라미터이어야 한다.

```javascript
function foo(...rest) {
  console.log(Array.isArray(rest)); // true
  console.log(rest); // [ 1, 2, 3, 4, 5 ]
}

function bar(param1, param2, ...rest) {
  console.log(param1); // 1
  console.log(param2); // 2
  console.log(rest);   // [ 3, 4, 5 ]
}

foo(1, 2, 3, 4, 5);
bar(1, 2, 3, 4, 5);
```

### arguments

ES5에서는 가변 인자 함수의 경우, arguments 객체를 통해 인수를 확인한다. arguments 객체는 함수 호출 시 전달된 인수(argument)들의 정보를 담고 있는 순회가능한 유사 배열 객체이다.arguments로 배열 메소드를 사용하려면 `Function.prototype.call`을 사용해야 한다.

```javascript
// ES5
var foo = function () {
  console.log(arguments);
};

foo(1, 2); // { '0': 1, '1': 2 }

function sum() {
  /*
  가변 인자 함수는 arguments 객체를 통해 인수를 전달받는다.
  유사 배열 객체인 arguments 객체를 배열로 변환한다.
  */
  var array = Array.prototype.slice.call(arguments);
  return array.reduce(function (pre, cur) {
    return pre + cur;
  });
}

console.log(sum(1, 2, 3, 4, 5)); // 15
```

ES6에서는 rest 파라미터를 사용하여 배열로 전달할 수 있다. 그러면 유사 배열인 arguments 객체를 배열로 변환할 필요가 없다.

화살표 함수에는 함수 객체의 arguments 프로퍼티가 없다. 그래서 화살표 함수를 통해 arguments 배열을 전달하려면 rest 파라미터를 꼭 써야한다.

```javascript
// ES6
function sum(...args) {
  console.log(arguments); // Arguments(5)&nbsp;[1, 2, 3, 4, 5, callee: (...), Symbol(Symbol.iterator): ƒ]
  console.log(Array.isArray(args)); // true
  return args.reduce((pre, cur) => pre + cur);
}
console.log(sum(1, 2, 3, 4, 5)); // 15

// 화살표 함수엔 arguments 프로퍼티가 없다.
var normalFunc = function () {};
console.log(normalFunc.hasOwnProperty('arguments')); // true

const arrowFunc = () => {};
console.log(arrowFunc.hasOwnProperty('arguments')); // false
```

## Spread 연산자

Spread 연산자인 `...`는 연산자의 대상 배열이나 Iterable을 개별 요소로 분리한다.

```javascript
// ...[1, 2, 3]는 [1, 2, 3]을 개별 요소로 분리한다(→ 1, 2, 3)
console.log(...[1, 2, 3]) // 1, 2, 3

// 문자열은 이터러블이다.
console.log(...'Hello');  // H e l l o

// Map과 Set은 이터러블이다.
console.log(...new Map([['a', '1'], ['b', '2']]));  // [ 'a', '1' ] [ 'b', '2' ]
console.log(...new Set([1, 2, 3]));  // 1 2 3
```

ES5에서는 배열의 각 요소를 파라미터로 전달하거나, 배열을 함수의 인자로 사용하는 경우 `Function.prototype.apply`를 사용한다.

ES6에서는 spread 연산자를 사용하여 배열의 각 요소를 개별적으로 전달한다.

```javascript
// ES5
function foo(x, y, z) {
  console.log(x); // 1
  console.log(y); // 2
  console.log(z); // 3
}

// 배열을 foo 함수의 인자로 전달하려고 한다.
const arr = [1, 2, 3];

// apply 함수의 2번째 인자(배열)는 호출하려는 함수(foo)의 개별 인자로 전달된다.
foo.apply(null, arr);
// foo.call(null, 1, 2, 3);

// ES6
/* ...[1, 2, 3]는 [1, 2, 3]을 개별 요소로 분리한다(→ 1, 2, 3)
   spread 연산자에 의해 분리된 배열의 요소는 개별적인 인자로서 각각의 매개변수에 전달된다. */
foo(...arr);
```

spread 연산자를 사용하여 파라미터를 정의한 것과, 배열 인수가 분리되어 순차적으로 매개변수에 할당되는 것은 다른 역할이다. 주의주의!!

```javascript
/* Spread 연산자를 사용한 매개변수 정의 (= Rest 파라미터)
   ...rest는 분리된 요소들을 함수 내부에 배열로 전달한다. */
function foo(param, ...rest) {
  console.log(param); // 1
  console.log(rest);  // [ 2, 3 ]
}
foo(1, 2, 3);

/* Spread 연산자를 사용한 인수
  배열 인수는 분리되어 순차적으로 매개변수에 할당 */
function bar(x, y, z) {
  console.log(x); // 1
  console.log(y); // 2
  console.log(z); // 3
}

// ...[1, 2, 3]는 [1, 2, 3]을 개별 요소로 분리한다(-> 1, 2, 3)
// spread 연산자에 의해 분리된 배열의 요소는 개별적인 인자로서 각각의 매개변수에 전달된다.
bar(...[1, 2, 3]);
```

## Spread 연산자를 배열에서 사용하기

### concat

```javascript
// ES5
var arr = [1, 2, 3];
console.log(arr.concat([4, 5, 6])); // [ 1, 2, 3, 4, 5, 6 ]

// ES6
const arr = [1, 2, 3];
// ...arr은 [1, 2, 3]을 개별 요소로 분리한다
console.log([...arr, 4, 5, 6]); // [ 1, 2, 3, 4, 5, 6 ]
```

ES5에서 기존 배열의 요소를 새로운 배열의 요소로 만들고 싶을 때, concat 메소드를 사용한다. 이 또한 ES6에서 Spread 연산자를 사용하면 기존 배열의 요소를 새로운 배열의 요소의 일부로 만들 수 있다.

### push

```javascript
// ES5
var arr1 = [1, 2, 3];
var arr2 = [4, 5, 6];

// apply 메소드의 2번째 인자는 배열. 이것은 개별 인자로 push 메소드에 전달된다.
Array.prototype.push.apply(arr1, arr2);

console.log(arr1); // [ 1, 2, 3, 4, 5, 6 ]

// ES6
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

// ...arr2는 [4, 5, 6]을 개별 요소로 분리한다
arr1.push(...arr2); // == arr1.push(4, 5, 6);

console.log(arr1); // [ 1, 2, 3, 4, 5, 6 ]
```

### splice

```javascript
// ES5
var arr1 = [1, 2, 3, 6];
var arr2 = [4, 5];

/*
apply 메소드의 2번째 인자는 배열. 이것은 개별 인자로 push 메소드에 전달된다.
[3, 0].concat(arr2) → [3, 0, 4, 5]
arr1.splice(3, 0, 4, 5) → arr1[3]부터 0개의 요소를 제거하고 그자리(arr1[3])에 새로운 요소(4, 5)를 추가한다.
*/
Array.prototype.splice.apply(arr1, [3, 0].concat(arr2));

console.log(arr1); // [ 1, 2, 3, 4, 5, 6 ]

// ES6
const arr1 = [1, 2, 3, 6];
const arr2 = [4, 5];

// ...arr2는 [4, 5]을 개별 요소로 분리한다
arr1.splice(3, 0, ...arr2); // == arr1.splice(3, 0, 4, 5);

console.log(arr1); // [ 1, 2, 3, 4, 5, 6 ]
```

### copy

ES5에서 기존 배열을 복사하기 위해서는 slice 메소드를 사용한다. ES6에서는 Spread 연산자를 사용하면 더욱 간편하게 배열을 복사할 수 있다.

```javascript
// ES5
var arr  = [1, 2, 3];
var copy = arr.slice();

console.log(copy); // [ 1, 2, 3 ]

// copy를 변경한다.
copy.push(4);

console.log(copy); // [ 1, 2, 3, 4 ]
// arr은 변경되지 않는다.
console.log(arr);  // [ 1, 2, 3 ]

// ES6
const arr = [1, 2, 3];
// ...arr은 [1, 2, 3]을 개별 요소로 분리한다
const copy = [...arr];

console.log(copy); // [ 1, 2, 3 ]

// copy를 변경한다.
copy.push(4);

console.log(copy); // [ 1, 2, 3, 4 ]
// arr은 변경되지 않는다.
console.log(arr);  // [ 1, 2, 3 ]
```

### 객체를 사용하는 경우

Spread 연산자를 사용하면 객체 또한 손쉽게 병합 또는 변경할 수 있다. Object.assign 메소드를 사용해도 동일한 작업을 할 수 있다. 또한 유사 배열 객체도 Spread 연산자를 사용하면 배열로 손쉽게 변환할 수 있다.

```javascript
// Spread 연산자 사용하여 객체 변경
// 객체의 병합
const merged = { ...{ x: 1, y: 2 }, ...{ y: 10, z: 3 } };
console.log(merged); // { x: 1, y: 10, z: 3 }

// 특정 프로퍼티 변경
const changed = { ...{ x: 1, y: 2 }, y: 100 };
// changed = { ...{ x: 1, y: 2 }, ...{ y: 100 } }
console.log(changed); // { x: 1, y: 100 }

// 프로퍼티 추가
const added = { ...{ x: 1, y: 2 }, z: 0 };
// added = { ...{ x: 1, y: 2 }, ...{ z: 0 } }
console.log(added); // { x: 1, y: 2, z: 0 }

// Object.assign 메소드 사용하여 객체 변경
// 객체의 병합
const merged = Object.assign({}, { x: 1, y: 2 }, { y: 10, z: 3 });
console.log(merged); // { x: 1, y: 10, z: 3 }

// 특정 프로퍼티 변경
const changed = Object.assign({}, { x: 1, y: 2 }, { y: 100 });
console.log(changed); // { x: 1, y: 100 }

// 프로퍼티 추가
const added = Object.assign({}, { x: 1, y: 2 }, { z: 0 });
console.log(added); // { x: 1, y: 2, z: 0 }

// 유사 배열 객체를 배열로 변환하기
const htmlCollection = document.getElementsByTagName('li');

// 유사 배열인 HTMLCollection을 배열로 변환한다.
const newArray = [...htmlCollection]; // Spread 연산자

// ES6의 Array.from 메소드를 사용할 수도 있다.
// const newArray = Array.from(htmlCollection);
```
