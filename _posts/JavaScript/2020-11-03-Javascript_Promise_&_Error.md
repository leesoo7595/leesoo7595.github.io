---
layout: post
title: "Javascript - Promise & Error Handling"
date: 2020-11-03
tags: [JavaScript]
categories:
- Javascript
comments: true
---

## 프로미스와 에러 핸들링

프로미스에서 에러가 발생하면 reject() 상태로 넘어가는데, 프로미스에서 `catch`문이 없음으로인해 처리해주지 못하게되면, 에러가 갇혀버리는 형태가 된다. 에러를 처리할 코드가 없기 때문이다.

이런 경우 스크립트가 죽고 콘솔 창에 메세지가 출력되게 되는데, 자바스크립트 엔진의 경우에는 에러가 위와 같이 처리가 되지못할 경우 전역 에러를 생성한다. 

![image](https://user-images.githubusercontent.com/39291812/98008011-65823a00-1e37-11eb-8457-3a3425e7ee5c.png)

브라우저 환경에서는 `unhandledrejection` 이벤트로 `catch`가 없는 프로미스 에러를 잡을 수 있다. 이 이벤트는 에러 정보가 담긴 이벤트 객체를 받아서 이 이벤트 핸들러로 원하는 작업을 할 수 있다.

```javascript
window.addEventListener('unhandledrejection', function(event) {
  // 이벤트엔 두 개의 특별 프로퍼티가 있습니다.
  alert(event.promise); // [object Promise] - 에러를 생성하는 프라미스
  alert(event.reason); // Error: 에러 발생! - 처리하지 못한 에러 객체
});

new Promise(function() {
  throw new Error("에러 발생!");
}); // 에러 처리 핸들러, catch가 없음
```

## 프로미스 API

### Promise.all

프로미스의 정적 메소드로, 여러 개의 프로미스를 동시에 실행시키고 모든 프로미스가 준비될 때까지 기다린다.

이 메소드는 프로미스로 이루어진 배열(이터러블 객체)를 받고 새로운 프로미스를 반환한다.

**Promise.all에 전달되는 프로미스 중 하나라도 Reject이 된다면, Promise.all이 반환하는 프로미스는 에러와 함께 Reject이 된다.**

```javascript
Promise.all([
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
  new Promise((resolve, reject) => setTimeout(() => reject(new Error("에러 발생!")), 2000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
]).catch(alert); // Error: 에러 발생!
```

## Reference

* [모던자바스크립트 - 프로미스 API](https://ko.javascript.info/promise-api)