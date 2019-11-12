---
layout: post
title: "[React] Tutorials - 1st week"
date: 2019-11-11
tags: [React]
comments: true
---

# React 1주차 강의

일단 나도 실무에서는 진짜 병아리 중의 햇병아리고 자바스크립트, 리액트에 관해서 아직도 공부할 것이 많은 사람이라 혹시 내가 설명이 좀 부족할 수 있고 그래도 좀 양해해주고 잘 이해가 안가거나 하면 질문 많이 해줘! 내가 아는 선에서는 다 얘기해줄 수 있도록 노력할게.

### React의 원리 (pass)

일단 우리 멋사에서 이제껏 해온 프론트 개발 작업은 실제 DOM 요소를 건들여서 작업을 해왔어. 여기서 DOM이란 우리가 브라우저를 볼때 html애들 있자나 걔네들이라고 생각하면 되고, 
페이스북에서 리액트를 만들기전의 프론트엔드 프레임워크들은, 어떤 데이터에 따라 뷰를 그리고, 해당 데이터가 바뀌면 그 바뀐 데이터에 따라 변화를 하는 식으로 뷰를 업데이트했는데, 여기서 변화라는 의미는 상당히 복잡하대. 그래서 페이스북에서는 그냥 변화라는 것을 주지말고, 데이터가 바뀌는 부분의 컴포넌트 자체를 날려버리고 다시 새로 뷰를 그리는 식으로 하면 어떨까 하는 생각을 하게 되었대.

그렇게하면 매우 간단해져. 그런데 실제 브라우저의 돔을 매번 데이터에 따라 뷰를 새로 그리게되는 작업을 직접적으로 하게되면 성능상의 이슈가 생겨. 그래서 React는 Virtual DOM이라는 가상돔을 그려서 그 안에 있는 데이터의 흐름에 따라 뷰를 날리고 새로 그리는 작업을 진행하게 하눈거야. 즉 React에서 가상돔으로 한번 뷰를 그려서 데이터의 변화를 비교한 다음에, 실제돔에 정말로 바뀐부분만 찾아서 업데이트할 수 있도록 한 단계 거쳐갈 수 있게 하는고지..

여기서 즉, Virtual DOM은 실제 DOM의 변화를 최소화 시켜주는 작업을 해줘!

## Reac란 ?

react는 페이스북에서 제공해주는 프론트엔드 라이브러리야.

## React를 시작하기까지

react를 쓰려면 사실 `npm install react`로도 가능하지만 react는 `es6` 기반으로 돌아가기 때문에 `babel`과 `webpack` 설정을 해주어야해
여기서 `babel`이란 `es6`기반으로 돌아가는 react 코드를 여러 브라우저의 호환성(IE) 때문에 자스 언어 버전을 `es5`으로 다운그레이드 해주는 트랜스컴파일러야
그리고 `webpack`은 나도 공부를 거의 안해봐서 설명해주기가 어려운데, 그냥 많은 종류의 파일들을 하나의 js파일로 합쳐주는 역할을 한다고 보면 될 것같아ㅠ

그런 작업들을 다 해주어야 react 라이브러리를 통해 프론트엔드 개발이 가능한데, 이를 진짜 딱 다운만해서 쓸수 있도록 프로젝트를 만든 것이 (보일러플레이트) `CRA(Create-react-app)`이야

## CRA 설치

1. nodejs 버전 확인
2. visual code editor

* nodejs 다운받기 : https://nodejs.org/en/
* CRA에서 hello world 찍기 : https://velopert.com/3621

## CRA 구조 파헤치기

1. App.js

구조 설명

컴포넌트를 만드는 방법
* 클래스 컴포넌트
* 함수형 컴포넌트

2. 브라우저상에 컴포넌트를 보여주기 위한 
`ReactDOM.render`

`ReactDOM.render(<App />, document.getElementById('root'));`

첫번째 파라미터는 렌더링 할 결과물이고, 두번째 파라미터는 컴포넌트를 어떤 DOM 에 그릴지 정해준다.

참고: https://velog.io/@godori/DOM%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80

## JSX 개념
￼

위 이미지는 jsx를 `babel`로 이용하여 트랜스컴파일한 것인데, 원래 이런식으로 입력해야 돌아가게되는건데, 이렇게 길고 복잡하게 입력하기 싫잖아.

그래서 리액트 개발을 쉽게 하기 위해 html 태그와 비슷하게 생긴 형태의 문법으로 (jsx) 작성해주면 알아서 babel이 저런 식으로 컴파일 해주도록 CRA에서 잘 만들어놨어.

여기서 XML 같이도 생긴 JSX를 바벨에서 잘 변환시킬 수 있도록 JSX 문법을 따라야하는 것이 있어

1. 열린태그, 닫힌 태그

우선 JSX는 html 태그처럼 열려있는 태그로 시작해야하고, 그 태그는 무조건 닫혀 있어야해

```javascript
<div>
    hello
</div>
```

2. 감싸져 있는 엘리먼트

리액트에서 컴포넌트를 그릴때, return 값으로 나와야할 값은 무조건 하나의 태그로 감싸져있는 요소가 나와야한다.


3. JSX 내부엔 자바스크립트 값을 사용하기
4. style, className 설정
    - 컴포넌트 내에서 바로 스타일 지정하기
    - css 파일을 따로 만들어서 스타일 지정하기

## component 만들기

### 컴포넌트 분리하기

### props 전달
부모 컴포넌트가 자식 컴포넌트에게 전달해주는 값

자식 컴포넌트 생성
```javascript
import React, { Component } from 'react';

class MyName extends Component {
  render() {
    return (
      <div>
        안녕하세요! 제 이름은 <b>{this.props.name}</b> 입니다.
      </div>
    );
  }
}

export default MyName;
```

자식 컴포넌트 사용 및 props 전달
```javascript
import React, { Component } from 'react';
import MyName from './MyName';

class App extends Component {
  render() {
    return (
      <MyName name="리액트" />
    );
  }
}

export default App;
```

1. static defaultProps = {}

```javascript
static defaultProps = {
    name: '기본이름'
  }
```

2. 함수형 컴포넌트 (단순한 props 전달)
3. 함수형 컴포넌트와 클래스 컴포넌트의 차이점 lifecycle의 유무

### state 전달
컴포넌트 내부에서 선언하며, 값을 변경할 수 있다.

1. state 

```javascript
state = {
    number: 0
  }
```

생성자 필드에 넣어서 사용할 수도 있지만, 편의를 위해 class fields를 사용한다.

2. 메소드 작성시 -> arrow function 과 this


3. setState
setState 함수가 호출되면 컴포넌트가 리렌더링되도록 설계되어있다.

`const { number } = this.state;`

```javascript
const { number } = this.state;
this.setState({
  number: number + 1
})
```

