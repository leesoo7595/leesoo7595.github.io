---
layout: post
title: "Javascript - Callback"
date: 2020-10-26
tags: [JavaScript]
categories:
- Javascript
comments: true
---

## 콜백 기반 비동기 프로그래밍

비동기적으로 수행하는 함수의 경우, 함수 내 동작이 모두 처리된 후 실행되어야하는 함수가 들어갈 **콜백**을 인수로 제공해야한다.

하지만 수행해야할 비동기 함수가 여러 개가 있는 경우, 콜백이 중첩으로 존재해야되기 때문에 이런 방식으로 코드를 작성하는 것은 좋지 않다.

## 에러 핸들링

비동기함수가 비동기적인 로직이 예상대로 동작하지 않을 수 있는데, 이럴 때 에러 핸들링은 필수적이다.

```javascript
function loadScript(src, callback) {
  let script = document.createElement('script');
  script.src = src;

  script.onload = () => callback(null, script);
  script.onerror = () => callback(new Error(`${src}를 불러오는 도중에 에러가 발생했습니다.`));

  document.head.append(script);
}

// 오류 우선 콜백
loadScript('/my/script.js', function(error, script) {
  if (error) {
    // 에러 처리
  } else {
    // 스크립트 로딩이 성공적으로 끝남
  }
});
```

오류 우선 콜백 함수는 첫 번째 인수는 에러처리를 위해 남겨두고, 에러 발생시 첫 번째 인수를 통해 에러 정보를 받을 수 있다.

두 번째 인수를 에러가 발생하지 않았을 경우에 사용할 수 있다. 즉, 에러 없이 성공한 경우 첫 번째 인자가 null이 들어오게 된다.

## 콜백지옥

비동기 동작이 많아지면 콜백에 콜백을 물고 점점 뎁스가 깊어지는 현상을 만나게 된다. 이런 현상을 해결하기 위해 아래와 같이 코드를 짜는 것이 좋다.

```javascript
loadScript('1.js', step1);

function step1(error, script) {
  if (error) {
    handleError(error);
  } else {
    // ...
    loadScript('2.js', step2);
  }
}

function step2(error, script) {
  if (error) {
    handleError(error);
  } else {
    // ...
    loadScript('3.js', step3);
  }
}

function step3(error, script) {
  if (error) {
    handleError(error);
  } else {
    // 모든 스크립트가 로딩되면 다른 동작을 수행합니다. (*)
  }
};
```

첫 비동기함수의 두 번째 인자에 이미 선언한 비동기 함수 이름을 넣어주는 방식으로 각 동작을 분리하여 깊은 중첩이 없도록 할 수 있다. 또한 콜백 기반으로 돌아가게끔 할 수도 있다. 

하지만 이런식으로 코드를 작성하게 되면 깔끔해보이는 대신 가독성이 떨어지게 된다. 

## 프로미스

```javascript
let promise = new Promise(function(resolve, reject) {
  // executor (제작 코드, '가수')
});
```

`new Promise`에 인자로 전달되는 함수는 실행 함수라고 한다. 이 함수는 `new Promise`가 만들어질 때 자동으로 실행된다. 실행 함수의 첫 번째 인자인 `resolve`, 두 번째 인자인 `reject`는 자바스크립트에서 자체적으로 제공하는 **콜백 함수**이다. 개발자는 `new Promise` 안에 `resolve`, `reject`를 신경쓰지않고 함수를 만들면 된다.

대신 실행 함수에서 자바스크립트 자체적으로 받아오는 콜백 중 하나를 호출하여야한다. 첫 번째 인자인 `resolve`의 경우, `new Promise`로 넘어간 실행 함수가 성공적으로 끝났을 때를 호출한다. `reject`는 에러 발생시 에러 객체를 나타내는 `error`와 함께 호출된다.

즉, `new Promise`로 넘긴 실행 함수는 처리가 끝나면 성공실패에 따라 첫 번째 인자나 두 번째 인자의 콜백이 호출된다.

### promise 객체

* state
  - pending
  - fulfilled (resolve)
  - rejected (reject)

* result
  - undefined
  - value (resolve)
  - error (reject)