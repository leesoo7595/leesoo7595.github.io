---
layout: post
title: "[JavaScript] function"
date: 2019-11-20
tags: [JavaScript]
comments: true
---

# 함수

자바스크립트의 함수는 객체이다. 즉, 호출할 수 있다. 다른 값들처럼 사용할 수 있어 변수나 객체, 배열 등에 저장할 수 있고, 다른 함수에 전달되는 인수로도 사용할 수 있고 반환값도 될 수 있다.

## 함수 정의

* 함수 선언문

```javascript
function makeFunc() {
    console.log('function');
}
```
* 함수 표현식

자바스크립트의 함수는 일급 객체이다.

- 변수 등에 저장할 수 있다.
- 함수의 파라미터로 전달할 수 있다.
- return value로 사용할 수 있다.

```javascript
const makeFunc = function() {
    console.log('function');
}
```

익명 함수라고도 불리는 함수 표현식으로 정의한 함수는 함수명을 생략할 수 있다.

함수를 변수에 할당하게 되면 이 변수는 할당된 함수를 가리키는 참조값을 저장하게 된다.

함수 호출시 함수명이 아닌 함수를 가리키는 변수명을 사용하여야한다.

```javascript
const onClick = () => {
    console.log('click');
}
```
**함수 표현식으로 쓰인 함수가 기명함수일 경우에 기명함수의 함수명으로 호출하면 에러가 발생한다.**

함수 표현식에 사용한 함수명은 외부에서 접근이 불가능하기 때문이다!!!!!

함수 선언문으로 정의한 경우 함수명으로 호출할 수 있다. 그 이유는 함수 선언문으로 정의하였을 때 자바스크립트 엔진이 다음과 같이 변경시키기 때문이다.

```javascript
var func = function func() {
    console.log('func');
}
```

함수명과 함수 참조값을 가진 변수명을 똑같이 만들어 함수명으로 호출되는 것으로 보이지만, 사실은 **변수명**으로 호출된 것이라는 것!!!!!

즉, 함수 선언문도 함수 표현식과 동일하게 함수 **리터럴 방식**으로 정의되는 것이다.

* Function 생성자 함수

함수 표현식도 선언문도 함수 리터럴 방식을 사용한다. 즉, 내장 함수 Function 생성자 함수로 함수를 생성하는 것을 단순화시킨 것이다.

`Function.prototype.constructor`

```javascript
const func = new Function('a', "console.log('a', a)");
```

## 함수 호이스팅

함수 정의 방식은 다 달라도 결국 Function 생성자 함수를 통해 함수를 생성한다. 하지만 동작 방식에 차이가 있다.

함수 선언문의 경우, 함수 선언의 위치와 상관없이 함수를 호출할 수 있다. 이를 호이스팅이라 한다.

**호이스팅이란, 모든 선언문이 해당 스코프의 선두로 옮겨진 것처럼 동작하는 특성을 말한다.**

자바스크립트의 모든 선언문은 선언되기 이전에 참조 가능하다.

함수 선언문으로 정의된 함수는 함수 선언, 초기화, 할당이 한번에 이루어진다.

반대로, 함수 표현식으로 함수를 정의할 경우, 함수 표현식 이전에 함수를 호출하면 `TypeError`가 발생한다.

이는 함수 표현식의 경우에 함수 호이스팅이 아니라 `변수 호이스팅`이 발생하기 때문이다.

변수 호이스팅은 함수와 달리 **변수 생성, 초기화, 할당이 분리되어 진행**된다. 호이스팅된 변수는 `undefined`로 초기화되고, 실제값 할당은 할당문에서 이루어지게 된다.

함수 선언문은(함수 호이스팅) 자바스크립트 엔진이 스크립트 로딩 시점에 바로 초기화하고 변수 객체(`Variable Object`)에 저장하는데, 이는 함수 선언, 초기화, 할당이 한번에 이루어지는 것을 뜻한다.

하지만 함수 표현식은 스크립트 로딩 시점에 변수 객체(`Variable Object`)에 함수를 할당하지 않고, `runtime`에 해석되고 실행된다.

그래서 함수 선언문보다는 함수 표현식을 사용할 것을 권장한다고 한다.... 함수 호이스팅이 코드의 구조를 엉성하게 만들 수 있고, 함수 선언문을 사용할 경우 인터프리터가 너무 많은 코드를 변수 객체(VO)에 저장하여 애플리케이션 응답 속도를 떨어뜨릴 수 있기 때문이다.

### 정리

* 함수 선언문

1. 자바스크립트 엔진에서 스크립트가 로딩될 때 변수 객체에 함수를 저장한다.
2. 즉, 함수 선언, 초기화, 할당이 한번에 이루어진다.
3. 함수 호이스팅이라 한다.
4. 이는 코드의 구조를 엉성하게 만든다.
5. 인터프리터가 너무 많은 코드를 변수 객체에 저장하여 응답 속도를 떨어뜨릴 수 있다.

* 함수 표현식

1. 자바스크립트 엔진에서 스크립트가 로딩될 때 `runtime`에 해석되고 실행되도록 한다.
2. 즉, VO에 저장하지 않는다.
3. 변수 생성, 초기화, 할당이 분리되어 진행된다.
4. 변수 호이스팅이라 한다.
5. 호이스팅된 변수는 `undefined`로 초기화되고, 실제값은 할당문에서 할당한다.
6. 함수 표현식 추천!!

## 일급 객체

프로그래밍 언어의 기본 조작을 제한없이 사용할 수 있는 대상을 말한다.

1. 무명의 리터럴로 표현이 가능하다.
2. 변수나 자료 구조(객체, 배열 등)에 저장할 수 있다.
3. 함수의 매개변수에 전달할 수 있다.
4. 반환값으로 사용할 수 있다.

Javascript 함수는 변수와 같이 사용할 수 있고 어디에서든 정의할 수 있다.

차이는 호출할 수 있다는 것!

## 매개변수 (parameter, 인자)

### 인자, 인수

매개변수는 변수와 동일하게 메모리 공간을 확보한다. 인수를 전달하지 않았을 때 `undefined`로 초기화된다.


### call-by-value

원시타입 인수는 `call-by-value`(값에 의한 호출)로 동작한다. 

함수 호출시 원시 타입 인수를 함수에 매개변수로 전달할 때, 매개변수에 값을 복사하여 전달하게 된다.

함수 내에서 매개변수를 통해 값이 변경되어도 전달이 완료된 원시 타입 값은 변경되지 않는다.

```javascript
function foo(primitive) {
  primitive += 1;
  return primitive;
}

var x = 0;

console.log(foo(x)); // 1
console.log(x);      // 0
```

### call-by-reference

객체형 인수는 `call-by-reference`(참조에 의한 호출)로 동작한다.

함수 호출시 참조 타입 인수를 함수에 매개변수로 전달할 때, 값이 복사되지 않고 객체의 참조값이 매개변수에 저장되어 함수로 전달된다.

즉, 함수 내에서 매개변수로 해당하는 객체의 값이 변경되었을 때, 전달되어진 참조형 인수값도 변경된다.

객체형 인수는 참조값을 전달하기 때문에 함수 값이 변경되면 원본 객체가 변경되는 `side-effect`가 발생한다. 이를 비순수 함수라고 하며, 이는 복잡성을 증가시킨다. 이러한 비순수 함수를 최대한 줄이는 것이 디버깅을 쉽게 만든다.

![image](https://user-images.githubusercontent.com/39291812/69392464-7070b700-0d19-11ea-9ddf-dad4dc261b3a.png)


## 반환값
