---
layout: post
title: "ES5 간단히 보기 (2)"
date: 2020-02-16
tags: [Class]
categories:
- Class
comments: true
---

## 자바스크립트의 this

자바스크립트에서 함수는 호출되는 순간, 파라미터 인자값 외에도 arguments 객체와 this를 함께 전달받게 된다.

```javascript
function func(number) {
    console.log(arguments);
    console.log(this);
    console.log(number);
}

func(5);
```

<img width="377" alt="Screen Shot 2020-02-17 at 12 49 29 AM" src="https://user-images.githubusercontent.com/39291812/74607815-634c4400-511f-11ea-9948-5e215e36aa1f.png">

**명심하자. 자바스크립트의 this는 함수 호출 방식에 따라 바인딩되는 객체가 달라진다.**

### 함수 호출 방식

즉, this에 바인딩되는 객체는 호출 방식에 따라 **동적**으로 결정된다. 함수 호출 방식의 종류는 다양하다.

* 함수 호출
* 메소드 호출
* 생성자 함수 호출
* apply/call/bind 호출

#### 함수 호출

기본적으로 함수의 `this`는 전역객체(window)에 바인딩된다. 콜백함수나 내부 함수의 경우에도 마찬가지이다. 특히 내부함수의 경우에 어디에서 선언된 것과 관계없이 전역객체에 this를 바인딩한다. 

```javascript
function foo() {
  console.log("foo's this: ",  this);  // window
  function bar() {
    console.log("bar's this: ", this); // window
  }
  bar();
}
foo();
```

`this`가 전역객체를 참조하는 것을 피하는 것은 this가 참조하고 있는 값을 수동적으로 바꾸는 것이다.

```javascript
var obj = {
    foo: function() {
        var that = this;

        console.log("foo's this: ", this);  // obj
        function bar() {
            console.log("bar's this: ", this);  // window
            console.log("bar's that: ", that);  // obj
        }
        bar();
    }
};
```

이외에는 자바스크립트의 this를 명시적으로 바인딩할 수 있는 `apply`, `call`, `bind` 메소드가 존재한다.

### 메소드 호출

함수가 객체의 프로퍼티 값으로 호출된다는 뜻은, 객체의 메소드로서 호출된다는 뜻이다. 메소드 내부의 `this`는 메소드를 가지고 있는 객체에 바인딩된다. 프로토타입의 메소드도 마찬가지로 호출한 객체에 바인딩된다.

```javascript
// 메소드
var obj1 = {
  name: 'Lee',
  sayName: function() {
    console.log(this.name);
  }
}

obj1.sayName();

// 프로퍼티 메소드
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

생성자 함수를 호출한다는 의미는 생성자 함수를 기반하여 객체를 생성한다는 의미이다. 생성자 함수를 통해 객체를 만들 때, 빈 객체가 먼저 생성되면서 생성자 함수 내에서 사용되는 this가 이 비어있는 객체를 가리키게 된다. 그리고 빈 객체는 생성자 함수의 prototype 프로퍼티가 가리키는 객체를 자신의 프로토타입 객체로 설정하도록 한다.

그렇게 생성자 함수를 통한 빈 객체 및 this 바인딩과 프로토타입 상속 후, 바인딩된 this를 통해 생성자 함수에서 가지고 있는 프로퍼티와 메소드를 새로 생성된 객체에 추가된다.

**생성자 함수를 통해 생성하는 인스턴스의 this는 해당 인스턴스를 가리킨다.**

### apply, call, bind

자바스크립트 엔진은 함수를 호출하면서 해당 패턴에 의해 this에 바인딩될 객체를 결정해준다. 이것은 자바스크립트 엔진이 암묵적으로 수행해주는 방식인데, 그 외에도 우리가 직접 명시적으로 this 바인딩을 특정 객체에 수행해주는 방식도 존재한다. 

해당 방식을 제공해주는 메소드가 바로 `apply`, `call`, `bind` 메소드이다. 이 아이들은 `Function.prototype`에서 프로퍼티 메소드로 가지고 있다.

```javascript
Array.prototype.map.call('string', e => e);
// ['s', 't', 'r', 'i', 'g']
```

## 클로저

클로저란, 리턴된 내부함수가 자신이 선언되었을 때의 스코프를 기억하여, 자신이 선언된 순간의 스코프 밖에서 호출되었어도 해당 스코프에 접근할 수 있도록 하는 함수를 말한다.

```javascript
function outerFunc() {
  var x = 10;
  return () => console.log(x);
}

/**
 *  함수 outerFunc를 호출하면 내부 함수가 반환된다.
 *  그리고 함수 outerFunc의 실행 컨텍스트는 소멸한다.
 */
var inner = outerFunc();  // 여기서 함수를 반환하고 사라진다.  
inner(); // 10  그런데 outerFunc 내부에서 선언한 x 값이 나옴
```

위처럼 자신을 포함하고 있는 외부함수보다 내부함수가 더 오래 유지되면, 외부함수의 지역변수를 접근할 수 있게된다. 이를 클로저라고 한다.

이렇게 동작할 수 있는 이유는, 내부함수가 스코프 체인(`[[Scope]]`)이라는 인터널 슬롯을 통해서 자신이 있는 스코프를 기억하고 있기 때문이다. 그래서 이 스코프 체인을 통해 외부함수가 이미 종료되었어도, 외부함수 내의 변수들은 자신을 필요로 하는 내부 함수가 존재하기 때문에 계속 유지된다.(여기서 외부함수 내의 AO(활성객체)가 유효하기 때문에) 이는 실제 변수에 접근한다는 것에 주의해야한다.

<img width="499" alt="Screen Shot 2020-02-20 at 6 24 46 PM" src="https://user-images.githubusercontent.com/39291812/74919683-4c9f3900-540e-11ea-8e6f-c3cf264c60fc.png">

### 활용

* 상태 유지

전역 객체를 활용하지 않고도, 현재 상태를 기억하고 변경된 최신 상태를 유지하는 방법으로 사용될 수 있다. 자바스크립트 내부에서는 모든게 객체(함수)이므로, 이 함수가 호출되어 실행 컨텍스트에서 사라지고 나면, 해당 함수의 내부 변수도 사라지게 되는데, 클로저를 사용하게 되면 해당 변수를 지속적으로 유지하면서 사용할 수 있게 된다.

* 전역 변수 사용 억제

위에서 얘기했듯이 클로저가 없었다면 전역 객체를 사용하여 상태를 유지할 수 밖에 없는데, 이는 좋은 방법이 아니다. 그래서 클로저가 이를 대체한다.

* 정보 은닉

```javascript
function Counter() {
  // 카운트를 유지하기 위한 자유 변수
  var counter = 0;

  // 클로저
  this.increase = function () {
    return ++counter;
  };

  // 클로저
  this.decrease = function () {
    return --counter;
  };
}

const counter = new Counter();

console.log(counter.increase()); // 1
console.log(counter.decrease()); // 0
```

counter라는 변수 자체에 직접적으로 접근할 방법이 없다! 단지 클로저를 통해 접근할 수 있을 뿐이다. 이를 통해 `private`한 걸 흉내낼 수 있다.

## 참고

- [poiemaweb](https://poiemaweb.com)