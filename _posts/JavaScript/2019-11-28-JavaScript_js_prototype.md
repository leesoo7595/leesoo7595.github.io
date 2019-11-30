---
layout: post
title: "[JavaScript] function"
date: 2019-11-28
tags: [JavaScript]
comments: true
---

# 프로토타입

## 프로토타입 객체

자바스크립트는 **프로토타입 기반 객체지향 프로그래밍** 언어이다.

 클래스 기반 객체지향 프로그래밍 언어는 객체 생성 이전에 클래스를 정의하고 이를 통해 객체를 생성한다. **하지만 프로토타입 기반 객체지향 프로그래밍 언어는 클래스 없이도 객체를 생성할 수 있다.** ES6에서 클래스가 추가되긴 했다.

 **자바스크립트의 모든 객체는 자신의 부모 역할을 담당하는 객체와 연결(체이닝)되어 있다.** 이는 객체 지향의 상속 개념과 비슷하게 부모 객체의 프로퍼티 또는 메소드를 상속받아 사용할 수 있다. 이런 부모 객체를 프로토타입 객체라고 한다.

Prototype 객체는 생성자 함수(New Function)에 의해 생성된 객체에 공유 프로퍼티를 제공하기 위해 사용한다.

```javascript
var student = {
    name: 'Tsun',
    age: 25,
};

console.log(student.hasOwnProperty('name'));    // true
console.dir(student);
```

ECMAScript spec

자바스크립트의 모든 객체는 `[[Prototype]]`이라는 인터널 슬롯을 가진다. 이는 상속을 구현하는 데에 사용된다. `[[Prototype]]` 값은 상속을 구현하는 데에 사용되는 객체이다. `[[Prototype]]` 객체의 데이터 프로퍼티는 get 엑세스를 위해 상속되어 자식 객체의 프로퍼티처럼 사용할 수 있다. set 엑세스는 허용되지 않는다.

`[[Prototype]]`의 값은 프로토타입 객체이다. `__proto__`로 접근할 수 있다. `__proto__` 프로퍼티에 접근하면 내부적으로 `Object.getPrototypeOf`가 호출되어 프로토타입 객체를 반환한다.

```javascript
var student = {
  name: 'Lee',
  score: 90
}
console.log(student.__proto__ === Object.prototype); // true
```

위 예제에서 student 객체는 `__proto__`의 프로퍼티로 자신의 부모 객체인 `Object.prototype`을 가리키고 있다.

**객체를 생성할 때 프로토타입이 결정되지만, 이는 다른 객체로 변경할 수 있다.** 즉, 부모 객체인 프로토타입을 동적으로 변경할 수 있다는 뜻이다. 이를 활용하여 객체의 상속을 구현할 수 있다.

## [[Prototype]] vs prototype 프로퍼티

모든 객체는 자신의 프로토타입 객체를 가리키는 `[[Prototype]]`을 가지며, 이는 상속을 위해 사용된다. 여기서 함수 또한 객체이므로 `[[Prototype]]`을 갖는다. 여기서 **함수 객체는 prototype 프로퍼티도 소유하게 된다.**

**prototype 프로퍼티와 [[Prototype]]은 다르다!**

```javascript
function Person(name) {
  this.name = name;
}

var foo = new Person('Lee');

console.dir(Person); // prototype 프로퍼티가 있다.
console.dir(foo);    // prototype 프로퍼티가 없다.
```

<img width="335" alt="Screen Shot 2019-11-29 at 12 50 46 AM" src="https://user-images.githubusercontent.com/39291812/69819496-74816500-1242-11ea-8b13-d150068dcce5.png">

<img width="337" alt="Screen Shot 2019-11-29 at 12 50 56 AM" src="https://user-images.githubusercontent.com/39291812/69819498-74816500-1242-11ea-8aa2-a068a9e619e5.png">


* `[[Prototype]]`

    - 함수를 포함한 모든 객체가 가지고 있다.
    - 객체의 부모 역할을 하는 프로토타입 객체를 가리킨다.
    - 함수의 경우 `Function.prototype`을 가리킨다.

`console.log(Person.__proto__ === Function.prototype);`

* prototype 프로퍼티

    - **함수 객체만** 가지고 있는 프로퍼티이다.
    - 함수 객체가 생성자로 사용될 때, 이 함수를 통해 생성될 객체의 부모 역할을 하는 프로토타입 객체를 가리킨다.

`console.log(Person.prototype === foo.__proto__);`

함수가 생성자로 사용되어 생성된 객체는 해당 객체의 프로토타입 객체는 함수의 프로토타입 객체를 가리킨다..(뭔소리여..)

## constructor 프로퍼티

프로토타입 객체는 `constructor 프로퍼티`를 가진다. 이는 객체를 생성한 또 다른 객체를 가리킨다.

위 예제에서 Person()이라는 생성자 함수에 의해 생성된 foo라는 객체를 예로 들면,

foo라는 객체를 생성한 객체는 Person() 생성자 함수이다. 

```javascript
function Person(name) {
  this.name = name;
}

var foo = new Person('Lee');

// Person() 생성자 함수에 의해 생성된 객체를 생성한 객체는 Person() 생성자 함수이다.
console.log(Person.prototype.constructor === Person);

// foo 객체를 생성한 객체는 Person() 생성자 함수이다.
console.log(foo.constructor === Person);

// Person() 생성자 함수를 생성한 객체는 Function() 생성자 함수이다.
console.log(Person.constructor === Function);
```

엄청 헷갈리는데....천천히 읽어보면 이해가 된다. 여러 차례 읽어보잣...

## Prototype chain

자바스크립트는 객체의 프로퍼티나 메소드에 접근할 때, 객체에 해당 프로퍼티나 메소드가 존재하지 않으면 `[[Prototype]]`이 가리키는 링크를 따라서 자신의 부모 역할을 하는 프로토타입 객체의 프로퍼티, 메소드를 검색한다. 이를 프로토타입 체인이라고 한다.

```javascript
var student = {
  name: 'Lee',
  score: 90
}

console.dir(student);
console.log(student.hasOwnProperty('name')); // true
console.log(student.__proto__ === Object.prototype); // true
// Object.prototype.hasOwnProperty()
console.log(student.hasOwnProperty('name')); // true
```

예를 들면 `hasOwnProperty` 메소드이다. 객체를 등록할 때 해당 메소드를 가지고 있지않지만, student 객체의 `[[Prototype]]`이 가리키는 링크를 따라가서 객체의 부모 역할을 하는 프로토타입 객체(`Object.prototype`)의 메소드를 호출하였기 때문에 정상 출력된다.

### 객체 리터럴 방식으로 생성된 객체의 프로토타입 체인

객체 생성 방법

* 객체 리터럴
* 생성자 함수
* Object() 생성자 함수

`객체 리터럴 방식`으로 만들어진 객체는 내장 함수의 `Object() 생성자 함수`로 객체를 생성하는 것을 단순화시킨 것이다.

자바스크립트 엔진은 `객체 리터럴 방식`으로 객체를 생성하는 코드를 만나면 내부적으로 `Object() 생성자 함수`를 사용하여 객체를 생성한다.

`Object() 생성자 함수`는 함수라서, 일반 객체와 달리 `prototype 프로퍼티`가 있다.

* prototype 프로퍼티는 함수 객체가 생성자로 사용되는 경우 이 함수를 통해 생성된 객체의 부모 역할을 하는 객체를 가리킨다.(프로토타입 객체)
  - 결국 prototype 프로퍼티는 해당 객체의 부모 객체가 가지고 있는 프로퍼티를 말하는 것이라고 생각된다.
* `[[Prototype]]`은 객체의 부모 역할을 하는 객체를 가리킨다. 

<img width="403" alt="Screen Shot 2019-11-30 at 9 34 31 PM" src="https://user-images.githubusercontent.com/39291812/69900632-44100700-13b9-11ea-8f27-ec35979bd51f.png">

내가 이해한 prototype과 __proto__의 차이....

```javascript
var person = {
  name: 'Lee',
  gender: 'male',
  sayHello: function(){
    console.log('Hi! my name is ' + this.name);
  }
};

console.dir(person);

// person 객체의 부모 === Object 함수의 prototype
console.log(person.__proto__ === Object.prototype);   // ① true
// person 객체의 부모의 생성자(Object 함수의 prototype의 생성자)
console.log(Object.prototype.constructor === Object); // ② true
// Object 함수의 부모 === Function 함수의 prototype
console.log(Object.__proto__ === Function.prototype); // ③ true
// Object 함수의 부모의 부모????? === Function 함수의 prototype의 부모 === Object 함수의 prototype???
console.log(Function.prototype.__proto__ === Object.prototype); // ④ true
```

![image](https://user-images.githubusercontent.com/39291812/69900980-2002f480-13be-11ea-9e34-16e05923e42c.png)

너무너무너무 헷갈린다...

내 생각에 이것의 포인트는

* 결국 모든 것은 Object.prototype에서 시작한다.
* function로 만들어진 객체의 __proto__는 function 자체의 prototype이다.
* 객체의 prototype의 __proto__는 Object의 prototype이다.
* Object 자체도 함수이다. 그래서 Object 함수의 __proto__ 또한 Function.prototype이다.
* Function도 함수이다. 그래서 Function.__proto__ 는 Function.prototype이다.
* Function.prototype은 또 다른 객체이기 때문에, Function.prototype의 __proto__는 Object.prototype이다.......
* `function.prototype`이란, f`unction()`의 인스턴스의 `__proto__`이다.

### 생성자 함수로 생성된 객체의 프로토타입 체인

생성자 함수로 객체를 생성하려면 생성자 함수를 정의하여야 한다.

함수를 정의하는 방식

* 함수선언식
* 함수표현식
* Function() 생성자 함수

함수표현식, 함수선언식 모두 함수 리터럴 방식을 사용한다. 이는 즉, Function() 생성자 함수로 함수를 정의하는 것을 단순화시킨 것이다.

즉, 어떤 방식으로 함수를 생성하든 모든 함수 객체의 프로토타입 객체(__proto__)는 `Function.prototype`이 될 것이다. 생성자 함수(`Function()`) 또한 함수이므로 생성자 함수의 프로토타입 객체(__proto__)도 `Function.prototype`일 것이다.

`console.log(Function.__proto__ === Function.prototype);`

```javascript
function Person(name, gender) {
  this.name = name;
  this.gender = gender;
  this.sayHello = function(){
    console.log('Hi! my name is ' + this.name);
  };
}

var foo = new Person('Lee', 'male');

console.dir(Person);
console.dir(foo);

console.log(foo.__proto__ === Person.prototype);                // ① true
console.log(Person.prototype.__proto__ === Object.prototype);   // ② true
console.log(Person.prototype.constructor === Person);           // ③ true
console.log(Person.__proto__ === Function.prototype);           // ④ true
console.log(Function.prototype.__proto__ === Object.prototype); // ⑤ true
```

foo 객체의 __proto__는 Person.prototype이고, Person 생성자 함수(constructor)의 __proto__인 것은 Function.prototype이다. 그리고 Function.prototype의 프로토타입 객체는 결국 Object.prototype이다.

결국 이는 객체 리터럴이든, 생성자 함수든 그 어떤 방식으로 만든 객체 모두 부모 객체가 Object.prototype 객체에서 프로토타입 체인이 끝나게 된다. 그래서 Object.prototype 객체를 프로토타입 체인의 종점(`End of prototype chain`)이라 한다.

## 프로토타입 객체의 확장

프로토타입 객체(__proto__)도 객체이다. 그래서 일반 객체와 같이 프로퍼티를 추가/삭제할 수 있고, 이는 즉시 프로토타입 체인에 반영된다.

<img width="320" alt="Screen Shot 2019-11-30 at 10 40 25 PM" src="https://user-images.githubusercontent.com/39291812/69901288-dd8fe680-13c2-11ea-82d6-f74aa95b7995.png">

## 원시 타입(Primitive data type)의 확장

자바스크립트의 원시 타입을 제외하고는 모두 객체이다. 그래서 원시 타입은 프로퍼티나 메소드를 가질 수 없다.

하지만 원시 타입으로 프로퍼티나 메소드를 호출하게 되면, 원시 타입과 연관된 객체로 일시적으로 변환되어 프로토타입 객체(__proto__)를 공유하게 된다.

<img width="336" alt="Screen Shot 2019-11-30 at 10 50 30 PM" src="https://user-images.githubusercontent.com/39291812/69901369-0a90c900-13c4-11ea-84e2-201d1e874c86.png">

원시 타입 자체에 프로퍼티나 메소드를 가질 수 없기 때문에, 대신 String 객체의 프로토타입 객체(prototype)에 메소드를 추가하면 원시 타입에도 해당 메소드를 사용할 수 있다.

어...근데 __proto__에 메소드를 넣으니까 원시 타입에 먹히지 않는다...!!!

왜나면 String.__proto__는 Function.prototype이라서! Function에 메소드가 들어간거다...!!
String.prototype은 String 객체로 만든 인스턴스의 __proto__이니까 다름다름...!!

`Built-in object`의 `Global objects`인 String, Number, Array 객체 등이 가지고 있는 표준 메소드는 프로토타입 객체인 String.prototype, Number.prototype...등에 정의되어있다. 이들 또한 Object.prototype을 프로토타입 체인에 의해 자신의 프로토타입 객체로 연결한다.

## 프로토타입 객체의 변경

객체를 생성할 때 프로토타입은 결정된다. 하지만 프로토타입 객체는 다른 객체로 변경할 수 있는데, 이를 활용하여 객체의 상속을 구현할 수 있다.

프로토타입 객체를 변경할 경우

* 프로토타입 객체 변경 시점 이전에 생성된 객체
  - 기존 프로토타입 객체를 __proto__에 바인딩한다.
* 프로토타입 객체 변경 시점 이후에 생성된 객체
  - 변경된 프로토타입 객체를 __proto__에 바인딩한다.

```javascript
function Person(name) {
  this.name = name;
}

Person.prototype.gender = 'male';

var foo = new Person('lee');
var bar = new Person('kim');

console.log(foo.gender); // ① 'male'
console.log(bar.gender); // ① 'male'

// 1. foo 객체에 gender 프로퍼티가 없으면 프로퍼티 동적 추가
// 2. foo 객체에 gender 프로퍼티가 있으면 해당 프로퍼티에 값 할당
foo.gender = 'female';   // ②

console.log(foo.gender); // ② 'female'
console.log(bar.gender); // ① 'male'
```

당연한 얘기인듯~
프로토타입은 지속적으로 복습을 해야할 듯하다.. 너무 복잡해서 까먹기 쉬운 내용일 듯

