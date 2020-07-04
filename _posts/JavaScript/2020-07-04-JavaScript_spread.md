---
layout: post
title: "Javascript - Spread"
date: 2020-07-04
tags: [JavaScript]
categories:
- Javascript
comments: true
---

## Rest Parameter

자바스크립트에서는 함수에 넘겨주는 인수에 제약이 없다.

그래서 특정 인수들을 제외한 나머지 인수들을 담아서 배열로 넘겨줄 수 있다.

```javascript
function sumAll(...args) { // args는 배열의 이름입니다.
  let sum = 0;

  for (let arg of args) sum += arg;

  return sum;
}

alert( sumAll(1) ); // 1
alert( sumAll(1, 2) ); // 3
alert( sumAll(1, 2, 3) ); // 6
```

주의할 것은, rest 파라미터는 항상 마지막 인자로 넣어주어야한다는 점이다.

## arguments 변수

`arguments`라는 유사 배열 객체가 있다. 이 변수의 인덱스를 사용하여 모든 인수에 접근이 가능하다.

```javascript
function showName() {
  alert(arguments.length)
  alert(arguments[0])
  alert(arguments[1])
}
```

`arguments` 객체는 유사 배열 객체이자 이터러블 객체이다. 즉, 배열이 아니다. 그래서 `arguments`에는 배열 메서드를 사용할 수 없다. 그리고 인수의 일부만 사용하고 싶을때 `arguments`를 사용할 수 없어서 일부 인수만 사용하고자할 땐 `rest` 파라미터를 사용하는 것이 좋다.

**화살표 함수에는 arguments 객체가 없다.** 화살표 함수에서 `arguments` 접근시, 외부의 일반 함수 `arguments` 객체에 접근한다.

## spread

인수를 배열로 가져오는 방법은 rest parameter를 사용하지만, 반대로 배열을 인수로 뿌려주는 경우에는 `spread`를 사용할 수 있다. 

정확히는 특정 메소드를 사용하고자 할 때, 배열을 통째로 넘기면 안되는 경우, **배열이나 이터러블한 객체** 앞에 `spread` 연산자를 사용하면 안에 있던 값들이 흩어져서 들어가게 된다.

```javascript
let arr = [3, 5, 1];

alert( Math.max(...arr) ); // 5 (전개 문법이 배열을 인수 목록으로 바꿔주었습니다.)
```

특징은 이터러블한 객체, 즉 `for ...of`를 사용할 수 있는 객체들은 모두 `spread` 연산자를 사용할 수 있다는 점이다.

```
[...arr] = Array.from(arr)

위 공식은 둘 다 같은 동작을 하지만, spread 연산자는 이터러블 객체에만 사용할 수 있고, Array.from은 유사배열 객체와 이터러블 객체 둘 다 사용할 수 있다.
```

## 실무 팁

spread 연산자를 실무에서 제일 자주 사용하는 경우는 객체를 다른 데에 복사할 때이다.

```javascript
let obj = { a: 1, b: 2, c: 3 };
let objCopy = { ...obj }; // spread the object into a list of parameters
                          // then return the result in a new object

// do the objects have the same contents?
alert(JSON.stringify(obj) === JSON.stringify(objCopy)); // true

// are the objects equal?
alert(obj === objCopy); // false (not same reference)

// modifying our initial object does not modify the copy:
obj.d = 4;
alert(JSON.stringify(obj)); // {"a":1,"b":2,"c":3,"d":4}
alert(JSON.stringify(objCopy)); // {"a":1,"b":2,"c":3}
```