---
layout: post
title: "[React] Tutorials - 2st week"
date: 2019-11-11
tags: [React]
comments: true
---

# React 2주차

* Event
* LifeCycle

## Event

**이벤트란?**
웹 브라우저가 알려주는 사건의 발생을 의미한다. || 특정 대상에게 가하는 어떤 행동이다.
ex) 마우스 클릭, 터치를 시작, ESC 키를 누르다 등

### Event handler와 Event listener


* Event Listener

특정 이벤트에 대해서 일어날 동작을 정의
지정된 타입의 이벤트가 특정 요소에서 발생하면, 웹 브라우저에서 그 요소에 등록된 해당 이벤트 리스너를 실행시킨다.
이벤트 리스너 + 이벤트 핸들러를 통틀어 이벤트 리스너라고 하기도함

* Event Handler

이벤트에 대한 다음 처리 과정을 의미

**이벤트가 발생하고 그 후 동작을 처리하기위한 이벤트 핸들러를 만들 경우, 발생한 해당 이벤트가 항상 이벤트핸들러 함수의 파라미터로 이벤트 객체를 갖게 된다!**

ex) 버튼을 누렀다 -> alert('눌렀다!') 띄우기 (**Event Handler**)

* event.target

`event.target`으로 이벤트 객체를 받아올 수 있다.

 // e.target 배운것을 활용하여 만들어보는 Input!

### React의 이벤트 처리

* React의 Event 처리는 소문자 대신 CamelCase를 사용하여 처리한다.
* JSX 문법을 사용하여 문자열이 아닌 함수를 통해 이벤트를 전달한다.

돔의 직접적인 접근을 활용해서 이벤트 처리를 해본적이 있어?

```javascript
<button onclick="activateLasers()">
  Activate Lasers
</button>
```

위의 예시는 돔의 직접적인 접근, 즉 HTML 상에서 바로 이벤트 처리를 해줄 때!
React는 좀 달라

```javascript
<button onClick={activateLasers}>
  Activate Lasers
</button>
```

## LifeCycle

<img width="769" alt="Screen Shot 2019-11-16 at 4 04 45 PM" src="https://user-images.githubusercontent.com/39291812/68989560-01641000-088c-11ea-8c35-d5e66a0ee556.png">

```javascript
class Content extends React.Component {
  constructor(props) {
    super(props);
  }

  // deprecated
  componentWillMount() {
    console.log('Component WILL MOUNT!')
  }
  componentDidMount() {
    console.log('Component DID MOUNT!')
  }
  // deprecated
  componentWillReceiveProps(nextProps) {    
    console.log('Component WILL RECIEVE PROPS!')
  }
  shouldComponentUpdate(nextProps, nextState) {
    return true;
  }
  // deprecated
  componentWillUpdate(nextProps, nextState) {
    console.log('Component WILL UPDATE!');
  }
  componentDidUpdate(prevProps, prevState) {
    console.log('Component DID UPDATE!')
  }
  componentWillUnmount() {
    console.log('Component WILL UNMOUNT!')
  }
  render() {
    return (
        <div>
          <h1>{this.props.sentDigit}</h1>
        </div>
    );
  }
}
```

<img width="725" alt="Screen Shot 2019-11-15 at 1 29 33 AM" src="https://user-images.githubusercontent.com/39291812/68876169-6530f080-0747-11ea-8614-f32abefc6073.png">

위 사진은 React의 LifeCyle에 관련된 메소드 정리를 해놓은 것들이야.

위 그림에 써져있는 용어들을 이해하고 있고, 제대로 활용할 줄 알아야 React를 잘 쓴다라고 할 수 있을 것 같아.

* Mount

초기 렌더링이 된 경우를 Mount가 되었다라고 한다.

* Update

초기 렌더링이 된 이후 컴포넌트가 변경이 되어 재렌더링이 될 때 Update한다라고 한다.

* Unmount

컴포넌트를 제거하게 될 때, Unmount한다라고 한다.

### render()

컴포넌트 렌더링 담당!

렌더링은 DOM을 그려준다라는 의미로 보면 될듯!

### componentDidMount

컴포넌트가 만들어지고, `render()`이 호출된 이후에 나타나는 메소드이다.
즉, 첫 렌더링 이후에 할 작업이 있을 때 활용할 수 있다.

```javascript
componentDidMount(){
  console.log("componentDidMount");
}
```

### getDerivedStateFromProps

```javascript
static getDerivedStateFromProps(nextProps, prevState) {
  // 여기서는 setState 를 하는 것이 아니라
  // 특정 props 가 바뀔 때 설정하고 설정하고 싶은 state 값을 리턴하는 형태로
  // 사용됩니다.
  /*
  if (nextProps.value !== prevState.value) {
    return { value: nextProps.value };
  }
  return null; // null 을 리턴하면 따로 업데이트 할 것은 없다라는 의미
  */
}
```

`componentWillReceiveProps`의 대체 메소드로, 컴포넌트가 인스턴스화 된 후, **새 props**를 받았을 때 호출되는 경우 쓰이는 메소드이다.

쉽게 말하면, 초기 렌더링이 된 이후에 새 props를 받아 update 되기 직전에 쓰는 메소드!!!

setState를 쓸 수 없고 값으로만 업데이트한다. 

### shouldComponentUpdate

```javascript
shouldComponentUpdate(nextProps, nextState) {
  // return false 하면 업데이트를 안함
  // return this.props.checked !== nextProps.checked
  return true;
}
```

컴포넌트 업데이트 직전에 호출되는 매소드이다. `props` 또는 `state`가 변경되었을 때, 재랜더링 여부를 `return`으로 결정한다. 컴포넌트 최적화 작업에 매우 유용하다.

### getSnapshotBeforeUpdate

**해당 메소드가 호출되는 시점 순서**

1. render()
2. getSnapshotBeforeUpdate()
3. 실제 DOM 변화
4. componentDidUpdate()

`componentWillUpdate`의 대체 역할로 만들어진 메소드이다.

DOM이 업데이트되기 직전에 호출되는 메소드이다. 잘 쓰이지 않는대용..

### componentDidUpdate

컴포넌트 업데이트 직후에 호출되는 메소드이다.

### componentWillUnmount

컴포넌트가 소멸된 시점에(DOM에서 삭제된 이후) 실행되는 메소드이다. 컴포넌트 내부에서 타이머나, 비동기 API를 사용하고 있을 때 이를 제거하기 위해 사용된다.

### componentDidCatch

render 함수에서 에러가 발생하면, 리액트앱이 깨져버리는데, 그럴때 깨지지않도록 에러핸들링을 도와주는 메소드이다.

```javascript
componentDidCatch(error, info) {
  this.setState({
    error: true
  });
}
```

에러가 발생하면 state.error 값을 true로 설정하고, render 함수에서 커스텀한 에러창을 띄우도록 하면 된닷..
