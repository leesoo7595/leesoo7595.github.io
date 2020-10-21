---
layout: post
title: "Javascript - Error Handling"
date: 2020-10-11
tags: [JavaScript]
categories:
- Javascript
comments: true
---

## try catch

```javascript
try {

} catch (err) {

}
```

`try catch` 동작 순서

1. `try { ... }` 안의 코드가 실행된다.
2. 에러가 없으면 `catch` 블록은 건너뛰게 된다.
3. 에러가 있으면, `try` 코드 실행이 중단되고 `catch(err)` 블록으로 코드 실행이 넘어가서 변수 `err`에 어떤 에러가 발생했는지에 대한 에러 객체를 담긴 상태로 실행된다.

이렇게 되면, `try {...}` 블록 안에서 에러가 발생하여도 `catch` 에서 에러를 처리하기 때문에 스크립트가 멈추지 않는다.

* `try...catch`는 런타임 에러에서만 동작한다.
* `try...catch`는 동기적으로 동작한다.

setTimeout 같은 비동기적인 코드에서는 `try...catch`로 잡아낼 수 없다. 그래서 비동기적인 코드의 예외를 잡으려면 비동기적인 코드 내부에서 `try...catch`를 구현하여야 한다.

## 에러 객체

자바스크립트는 에러가 발생하면 에러 내용이 담긴 객체를 생성한다. 에러 객체에는 `name`, `message` 프로퍼티가 주로 존재하고 있다.

정확히는 `Error.prototype`으로 존재하고 있다고 한다. 그래서 기본적으로 `constructor`(에러 객체의 프로토타입을 담당하는 생성자 함수)도 가지고 있다.

`constructor`, `name`, `message` 프로퍼티들은 모두 표준 프로퍼티라고 하는데, 여러 브라우저에서 모두 통틀어 가지고 있는 프로퍼티를 표준 프로퍼티라고 하는 것 같다. 즉, 특수한 프로퍼티가 존재하다는 의미인다. 특정 환경(노드, 파이어폭스, 크롬, 엣지, ie10+, 오페라, 사파리6+)에서는 `stack`이라는 비표준 프로퍼티를 가진다. 이 프로퍼티는 에러의 스택 트레이스를 포함한다. 다른 말로 하면 에러를 유발한 중첩 호출들의 순서 정보를 가지고 있는 문자열을 가지고 있으며, 디버깅 목적으로 사용된다.

**stack과 같은 비표준 프로퍼티는 크로스 브라우징 같은 이슈에 문제가 될 수 있으므로, 프로덕션 모드에서는 사용하지 않는 것이 좋다. 모든 사용자들에게 동작하지 않을 것이기 때문이다.**

## catch의 에러 객체 바인딩 유무

최근에 추가된 문법이다.

`try...catch`를 사용할 때, 에러에 대한 정보가 필요하지 않을 경우 `catch`에서 생략이 가능하다.

## throw 연산자

throw 연산자는 에러를 생성한다.

```javascript
throw <error object>
```

throw 뒤에는 에러 객체를 넣는 식으로 문법이 되어있는데, 에러 객체로는 어떤 자료형이라도 넣을 수 있지만, 보통 내장 에러엔 기본적으로 `name`, `message` 프로퍼티를 포함하므로 이 형식을 지키는 것이 좋다.

자바스크립트에서 기본적인 에러 객체를 만들어 낼 수 있는 7가지 에러 생성자가 존재한다. 이 생성자들을 활용하여 에러 객체를 만들 수 있다.

`try...catch`에서 `try` 구문 안에 인위적으로 에러를 생성하면, `catch`문에서 처리할 수 있게 활용할 수 있다.

```javascript
let json = '{ "age": 30 }'; // 불완전한 데이터

try {

  let user = JSON.parse(json); // <-- 에러 없음

  if (!user.name) {
    throw new SyntaxError("불완전한 데이터: 이름 없음"); // (*)
  }

  alert( user.name );

} catch(e) {
  alert( "JSON Error: " + e.message ); // JSON Error: 불완전한 데이터: 이름 없음
}
```

**catch 문에서 나오는 에러는 개발자가 알고 있는 에러여야 한다. 다른 모든 에러는 다시 throw하도록 하여야 한다.**

예상치 못한 에러가 발생하였는데 그것을 `catch`문으로 잡아버리면 디버깅이 어려워진다. 그렇게 되면 밖에서 한 번 더 예상치 못한 에러를 잡아서 에러 객체를 읽을 수 있도록 처리하면 좋다...?

throw 연산자는 위에서 말했듯이 오류 객체 외에도 다른 원시형 자료도 들어갈 수 있다. 하지만 에러 객체가 아닌 다른 자료형으로 전달하였을 때 Error 객체로 사용함으로써 생기는 유용한 속성을 통한 자세한 정보를 알기 어렵다. 그 자세한 정보들은 뭐가 있을까?

### v8의 stack 속성

```javascript

// error.js
var err = new Error();
console.log(typeof err.stack);
console.log(err.stack);

∞ node error.js
string
Error
    at Object.<anonymous> (/private/tmp/error.js:2:11)
    at Module._compile (module.js:411:26)
    at Object..js (module.js:417:10)
    at Module.load (module.js:343:31)
    at Function._load (module.js:302:12)
    at Array.0 (module.js:430:10)
    at EventEmitter._tickCallback (node.js:126:26)
```

위처럼 에러 객체에 존재하는 stack을 사용했을 때, v8의 경우 객체가 어디에서 생성되고 전달되었는지 정확히 내역을 볼 수 있게 된다. v8은 심지어 스택 트레이스의 크기를 `Error.stackTraceLimit`을 통해 제한 값을 변경할 수도 있다.

### 커스텀 에러와 스택 트레이즈 조작

스택 트레이스에서 쌓이는 기록을 차단하기위해(불필요한 정보를 지우기) 커스텀 에러 객체를 만들 수 있다. 

```javascript
function MongooseError (msg) {
  Error.call(this);
  Error.captureStackTrace(this, arguments.callee);
  this.message = msg;
  this.name = 'MongooseError';
};

MongooseError.prototype.__proto__ = Error.prototype;
```

위 커스텀 에러는 MongooseError 생성자 호출을 스택에서 감추어서 불필요한 정보를 없애기 위해 captureStackTrace에 두 번째 아규먼트를 전달했다.

`Error.captureStackTrace` 함수는 첫 번쨰 인자로는 object를 받고, 두 번째 인자로는 function을 선택적으로 넘긴다. 이 함수는 현재의 스택 트레이스를 캡쳐하고 대상 객체 안에 stack 프로퍼티를 만들어 저장하는 역할을 한다. 두 번째 인자가 있다면 해당 함수는 호출 스택의 끝점으로 간주되어. 스택 트레이스는 콜백 함수가 호출되기 전에 발생한 호출만 표시하게 된다.

간단하게 말하면, `Error.captureStackTrace`에 두번째 인자가 없다면 그 전에 실행되었던 함수들과 해당 `Error.captureStackTrace`를 호출했던 함수를 마지막으로 찍히게되지만, 두 번째 인자에 함수를 넘기게 되면 해당 함수를 호출한 함수까지만 스택 트레이스에 보이게 되고 그 위의 모든 실행된 함수들은 숨기도록 된다. 이런식으로 스택 트레이스를 조작하여 에러 객체에서 남는 기록들을 조절할 수 있다.

## try...catch...finally

```javascript
try {
   ... 코드를 실행 ...
} catch(e) {
   ... 에러 핸들링 ...
} finally {
   ... 항상 실행 ...
}
```

해당 블록 안의 변수들은 지역 변수이다. 그래서 밖에서 영향을 받을 변수가 있다면 `try...catch` 밖에서 선언을 해주어야할 것이다.

`finally` 절은 항상 실행된다. `try...catch` 안에서 `return`을 사용하여도 `finally` 절은 실행된다.

`try...finally` 구문도 존재한다. 이는 에러 처리를 할 일이 없지만, 시작한 일이 마무리되었는지 확인할 때 쓰인다. 대신 `catch`가 없어서 `try` 안에서 에러가 발생하면 스크립트가 죽을 것이다. **`finally`절은 스크립트가 죽더라도 완료된다.**

### 만약 try...catch 바깥에서 에러가 발생하여 스크립트가 죽었다면?

브라우저 환경에서는 `window.onerror`라는 함수를 사용하여 에러를 처리할 수가 있다. 해당 함수의 프로퍼티에 함수를 할당할 수 있는데, 예상치 못한 에러가 발생하면 할당한 함수가 실행된다.

```javascript
window.onerror = function(message, url, line, col, error) {
  // ...
  // message : 에러메세제
  // url : 에러가 발생한 스크립트의 URL
  // line, col : 에러가 발생한 곳의 줄과 열 번호
  // error : 에러 객체
};
```

## Reference

* ['try..catch'와 에러 핸들링](https://ko.javascript.info/try-catch#ref-365)
* [문자열은 Error가 아닙니다.](https://blog.outsider.ne.kr/722)
* [자바스크립트 에러와 스택 트레이스 심화](https://ui.toast.com/weekly-pick/ko_20170306/)
* [Error types](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error)