---
layout: post
title: "[ES6] Extended Parameter Handling"
date: 2020-01-12
tags: [JavaScript]
comments: true
---

# 매개변수 기본값

함수를 호출할 때, 매개변수에 적절한 인수가 전달되었는지 함수 내부에서 확인을 하지 않으면, 에러가 발생하기 쉽다.

그래서 ES6 이전까지는 함수 내부에서 매개변수 인수를 체크 및 기본값 부여 로직이 들어가 있어야 했다. ES6부터는 매개변수 기본값을 사용할 수 있게 되어 함수 호출시 매개변수에 인수 전달이 되지않았을 때의 체크를 할 수 있게 되었다.

```javascript
function sum(x = 0, y = 0) {
  return x + y;
}

console.log(sum(1));    // 1
console.log(sum(1, 2)); // 3

function foo(x, y = 0) {
  console.log(arguments);
}

console.log(foo.length); // 1

sum(1);    // Arguments { '0': 1 }
sum(1, 2); // Arguments { '0': 1, '1': 2 }
```

매개변수 기본값은 함수 정의 시 선언한 매개변수 개수를 나타내는 함수 객체의 length 프로퍼티, arguments 객체에 영향을 주지 않는다.

# Rest 파라미터

Rest 파라미터는 나머지 매개변수라는 뜻이다. 매개변수 이름 앞에 `...`을 붙여서 정의한 매개변수를 의미한다.

Rest 파라미터는 해당 매개변수들의 리스트를 배열로 전달 받는다.

```javascript
function foo(...rest) {
  console.log(Array.isArray(rest)); // true
  console.log(rest); // [ 1, 2, 3, 4, 5 ]
}

foo(1, 2, 3, 4, 5);
```

Rest 파라미터는 먼저 선언된 인수를 제외한 나머지 인수들이 배열로 담겨서 Rest 파라미터에 할당된다. 그래서 **Rest 파라미터는 마지막 파라미터여야 한다.**

Rest 파라미터는 함수 정의 시 선언한 매개변수 개수를 의미하는 length 프로퍼티에 영향을 미치지 않는다.

```javascript
function foo(...rest) {}
console.log(foo.length); // 0

function bar(x, ...rest) {}
console.log(bar.length); // 1
```

## arguments와 rest

ES5에서는 매개변수 개수가 가변적인 함수의 경우, arguments 객체를 통해 인수를 확인한다. arguments 객체는 함수 호출시 전달되는 인수들의 정보를 담고 있는 이터러블한 유사 배열 객체이다. 함수 내부에서 한해 지역 변수처럼 사용 가능하다.

ES5에서는 가변 인자 함수에서 파라미터를 통해 인수를 전달받는 것이 불가능하다. 그래서 arguments 객체를 활용하여 인수를 전달받는데, arguments가 유사배열 객체라서 배열 메소드를 사용하려면 Array에 this값을 바인딩해줘야한다.

하지만 ES6에서는 rest 파라미터를 사용하여 곧바로 배열로 전달 받을 수 있다.

```javascript
// ES6
function sum(...args) {
  console.log(arguments); // Arguments(5) [1, 2, 3, 4, 5, callee: (...), Symbol(Symbol.iterator): ƒ]
  console.log(Array.isArray(args)); // true
  return args.reduce((pre, cur) => pre + cur);
}
console.log(sum(1, 2, 3, 4, 5)); // 15
```

여기서 ES6의 화살표 함수는 arguments 프로퍼티가 없다. 그래서 화살표 함수로 매개변수가 가변적인 함수를 만들 경우, rest 파라미터를 꼭 사용해야 한다.

# Spread

Spread 연산자는 대상을 개별 요소로 분리한다. 여기서 Spread 연산자를 사용하는 대상은 이터러블이어야 한다.

```javascript
console.log(...'Hello'); // H e l l o
```

## 함수 인수

배열을 분해하여 각 요소들을 함수의 파라미터에 전달하고 싶은 경우, ES5 때엔 apply 메소드로 this 바인딩을 하여 사용해야 한다.

그런데 ES6의 Spread 연산자를 사용한 배열을 인수로 그냥 전달하면 Spread 연산자가 배열을 분해하여 각 요소를 파라미터에 자동으로 할당한다.

```javascript
// ES5
function foo(x, y, z) {
  console.log(x); // 1
  console.log(y); // 2
  console.log(z); // 3
}

// 배열을 분해하여 배열의 각 요소를 파라미터에 전달하려고 한다.
const arr = [1, 2, 3];

// apply 함수의 2번째 인수(배열)는 분해되어 함수 foo의 파라이터에 전달된다.
foo.apply(null, arr);
// foo.call(null, 1, 2, 3);

// ES6
// 배열을 foo 함수의 인자로 전달하려고 한다.
const arr = [1, 2, 3];

/* ...[1, 2, 3]는 [1, 2, 3]을 개별 요소로 분리한다(→ 1, 2, 3)
   spread 문법에 의해 분리된 배열의 요소는 개별적인 인자로서 각각의 매개변수에 전달된다. */
foo(...arr);
```

Rest 파라미터는, Spread 연산자를 사용하여 파라미터를 정의한 것을 의미한다. 

Rest는 반드시 함수의 마지막 파라미터에 들어가야하지만, Spread 연산자를 사용한 인수(배열)은 파라미터의 위치와 상관없이 사용할 수 있다.

## 배열

Spread 연산자를 사용한 배열의 경우, spread 연산자를 사용한 대상(배열)이 각 요소로 분해되어 들어가므로 배열 메소드에 spread 연산자를 사용할 경우 하나의 잘 정리된 리스트로 정리가 된다.

ES5 때에는 concat 메소드를 주로 사용해야했지만, ES6부터는 spread 연산자를 활용하여 간단히 배열 요소의 일부로 합쳐 사용할 수 있게 되었다.

```javascript
// ES6
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

// ...arr2는 [4, 5, 6]을 개별 요소로 분리한다
arr1.push(...arr2); // == arr1.push(4, 5, 6);

console.log(arr1); // [ 1, 2, 3, 4, 5, 6 ]

const arr = [1, 2, 3];
// ...arr은 [1, 2, 3]을 개별 요소로 분리한다
console.log([...arr, 4, 5, 6]); // [ 1, 2, 3, 4, 5, 6 ]
```

기존 배열을 복사하는 경우, ES5에서는 slice 메소드를 사용했었다. 그런데 spread 연산자를 사용하는 경우 배열을 더욱 간단하게 복사할 수 있다.

다만, slice 메소드나 spread 연산자를 사용하여 복사하면, 이는 원본 배열의 각 요소를 얕은 복사를 하여 새로운 복사본을 생성한다.

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