---
layout: post
title: "Javascript - toPrimitive"
date: 2020-06-05
tags: [JavaScript]
categories:
- Javascript
comments: true
---

## 객체를 원시형으로

객체를 제외한 원시형 자료가 문자, 숫자, boolean형으로 변환되는지 전에 공부하였다. 하지만 메소드와 심볼의 지식을 갖춘다면, 객체를 원시형으로 어떻게 변환을 할 수 있는지를 공부해볼 수 있다.

그 전에 짚고 갈 기본적인 상식!

* 객체는 boolean 평가시 true를 반환한다. 즉, 객체는 숫자, 문자형으로 형 변환이 일어날 수 있다.

* 객체에서 숫자형으로 형 변환을 할 땐, 객체끼리 빼는 연산을 하거나 수학 관련 함수를 적용할 때 일어난다.

ex) Date 함수

* 문자형으로 객체를 변환시킬 땐, 객체를 출력하려할 때 일어난다.

### ToPrimitive

`ToPrimitive`는 명세서에서 `Abstract Operations` 중 하나인데, 이는 ECMAScript 언어가 아닌 의미론을 명시적으로 하기 위해서만 정의된다. 자바스크립트는 동적 언어이기 때문에 어느 일정 부분은 자바스크립트가 알아서 처리를 해주는 부분이 있다. 이 부분을 `Abstract Operations`라고 나는 이해하고 있고, 이 중 하나인 `Type Conversion` 즉, 형 변환을 자동으로 해주는 메소드 중 하나가 `ToPrimitive`이다. 여기서 객체 형 변환은 세 종류로 구분 된다. 이 구분 기준은 `hint`라는 값인데, 이는 간단히 말하면 목표로 하는 자료형을 의미한다.

1. String

`hint`가 `string`이 되는, 즉 객체를 문자형으로 변환시킬 때를 의미한다. `alert` 함수 같은 문자열을 활용해야하는 연산을 수행할 때 쓰이는 객체 형 변환이다.

```javascript
// 객체를 출력하려고 함
alert(obj);

// 객체를 프로퍼티 키로 사용하고 있음
anotherObj[obj] = 123;
```

2. Number

수학 연산을 수행할 때, `hint`가 `number`가 된다. 여기서 객체는 숫자형으로 변환된다.

```javascript
// 명시적 형 변환
let num = Number(obj);

// (이항 덧셈 연산을 제외한) 수학 연산
let n = +obj; // 단항 덧셈 연산
let delta = date1 - date2;

// 크고 작음 비교하기
let greater = user1 > user2;
```

3. Default

연산자가 기대하는 자료형이 확실하지 않을 때, `hint`는 `default`가 된다. 예를 들어 이항 덧셈 연산자인 `+`는 피연산자의 자료형에 따라 문자열을 합칠 수도 있고, 숫자를 더해줄 수도 있다. 이런 경우에 `hint`는 `default`가 된다.

* 자바스크립트에서 형 변환이 필요한 경우, 알고리즘 실행 흐름

1. 상대 객체에 `obj[Symbol.toPrimitive](hint)`가 있는지 찾고, 호출한다. `Symbol.toPrimitive`는 시스템 심볼이고, 이를 키로 사용된다.
2. `hint`가 `string`이라면, `obj.toString()`이나 `obj.valueOf()`를 호출한다. (존재하는 메소드 우선 실행)
3. `hint`가 `number`, `default`일 경우, `obj.valueOf()`나 `obj.toString()`을 호출한다. (존재하는 메소드 우선 실행)

### Symbol.toPrimitive

자바스크립트에는 `Symbol.toPrimitive`이라는 내장 심볼이 존재하는데, 이 심볼은 형 변환할 때 바꾸게될 목표 자료형(`hint`)을 정하는 데에 사용된다.

```javascript
obj[Symbol.toPrimitive] = function(hint) {
  // 무조건 원시값 리턴 
  // hint -> string, number, default
}
```

아래는 활용 예시이다! user라는 객체에 name과 money라는 프로퍼티가 있는데, 해당 심볼의 hint가 string이면 user라는 객체를 출력할 때 name프로퍼티를, number면 money 프로퍼티 값을 반환하는 예제이다.

```javascript
let user = {
  name: "John",
  money: 1000,

  [Symbol.toPrimitive](hint) {
    alert(`hint: ${hint}`);
    return hint == "string" ? `{name: "${this.name}"}` : this.money;
  }
};

// 데모:
alert(user); // hint: string -> {name: "John"}
alert(+user); // hint: number -> 1000
alert(user + 500); // hint: default -> 1500
```

이처럼 `Symbol.toPrimitive`를 활용하면 해당 심볼 메소드 하나로 객체의 형 변환을 다룰 수 있다.

### toString, valueOf

해당 메소드들은 심볼이 생기기 전부터 있었던 메소드이다. 이 메소드를 사용하면 형 변환을 직접적으로 구현할 수 있다. 자바스크립트에서 실행하는 순서는 객체에 `Symbol.toPrimitive`가 없으면 자바스크립트에서 위의 메소드들을 호출한다.

* hint: string => toString -> valueOf 순으로 메소드 호출
* 그 외 => valueOf -> toString 호출

이 메소드들도 위의 심볼과 마찬가지로 원시값을 반환해야한다. 만약 해당 메소드들이 객체를 반환한다면 무시된다. 일반 객체의 경우, toString은 문자열인 `"[object Object]"`를 반환한다. valueOf는 객체 자신을 반환한다.

```javascript
let user = {name: "John"};

alert(user); // [object Object]
alert(user.valueOf() === user); // true
```

위와 같은 이론으로 객체 자체를 출력할 때 위처럼 `[object Object]`가 출력된다.
아래 로직은 `toString`, `valueOf`를 활용하여 `Symbol.toPrimitive`와 똑같이 구현한 것이다.

```javascript
let user = {
  name: "John",
  money: 1000,

  // hint가 "string"인 경우
  toString() {
    return `{name: "${this.name}"}`;
  },

  // hint가 "number"나 "default"인 경우
  valueOf() {
    return this.money;
  }

};
```

![image](https://user-images.githubusercontent.com/39291812/83874304-383e3c80-a770-11ea-9a04-820910d1d6d5.png)

### Reference

- (자바스크립트에서 [object Object] 가 대체 뭘까?)[https://medium.com/%EC%98%A4%EB%8A%98%EC%9D%98-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EC%97%90%EC%84%9C-object-object-%EA%B0%80-%EB%8C%80%EC%B2%B4-%EB%AD%98%EA%B9%8C-fe55b754e709]
- (모던자스 - toPrimitive)[https://ko.javascript.info/object-toprimitive]
- (tc39)[https://tc39.es/ecma262/#sec-toprimitive]