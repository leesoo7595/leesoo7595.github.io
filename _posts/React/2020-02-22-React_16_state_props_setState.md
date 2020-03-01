---
layout: post
title: "[React] state & props"
date: 2020-03-01
tags: [React]
categories:
- React
comments: true
---

해당 글은 실전 **리액트 프로그래밍 3장**을 읽고 정리한 글 입니다.

## 리액트 헷갈리는 개념 다시 잡기

### state & props

`state` 값을 변경하려면 무조건 `setState`를 거쳐서 업데이트를 해주어야한다. `setState` 메소드가 호출되면 `state` 값을 변경 후, 해당 컴포넌트를 다시 렌더링하도록 동작한다.

`props` 값을 받는 컴포넌트가 있을 때, `props`를 내려주는 부모 컴포넌트가 렌더링될 때마다 같이 렌더링된다. 즉, `props` 값이 바뀌지 않아도 부모 컴포넌트가 렌더링되면 자식 컴포넌트 또한 다시 렌더링된다. 만약 자식 컴포넌트의 `props` 값이 변경될 때만 렌더링이 되길 원한다면, `React.memo`나 `React.PureComponent`를 사용할 수 있다.

#### setState

`setState` 메소드는 비동기로 `state` 값을 변경한다. 그래서 `setState` 메소드를 연속적으로 호출할 경우 의도대로 동작하지 않는다. 리액트는 효율적으로 렌더링하기 위해 여러 개의 `setState` 메소드를 배치처리한다. 여기서 배치 처리란, 일괄 처리를 말한다.

리액트의 `setState`는 비동기로 여러 개를 일괄 처리하기 때문에, 동시에 `setState`가 호출되더라도 한 번만 반영이 된다. 이를 해결하기 위해 다음과 같이 `setState`에 함수를 넣어줄 수 있다.

```javascript
onClick = () => {
    this.setState(prevState => ({count: prevState.count + 1}));
    this.setState(prevState => ({count: prevState.count + 1}));
}
```

위처럼 `setState` 메소드에서 인자로 들어갈 함수는 자신이 호출되기 직전의 상태값(`prevState`)을 파라미터로 받는다. 두 번째 `setState` 호출은 첫 번째에서 이미 변경한 `state` 값이 인자로 들어가는 함수의 파라미터에 들어간다. 그래서 위와 같이 쓰이면 `count` 라는 `state` 값이 2가 증가한다.

이를 활용해서 `state`를 관리하는 로직을 분리해서 사용할 수도 있다.

```javascript
const actions = {
    init() {
        return { count: 0 };
    },
    increment(state) {
        return { count: state.count + 1 };
    },
    decrement(state) {
        return { count: state.count - 1 };
    },
}

class MyComponent extends React.Component {
    state = actions.init();

    onIncrement = () => {
        this.setState(actions.increment);
    };

    onDecrement = () => {
        this.setState(actions.decrement);
    };
}
```

위와 같이 state 값을 관리하는 로직을 따로 분리하여 사용할 수도 있다. 난 이게 개인적으로 Redux가 위에서 파생된 개념이지 않을까 생각이 든다. 그런데 어떤 의미로 이런 이야기를 책에 써놨는지는 이해가 가지만 개인적으로 조금 코드 가독성이 떨어지는 것 같은 느낌이 들기도 한다.... 내 실력이 아직 못미치는 걸로....

그 다음으로, `setState` 메소드는 비동기로 처리되지만, 호출 순서가 보장된다. `setState` 호출 순서대로 `state` 값이 변경된다.

그리고 `setState` 메소드가 비동기로 처리되기 때문에, 처리되는 시점을 정확히 알고 싶을 수 있다. 그래서 `setState` 메소드의 두 번째 파라미터로 받는 값은 **처리가 끝났을 때 호출되는 콜백 함수**이다. 그렇게 `setState` 메소드의 두 번째 파라미터를 이용해서 `state` 값이 변경 된 후의 작업을 할 수 있다. 즉, 이는 변경된 값을 기반으로 다음 작업을 처리할 때도 유용하게 쓰인다.

