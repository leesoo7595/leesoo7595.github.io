---
layout: post
title: "Javascript - Object"
date: 2020-05-27
tags: [JavaScript]
categories:
- Javascript
comments: true
---

## 객체

자바스크립트에는 8가지의 자료형이 존재한다. 그 중 7개는 원시형이라고 부르며 형태가 변하지않는 한 가지의 데이터를 담을 수 있는 형태를 말하지만, 나머지 한 가지인 객체형은 다양한 데이터를 담는 것이 가능하다.

자바스크립트에서 객체는 모든 곳에 존재하는 개념이라고 할 수 있어서 객체를 이해하는 것이 중요할 것이다.

### 선언

```javascript
let obj = {}  // 객체 리터럴
let obj = new Object()  // 객체 생성자
```

### in & of 연산자

자바스크립트에는 `in`과 `of`의 연산자가 있다. 이 둘의 차이를 아는 것도 매우 중요하다.

우선 객체의 키 값을 알고 싶을 때는 연산자 `in`을 사용하면 쉽게 알아낼 수 있다. 반대로 객체의 value 값을 알고 싶을 땐, `of` 연산자를 사용하면 된다.

```javascript
for (let key in object) {
  // 각 프로퍼티 키(key)를 이용하여 본문(body)을 실행합니다.
}

for (let value of object) {
  // 각 프로퍼티 값(value)를 이용하여 본문(body)을 실행합니다.
}
```

### 객체 프로퍼티 삭제

객체의 프로퍼티를 삭제하는 방법은 `delete`를 사용하면 된다. 하지만 그렇다고 delete를 사용하여 프로퍼티를 삭제하는 방법을 추천하지는 않는다고 한다.

![image](https://user-images.githubusercontent.com/39291812/83161753-cfd5d680-a143-11ea-8a5d-582cac0dcb49.png)

이유는 위와 같이 배열에서 `delete`를 사용할 경우, 해당 값이 아예 사라지는 것이 아닌 empty 값으로 바뀌는 것과 비슷하다.

`delete` 연산자는 피연산자가 성공적으로 삭제되었을 때, `true`를 반환하고, 아니면 `false`를 반환한다. 여기서 중요한 사실은 모든 변수나 프로퍼티를 삭제할 수 없다는 것이다. 이게 무슨 뜻이냐, 객체 자체는 삭제할 수 없지만 객체의 프로퍼티는 삭제할 수 있다는 의미이고, 또한 객체의 프로퍼티 중에서도 몇몇 중요한 프로퍼티의 경우는 삭제할 수 없고, `var`를 사용한 변수들도 삭제할 수 없다.

또한 존재하지 않는 프로퍼티를 `delete`를 사용하여 삭제할 경우 `true`가 반환되는 것도 오점 중 하나라고 얘기할 수 있다.

전반적으로 객체 자체는 삭제할 수 없으면서 프로퍼티만 사용할 수 있고, 배열 자체에서 요소를 삭제하려면 또 다른 방법을 사용해야한다는 불편함 등 신경써야할 오점이나 특징들이 있기 때문에 `delete` 연산자는 자바스크립트에서는 많이 쓰이지 않는다.

#### Object Internal Methods & Internal Slots

ECMAScript에서 객체의 시맨틱한 의미는, 인터널 메소드라고 불리는 특정 알고리즘에 의해 동작하는 것이다. 객체의 ECMAScript 엔진은 런타임 동작을 정의하는 인터널 메소드와 연관되어있다. 이 인터널 메소드는 ECMAScript 언어로 되어있지않다. 인터널 메소드는 어떻게 구현되어있는지는 설명하지 않겠지만, 이 인터널 메소드들로 인해 ECMAScript로 구현되어있는 객체는 연관된 인터널 메소드로 지정된 대로 작동해야 한다.

인터널 메소드 이름들은 다형성을 가지고 있는데, 이 뜻은 공통 이름을 가진 인터널 메소드라도 이 메소드가 호출될 때 다른 객체가 다른 알고리즘을 수행할 수 있다는 것을 의미한다. 인터널 메소드가 호출될 때, 호출의 대상이 해당 객체이다. 만약, 런타임시 지원되지 않는 인터널 메소드의 알고리즘을 실행시키려고 할 때, `TypeError`가 일어난다.

여기서 객체가 기본적으로 꼭 가져야하는 인터널 메소드들 중에 `[[Delete]]`라는 인터널 메소드가 존재한다.

<img width="985" alt="Screen Shot 2020-05-29 at 1 28 53 AM" src="https://user-images.githubusercontent.com/39291812/83167977-c51f3f80-a14b-11ea-9c6d-939d564dfbfa.png">

위의 사진은 명세서의 `[[Delete]]` 인터널 메소드를 설명한 것인데, 위의 내가 적어놓은 설명과 거의 똑같은 얘기를 하고 있다.

#### 잘 알려진 Intrinsic Objects

intrinsic Objects는 빌트인 객체들을 의미한다. 현제 명세에 적혀있는 정도의 알고리즘으로 참조되고, 일반적으로 영역별(realm-specific) 특징(아이덴티티)를 가지고 있는 객체를 말한다. 해당 Instrinsic Objects는 한 realm(영역) 당 유사한 객체 덩어리에 해당한다.

```
// Array Intrinsic Objects
%Array%, %ArrayBuffer%, %ArrayBufferPrototype%, %ArrayIteratorPrototype%, %ArrayPrototype%, %ArrayProto_entries%, %ArrayProto_forEach%, %ArrayProto_keys%, %ArrayProto_values%	

// Async
%AsyncFromSyncIteratorPrototype%, %AsyncFunction%, %AsyncFunctionPrototype%, %AsyncGenerator%, %AsyncGeneratorFunction%, %AsyncGeneratorPrototype%, %AsyncIteratorPrototype%

%Atomics%
... 수 없이 많다. 내장 객체면 다 있다고 봐야..
```

### 객체가 비었을 땐?

`isEmpty(obj)`를 사용하면, 해당 객체에 프로퍼티가 한 개도 없으면 `true`, 아니면 `false`를 뱉는다!

### 참조!

객체와 원시 타입의 가장 근본적인 차이는 객체는 **참조에 의해** 저장되고 복사된다는 것이다. 원시값은 반면에 **값 그대로** 저장되고 복사된다.

변수에 원시값을 넣을 때는 값 그대로가 저장되지만, 변수에 객체를 넣을 땐, 객체 자체가 그대로 저장되는 것이 아닌 **객체가 저장되어있는 메모리 주소 값**이 저장된다. 그리고 이를 **객체에 대한 참조 값**이라고 한다.

즉, 객체는 메모리 내 어딘가에 저장되고, 변수는 객체를 참조할 수 있는 값이 저장된다. 그래서 객체가 할당된 변수를 복사하게되면, 참조값이 복사되고 객체 자체는 복사되지 않는다.

**또 한번 즉, 객체를 조작하게 되면 해당 객체를 참조하고 있는 변수들이 값이 반영된다고 볼 수 있다.**

#### 참조 비교

```javascript
let a = {};
let b = a; // 참조에 의한 복사

alert( a == b ); // true, 두 변수는 같은 객체를 참조합니다.
alert( a === b ); // true
```

```javascript
let a = {};
let b = {}; // 독립된 두 객체

alert( a == b ); // false
```

### 객체 복사, 병합

객체를 똑같이 복제하고 싶으면서 기존의 객체를 건드리고 싶지 않을 땐, 새로운 객체를 만든 후 기존 객체의 프로퍼티를 순회해서 원시값 수준까지 프로퍼티를 복사하면 된다.

```javascript
let user = {
  name: "John",
  age: 30
};

let clone = {}; // 새로운 빈 객체

// 빈 객체에 user 프로퍼티 전부를 복사해 넣습니다.
for (let key in user) {
  clone[key] = user[key];
}

// 이제 clone은 완전히 독립적인 복제본이 되었습니다.
clone.name = "Pete"; // clone의 데이터를 변경합니다.

alert( user.name ); // 기존 객체에는 여전히 John이 있습니다.
```

for 문으로 순회하는 방법도 있지만, `Object.assign`을 사용해도 마찬가지 결과가 나온다. 또한 이 메소드는 여러 객체를 하나도 병합하는 것도 가능하다.

```javascript
let user = {
  name: "John",
  age: 30
};

let clone = Object.assign({}, user);
```

위의 예시를 실행하면, user 객체 내의 프로퍼티가 모두 빈 배열에 복사되고 변수에 할당된다.

### 중첩 객체 복사

중첩 객체 복사의 경우 더욱 문제는 까다로워진다. 객체 내부에 객체가 또 존재하기 때문에, 객체 내부만을 복사하는 것만으로는 모두 복제할 수 없기 때문이다. 이 경우, for문을 돌면서 각 키 값을 또 한번 검사하여 해당 키 값이 객체인 경우 이 객체도 for문을 돌리는 식으로 깊은 복사를 해야한다.

간단하게 자바스크립트의 유명한 라이브러리인 `lodash`의 메서드인 `_.cloneDeep(obj)`을 사용하면 직접 구현하지 않아도 중첩객체를 깊은 복사 처리할 수 있다.
