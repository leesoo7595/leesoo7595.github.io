---
layout: post
title: "Javascript - prototype"
date: 2020-08-22
tags: [JavaScript]
categories:
- Javascript
comments: true
---

# 프로토타입 상속

## [[Prototype]]

자바스크립트에는 `[[Prototype]]`라는 숨김 프로퍼티를 가진다. 이 값은 null이거나, 다른 객체에 대한 참조를 가지게 되는데, 다른 객체를 참조하는 경우 참조 대상을 프로토타입이라고 부른다.

자바스크립트의 프로토타입 기능은, 객체에서 프로퍼티를 읽으려고 할 때 해당 프로퍼티가 존재하지 않으면, 자바스크립트는 자동으로 해당 객체의 프로토타입에서 프로퍼티를 찾는다. 이러한 동작 방식을 **프로토타입 상속**이라고 부른다. 이 숨김 프로퍼티를 이용하여 값을 설정할 수 있다.

```javascript
let animal = {
  eats: true
};
let rabbit = {
  jumps: true
};

rabbit.__proto__ = animal;
```

`[[Prototype]]` 프로퍼티를 이용하는 방법은 코드에서 `__proto__`를 활용하여 사용할 수 있다. `__proto__`는 getter, setter 설정자이다. 하지만 요새에는 `__proto__`를 사용해서 설정한다기보다, `Object.getPrototypeOf`나 `Object.setPrototypeOf`를 써서 프로토타입을 설정하거나 가져온다. 

프로토타입 상속을 이용하게되면, 지속적으로 프로토타입을 활용하여 상속을 받는 경우가 생긴다. 이를 **프로토타입 체이닝**이라고 한다. 프로토타입 체이닝은 순환 참조를 허용하지 않고, `__proto__`를 이용하여 다른 형태를 참조하지 못한다.(?) 그리고 `__proto__`의 값은 객체나 null만 가능하다.

프로토타입은 프로퍼티를 읽을 때만 사용하고, 프로퍼티를 추가, 수정, 지우는 연산의 경우 객체에서 직접 해주어야 한다. 

```javascript
let animal = {
  eats: true,
  walk() {
    /* rabbit은 이제 이 메서드를 사용하지 않습니다. */
  }
};

let rabbit = {
  __proto__: animal
};

rabbit.walk = function() {
  alert("토끼가 깡충깡충 뜁니다.");
};

rabbit.walk(); // 토끼가 깡충깡충 뜁니다.
```

* 프로토타입에서 상속받은 메소드라도, 해당 객체의 메소드를 호출하면 메소드 안의 this는 호출 대상 객체를 가리킨다.

즉, 상속받은 메소드라도 그 메소드를 사용하는 객체가 this가 된다.

### Reference

[모던 자바스크립트 - 프로토타입](https://ko.javascript.info/prototype-inheritance)