---
layout: post
title: "[React] LifeCycle"
date: 2018-12-19
tags: [React]
comments: true
---

# 컴포넌트 초기 생성

컴포넌트가 브라우저에 나타나기 전, 후에 호출되는 API

## constructor

컴포넌트 생성자 함수이다. 컴포넌트가 새로 만들어질 때마다 이 함수가 호출된다.

## componentWillMount

컴포넌트가 화면에 나가기 직전에 호출되는 API이다. v16.3에는 deprecated됨

## componentDidMount

컴포넌트가 화면에 나타나게 됐을 때 호출되는 것이다. DOM을 사용해야하는 외부 라이브러리 연동을 하거나, 해당 컴포넌트에서 필요로 하는 데이터를 요청하기 위해 ajax 요청을 하거나, DOM의 속성을 읽거나 직접 변경하는 작업을 한다.

# 컴포넌트 업데이트

props, state의 변화에 따라 결정된다.

## componentWillReceiveProps

컴포넌트가 새로운 props를 받게 됐을 때 호출된다. state가 props에 따라 변해야하는 로직을 작성한다. 새로 받게될 props는 nextProps로 조회할 수 있고, this.props를 조회하면 업데이트되기 전의 API이다. v16.3에는 deprecated될 예정이다.

## static getDerivedStateFromProps()

v16.3 이후에 만들어진 라이프사이클 API이다. props로 받아온 값을 state로 동기화하는 작업을 해줘야하는 경우에 사용된다.

## souldComponentUpdate

컴포넌트를 최적화하는 작업에서 활용된다. 리액트는 변화가 발생하는 부분만 업데이트를 해줘서 성능이 좋다. 이렇게 동작하게하려면, virtual DOM에 그려줘야한다. virtual DOM이 뭐지;

변화가 없을 경우엔 DOM 조작을 하지않게 되고, virtual DOM에만 렌더링할 뿐인데, 컴포넌트가 무수히 많은 경우 CPU자원을 낭비하기 때문에 이를 쓴다.

이 함수는 기본적으로 true를 반환한다. 우리가 따로 작성을 해주어 조건에 따라 false로 반환하면 해당 조건에는 render 함수를 호출하지 않는다.

## componentWillUpdate

이후에 deprecated되는 API이다. true를 반환했을 때만 호출되는 것이다. 만약 false를 반환하면 이 함수는 호출되지 않는다. 주로 애니메이션 효과를 초기화하거나 이벤트리스너를 없애는 작업을 한다. 이 함수 호출 후에는 render()가 호출된다.

## getSnapshotBeforeUpdate()

이 API가 발생하는 시점 :

render() -> getSnapshotBeforeUpdate() -> 실제 DOM에 변화 -> componentDidUpdate

DOM 변화 일어나기 직전의 DOM 상태를 가져오고, 여기서 리턴하는 값을 componentDidUpdate에서 3번째 파라미터로 받아올 수 있다.

## componentDidUpdate

이 API는 render() 호출 후 발생한다. 이 시점에서는 props와 state가 바뀌어있다. 이 API의 파라미터로 이전의 값인 prevProps, prevState를 조회할 수 있다. getSnapshotBeforeUpdate에서 반환한 snapshot 값은 세번째 값으로 받아온다.

# 컴포넌트 제거

## componentWillUnMount

이 API에서는 주로 등록했었던 이벤트를 제거하고, setTImeout을 걸은 것이 있으면 제거를하는 등 라이브러리에 dispose 기능이 있다면 여기서 호출한다.
