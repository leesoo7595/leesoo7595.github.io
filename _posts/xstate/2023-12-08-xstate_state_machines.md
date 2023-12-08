---
layout: post
title: "[XState] state machines"
date: 2023-12-08
tags: [xstate]
comments: true
---

이 글은 [XState 공식문서](https://stately.ai/docs/machines#specifying-types)를 읽고 정리한 글이다.

# State machines - 상태 머신

state machine(이하 상태 머신)은 actor 같은 무언가의 행동을 묘사하는 모델이다. Finite state machines는 actor의 state가 어떤 event 발생한 때에 다른 state로 트랜지션되는지를 묘사한다. (finite states(유한 상태)란 주어진 시간에서 있을 수 있는 가능한 state 중 하나이다.)

## 상태 머신의 이점

상태 머신은 안정적이고 강력한 소프트웨어를 구축하는데 도움이 된다.

## 상태 머신 생성하기

XState에서는 상태 머신을 `createMachine(config)` 함수를 사용하여 생성한다:

```javascript
import { createMachine } from "xstate";

const feedbackMachine = createMachine({
  id: "feedback",
  initial: "question",
  states: {
    question: {
      on: {
        "feedback.good": {
          target: "thanks",
        },
      },
    },
    thanks: {
      // ...
    },
    // ...
  },
});
```

이 예시에서는 machine은 `question`과 `thanks` 두 개의 state를 가지고 있다: `question` state는 `feedback.good` 이벤트가 머신에 전송될 때 `thanks` state로 트랜지션한다.

```javascript
const feedbackActor = createActor(feedbackMachine);

feedbackActor.subscribe((state) => {
  console.log(state.value);
});

feedbackActor.start(); // logs 'question'
feedbackActor.send({ type: "feedback.good" }); // logs 'thanks'
```

## 머신에서 actors 생성하기

머신은 actor의 로직을 포함한다. actor란 실행중인 머신의 인스턴스이다; 다시 말하자면, 머신에 의해 묘사된 로직을 가지고 있는 개체이다. 여러 actors가 같은 머신에서 생성될 수 있고, 각각의 actors는 같은 행동을 보여주지만(수신된 이벤트에 따른 반응), 그것들은 각기에 대해 독립적이고, 각자의 상태를 가지고 있다.

actor를 생성하기 위해서는 `createActor(machine)` 함수를 사용한다:

```javascript
import { createActor } from "xstate";

const feedbackActor = createActor(feedbackMachine);

feedbackActor.subscribe((state) => {
  console.log(state.value);
});
feedbackActor.start(); // logs 'question'
```

다른 타입의 로직을 가지고 있는 actor도 생성할 수 있다. (예를 들면 functions, promises, 그리고 observables 같은 로직)

## Providing implementations

이는 머신 생성시 실행되지마느 머신의 state, transition과 직접적으로 관련이 없는 language-specific 코드이다. 이 머신 구현체들은 아래를 포함한다:

- Actions : 실행되고 side-effects가 잊혀진다.
- Actors : 머신 actor과 상호작용할 수 있는 개체들
- Guards : 트랜지션 여부를 결정하는 조건
- Delays : 지연된 트랜지션이 수행되거나, 지연된 이벤트가 전송되기까지의 시간을 지정한다.

이 기본적인 구현체들은 머신을 생성할 때의 `setup({...})` 함수에 제공된다. 그리고 JSON 직렬화 가능한 문자열 또는 객체를 사용하여 해당 구현을 참조할 수 있다.

```javascript
import { setup } from "xstate";

const feedbackMachine = setup({
  // Default implementations
  actions: {
    doSomething: () => {
      console.log("Doing");
    },
  },
  actors: {},
  guards: {},
  delays: {},
}).createMachine({
  entry: { type: "doSomething" },
});

const feedbackActor = createActor(feedbackMachine);
feedbackActor.start(); // logs 'Doing';
```

기본 구현체들을 제공해주는 `machine.provide(...)` 기능들로 하여금 오버라이딩할 수 있다. 이 함수는 같은 config로 하지만 제공된 기능들로 새로운 머신을 생성할 것이다.

```javascript
// 기존에 feedbackMachine이 있음 (위)
const customFeedbackMachine = feedbackMachine.provide({
  actions: {
    doSomething: () => {
      console.log("Doing something");
    },
  },
});

const feedbackActor = createActor(customFeedbackMachine);
feedbackActor.start(); // logs 'Doing something'
```

## 타입 지정하기

머신 config 안에 `.types` 프로퍼티를 사용하여 Typescript 타입을 지정할 수 있다.

```javascript
import { createMachine } from 'xstate';

const feedbackMachine = createMachine({
  types: {} as {
    context: { feedback: string };
    events: { type: 'feedback.good' } | { type: 'feedback.bad' };
    actions: { type: 'logTelemetry' };
  },
});
```

이 타입들은 머신 config 전체와 생성된 머신, actor에서 추론되므로 `machine.transition(...)`, `actor.send(...)` 등의 메서드가 타입안전하다.

`setup(...)` 함수를 사용하는 경우, `setup` 함수 내부의 `.types` 프로퍼티에 타입을 제공해주어야한다:

```javascript
import { setup } from 'xstate';

const feedbackMachine = setup({
  types: {} as {
    context: { feedback: string };
    events: { type: 'feedback.good' } | { type: 'feedback.bad' };
  },
  actions: {
    logTelemetry: () => {
      // TODO: implement
    }
  }
}).createMachine({
  // ...
})
```

## Typegen

Typegen은 XState v5에서 아직 제공하지 않는다. `setup(...)` 함수에서 `.types` 프로퍼티를 기재해놓는다면, 머신에 강한 타입을 제공해줄 수 있다.

## Typescript

머신에 강한 타입을 제공하는 방법은 `setup(...)` 함수를 사용하여 `.types` 프로퍼티를 넣는 것이다.

```javascript
import { setup, fromPromise } from 'xstate';

const someAction = () => {};
const someGuard = ({ context }) => context.count <= 10;
const someActor = fromPromise(async () => {
  return 42;
});

const feedbackMachine = setup({
  types: {
    context: {} as { count: number };
    events: {} as { type: 'increment' } | { type: 'decrement' };
  },
  actions: {
    someAction,
  },
  guards: {
    someGuard,
  },
  actors: {
    someActor
  }
}).createMachine({
  initial: 'counting',
  states: {
    counting: {
      entry: { type: 'someAction' }, // strongly-typed
      invoke: {
        src: 'someActor', // strongly-typed
        onDone: {
          actions: ({event}) => {
            event.output;
          }
        }
      },
      on: {
        increment: {
          guard: { type: 'someGuard' }, // strongly-typed
          actions: assign({
            count: ({ context }) => context.count + 1
          })
        }
      }
    }
  }
})
```
