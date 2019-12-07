---
layout: post
title: "[JavaScript] this"
date: 2019-12-05
tags: [JavaScript]
comments: true
---

# 함수 호출 방식에 따라 결정되는 this

자바스크립트 함수는 호출시 전달받는 것.

* 매개변수 값
* arguments 객체
* this

<img width="502" alt="Screen Shot 2019-12-05 at 7 33 07 PM" src="https://user-images.githubusercontent.com/39291812/70227645-48139e80-1796-11ea-8e67-840a53214a02.png">

인스턴스 자신을 가리키는 참조변수를 의미하는 Java의 this와는 달리, </br>
자바스크립트의 this는 **해당 함수 호출 방식에 따라 this에 바인딩되는 객체가 달라진다.**

## 함수 호출 방식

자바스크립트는 **함수 호출 방식**에 의해 `this`에 바인딩할 객체가 동적으로 결정된다.

`렉시컬 스코프`는 함수를 선언할 때 결정된다는 점에서 </br>
함수 호출 방식에 바인딩 객체가 동적으로 결정되는 `this`와 혼동할 수 있으니 주의하자.

* 함수 호출 방식

1. 함수 호출
2. 메소드 호출
3. 생성자 함수 호출
4. `apply/call/bind` 호출

### 함수 호출

전역 객체는 전역 스코프를 갖는 전역 변수를 프로퍼티로 소유한다. 글로벌 영역에 선언한 함수는 전역객체의 프로퍼티로 접근할 수 있다. 이는 즉, 전역 변수의 메소드가 된다는 것이다.

기본적으로 this는 전역객체에 바인딩된다. 내부함수(어디의 내부함수이든) 또한 어디에 선언되었든지 this는 전역객체를 바인딩한다.(일반 함수, 메소드, 콜백 함수 안에 들어있는 내부함수는 this가 전역객체를 바인딩한다.)

내부 함수의 this가 전역객체를 참조하는 것을 회피하는 방법은, **내부함수에서 this를 지역 변수에 묶어주는 것이다.**

![image](https://user-images.githubusercontent.com/39291812/70229329-53b49480-1799-11ea-8b29-54b68e6b4fa3.png)

위 방법 이외에도 this를 명시적으로 바인딩할 수 있는 `apply`, `call`, `bind` 메소드가 있다.

## 메소드 호출

객체의 프로퍼티에 함수가 있으면, 이 함수는 메소드이다. 메소드 내부의 this는 해당 메소드를 소유한 객체에 바인딩된다.

프로토타입 객체도 메소드를 가질 수 있다. 그리고 프로토타입 객체 메소드에서 사용된 this도 일반 메소드 방식과 같이 해당 메소드를 호출한 객체에 바인딩된다.

(헷갈릴까봐 적어놓는건, 메소드 안의 내부함수는 this를 전역객체에 바인딩한다는 것!)

## 생성자 함수 호출

자바스크립트 생성자 함수는 객체를 생성하는 역할을 한다. 기존 함수에 new 연산자를 붙여서 호출하면 생성자 함수로 동작한다. 이는 함수 내부에 constructor라는 아이가 있기 때문일 것이다...!!! 함수의 __proto__인 (Function.prototype에 붙어있는 아이)

어떤 함수든 생성자 함수처럼 동작할 수 있으므로, 생성자 함수명은 첫 문자를 대문자로 기술하도록 한다.

**new 연산자와 함께 생성자 함수를 호출하면 this 바인딩이 메소드나 함수 호출 때와는 다르게 동작한다.**

### 생성자 함수 동작 방식

new 연산자와 함께 생성자 함수를 호출할 경우

* 빈 객체 생성 및 this 바인딩

1. 생성자 함수의 코드가 실행되기 전에, 빈 객체가 생성된다.
2. 해당 빈 객체가 생성자 함수가 생성하는 객체이다.
3. 생성자 함수 내에서 사용되는 this는 이 빈 객체를 가리킨다.
4. 생성자 함수의 prototype 프로퍼티가 가리키는 갹체를 자신의 프로토타입 객체로 설정한다.

<img width="336" alt="Screen Shot 2019-12-05 at 9 20 31 PM" src="https://user-images.githubusercontent.com/39291812/70234917-1950f480-17a5-11ea-9225-380e9d2678dc.png">

* this를 통한 프로퍼티 생성

1. 빈 객체에 this를 사용하여 동적으로 프로퍼티나 메소드를 생성할 수 있다
2. 이를 통해 생성된 것은 위의 객체에 추가된다.

* 생성된 객체 반환

1. 반환문이 없을 경우, 새로 생성한 객체 자체가 반환된다. this를 리턴하여도 같다.
2. 반환문이 this 이외인 다른 객체인 경우, 해당 객체가 반환된다. 하지만 이를 생성자 함수로 사용하지 않는다.

### 객체 리터럴과 생성자 함수의 차이

객체 리터럴 방식과 생성자 함수 방식의 차이는 **프로토타입 객체**이다.

- 객체 리터럴로 생성된 객체의 __proto__(프로토타입 객체)는 `Object.prototype`이다.
- 생성자 함수로 생성된 객체의 __proto__(프로토타입 객체)는 `Person(생성자함수 이름).prototype`이다.

### 생성자 함수에 new 연산자를 붙이지 않고 호출

일반함수와 생성자 함수는 구별 방법이 없다. 단지 함수에 new 연산자를 붙여 호출하면 생성자 함수로 동작한다.

생성자 함수로 쓸 함수를 new 없이 호출하거나, 일반 함수에 new를 붙이면 오류가 생길 수 있다. this 바인딩 방식이 각각 다르기 때문이다.

일반 함수 호출시 this는 전역 객체에 바인딩이 된다. new 연산자로 생성자 함수를 호출하면 this는 생성된 빈 객체에 바인딩된다.

일반 함수와 생성자 함수는 특별한 구별 방법이 없어서 혼동을 일으킬 수 있다. 그래서 `Scope-Safe Constructor`라는 패턴이 사용된다. 

**arguments.callee** : 현재 실행 중인 함수를 포함한다. 실행 중인 함수의 이름을 알 수 없는 경우에 유용하다. 재귀함수로 쓰이는 익명함수의 경우에 쓰이지만 이는 안좋은 사용 방법이라고 한다. arguments.callee를 쓰지말고 함수명으로 재귀를 시키도록 하는 것이 더욱 유용하다. 이의 한계점은 callee로 호출한 함수의 this 값이 바뀌는 문제점이 있을 수 있다.

## apply/call/bind 호출

자바스크립트 엔진은 암묵적으로 this 바인딩을 한다.(함수 호출 패턴으로 인해) 그 외에 this를 특정 객체에 명시적으로 바인딩하는 방법도 존재한다.

함수 객체의 프로토타입 객체인 `Function.prototype` 객체의 매소드인 apply, call 메소드이다.

### apply()

`func.apply(thisArg, [argsArray])`

* `thisArg`: 함수 내부의 this에 바인딩할 객체
* `argsArray`: 함수에 전달할 argument의 배열

apply() 메소드를 호출하는 주체는 함수이고, 해당 메소드는 this를 특정 객체에 바인딩을 해주는 역할만 수행한다. apply() 메소드는 arguments 객체와 같은 유사 배열 객체에 메소드를 사용하는 경우에 쓰인다.

즉, 유사 배열 객체는 배열이 아니라서 배열의 메소드를 사용할 수 없는데, apply() 메소드를 활용하여 해당 객체에 동적으로 배열 프로퍼티가 추가되도록 하여 사용할 수 있게 하는 것이다.

<img width="273" alt="Screen Shot 2019-12-07 at 5 27 25 PM" src="https://user-images.githubusercontent.com/39291812/70371488-e9c1f980-1916-11ea-8fcb-f134174df946.png">

그래서 위의 예시에는 Person 함수에 foo 객체를 묶어줌으로써 foo객체가 Person의 name 프로퍼티를 쓸 수 있는 것이다.

마찬가지로 유사배열객체도 배열 객체의 메소드를 사용해야할 때, `Array.prototype.slice.apply(arguments)`로 apply() 메소드를 사용해여 arguments 객체의 this를 묶고, Array 함수의 slice 메소드를 사용하도록 하는 것이다.

**여기서 주의할 것은 apply() 메소드는 해당 메소드를 쓰는 함수를 곧바로 호출한다. 단지 호출하기 전에 this를 특정 객체에 바인딩을 해줄 뿐이다.**

### call()

call() 메소드는 , apply()와 기능은 같지만, 배열 형태로 넘기는 프로퍼티를 각각 인자로 넘겨서 사용한다.

```javascript
Person.apply(foo, [1, 2, 3]);

Person.call(foo, 1, 2, 3);
```

apply, call 메소드는 콜백 함수의 this를 위해서 사용되기도 한다. 콜백함수를 호출하는 외부 함수의 this와 콜백함수의 this가 다르기 때문에, 이를 통일해주어야한다.

```javascript
function Person(name) {
    this.name = name;
}

Person.prototype.doSomething = function (callback) {
    if (typeof callback == function) {
        // this가 Person 객체이다.
        callback.call(this); // window를 가리키는 this를 Person을 가리키도록 해준다
    }
};

function foo() {
    console.log(this.name); // this가 window이다.
}

var p = new Person('Lee');
p.doSomething(foo); // 'Lee'
```

### bind()

ES5에는 `Function.prototype.bind`가 추가되었다. bind는 **함수에 인자로 전달한 this가 바인딩된 새로운 함수를 리턴하도록 한다.** `Function.prototype.apply`, `Function.prototype.call` 메소드와는 달리 함수를 실행하지 않기 때문에 명시적으로 함수를 호출해야하는 것에 차이가 있다.

위 예제와 다른 부분 -> `callback.bind(this)();`