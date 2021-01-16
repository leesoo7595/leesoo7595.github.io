---
layout: post
title: "[React] Immer 라이브러리"
date: 2021-01-15
tags: [React]
categories:
- React
comments: true
---

오늘은 Immer 라이브러리에 대해서 공부해보려고 한다.

리액트에서는 객체를 수정할 때 불변성을 지켜주면서 수정을 해야한다.

```javascript
const object = {
  a: 1,
  b: 2,
}

const updatedObject = {
  ...object,
  b: 3,
}
```
배열의 경우에도 직접적으로 배열을 수정하는 메소드를 사용하는 것 보다 카피본을 새로 만들어주는 메소드를 사용해야한다.

하지만 객체의 복잡성이 증가하여 이와 같이 불변성을 지켜주기가 어려워질 경우, 코드의 가독성이 떨어지고 코드의 구조가 복잡해지게 된다. 

이런 경우, `immer`라는 라이브러리를 사용하면 코드의 가독성도 올라가고 간편하게 구현이 가능해진다.

```javascript
import produce from 'immer'

const nextState = produce(state, draft => {
  const post = draft.posts.find(post => post.id === 1)
  post.comments.push({
    id: 3,
    text: 'asdf',
  })
})
```

immer를 사용하면 상태를 업데이트할 때 불변성을 신경쓰지 않게 해준다. 즉, immer가 불변성 관리를 대신 해준다.

## Immer

```javascript
const state = {
  number: 1,
  dontChangeMe: 2
};

const nextState = produce(state, draft => {
  draft.number += 1;
});

console.log(nextState);
// { number: 2, dontChangeMe: 2 }
```

immer를 사용하면 `produce`라는 함수를 사용하게 되는데, `produce`의 첫 번째 파라미터로는 수정하고 싶은 상태를 넣어주고, 두 번째 파라미터로는 어떻게 업데이트하고 싶을지 정의하는 함수를 넣어주어 사용하게 된다.

## Reference

- [Immer를 사용한 쉬운 불변성 관리](https://react.vlpt.us/basic/23-immer.html)
