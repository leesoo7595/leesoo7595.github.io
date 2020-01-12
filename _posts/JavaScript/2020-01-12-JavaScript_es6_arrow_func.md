---
layout: post
title: "[ES6] Arrow Function"
date: 2020-01-12
tags: [JavaScript]
comments: true
---


# Arrow Function

ES6 이후부터는 function 키워드 대신 화살표(=>)를 사용하여 더욱 간략한 방법으로 함수를 선언할 수 있게 되었다. 이번 주제는 화살표 함수(arrow function)의 사용방법 및 규칙에 대해서 정리해보려고 한다.

## 호출

화살표 함수는 익명 함수로만 사용할 수 있다. 즉, 화살표 함수를 호출하기 위해서는 함수 표현식을 사용하여야 한다. 또한 화살표 함수는 콜백 함수로도 더욱 간결하게 사용할 수 있다.

```javascript
// ES5
var pow = function (x) { return x * x; };
console.log(pow(10)); // 100

// ES6
const pow = x => x * x;
console.log(pow(10)); // 100
```

## this

화살표 함수와 function 키워드로 생성한 일반 함수의 차이점은 this이다.

### function의 this

자바스크립트의 this는 호출 방식에 의해 특정 객체가 동적으로 바인딩된다. 즉, 함수를 선언할 때 결정되지 않고, 호출할 때에 따라 this에 바인딩되는 객체가 달라진다.

### arrow function의 this

일반 함수는 함수를 호출할 때 당시의 this에 바인딩할 객체가 동적으로 결정되지만, **화살표 함수는 함수를 선언할 때 this에 바인딩하는 객체가 정적으로 결정된다.**

**화살표 함수의 this는 항상 상위 스코프의 this를 가리킨다.** 이를 `Lexical this`라고 한다. 그래서 화살표 함수의 this 바인딩 객체 결정 방식은, 함수의 상위 스코프를 결정하는 방식인 `Lexical Scope`와 비슷하다.

화살표 함수의 this는 선언시 바로 정적으로 결정되기 때문에, `call, apply, bind` 등의 메소드를 사용하여 this를 변경할 수 없다.

## 화살표 함수를 사용하면 안되는 경우

화살표 함수는 정적으로 this를 바인딩할 객체를 정해주기 때문에 콜백 함수로 사용하기 편리하다. 하지만 화살표 함수를 사용하면 오히려 혼란을 만드는 경우가 있다.

### 메소드

화살표 함수로 메소드를 정의하는 방식은 피해야한다.

메소드로 정의한 화살표 함수 내부의 this는 해당 메소드를 소유한 객체의 상위 컨텍스트를 가리킨다. 그러므로 메소드에 함수를 정의할 때, 일반 함수로 정의하는 것이 올바르다.

```javascript
// Bad
const person = {
  name: 'Lee',
  sayHi: () => console.log(`Hi ${this.name}`)
};

person.sayHi(); // Hi undefined

// Good
const person = {
  name: 'Lee',
  sayHi() { // === sayHi: function() {
    console.log(`Hi ${this.name}`);
  }
};

person.sayHi(); // Hi Lee
```

### prototype

메소드를 정의하는 것과 마찬가지로, prototype에 화살표 함수로 할당하는 것 또한 동일한 문제가 발생한다. 그래서 이 또한 일반 함수를 사용하여 prototype에 메소드를 할당하도록 한다.

### 생성자 함수

**화살표 함수는 생성자 함수로 사용할 수 없다.** 생성자 함수는 prototype 프로퍼티(`__proto__`가 아닌, 함수 객체만 가지고 있는 프로퍼티를 의미)를 가진다.

함수 객체가 생성자로 사용될 때, 이 함수를 통해 생성될 객체(인스턴스)의 부모 역할을 하는 객체를 의미하는 prototype 프로퍼티. 자세한건 프로토타입을 복습하도록!!!! 까묵는다잉...

그리고 prototype 프로퍼티는 프로토타입 객체의 constructor를 사용하여 인스턴스를 생성하게 된다. 그래서 생성자 함수는 prototype 프로퍼티를 가지는 것..!!

하지만 화살표 함수는 **prototype 프로퍼티를 가지고 있지 않기 때문에** 생성자 함수로 사용할 수 없는 것이다.

```javascript
const Foo = () => {};

// 화살표 함수는 prototype 프로퍼티가 없다
console.log(Foo.hasOwnProperty('prototype')); // false

const foo = new Foo(); // TypeError: Foo is not a constructor
```

### addEventListener 함수의 콜백 함수

addEventListener 함수의 콜백 함수를 화살표 함수로 정의할 경우, 화살표 함수의 this가 메소드의 경우와 마찬가지로 상위 컨텍스트에 잡히게 된다. addEventListener의 경우 상위 컨텍스트는 window를 가리키게 된다.

그래서 addEventListener 함수의 콜백함수를 주어야하는 부분에서는 function 키워드를 사용하여 일반 함수로 정의하여야 한다. 그렇게 되면 해당 콜백 함수 내부의 this는 이벤트 리스너에 바인딩된 현재 요소인 `currentTarget을` 가리키게 된다.

```javascript
// Bad
var button = document.getElementById('myButton');

button.addEventListener('click', () => {
  console.log(this === window); // => true
  this.innerHTML = 'Clicked button';
});

// Good
var button = document.getElementById('myButton');

button.addEventListener('click', function() {
  console.log(this === button); // => true
  this.innerHTML = 'Clicked button';
});
```

## 참고 링크

- [poiemaweb - arrow function](https://poiemaweb.com/es6-arrow-function)