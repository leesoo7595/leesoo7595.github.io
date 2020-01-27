---
layout: post
title: "[ES6] Promise"
date: 2020-01-26
tags: [JavaScript]
comments: true
---

# Promise

자바스크립트 비동기 처리를 위한 콜백 함수가 있다. 그런데 이 콜백 함수는 가독성이 좋지 않고 예외 처리 또한 힘들어서 여러 개의 비동기 함수를 한꺼번에 처리하는 것이 힘들다. 그래서 ES6에서 Promise가 나왔다. 

## 콜백 지옥

### 동기식 처리 모델

동기식 처리 모델은 태스크가 순차적으로 진행이 된다. 어떤 작업이 수행 중일 경우 다음 태스크는 대기하게 된다. 

즉, 서버에서 데이터를 가져와서 화면에 표현해줄 때, 서버에 요청하고 응답을 받을 때까지 이후 태스크들은 블로킹 된다.

### 비동기식 처리 모델

비동기식 처리 모델은 태스크를 병렬적으로 수행한다. 태스크가 수행중이더라도 그 다음 태스크를 실행한다. 이것을 Non-Blocking 상태라고 한다.

자바스크립트의 비동기식 처리 모델은 요청을 병렬로 처리하여 Non-Blocking이 된다. 콜백 패턴을 사용하면 콜백 패턴 후의 처리를 하기 위해 그 안에 콜백함수가 네스팅된다. 즉, 비동기 함수의 처리 결과를 가지고 다른 비동기 함수를 호출해야하는 경우일 때, 함수가 네스팅된다고 한다. 이것을 콜백 지옥이라 하는데, 콜백 지옥은 가독성을 나쁘게 하고, 실수를 만드는 원인이 된다.

콜백 지옥이 발생하는 이유는, 비동기 처리 로직이 실행되면 그 다음 로직은 비동기 처리 로직이 완료되기까지 기다리지 않고 실행하기 때문에, 코드 자체는 동작이 달리 움직일 수 있다.

## 에러 처리의 한계

콜백 함수는 에러 처리가 힘들다. 비동기 함수가 실행될 경우, 그 다음 로직이 비동기 함수가 실행이 끝날 때까지 기다리지 않고 바로 실행되므로 try catch 블럭에서 캐치되지 않는다. 

그 이유는 try catch 블럭에서 에러를 감지하기 위해서는 비동기 함수에서 뱉는 콜백 함수를 호출한 것은 해당 비동기 함수가 콜백 함수를 호출한 것이 아니기 때문이다. 실행 컨텍스트에서 스택으로 쌓이는 함수 기준으로 try catch 블럭에서 에러를 캐치할 수 밖에 없기 때문에, 이벤트 루프로 인해 다른 태스크 큐로 옮겨가는 콜백 함수의 경우 try catch에서 감지할 수 있는 에러의 범위를 넘어서기 때문이다.

이러한 문제를 극복하기 위해 Promise가 나오게 되었다. 현재 Promise는 IE를 제외한 대부분의 브라우저가 지원하고 있다.

## new Promise

```javascript
const promise = new Promise((resolve, reject) => {
    if (somethingIsSuccess) {
        resolve('result');
    }
    else (somethingIsFail) {
        reject('failure reason');
    }
});
```

promise는 비동기 처리가 성공(fulfilled)하였는지, 실패(rejected)하였는지 state 정보를 갖는다. Promise 생성자 함수를 인자로 전달받은 콜백 함수는 내부에서 비동기 작업을 수행한다. 

Promise 객체를 통해서 비동기 처리에 성공하면 resolve 메소드의 인자로 비동기 처리 결과를 전달한다. 실패하면 reject 메소드를 통해 에러 메세지를 전달한다.

## Promise 이후 처리 메소드

Promise로 구현된 비동기 함수는 Promise 객체를 반환한다. Promise로 구현된 비동기 함수를 호출하는 이벤트 루프는 Promise의 후처리 메소드인 then, catch를 통해 비동기의 처리 결과나 에러 메세지를 전달받아 처리한다. Promise 객체는 상태를 가지는데, 이 상태에 따라 then, catch의 체이닝 방식으로 호출한다.

### then

then은 두 개의 콜백 함수를 인자로 전달 받는다. 여기서 then은 Promise를 반환한다.

1. fulfulled 상태 즉, resolve 함수가 호출된 상태시 호출되는 인자
2. rejected 상태 즉, reject 함수가 호출된 상태시 호출된다.

### catch

비동기 처리에서 발생한 에러나, then에서 발생한 에러가 나오면 호출된다. catch도 마찬가지로 Promise를 반환한다.

## Promise 에러 처리

비동기 로직을 처리할 때, 발생한 에러 메세지는 then 메소드의 두 번째 파라미터에 콜백 함수로 들어간다. catch 메소드 또한 에러를 처리한다는 점에서 then의 두번째 파라미터의 역할과 비슷하다. 차이점은, then 메소드의 두 번째 콜백 함수는 비동기 처리에서 발생한 에러만을 캐치하게 된다. 반대로 catch 메소드는 비동기 처리에서 발생한 에러와 then 메소드에서 발생한 에러까지 캐치할 수 있다. 

**그래서 Promise의 에러 처리는 catch 메소드를 사용하는 것이 더욱 효율적이다.**

## Promise Chaining

프로미스 체이닝이란, 비동기 함수의 처리 결과를 가지고 다른 비동기 함수를 호출해야할 때, 함수 호출이 nesting되어 복잡도가 높아지게 되는데(콜백 지옥), Promise를 사용하게 될 경우, `then`이나 `catch`를 이용해서 여러 개의 Promise를 이용하여 사용할 수 있다. 이를 통해서 콜백 지옥 문제를 해결할 수 있다.

## staticMethod of Promise

Promise 객체는 4가지 정적 메소드가 있다.

### Promise.resolve / reject

해당 정적 메소드는 특정 값을 Promise로 래핑하기 위해 사용하는 메소드이다.

```javascript
const resolvedPromise = Promise.resolve([1, 2, 4]);
resolvedPromise.then(console.log);

// 위 로직과 똑같이 동작한다.
const resolvedPromise = new Promise(resolve => resolve([1, 2, 4]));
resolvedPromise.then(console.log);
```

### Promise.all

`Promise.all` 메소드는 Promise가 담겨있는 배열 등의 **이터러블**을 인자로 전달 받는다. 그리고 전달받은 모든 Promise를 병렬로 처리하고, 그 결과를 resolve하는 새로운 Promise를 반환하도록 한다.

`Promise.all`은 배열로 전달받은 Promise를 병렬로 처리하기 때문에, 모든 Promise의 처리가 끝날 때까지 대기한 후에 모든 처리 결과를 resolve, reject 한다.

여기서 모든 Promise의 처리 결과가 resolve한 처리 결과를 배열에 담아 다시 resolve하는 새로운 Promise를 반환하는데, 첫 번째 Promise가 가장 처리 결과가 늦어도 반환된 Promise에서는 순서대로 처리 결과가 나온다. **즉, 처리 순서가 보장된다.**

반대로 Promise의 처리가 하나라도 실패하면 가장 먼저 실패한 Promise가 reject한 에러를 reject 메소드에 Promise로 반환하게 된다.

```javascript
Promise.all([
  new Promise(resolve => setTimeout(() => resolve(1), 3000)), // 1
  new Promise(resolve => setTimeout(() => resolve(2), 2000)), // 2
  new Promise(resolve => setTimeout(() => resolve(3), 1000))  // 3
]).then(console.log) // [ 1, 2, 3 ]
  .catch(console.log);

Promise.all([
  new Promise((resolve, reject) => setTimeout(() => reject(new Error('Error 1!')), 3000)),
  new Promise((resolve, reject) => setTimeout(() => reject(new Error('Error 2!')), 2000)),
  new Promise((resolve, reject) => setTimeout(() => reject(new Error('Error 3!')), 1000))
]).then(console.log)
  .catch(console.log); // Error: Error 3!
```

Promise.all 메소드에는 전달받는 요소가 이터러블이 아닐 경우에, `Promise.resolve`를 통해서 Promise로 래핑된다.

### Promise.race

`Promise.race` 메소드는 `Promise.all` 메소드와 동일하게 Promise가 담겨있는 이터러블을 인자로 전달 받는다. 그런데, `Promise.race`는 모든 Promise를 병렬 처리하지 않고, 가장 먼저 처리된 프로미스가 resolve한 처리 결과를 resolve하는 새로운 Promise를 반환한다.

```javascript
Promise.race([
  new Promise(resolve => setTimeout(() => resolve(1), 3000)), // 1
  new Promise(resolve => setTimeout(() => resolve(2), 2000)), // 2
  new Promise(resolve => setTimeout(() => resolve(3), 1000))  // 3
]).then(console.log) // 3
  .catch(console.log);
```

반대로 에러가 발생한 경우엔, `Promise.all` 메소드와 동일하게 처리된다.

## 참고

* [promise - poiemaweb](https://poiemaweb.com/es6-promise)
