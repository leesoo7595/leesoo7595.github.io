---
layout: post
title: "[JavaScript] Scope"
date: 2018-12-23
tags: [JavaScript]
comments: true
---

참고도서 : 러닝 자바스크립트

스코프는 변수와 상수, 매개변수가 언제 어디서 정의되는지 결정한다. 예를 들어 함수의 매개변수가 함수 안에서만 존재하는 것이다.

```javascript
function f(x) {
  return x + 3;
}

f(5);     // 8
x;        // ReferenceError: x is not defined
```

함수 바디를 벗어나면 x는 존재하지 않는 것처럼 보인다. 그래서 x의 스코프가 함수 f라고 말한다.

## 스코프와 존재

변수가 존재하지 않으면 그 변수는 스코프 안에 있지 않음을 알 수 있다. 아직 선언하지 않은 변수나 함수가 종료되면서 존재하지 않게 된 변수는 분명 스코프안에 있지 않다.

스코프는 프로그램의 현재 실행 중인 부분, `실행 컨텍스트`에서 현재 보이고 접근할 수 있는 식별자를 말한다. 그리고 존재한다는 말은 그 식별자가 메모리가 할당된 무언가를 가리키고 있는 것을 말한다.

무언가가 더는 존재하지 않는다고 해서 자바스크립트는 메모리를 바로 회수하지는 않는다. 그것을 계속 유지할 필요가 없다고 표시해두면, 주기적으로 일어나는 `가비지 콜랙션 프로세스`에서 메모리를 회수한다.

## 정적 스코프 & 동적 스코프

프로그램의 소스 코드를 살펴보는 것은 정적 구조를 살펴보는 것이고, 실제 실행하면 실행 흐름은 움직인다. 자바스크립트의 스코프는 정적이다. 소스 코드만 봐도 변수가 스코프에 있는지 판단할 수 있다.

정적 스코프는 어떤 변수가 함수 스코프 안에 있는지 함수를 정의할 때 알 수 있다. 호출할 때 알 수 있는 것은 아니다.

```javascript
const x = 3;

function f() {
  console.log(x);
  console.log(y);
}

{ // 새 스코프
  const y = 5;
  f();
}
```

변수 x는 함수 f를 정의할 때 존재하지만, y는 다른 스코프에 존재한다. 다른 스코프에 y를 선언하고 그 스코프에서 f를 호출하더라도, x는 그 바디 안에 스코프 안에 있지만 y는 그렇지 않다.

함수 f는 자신이 정의될 때 접근할 수 있었던 스코프는 접근할 수 있지만, 호출시 스코프에 있는 식별자에 접근할 수 없다.

자바스크립트의 정적 스코프는 전역 스코프, 블록 스코프, 함수 스코프에 적용된다.

### 전역 스코프

스코프는 계층적이다. 프로그램을 시작할 때 암시적으로 주어지는 스코프가 필요하다. 이를 전역 스코프라고 한다. 자바스크립트 프로그램을 시작할 때 실행 흐름은 전역 스코프에 있다.

전역 스코프에서 선언된 것을 전역 변수라고 한다. 전역 변수를 많이 쓰는 것은 좋지 않다고 한다.. 전역 스코프에 의존하는 것을 피해야한다. 사용자 정보 같은 것은 단일 객체에 보관하는 것이 낫다.

### 블록 스코프

let, const는 식별자를 블록 스코프에서 선언한다. 블록 스코프는 그 안에서만 보이는 식별자를 의미한다.

```javascript
console.log('before block');
{
  console.log('inside block');
  const x = 3;
  console.log(x);                     // 3
}
console.log(`outside block; x=${x}`); // ReferenceError: x는 정의되지 않았다.
```

x는 블록 안에서 정의되었고, 블록을 나가는 즉시 x도 스코프 밖으로 사라지므로 정의되지 않은 것으로 간주한다.

## 변수 숨기기

다른 스코프에 있으면서 이름이 같은 변수나 상수는 혼란을 초래한다. 스코프 하나가 끝난 다음에 다른 스코프가 있는 식이면 비교적 단순하다.

```javascript
{
  // block 1
  const x = 'blue';
  console.log(x);   // "blue"
}
console.log(typeof x);  // "undefined"; x는 스코프 밖에 있다.
{
  // block 2
  const x = 3;
  console.log(x);       // 3
}
console.log(typeof x);  // "undefined"; x는 스코프 밖에 있다.

{
  // block 1
  const x = 'blue';
  console.log(x);   // "blue"
  {
    // block 2
    const x = 3;
    console.log(x);       // 3
  }
  console.log(x);   // "blue"
}
console.log(typeof x);  // "undefined"; x는 스코프 밖에 있다.
```

내부 블록의 x는 외부 블록에서 정의한 x와 이름만 같을 뿐 다른 변수이므로 외부 스코프의 x를 숨기는 효과가 있다.

실행 흐름이 내부 블록에 들어가 새 변수 x를 정의하는 순간, 두 변수가 모두 스코프 안에 있다. 변수의 이름이 같으므로 외부 스코프에 있는 변수에 접근할 방법이 없다.

## 함수, 클로저, 정적 스코프

함수가 특정 스코프에 접근할 수 있도록 의도적으로 그 스코프에서 정의하는 경우가 많다. 이런 것을 클로저라고 한다. 스코프를 함수 주변으로 좁히는 것이다.

```javascript
let globalFunc;           // 정의되지 않은 전역 함수
{
  let blockVar = 'a';
  globalFunc = function() {
    console.log(blockVar);
  }
}
globalFunc();             // "a"
```

globalFunc는 블록 안에서 값을 할당 받았다. 이 블록 스코프와 그 부모인 전역 스코프가 클로저를 형성한다. globalFunc를 어디서 호출하든 이 함수는 클로서에 들어가있는 식별자에 접근할 수 있다.

globalFunc를 호출하면, 이 함수는 스코프에서 빠져나왔음에도 불구하고 blockVar에 접근할 수 있다. 여기서는 함수를 스코프 안에서 정의하였고, 이 함수는 스코프 밖에서도 참조할 수 있기 때문에 자바스크립트는 스코프를 계속 유지한다.

스코프 안에서 함수를 정의하면 해당 스코프는 더 오래 유지된다. 그리고 일반적으로 접근할 수 없는 것에 접근할 수도 있다.

```javascript
let f;
{
  let o = { note: 'Safe' };
  f = function() {
    return o;
  }
}
let oRef = f();
oRef.note = "Not so safe after all";
```

일반적으로 스코프 바깥에 있는 것들에는 접근할 수 없다. 함수를 정의에 클로저를 만들면 접근할 수 없었던 것들에 접근할 방법이 생긴다.

## 즉시 호출하는 함수 표현식

함수 표현식을 사용하면 즉시 호출하는 함수 표현식(IIFE)을 만들 수 있다. IIFE는 함수를 선언하고 즉시 실행한다.

```javascript
(function() {
  // IIFE 바디
})();
```

IIFE의 장점은 내부에 있는 것들이 모두 자신만의 스코프를 가지지만, IIFE 자체는 함수이므로 그 스코프 밖으로 무언가를 내보낼 수 있다.

```javascript
const message = (function() {
  const secret = "I'm a secret!";
  return `The secret is ${secret.length} characters long.`;
})();
console.log(message);
```

변수 secret은 IIFE의 스코프 안에서 안전하게 보호되어서 외부에서 접근할 수 없다. IIFE는 함수이므로 무엇이든 반환할 수 있다.

## 함수 스코프와 호이스팅

let으로 변수를 선언하면 그 변수는 선언하기 전에는 존재하지 않는다. var로 선언한 변수는 현재 스코프 안이라면 어디서든 사용할 수 있고, 심지어 선언 전에도 사용할 수 있다.

```javascript
let var1;
let var2 = undefined;
var1;                 // undefined
var2;                 // undefined
undefinedVar;         // ReferenceError: undefinedVar는 정의되지 않았다.
```

let을 쓰면, 변수를 선언하기 전 사용하려할 때 에러가 나지만, var로 변수를 선언하면 선언하기 전에도 사용할 수 있다. var로 선언한 변수는 끌어올린다는 뜻의 호이스팅이라는 기능을 가지고 있다. 자바스크립트는 함수나 전역 스코프 전체를 살펴보고 var로 선언한 함수를 맨 위로 끌어올린다. **선언만 끌어올려지고, 할당은 끌어올려지지 않는다.**

```javascript
var x;      // 선언(할당은 아닌)이 끌어올려진다.
x;          // undefined
x = 3;      
x;          // 3
```

## 함수 호이스팅

var로 선언된 변수와 마찬가지로, 함수 선언도 스코프 맨 위로 끌어올려진다. 그래서 함수를 선언하기 전에 호출할 수 있다.

```javascript
f();              // 'f'
function f() {
  console.log('f');
}
```

하지만 변수에 할당한 함수 표현식은 호이스팅되지 않는다. 이는 변수의 스코프 규칙을 그대로 따른다.

```javascript
f();                  // ReferenceError: f는 정의되지 않았다.
let f = function() {
  console.log('f');
}
```

## 사각지대

let으로 선언하는 변수는 선언하기 전까지 존재하지 않는다. 스코프 안에서 변수의 사각지대는 변수가 선언되기 전의 코드이다.

typeof 연산자는 변수가 선언됐는지 알아볼 때 널리 쓰이고, 존재를 확인하는 안전한 방법으로 알려져 있다. let 키워드가 도입되고 변수의 사각지대가 생기기 전까지는 안전하였다. ES6에서는 typeof 연산자로 변수가 정의됐는지 확인할 필요가 거의 없어졌다.

```javascript
if (typeof x === "undefined") {
  console.log("x doesn't exist or is undefined");
} else {
  // x를 사용해도 안전한 코드
}

// let으로 변수 선언하면 안전하지 않다. 오류 발생
if (typeof x === "undefinesd") {
  console.log("x doesn't exist or is undefined");
} else {
  // x를 사용해도 안전한 코드
}
let x = 5;
```

## 스트릭트 모드

ES5 문법에서는 암시적 전역 변수라는 것이 생길 수 있다. 즉, var로 변수를 선언하는 것을 잊으면 자바스크립트는 전역 변수를 참조하려 한다고 간주하고 전역 변수가 없으면 스스로 만들었다.

이런 이유로 자바스크립트는 스트릭트 모드를 도입하였다. 이는 암시적 전역 변수를 허용하지 않는다. 스트릭트 모드를 사용하려면 `"use strict"`라고 이루어진 행을 코드 맨 앞에 쓰면 된다. 전역 스코프에서 사용하면 스크립트 전체가 스트릭트 모드로 실행되고, 함수 안에서 사용하면 해당 함수만 실행된다.
