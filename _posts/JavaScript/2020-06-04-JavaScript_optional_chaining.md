---
layout: post
title: "Javascript - Optional Chaining"
date: 2020-06-04
tags: [JavaScript]
categories:
- Javascript
comments: true
---

## 옵셔널 체이닝

`?.`이라는 옵셔널 체이닝을 사용하면, 해당 피연산자의 프로퍼티가 없는 중첩 객체에도 에러 없이 접근이 가능하다.

```javascript
let user = {}; // 주소정보가 없는 사용자

alert(user.address.street); // TypeError: Cannot read property 'street' of undefined
```
위와 같은 user라는 빈 객체에 없는 프로퍼티를 접근하면 자바스크립트 엔진은 에러를 뿜는다.

이러한 문제를 해결하려면 `&&` 연산자를 사용하여 일일이 해당 객체의 프로퍼티가 존재하는지 여부를 체크하였다. 하지만 이와 같이 사용할 경우, 쓸데없이 코드가 매우 길어진다.

```javascript
let user = {}; // 주소정보가 없는 사용자

alert( user && user.address && user.address.street ); // undefined, 에러가 발생하지 않습니다.
```

이를 해결하기 위해 Optional Chaining이라는 연산자가 등장하였다.

`?.`는 이것을 사용하는 앞의 대상이 `undefined`, `null`일 경우 평가를 멈추고 `undefined`를 반환하게 된다.

```javascript
let user = {}; // 주소정보가 없는 사용자

alert( user?.address?.street ); // undefined, 에러가 발생하지 않습니다.
```

### 옵셔널 체이닝 사용할 때 

* 남용 놉!

`?.`를 사용할 땐, 상대 객체가 존재하지 않아도 괜찮은 경우에만 사용하도록 한다.

* `?.` 앞의 변수는 선언되어있어야 한다.

```javascript
// ReferenceError: user is not defined
user?.address;
```

* 단락 평가

옵셔널 체이닝은 왼쪽 평가대상에 값이 없으면 평가를 멈춘다. 이를 단락 평가라고 한다.

그래서 옵셔널 체이닝 오른쪽에 있는 부가 동작들의 경우, `?.`의 실행이 멈췄을 경우 더 진행되지 않는다.

```javascript
let user = null;
let x = 0;

user?.sayHi(x++); // 아무 일도 일어나지 않습니다.

alert(x); // 0, x의 값은 증가하지 않습니다.
```

* `?.()` & `?.[]`

위에 계속 연산자라고 정의했는데,.... `?.`는 연산자가 아니다. **함수, 대괄호**와 함께 동작할 수 있는 특별 문법 구조체이다.

`?.()`는 존재 여부가 확실하지 않는 함수를 호출할 때 사용할 수 있다. `?.[]`의 경우, 존재 여부가 확실하지 않은 객체 프로퍼티에 접근하는 데에 쓰일 수 있을 것이다. 또한 옵셔널 체이닝은 `delete` 연산자와도 함께 사용할 수 있다.


### Reference

- [modern JS - Optional Chaining](https://ko.javascript.info/optional-chaining)


