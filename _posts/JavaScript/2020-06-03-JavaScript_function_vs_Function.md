---
layout: post
title: "Javascript - new 연산자 & 생성자 함수"
date: 2020-06-03
tags: [JavaScript]
categories:
- Javascript
comments: true
---

객체 리터럴을 사용하면 객체를 쉽게 사용할 수 있다. 하지만 특정 유사 객체를 복수로 만들어야할 때, new 연산자와 생성자 함수를 활용하면 해결할 수 있다.

### 생성자 함수

생성자 함수와 일반 함수는 기술적 차이가 없다. 단지 차이점이 존재한다.

* 함수 이름의 첫 글자는 대문자로 시작한다.
* new 연산자를 붙여 실행한다.

```javascript
function User(name) {
  this.name = name;
  this.isAdmin = false;
}

let user = new User('jack');
```

위 로직으로 `new User(...)`를 사용하여 함수를 실행하면

* 빈 객체를 만들어 this를 할당한다.
* 함수 본문을 실행하여 this에 새로운 프로퍼티를 추가한다.
* this를 반환한다.

```javascript
function User(name) {
  // this = {};  (빈 객체가 암시적으로 만들어짐)

  // 새로운 프로퍼티를 this에 추가함
  this.name = name;
  this.isAdmin = false;

  // return this;  (this가 암시적으로 반환됨)
}
```

이렇게 new 연산자를 사용하여 객체를 생성하면 쉽게 해당 사용자 객체를 여러 개 만들 수 있다. 생성자는 이럴 때 쓰인다. 

### new.target

`new.target` 프로퍼티를 사용하면 생성자 함수로 호출되었는지 여부를 알 수 있다. 일반적인 방법으로 함수 호출시 `new.target`은 `undefined`를 반환하지만, new와 함께 호출한 함수의 경우 `new.target`은 함수 자체를 반환한다.

```javascript
function User() {
  alert(new.target);
}

// "new" 없이 호출함
User(); // undefined

//"new"를 붙여 호출함
new User(); // function User { ... }
```

### 생성자 함수와 return

생성자 함수에는 `return`문이 존재하지 않는다. 반환해야할 것은 모두 `this`에 저장하여 this는 자동으로 반환되기 때문에 return을 써줄 필요가 없기 때문이다.

만약 생성자 함수에 return을 한다면, 객체를 리턴했을 경우 해당 return문이 this 객체 대신 반환된다. 반면 원시형의 경우 return 문 자체가 무시되고 this가 반환된다.

* 인수가 없는 생성자 함수의 경우, 괄호를 생략하여 호출할 수 있다.

