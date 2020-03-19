---
layout: post
title: "ES5 - Array method"
date: 2020-03-19
tags: [Class]
categories:
- Class
comments: true
---

# Array

배열은 다들 알다시피 1개의 변수를 선언해서 여러 개의 값을 저장할 수 있게 해주는 아이이다. 자바스크립트에서 배열은 Array 타입의 객체이고, 이 객체가 상속하고 있는 프로토타입은 `Array.prototype`이다.

여기서 Array도 객체로 Object인데, 일반 Object와의 차이가 궁금할 수 있다. 일반 Object가 상속하고 있는 프로토타입 (`__proto__`)는 `Object.prototype`이지만, array 객체의 `__proto__`는 `Array.prototype`이고, `Array.prototype`의 `__proto__`가 `Object.prototype`이 된다. 그래서 array 객체는 array 메소드를 `Array.prototype`을 통해 사용할 수 있게 되는 것이다.

```javascript
var arr = [];
var obj = {};

console.dir(arr.__proto__);
console.dir(obj.__proto__);
```

<img width="668" alt="Screen Shot 2020-03-19 at 10 18 54 AM" src="https://user-images.githubusercontent.com/39291812/77124609-b00c9d00-6a86-11ea-88a5-4a02a39b88a4.png">

위 이미지는 일반 object와 array의 `__proto__`를 콘솔로 찍어 비교해본 이미지이다.

또한 자바스크립트는 동적 언어이기 때문에, 다른 보통의 프로그래밍 언어와 달리 자바스크립트의 배열 안의 요소로 여러 데이터 타입의 값이 들어갈 수 있다.

## Array 요소 추가/삭제

```javascript
var arr = [];

// array 요소 추가
arr[1] = 0;
arr[2] = 2;

console.log(arr);

// array 요소의 값만 삭제
delete arr[1];
console.log(arr);

// array 요소 자체를 삭제
// splice(시작 인덱스, 삭제할 요소수)
arr.splice(1, 1);
console.log(arr);
```

## Array 순회

### for...in & for...of

* 객체의 프로퍼티 키를 순회하는 것이 `for...in`!
* 객체의 프로퍼티 값을 순회하는 것이 `for...of`!

Array는 딱 보기엔 배열 내부에 값만 가지고 있는 것 같지만, Array도 결국 객체이므로 프로퍼티 키 값을 가진다.

```javascript
var arr = [0, 1, 2, 3];
arr.foo = 10;

for (var key in arr) {
  console.log('key: ' + key, 'value: ' + arr[key]);
}
// key: 0 value: 0
// key: 1 value: 1
// key: 2 value: 2
// key: 3 value: 3
// key: foo value: 10 => 불필요한 프로퍼티까지 출력
```

위의 코드를 보면, Array에 key 값 또한 가지고 있는 것을 볼 수 있다. 이는 Array도 객체이기 때문에 배열이라는 이유로 겉으로는 보이지 않는 인덱스 키 값을 가지고 있는 것이다. 그래서 배열을 객체처럼 프로퍼티에 값을 집어넣는 것이 가능하다.

그래서 배열의 값을 순회할 경우엔 객체의 value 값을 순회하는 `for...of`나, `forEach` 메소드를 써야한다.

```javascript
var arr = [0, 1, 2, 3]

arr.forEach((item, index) => console.log(index, item));

for (var i = 0; i < arr.length; i++) {
  console.log(i, arr[i]);
}

for (const item of arr) {
  console.log(item);
}
```

## Array method

Array 메소드로는 크게 두 가지로 나뉘어진다.

* 원본 배열(`this`)을 변경한다.
* 원본 배열(`this`)을 변경하지 않는다.

그 외 메소드로는 이터러블한 객체나 특정 요소를 배열로 만들어주거나, 배열인지를 체크하는 메소드가 있다.

### isArray

해당 객체가 배열인지 `boolean`으로 체크한다.

```javascript
Array.isArray([]);
Array.isArray({});
```

### from - ES6

이터러블을 배열로 반환해준다. 이터러블이란, 단순히 말해서 순회가 가능한 객체를 말한다. 해당 개념은 ES6에서 나온 개념이다. 순회 가능한 객체란, `[Symbol.iterator]` 프로퍼티를 가지고 있는 아이이다. 다음 아이들은 빌트인 이터러블이다. 

```
Array, String, Map, Set, TypedArray(Int8Array, Uint8Array, Uint8ClampedArray, Int16Array, Uint16Array, Int32Array, Uint32Array, Float32Array, Float64Array), DOM data structure(NodeList, HTMLCollection), Arguments
```

```javascript
console.log(Array.from('hello'))
```

### of - ES6

주어진 인수들로 새로운 배열을 생성하여 반환한다.

```javascript
Array.of(1, 2);
```

### 원본 배열을 건드리지 않는 Array 메소드

* Array.prototype.indexOf

`indexOf` 메소드는 인자를 두 가지를 받을 수 있는데, 첫 번째 인자로 받은 요소를 배열에서 검색하여 해당 요소의 첫 번째로 검색되는 인덱스를 반환한다. 해당하는 요소가 없는 경우, -1을 반환한다. 두 번째 인자로는 배열 내부에서 첫 번째로 받는 요소가 중복되어 여러 번 나올 경우, 몇 번째에 나오는 요소의 인덱스를 반환할 것인지를 받는다.

```javascript
var arr = [1, 2, 2, 3];
console.log(arr.indexOf(2));    // 1
console.log(arr.indexOf(4));    // -1
console.log(arr.indexOf(2, 2)); // 2

// arr 배열에 3 요소가 존재하는지 확인
console.log(arr.indexOf(3) !== -1)  // true -> 3 요소 존재
```

* Array.prototype.concat


`concat` 메소드는 파라미터로 넘어오는 값들(배열이나 값)을 기존 배열의 복사본에 요소를 추가하고 복사본을 반환한다. 그래서 원본 배열은 변경되지 않는다.

```javascript
var a = ['a', 'b', 'c'];
var b = ['x', 'y', 'z'];

var c = a.concat(b);
console.log(c); // ['a', 'b', 'c', 'x', 'y', 'z']
```

* Array.prototype.join

`join` 메소드는 파라미터로 받는 값으로 배열 전체를 연결하여 문자열로 반환한다. 인자를 넣어주지않으면, 기본값으로 `,`가 들어간다.

```javascript
var arr = ['a', 'b', 'c', 'd'];

var x = arr.join();
console.log(x);  // 'a,b,c,d';

var y = arr.join('');
console.log(y);  // 'abcd'

var z = arr.join(':');
console.log(z);  // 'a:b:c:d'
```

* Array.prototype.slice

`slice` 메소드의 파라미터로 들어온 배열의 부분을 복사하여 해당 복사본을 반환한다.

```javascript
const items = ['a', 'b', 'c'];

// items[0]부터 items[1] 이전(items[1] 미포함)까지 반환
let res = items.slice(0, 1);
console.log(res);  // [ 'a' ]

// items[1]부터 이후의 모든 요소 반환
res = items.slice(1);
console.log(res);  // [ 'b', 'c' ]

// 인자가 음수인 경우 배열의 끝에서 요소를 반환
res = items.slice(-1);
console.log(res);  // [ 'c' ]

// 모든 요소를 반환 (= 복사본(shallow copy) 생성)
res = items.slice();
console.log(res);  // [ 'a', 'b', 'c' ]

```

* Array.prototype.forEach

`forEach` 메소드는 for문 대신 사용할 수 있다. 배열을 순회하며 배열의 각 요소에 대하여 인자로 받아오는 콜백함수를 실행한다. **반환값은 undefined이다.**

```javascript
// forEach 메소드로 순회
const numbers = [1, 2, 3];
let pows = [];

numbers.forEach(function (item) {
  pows.push(item ** 2);
});
```

* Array.prototype.map

`map` 메소드는 `forEach`와 마찬가지로 배열을 순회하는대신, 해당 반환값은 새로운 배열을 생성하여 그 배열을 반환한다. 원본 배열은 변경되지 않는다.

```javascript
const numbers = [1, 4, 9];

// 배열을 순회하며 각 요소에 대하여 인자로 주어진 콜백함수를 실행
const roots = numbers.map(function (item) {
  // 반환값이 새로운 배열의 요소가 된다. 반환값이 없으면 새로운 배열은 비어 있다.
  return Math.sqrt(item);
});

// 위 코드의 축약표현은 아래와 같다.
// const roots = numbers.map(Math.sqrt);

console.log(roots);   // [ 1, 2, 3 ]
console.log(numbers); // [ 1, 4, 9 ]
```

* Array.prototype.filter

`filter` 메소드를 사용하면, 메소드 인자로 콜백함수를 받는데, 콜백함수의 반환값이 true인 배열 요소 값만을 추출하여 새로운 배열을 반환한다. 즉, 특정 조건을 충족하는 요소만 다시 배열로 추출할 수 있도록 해주는 기능을 한다.

```javascript
const result = [1, 2, 3, 4, 5, 6];

result.filter(ele => ele % 2);  // ele가 홀수인 것 만을 필터링 한다.
```

* Array.prototype.reduce

`reduce` 메소드는 배열을 순회하면서 각 요소에 대하여 이전의 콜백함수 실행 반환값을 전달하여 같은 콜백함수를 실행하고 그 결과를 반환한다. 간단히 말하면, 누적함수 실행하여 최종값을 내게끔 기능을 하는 함수이다.

```javascript
const arr = [1, 2, 3, 4, 5, 6];

const sum = arr.reduce((previousValue, currentValue, currentIndex, self) => {
  return previousValue + currentValue;  // 반환값은 다음 reduce의 콜백함수가 실행될 때 첫 번째 인자로 전달된다.
});

console.log(sum);

// sum이 실행되는 과정

// 1: 1+2 = 3
// 2: 3+3 = 6
// 3: 6+4 = 10
// 4: 10+5 = 15
// 5: 15+6 = 21
```

<img width="527" alt="Screen Shot 2020-03-20 at 8 21 36 AM" src="https://user-images.githubusercontent.com/39291812/77123570-d5e47280-6a83-11ea-8169-c9877ff4b5da.png">

* Array.prototype.some

배열 내 일부 요소를 루핑하면서 `some` 메소드에서 인자로 받는 콜백함수를 실행하여 그 결과 중 하나라도 콜백함수의 반환값이 true이면 true 값, 모두 false면 false으로 반환한다.

```javascript
// 배열 내 요소 중 10보다 큰 값이 1개 이상 존재하는지 확인
let res = [2, 5, 8, 1, 4].some(function (item) {
  return item > 10;
});
console.log(res); // false
```

* Array.prototype.every

배열 내 모든 요소를 루핑하면서 `every` 메소드에서 인자로 받는 콜백함수의 반환값이 모두 true일 경우 true 값을 반환하고, 하나라도 false를 뱉을 경우 false로 반환한다.

```javascript
// 배열 내 모든 요소가 10보다 큰 값인지 확인
let res = [21, 15, 89, 1, 44].every(function (item) {
  return item > 10;
});
console.log(res); // false
```

* Array.prototype.find

`find` 메소드는 배열의 각 요소를 루핑하면서 인자로 받는 콜백함수를 실행하여 그 반환값이 true인 첫 번째 요소를 반환한다. 콜백함수의 실행결과가 true인 값이 존재하지 않으면 `undefined`를 반환한다.

```javascript
const users = [
  { id: 1, name: 'Lee' },
  { id: 2, name: 'Kim' },
  { id: 2, name: 'Choi' },
  { id: 3, name: 'Park' }
];

// 콜백함수를 실행하여 그 결과가 참인 첫번째 요소를 반환한다.
let result = users.find(function (item) {
  return item.id === 2;
});
// Array#find는 배열이 아니라 요소를 반환한다.
console.log(result); // { id: 2, name: 'Kim' }
```

* Array.prototype.findIndex

배열을 루필하면서 각 요소에 대하여 인자로 주어진 콜백함수를 실행하여 그 결과가 true인 첫 번째 요소의 인덱스를 반환한다. 콜백함수의 반환값이 true인 요소가 없으면 -1을 반환한다.

```javascript
const users = [
  { id: 1, name: 'Lee' },
  { id: 2, name: 'Kim' },
  { id: 2, name: 'Choi' },
  { id: 3, name: 'Park' }
];

// 콜백함수를 실행하여 그 결과가 참인 첫번째 요소의 인덱스를 반환한다.
let resultIdx = users.findIndex(function (item) {
  return item.id === 2;
});
// 배열이 아니라 요소의 인덱스를 반환한다.
console.log(resultIdx); // 1
```

### 원본 배열을 바꾸는 Array 메소드

* Array.prototype.pop

배열에서 마지막 요소를 제거하고 **제거한 요소**를 반환한다. 빈 배열의 경우엔 `undefined`를 반환한다. 

```javascript
var a = ['a', 'b', 'c'];
var c = a.pop();

// 원본 배열이 변경된다.
console.log(a); // a --> ['a', 'b']
console.log(c); // c --> 'c'
```

반대로 배열의 첫 요소를 제거하고 제거한 요소를 반환하는 것은 `shift` 메소드를 사용하면 된다.

* Array.prototype.push

`push` 메소드의 파라미터로 받는 값을 기존 배열의 마지막에 추가하고, 해당 변경된 배열의 `length`를 반환한다.

```javascript
var a = ['a', 'b', 'c'];
var b = ['x', 'y', 'z'];

// push는 원본 배열을 직접 변경하고 변경된 배열의 length를 반환한다.
var c = a.push(b);
console.log(a); // a --> ['a', 'b', 'c', ['x', 'y', 'z']]
console.log(c); // c --> 4;

var x = ['a', 'b', 'c'];
var y = ['x', 'y', 'z'];
// concat은 원본 배열을 직접 변경하지 않고 복사본을 반환한다.
console.log(x.concat(y)); // ['a', 'b', 'c', 'x', 'y', 'z']
console.log(x); // ['a', 'b', 'c']
console.log(y); // ['x', 'y', 'z']
```

여기서 배열의 맨 앞에 추가하고 싶으면 `unshift`, 중간에 추가할 경우 `splice` 메소드를 사용할 수 있다.

<img width="795" alt="Screen Shot 2020-03-20 at 12 01 40 AM" src="https://user-images.githubusercontent.com/39291812/77081458-0d303080-6a3e-11ea-866b-2e0598957ea1.png">

* Array.prototype.reverse

배열 요소의 순서를 반대로 변경한다.

```javascript
var a = ['a', 'b', 'c'];
var b = a.reverse();

// 원본 배열이 변경된다
console.log(a); // [ 'c', 'b', 'a' ]
console.log(b); // [ 'c', 'b', 'a' ]
```

* Array.prototype.splice

기존 배열의 요소를 제거하고 그 위치에 새로운 요소를 추가할 때 사용한다.

```javascript
const items1 = [1, 2, 3, 4];

// items[1]부터 2개의 요소를 제거하고 제거된 요소를 배열로 반환
const res1 = items1.splice(1, 2);

// 원본 배열이 변경된다.
console.log(items1); // [ 1, 4 ]
// 제거한 요소가 배열로 반환된다.
console.log(res1);   // [ 2, 3 ]

const items2 = [1, 2, 3, 4];

// items[1]부터 모든 요소를 제거하고 제거된 요소를 배열로 반환
const res2 = items2.splice(1);

// 원본 배열이 변경된다.
console.log(items2); // [ 1 ]
// 제거한 요소가 배열로 반환된다.
console.log(res2);   // [ 2, 3, 4 ]
```

* Array.prototype.sort

배열의 요소를 `sort` 메소드에 함수를 받아서 함수의 조건대로 적절히 정렬한다.

```javascript
const fruits = ['Banana', 'Orange', 'Apple'];

// ascending(오름차순)
fruits.sort();
console.log(fruits); // [ 'Apple', 'Banana', 'Orange' ]

// descending(내림차순)
fruits.reverse();
console.log(fruits); // [ 'Orange', 'Banana', 'Apple' ]

const points = [40, 100, 1, 5, 2, 25, 10];
// 비교 함수의 반환값이 0보다 작은 경우, a를 우선하여 정렬한다.
points.sort(function (a, b) { return a - b; });
// 비교 함수의 반환값이 0보다 큰 경우, b를 우선하여 정렬한다.
points.sort(function (a, b) { return b - a; });
```

## Reference

- [poiemaweb - Array](https://poiemaweb.com/js-array)
- [poiemaweb - HOC](https://poiemaweb.com/js-array-higher-order-function)

