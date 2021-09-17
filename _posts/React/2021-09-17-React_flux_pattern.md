---
layout: post
title: "[React] 리덕스와 플럭스 패턴 이해하기"
date: 2021-09-17
tags: [React]
categories:
  - React
  - TIL
comments: true
---

## Flux 패턴

**리액트, 리덕스로 프로젝트를 구성하는데 잘 이해가 가지 않아서 정리해보는 글** <br />

예전의 프론트엔드 패턴은 MVC 패턴이었다고 한다. 하지만 프로젝트가 점점 무거워지면서 수많은 View와 Model들이 생기고, 이들을 파악하기가 어렵게 되었다고 한다. <br />

그러다가 페이스북에서 Flux라는 패턴에 대해 발표하게 되었다. Flux 패턴은 어떤 액션이 발생하면 dispatcher에 의해 store에 변경된 사항이 저장된다. 그리고 그 저장된 데이터들에 의해 view가 변경되는 단방향 패턴이다. <br />

### dispatch 함수란?

dispatcher란, 어플리케이션에서 발생한 action들을 정리해주는 역할을 한다. 찾아보면서 공부하는데 나는 아직 dispatcher의 쓰임새 이해가 잘 가지 않았다.

다른 사이트를 참고해보니..

```
action === event
reducer === response of event
```

dispatch라는 함수에 단순 객체일 뿐인 action을 넣어서 action을 발생시키도록 만든다.
<br />
<br />

`dispatch(action) => reducer 동작하게됨 => store의 state 변경 => 변경된 state가 state를 subscription하고 있는 component에 정보 전달`
<br />

이런 로직으로 상태 변화가 일어난다.<br />

즉, 아무 기능도 없는 객체일 뿐인 event를 dispatch가 동작시키도록 도와주는 역할을 한다.

Flux 패턴을 쓰게되면, 개발 흐름이 단방향으로 흐르기 때문에 파악하기 용이하고 코드의 흐름이 예측 가능해진다고 한다.

### Flux 패턴과 Reducer 개념을 도입한 Redux

Flux패턴이 탄생하고, 댄아저씨가 Redux를 탄생시켜버린당.
Flux 패턴이 나왔지만, Flux패턴을 반영한 공식 구현체가 존재하지 않아서 그 중 수많은 것들 중 하나가 Redux인 것이었당

Flux 패턴의 동작 방식이 Reduce 함수와 비슷하다고 생각하여 Reducer 함수 개념을 도입한 것이 Redux이다.

**reduce 함수와 비슷하다구??**

모가 비슷행? 잘 모르겠어서 조금 더 서칭...

Flux 패턴의 경우 애플리케이션의 상태를 action이 바뀜에 따라 어떤식으로 반영해야하는지를 알려주지 않는다. 그것을 redux 프레임워크에서 reducer가 담당한다고 한다.

`Array.prototype.reduce(cb, initVal)`에 전달되는 리듀서콜백처럼 동작한다.

```javascript
// 상태 객체
initialState = {
  visibilityFilter: "SHOW_ALL",
  todos: [
    {
      text: "자료 준비하기",
      completed: true,
    },
    {
      text: "블로그 글쓰기",
      completed: false,
    },
  ],
};

//이 상태를 업데이트해주는 리듀서
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter,
      });
    default:
      return state;
  }
}
```

리듀서는 첫번째 인수로는 기존 상태를 전달받고, 두번째 인수로 액션 객체를 전달받는다. 리듀서를 작성할 때 첫 번째 인수로 전달받은 state는 수정하면 안되고, 반드시 `Object.assign`등을 사용하여 새로운 객체를 만들어서 반환해야 한다.

리듀서의 default 케이스에서 기존 state를 반환해야한다. 이유는 전달받은 액션 객체를 리듀서가 처리하지 못하는 때가 있을 때 default 케이스에서는 기존의 state를 그대로 반환하는 로직이 있어야한다.

## Reference

- [[React] 리덕스 (Redux) 이해하기](https://im-developer.tistory.com/158)
- [[Redux] dispatch란?](https://zereight.tistory.com/603)
- [Flux와 Redux](https://taegon.kim/archives/5288)
- ['데이터가 폭포수처럼 흘러내려' React의 flux 패턴](https://www.huskyhoochu.com/flux-architecture/)
- [Flux로의 카툰 안내서](https://bestalign.github.io/translation/cartoon-guide-to-flux/)
