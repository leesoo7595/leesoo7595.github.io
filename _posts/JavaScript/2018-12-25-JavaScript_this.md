---
layout: post
title: "[JavaScript] this"
date: 2018-12-25
tags: [JavaScript]
comments: true
---

## 함수의 호출 방식에 의해 결정되는 this

Java에서의 this는 인스턴스 자신을 가리키는 참조변수이다. this가 객체 자신에 대한 참조 값을 가지고 있다는 뜻이다.

하지만 자바스크립트의 경우 this에 바인딩되는 객체는 한 가지가 아니라 해당 함수 호출 방식에 따라 this에 바인딩되는 객체가 달라진다.

함수를 선언할 때 this에 바인딩할 객체가 정적으로 결정되는 것이 아닌, 함수를 호출할 때 함수가 어떻게 호출되었는지에 따라 this에 바인딩할 객체가 동적으로 결정된다.

함수를 호출하는 방식은 다양하다.

* 함수 호출
* 메소드 호출
* 생성자 함수 호출
* apply/call/bind 호출

```javascript
var foo = function () {
  console.dir(this);
};

// 1. 함수 호출
foo(); // window
// window.foo();

// 2. 메소드 호출
var obj = { foo: foo };
obj.foo(); // obj

// 3. 생성자 함수 호출
var instance = new foo(); // instance

// 4. apply/call/bind 호출
var bar = { name: 'bar' };
foo.call(bar);   // bar
foo.apply(bar);  // bar
foo.bind(bar)(); // bar
```

### 함수 호출

전역 객체는 모든 객체의 유일한 최상위 객체를 의미한다. 기본적으로 this는 전역객체에 바인딩된다. 전역함수는 물론이고 내부함수의 경우도 this는 외부함수가 아닌 전역객체에 바인딩된다.

내부함수는 어디에서 선언되었든 관계없이 this는 전역객체를 바인딩한다. 내부함수의 this가 전역객체를 참조하는 것을 회피하는 방법은 아래와 같다.

```javascript
var value = 1;

var obj = {
  value: 100,
  foo: function() {
    var that = this;  // Workaround : this === obj

    console.log("foo's this: ",  this);  // obj
    console.log("foo's this.value: ",  this.value); // 100
    function bar() {
      console.log("bar's this: ",  this); // window
      console.log("bar's this.value: ", this.value); // 1

      console.log("bar's that: ",  that); // obj
      console.log("bar's that.value: ", that.value); // 100
    }
    bar();
  }
};

obj.foo();
```

### 메소드 호출

함수가 객체의 프로퍼티 값이면 메소드로서 호출된다. 이때 메소드 내부의 this는 해당 메소드를 소유한 객체(해당 메소드를 호출한 객체)에 바인딩된다.

```javascript
var obj1 = {
  name: 'Lee',
  sayName: function() {
    console.log(this.name);
  }
}

var obj2 = {
  name: 'Kim'
}

obj2.sayName = obj1.sayName;

obj1.sayName();
obj2.sayName();
```

프로토타입 객체도 매소드를 가질 수 있다. 프로토타입 객체 메소드 내부에서 사용된 this도 일반 메소드 방식과 마찬가지로 해당 메소드를 호출한 객체에 바인딩된다.

```javascript
function Person(name) {
  this.name = name;
}

Person.prototype.getName = function() {
  return this.name;
}

var me = new Person('Lee');
console.log(me.getName());

Person.prototype.name = 'Kim';
console.log(Person.prototype.getName());
```

### 생성자 함수 호출

자바스크립트 생성자 함수는 객체를 생성하는 역할을 한다. 이는 기존 함수에 new 연산자를 붙여서 호출하면 해당 함수는 생성자 함수로 동작한다.

```javascript
// 생성자 함수
function Person(name) {
  this.name = name;
}

var me = new Person('Lee');
console.log(me); // Person&nbsp;{name: "Lee"}

// new 연산자와 함께 생성자 함수를 호출하지 않으면 생성자 함수로 동작하지 않는다.
var you = Person('Kim');
console.log(you); // undefined
```

new 연산자와 함께 생성자 함수를 호출하면 this 바인딩이 메소드나 함수 호출 때와는 다르게 동작한다.

#### 생성자 함수 동작 방식

1. 빈 객체 생성 및 this 바인딩

생성자 함수의 코드가 실행되기 전 빈 객체가 생성된다. 이후 생성자 함수 내에서 사용되는 this는 이 빈 객체를 가리킨다. 그리고 생성된 빈 객체는 생성자 함수의 prototype 프로퍼티가 가리키는 객체를 자신의 프로토타입 객체로 설정한다.

2. this를 통한 프로퍼티 생성

생성된 빈 객체에 this를 사용하여 동적으로 프로퍼티나 메소드를 생성할 수 있다. this는 새로 생성된 객체를 가리키므로 this를 통해 생성한 프로퍼티와 메소드는 새로 생성된 객체에 추가된다.

3. 생성된 객체 반환

- 반환문이 없는 경우, this에 바인딩된 새로 생성된 객체가 반환된다.
- 반환문이 this가 아닌 다른 객체를 명시적으로 반환하는 경우, this가 아닌 해당 객체가 반환된다.

```javascript
function Person(name) {
  // 생성자 함수 코드 실행 전 -------- 1
  this.name = name;  // --------- 2
  // 생성된 함수 반환 -------------- 3
}

var me = new Person('Lee');
console.log(me.name);
```

#### 객체 리터럴 방식과 생성자 함수 방식의 차이

```javascript
// 객체 리터럴 방식
var foo = {
  name: 'foo',
  gender: 'male'
}

console.dir(foo);

// 생성자 함수 방식
function Person(name, gender) {
  this.name = name;
  this.gender = gender;
}

var me  = new Person('Lee', 'male');
console.dir(me);

var you = new Person('Kim', 'female');
console.dir(you);
```

객체 리터럴 방식과 생성자 함수 방식의 차이는 `프로토타입 객체([[Prototype]])`에 있다.

- 객체 리터럴 방식의 경우, 생성된 객체의 프로토타입 객체는 `Object.prototype`이다.
- 생성자 함수 방식의 경우, 생성된 객체의 프로토타입 객체는 `Person.prototype`이다.

#### 생성자 함수에 new 연산자를 붙이지 않고 호출

일반함수와 생성자 함수의 호출시 this 바인딩 방식이 다르기 때문에 객체 생성 목적으로 작성한 생성자 함수를 new 없이 호출하거나 일반함수에 new를 붙여 호출하면 오류가 발생할 수 있다.

```javascript
function Person(name) {
  // new없이 호출하는 경우, 전역객체에 name 프로퍼티를 추가
  this.name = name;
};

// 일반 함수로서 호출되었기 때문에 객체를 암묵적으로 생성하여 반환하지 않는다.
// 일반 함수의 this는 전역객체를 가리킨다.
var me = Person('Lee');

console.log(me); // undefined
console.log(window.name); // Lee
```

생성자 함수를 new 없이 호출하면, Person 내부의 this는 전역객체를 가리키므로 name는 window에 바인딩된다. 또한 new와 함께 생성자 함수를 호출하는 경우에 this도 반환하지 않고, 반환문이 없으므로 undefined를 반환한다. 이를 회피하기 위해서는 `Scope-Safe Constructor` 패턴이 광범위하게 사용된다.

`callee`는 `arguments` 객체의 프로퍼티로서 함수 바디 내에서 현재 실행 중인 함수를 참조할 때 사용한다. 즉, 함수 바디내에서 현재 실행 중인 함수의 이름을 반환한다.

### apply/call/bind 호출

this에 바인딩될 객체는 함수 호출 패턴에 의해 결정된다. 암묵적 this 바인딩 이외에 this를 특정 객체에 명시적으로 바인딩하는 방법도 제공된다. 이가 가능하도록 하는 것이 `Function.prototype.apply, Function.prototype.call` 메소드이다.

이 메소드는 모든 함수 객체의 프로토타입 객체인 `Function.prototype` 객체의 메소드이다.

```javascript
func.apply(thisArg, [argsArray])

// thisArg: 함수 내부의 this에 바인딩할 객체
// argsArray: 함수에 전달할 argument의 배열
```
