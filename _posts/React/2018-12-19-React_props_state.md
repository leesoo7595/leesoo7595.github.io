---
layout: post
title: "[React] State & Props"
date: 2018-12-19
tags: [React]
comments: true
---

React Component에서 다루는 데이터는 두 개로 나뉜다. props와 state이다.

props는 부모 컴포넌트가 자식 컴포넌트에게 주는 값이고, 자식 컴포넌트는 props를 받아오지만 이를 직접 수정할 수 없다.

하지만 state는 컴포넌트 내부에서 선언하며 내부에서 값을 변경할 수 있다.

## 메소드

컴포넌트에 메소드를 작성할 때,  컴포넌트에서 매소드는 두 가지 방법으로 작성해줄 수가 있다.

```javascript
handleIncrease = () => {
  this.setState({
    number: this.state.number + 1
  });
}

handleDecrease = () => {
  this.setState({
    number: this.state.number - 1
  });
}
```
이 방법과

```javascript
handleIncrease() {
 this.setState({
   number: this.state.number + 1
 });  
}

handleDecrease() {
  this.setState({
    number: this.state.number - 1
  });
}
```
위와 같은 방법이 있다.

그런데 두 번째 방법으로 하면, 클릭이벤트가 발생했을 때 `this`가 `undefined`로 나타나서 제대로 처리가 되지 않는다. 그 이유는 함수가 클릭이벤트로 전달되는 과정에서 `this`와의 연결이 끊키기 때문이다.

이런 문제는 다음과 같이 해결한다.

```JavaScript
constructor(props) {
  super(props);
  this.handleIncrease = this.handleIncrease.bind(this);
  this.handleDecrease = this.handleDecrease.bind(this);  
}
```

이렇게 쓰거나, 맨처음 방법처럼 메소드를 작성한다면 this가 풀리지 않는다.
