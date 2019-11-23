---
layout: post
title: "[React] Tutorials - 3st week"
date: 2019-11-23
tags: [React]
comments: true
---

## Mini Project

* 부모 컴포넌트가 주는 색깔 이름에 따라서 버튼 색깔이 바뀌는 컴포넌트 만들기
* input 창 만들기


## 조건부 렌더링

* 색깔이 변하는 버튼 만들기

1. if로 나누기
2. 삼항연산자 사용하기

## Lists & keys

1. 하나의 컴포넌트를 여러개 그려야할 때??
2. list, maps
3. warning not given keys

```javascript
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li>{number}</li>
);
```

keys는 컴포넌트가 바뀌고, 생기고, 제거되는 데에 도움을 준다.
keys가 있음으로써 컴포넌트의 고유성이 주어지고, 안정성을 높여준다.

**keys는 unique해야한다**

## forms

* 간단한 회원가입 폼 만들어보기

### submit의 역할과 react

submit은 기본적으로 form 태그에 붙어있는 이벤트이다.

기본적으로 하는 일은 form이 서버에 보내지면서 refresh를 해주는 기능인데, react는 이러한 기능을 원하지않으므로(깜빡거리는거 최소화) 기능을 변경시켜줘야한다.

* e.preventDefault()

```javascript
class SignUpForm extends Component {
    state = {
        name: '',
        password: '',
    }

    onChangeName = e => {
        this.setState({name: e.target.value})
    }

    onChangePassword = e => {
        this.setState({password: e.target.value})
    }

    onSubmit = e => {
        e.preventDefault();
        // submit한 뒤에....
    }

    render() {
        const {name, password} = this.state;
        return (
            <form onSubmit={this.onSubmit}>
                <label>
                    Name 
                    <input type='text' value={name} onChange={this.onChangeName} />
                </label>
                <label>
                    Password 
                    <input type='password' value={password} onChange={this.onChangePassword} />
                </label>
                <button type='submit'>submit</button>
            </form>
        )
    }
}
```

## Composition vs Inheritance

React는 상속 대신 합성을 사용하여 컴포넌트 간에 코드를 재사용하는 것이 좋다.

### Containment

어떤 페이지를 만들 때, 주로 겹처지는 레이아웃들이 있다. 레이아웃은 겹쳐지지만 내용은 달라지는 경우!!! 즉, props가 무엇인지 예상할 수 없을 때 쓰이는 것!

* `props.children`

그 어떤 컴포넌트에는 props를 전달받을때 children이라는 값으로 모든 것을 내려받을 수 있다!

더 **구체적인** 컴포넌트가 **일반적인** 컴포넌트를 렌더링하고 props를 통해 내용을 구성할 수 있다.

```javascript
class Layout extends Component {
    render() {
        const {children, title, message} = this.props;
        const style = {
            width: '100%',
            height: '100%',
            backgroundColor: 'black',
        }
        return (
            <div style={style}>
                <div>
                    {title}
                </div>
                <div>
                    {message}
                </div>
                {children}
            </div>
        )
    }
}
```

## material-ui

https://material-ui.com/

```javascript
npm install @material-ui/core
```
* Button
* TextField
...