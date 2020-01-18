---
layout: post
title: "[ES6] Class"
date: 2020-01-16
tags: [JavaScript]
comments: true
---

## Javascript의 Class

javascript는 다른 언어와는 다르게 해당 객체의 유사객체, 인스턴스를 정의하는 대표적인 방법인 class를 지원해주지 않았다. ES6에서 뒤늦게 스펙에 class가 포함되도록 업데이트 되었지만, 이는 다른 언어의 클래스와 완전히 동일하지 않다고 한다. 

난 이전에 사람들과 class에 관해서 얘기하는 기회가 생겨 생각을 해보게 되었다. 그러다가 나는 javascript class가 나오게된 절차라던지, 다른 언어와의 차이점이 구체적으로 무엇인지, 차이점이 생김으로써 어떤 장단점이 있는지를 알아야겠다고 생각했다. 이번 기회로 javascript의 내부 동작 구조를 잘 알아가게 되었으면 한다.

우선 javascript class가 나오기 이전에는 많은 개발자들이 javascript 언어를 가지고 class과 비슷한 라이브러리를 만들었다고 한다.

그 라이브러리란, 클래스에 가장 가까운 방법이라고 할 수 있는데, 생성자를 만들고, 생성자의 프로토타입에 메소드를 할당하는 방법이다. 이를 **사용자 정의 타입 생성**이라고 한다.

```javascript
// ES5

function Person(name) {
    this.name = name;
}

Person.prototype.sayName = function() {
    console.log(this.name);
};

const leesoo = new Person('leesoo');

console.log(leesoo instanceof Person);  // True
console.log(leesoo instanceof Object);  // True
```

위의 코드가 class가 없었을 때, class처럼 동작하도록 만든 함수이다. 해당 함수는 생성자처럼 name프로퍼티를 받아 가지고 있도록 하고, prototype으로 메소드를 만든다. 그리고 해당 객체를 생성자로 생성하는 인스턴스들은 prototype 상속을 통해서 만들어진다. 

해당 인스턴스는 그래서 Person과 Object의 인스턴스로 간주된다.

```javascript
// ES6

class Person {
    constructor(name) {
        this.name = name
    }

    sayName() {
        console.log(this.name);
    }
}

const leesoo = new Person('leesoo');

console.log(leesoo instanceof Person);  // true
console.log(leesoo instanceof Object);  // true

console.log(typeof Person); // function
console.log(typeof Person.prototype.sayName);   // function
```

ES6부터 클래스가 지원이 된다. 위처럼 클래스 선언은 사실 ES5 때 했던 사용자 정의 타입 생성과 거의 유사하다. 겉으로 보이는 차이점이 있다면, 함수를 생성자로 정의하지 않고 클래스 선언을 하게되면 constructor 메소드 이름을 사용해야한다는 점이다. 클래스 메소드는 function 키워드를 사용하지 않아도 된다. 또한 메소드를 여러 개 추가하여도 된다.

클래스나 생성자 내부에서 프로퍼티를 선언할 때, 무조건 클래스 생성자 내부에서 만드는 것이 좋다. 이는 클래스의 사용법으로써 올바르기 때문이다. 클래스의 한 장소인 constructor에서 모든 프로퍼티가 표현되기 때문이다.

클래스 선언은 사실, ES5에서부터 class를 흉내내왔던 기존 사용자 정의 타입 선언의 `Syntatic Sugar` 이다.

실제로 내부적으로 Person이라는 class는 `constructor`이라는 메소드의 동작을 갖는 함수를 생성한다. 그 근거가 바로 `typeof Person`이 `function`으로 나오는 이유이다.

javascript의 class는 function이다. java와 비교하면, java는 class라는 데이터 타입이 존재한다. 하지만 javascript에는 class 타입이란건 없다. (너무 당연한 소리하나..)

그렇다면 javascript를 쓰는데 class를 써야하는 이유는 무엇일까??

### 클래스를 사용해야하는 이유

class와 사용자 정의 타입은 유사하다. 그럼에도 불구하고 class를 사용해야하는 이유가 있다.

class과 사용자 정의 타입의 차이점은,

* class 선언은 function 선언과는 달리 호이스팅되지 않는다. 이는 let 선언과 같이 동작한다. 즉, 선언이 되고 실행될 때까지 TDZ에 존재한다.
* class로 선언된 코드는 `strict` 모드로 실행된다. 이건 피할 수 없다.
* class 내부의 모든 메소드는 `Non-enumerable`이다.(비열거)
* 마찬가지로 class 내부의 메소드는 [[Construct]] 메소드가 존재하지 않아 new로 호출하면 에러가 생긴다.
* class 메소드 내에서 클래스 이름을 덮어쓸 경우, 오류가 생긴다.

```javascript
// PersonClass와 동일합니다.
let PersonType2 = (function() {
    "use strict";
    const PersonType2 = function(name) {
        // new를 이용하여 호출이 되었는지 확인
        if (typeof new.target === "undefined") {
            throw new Error("Constructor must be called with new.");
        }
        this.name = name;
    }
    Object.defineProperty(PersonType2.prototype, "sayName", {
        value: function() {
            // new를 이용하여 호출하지 않도록 함.
            if (typeof new.target !== "undefined") {
                throw new Error("Method cannot be called with new.");
            }
            console.log(this.name);
        },
        enumerable: false,
        writable: true,
        configurable: true
    });
    return PersonType2;
}());
```

### 참고

- [Javascript 클래스 소개](https://infoscis.github.io/2018/02/13/ecmascript-6-introducing-javascript-classes/)
