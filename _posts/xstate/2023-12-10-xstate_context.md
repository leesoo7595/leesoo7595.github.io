---
layout: post
title: "[XState] context"
date: 2023-12-10
tags: [xstate]
comments: true
---

이 글은 [XState 공식문서](https://stately.ai/docs/context)를 읽고 정리한 글이다.

# Context

XState에서 context는 상태 머신 actor에 데이터를 저장하는 방식이다.

context 프로퍼티는 모든 state에서 사용할 수 있고, actor와 관련된 데이터를 저장하는데 사용된다. context 객체는 immutable하고, 직접적으로 수정할 수 없다. 대신, 상태머신 로직의 경우 `assign` 액션을 사용하여 context를 업데이트할 수 있다.

context 프로퍼티는 옵셔널이다; 상태 머신이 finite states만을 명시하고 다른 외부 컨텍스트 데이터가 없다면, `context`는 필요없을 것이다.

```javascript
import { createMachine } from "xstate";

const feedbackMachine = createMachine({
  // Initialize the state machine with context
  context: {
    feedback: "Some feedback",
  },
});

const feedbackActor = createActor(feedbackMachine);

feedbackActor.subscribe((state) => {
  console.log(state.context.feedback);
});

feedbackActor.start();
// logs 'Some feedback'
```

## initial context

```javascript
import { createMachine } from "xstate";

const feedbackMachine = createMachine({
  context: {
    feedback: "Some feedback",
    rating: 5,
    // other properties
  },
});

// lazy initial context
const feedbackMachine = createMachine({
  context: () => ({
    feedback: "Some feedback",
    createdAt: Date.now(),
  }),
});

const feedbackActor = createActor(feedbackMachine).start();

console.log(feedbackActor.getSnapshot().context.createdAt);
// logs the current timestamp
```

lazy 초기 context는 actor 단위로 평가되므로, 각 actor는 자체 context 객체를 갖게 된다.

## input

input 데이터를 머신 초기 context로 주려면, `createActor(machine, {input})` 함수로 input 프로퍼티를 전달하고, context 함수의 첫번째 인수에서 input 프로퍼티를 사용할 수 있다:

```javascript
const feedbackMachine = createMachine({
  context: ({ input }) => ({
    feedback: "",
    rating: input.defaultRating,
  }),
});

const feedbackActor = createActor(feedbackMachine, {
  input: {
    defaultRating: 5,
  },
}).start();

console.log(feedbackActor.getSnapshot().context.rating);
// logs 5
```

## `assign(...)`으로 context 수정하기

```javascript
import { createMachine, assign } from "xstate";

const feedbackMachine = createMachine({
  context: {
    feedback: "Some feedback",
  },
  on: {
    "feedback.update": {
      actions: assign({
        feedback: ({ event }) => event.feedback,
      }),
    },
  },
});

const feedbackActor = createActor(feedbackMachine);

feedbackActor.subscribe((state) => {
  console.log(state.context.feedback);
});

feedbackActor.start();

// logs 'Some feedback'

feedbackActor.send({
  type: "feedback.update",
  feedback: "Some other feedback",
});

// logs 'Some other feedback'
```
