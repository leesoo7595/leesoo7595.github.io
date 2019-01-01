---
layout: post
title: "[React] Functional Component"
date: 2019-01-01
tags: [React]
comments: true
---

참고 : https://velopert.com/2994

React에서 함수 형태로 컴포넌트를 정의하는 방법

```javascript
import React, { Component } from 'react';

class Hello extends React.Component {
  render() {
    return (
      <div>Hello {this.props.name}</div>
    );
  }
}

export default Hello;
```
React에서 컴포넌트를 정의할 때, liftCycle API나 state를 사용해야하는 경우에는 위와 같이 정의하여야 한다.

하지만 만약 props만 전달할 경우 뷰를 렌더링만 해주는 역할이라면, **함수형 컴포넌트** 형식으로 컴포넌트를 정의할 수 있다.

```javascript
import React from 'react';

function Hello(props) {
  return (
    <div>Hello {props.name}</div>
  );
}

// ES6 화살표 함수
import React from 'react';

const Hello = (props) => {
  return (
    <div>Hello {props.name}</div>
  );
}

export default Hello;
```

위 코드를 **비구조화 할당 (Object Destructuring)** 문법을 사용하면 다음과 같이 바꿀 수 있다.

```javascript
import React from 'react';

const Hello = ({name}) => {
  return (
    <div>Hello {name}</div>  
  );
}
```
