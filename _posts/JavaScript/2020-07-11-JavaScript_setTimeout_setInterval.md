---
layout: post
title: "Javascript - setTimeout & setInterval"
date: 2020-07-11
tags: [JavaScript]
categories:
- Javascript
comments: true
---

일정 시간이 지난 후에 함수를 실행할 수 있게끔 동작하는 것을 **호출 스케줄링**이라고 한다.

호출 스케줄링을 구현할 때 사용하는 Web API는 두 가지가 있다.

* setTimeout
* setInterval

자바스크립트 명세서에는 위 함수에 대한 설명이 없다. 하지만 브라우저, Node.js 등 자바스크립트 호스트 환경에서는 이같은 스케줄러를 지원한다.

## setTimeout

```javascript
let timerId = setTimeout(func|code, [delay], [arg1], [arg2], ...)
```

`setTimeout` 함수의 첫 번째 인자로는 함수나 문자열 형태를 받을 수 있는데, 주로 함수를 넘겨주어 사용한다. 두 번째 인자로는 실행 전 대기 시간이다.

세 번째 인자부터는 함수에 전달할 인자들인데, IE9 이하에서는 지원하지 않는다.

```javascript
function sayHi() {
  alert('안녕하세요.');
}

setTimeout(sayHi, 1000);

function sayHi(who, phrase) {
  alert( who + ' 님, ' + phrase );
}

setTimeout(sayHi, 1000, "홍길동", "안녕하세요."); // 홍길동 님, 안녕하세요.

setTimeout(() => alert('안녕하세요.'), 1000);
```

`setTimeout` 함수를 사용할 때, 넘기는 함수는 실행시키는 상태가 아닌 참조 상태로 넘겨야한다. 실행시키는 상태로 전달할 경우, 함수 실행 결과가 전달된다. 즉, 리턴값이 전달되는데, 리턴값이 없을 경우 `setTimeout`은 스케줄링 대상을 찾지 못하기 때문에 코드가 다르게 동작할 것이다.

### clearTimeout으로 스케줄링 취소하기

setTimeout을 호출하게되면 타이머 식별자(`timer identifier`)가 반환되는데, 스케줄링을 취소하고 싶을 경우에는 setTimeout을 할당한 변수를 사용하면 된다.

```javascript
let timerId = setTimeout(...);
clearTimeout(timerId);
```

## setInterval

`setInterval` 함수는 `setTimeout` 함수와 동일한 문법을 가지고 있다.

`setTimeout`과 `setInterval` 차이는, `setInterval`은 함수를 주기적으로 실행하게 하는 것이다. 함수 호출을 중단하려면 `clearInterval(setInterval할당변수)`를 사용한다.

대부분의 브라우저는 alert창이 실행중에 있어도 내부적으로 타이머를 멈추지 않는다.

```javascript
// 2초 간격으로 메시지를 보여줌
let timerId = setInterval(() => alert('째깍'), 2000);

// 5초 후에 정지
setTimeout(() => { clearInterval(timerId); alert('정지'); }, 5000);
alert
```

## 중첩 setTimeout

`setInterval`을 사용하는 방법과, `setTimeout`을 중첩으로 두고 사용하는 방법이 있다.

중첩 `setTimeout`을 이용하는 방법은 `setInterval`을 사용하는 방법보다 유연하다. 그 이유는 호출 결과에 따라서 다음 호출을 조절할 수 있기 때문이다.

`setTimeout` 함수 호출을 시킬 때 서버 요청을 한다고 할 때, 서버가 과부하 상태일 경우 요청 시간을 증가시킬 수 있다.

마찬가지로 CPU 소모가 많은 작업을 주기적으로 실행하게 할 경우, 작업이 걸리는 시간에 따라서 다음 작업을 유동적으로 계획할 수 있다.

**즉, 중첩 `setTimeout`을 이용하는 방법은 지연간격을 보장하지만, `setInterval`은 이를 보장하지 않는다.**

```javascript
let i = 1;
setInterval(function() {
  func(i++);
}, 100);

let i = 1;
setTimeout(function run() {
  func(i++);
  setTimeout(run, 100);
}, 100);
```

`setInterval`을 사용하는 경우. 내부 `func` 함수는 100밀리초마다 실행한다. 

![image](https://user-images.githubusercontent.com/39291812/87219964-c4a8d400-c39a-11ea-8343-d36b317f12a6.png)

위 그림을 보면 알겠지만, `setInterval`을 사용하면 func 함수 호출 사이의 지연 간격이 실제 명시한 100밀리초보다 짧아지게 된다. 그러면 func 함수를 실행하는 데 걸리는 시간이 명시한 지연 간격보다 길게 되는 경우, 엔진이 func의 실행이 종료될 때까지 기다려주고 다음 호출을 시작한다.

<img width="603" alt="스크린샷 2020-07-11 오후 5 28 51" src="https://user-images.githubusercontent.com/39291812/87220086-ff5f3c00-c39b-11ea-8592-575e42590d83.png">

위 그림에서처럼 `setInterval`과는 달리 중첩 `setTimeout` 함수를 실행한 경우, 100밀리초라는 명시한 지연이 보장된다. 그 이유는 이전 함수의 실행이 종료된 이후에 다음 함수 호출이 정해지기 때문이다.
  
### 가바지 컬렉션과 setInterval, setTimeout

`setInterval`, `setTimeout`에 함수를 넣어 사용하게될 경우, 함수에 대한 내부 참조가 새롭게 만들어지고 이 참조 정보는 스케줄러에 저장된다. 그래서 해당 함수를 참조하는 것이 없어도 가바지 컬렉션의 대상이 되지 않는다.

`setInterval`의 경우, `clearInterval`이 호출되기 전까지 함수의 참조 정보가 메모리에 유지된다. 또한 외부 렉시컬 환경을 참조하는 함수가 있어서, 위 api에 걸려있는 함수 참조가 남아있는 경우엔 외부 변수에도 메모리에 남아있게 되어 더 많은 메모리 공간이 사용되게 된다. 

그러므로 스케줄링할 필요가 없어진 함수는 꼭 clear해줘야한다.


## Reference

- [모던자스 - setTimeout & setInterval](https://ko.javascript.info/settimeout-setinterval)

