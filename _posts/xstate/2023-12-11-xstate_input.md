---
layout: post
title: "[XState] input"
date: 2023-12-11
tags: [xstate]
comments: true
---

이 글은 [XState 공식문서](https://stately.ai/docs/input)를 읽고 정리한 글이다.

# Input

Input은 상태 머신에 제공되어 그 동작에 영향을 주는 데이터를 말한다.. XState에서는 **actor를 생성할 때** `createActor(machine, {input})` 함수의 두 번째 인수를 사용하여 input을 사용한다:

```javascript
import { createActor, createMachine } from "xstate";

const feedbackMachine = createMachine({
  context: ({ input }) => ({
    userId: input.userId,
    feedback: "",
    rating: input.defaultRating,
  }),
  // ...
});

const feedbackActor = createActor(feedbackMachine, {
  input: {
    userId: "123",
    defaultRating: 5,
  },
});
```

## input과 같이 actors 생성하기

fromPromise(), fromTransition(), fromObservable() 등의 actor 로직 생성자의 첫 번째 인수의 input 프로퍼티를 통해 전달해서 이 input을 읽어 모든 종류의 actor에 input을 전달할 수 있다.

### `fromPromise()`

```javascript
import { createActor, fromPromise } from "xstate";

const userFetcher = fromPromise(({ input }) => {
  return fetch(`/users/${input.userId}`).then((res) => res.json());
});

const userFetcherActor = createActor(userFetcher, {
  input: {
    userId: "123",
  },
}).start();

userFetcherActor.onDone((data) => {
  console.log(data);
  // logs the user data for userId 123
});
```

### `fromTransition()`

```javascript
import { createActor, fromTransition } from 'xstate';

const counter = fromTransition((state, event) => {
  if (event.type === 'INCREMENT') {
    return { count: state.count + 1 };
  }
  return state;
}, ({ input }) => ({
  count: input.startingCount ?? 0,
});

const counterActor = createActor(counter, {
  input: {
    startingCount: 10,
  }
});
```

### `fromObservable()`

```javascript
import { createActor, fromObservable } from "xstate";
import { interval } from "rxjs";

const intervalLogic = fromObservable(({ input }) => interval(input.interval));

const intervalActor = createActor(intervalLogic, {
  input: {
    interval: 1000,
  },
});

intervalActor.start();
```

## 초기 이벤트 input

actor가 시작되면, 자동적으로 `xstate.init`라는 이름의 이벤트를 스스로에게 전송한다. `createActor(log, { input })` 함수에 input이 제공되면, `xstate.init` 이벤트에 포함되어있을 것이다:

```javascript
import { createActor, createMachine } from "xstate";

const feedbackMachine = createMachine({
  entry: ({ event }) => {
    console.log(event.input);
    // logs { userId: '123', defaultRating: 5 }
  },
  // ...
});

const feedbackActor = createActor(feedbackMachine, {
  input: {
    userId: "123",
    defaultRating: 5,
  },
}).start();
```

## input으로 actor 호출하기

actors가 호출될때 input을 제공해줄 수도 있다. invoke 설정에 input 프로퍼티를 추가해주면 된다.

```javascript
const machine = createMachine({
  invoke: {
    src: "liveFeedback",
    input: {
      domain: "stately.ai",
    },
  },
}).provide({
  actors: {
    liveFeedback: fromPromise(({ input }) => {
      return fetch(`https://${input.domain}/feedback`).then((res) =>
        res.json()
      );
    }),
  },
});
```

`invoke.input` 프로퍼티는 스태틱한 input 값 또는 input 값을 리턴해주는 함수가 될 수 있다. 함수는 현재상태의 context, event 객체를 포함한 함수이다:

```javascript
import { createActor, createMachine } from "xstate";

const feedbackMachine = createMachine({
  context: {
    userId: "",
    feedback: "",
    rating: 0,
  },
  invoke: {
    src: "fetchUser",
    input: ({ context }) => ({ userId: context.userId }),
  },
  // ...
}).provide({
  actors: {
    fetchUser: fromPromise(({ input }) => {
      return fetch(`/users/${input.userId}`).then((res) => res.json());
    }),
  },
});
```

## spawn 상태인 actors에 input 넘기기

`spawn` 설정에 input 프로퍼티를 통해 스폰된 액터에 입력을 제공할 수 있다:

```javascript
import { createActor, createMachine } from "xstate";

const feedbackMachine = createMachine({
  context: {
    userId: "",
    feedback: "",
    rating: 0,
    emailRef: null,
  },
  // ...
  on: {
    "feedback.submit": {
      actions: assign({
        emailRef: ({ context, spawn }) => {
          return spawn("emailUser", {
            input: { userId: context.userId },
          });
        },
      }),
    },
  },
  // ...
}).provide({
  actors: {
    emailUser: fromPromise(({ input }) => {
      return fetch(`/users/${input.userId}`, {
        method: "POST",
        // ...
      });
    }),
  },
});
```

## Use-cases

Input은 다른 input 값들을 설정할 필요가 있을 때 재사용된 machine을 생성하는데 유용하다.

- machine 관련된 기계적으로 함수를 작성하는 옛날 방식을 대체한다.

```javascript
// Old way: using a factory function
const createFeedbackMachine = (userId, defaultRating) => {
  return createMachine({
    context: {
      userId,
      feedback: "",
      rating: defaultRating,
    },
    // ...
  });
};

const feedbackMachine1 = createFeedbackMachine("123", 5);

const feedbackActor1 = createActor(feedbackMachine1).start();

// ===========
// New way: using input
const feedbackMachine = createMachine({
  context: ({ input }) => ({
    userId: input.userId,
    feedback: "",
    rating: input.defaultRating,
  }),
  // ...
});

const feedbackActor = createActor(feedbackMachine, {
  input: {
    userId: "123",
    defaultRating: 5,
  },
});
```

### actor에 새로운 데이터 전달

input이 바뀌어도 actor가 다시 시작하진 않는다. actor에 새로운 데이터를 전달하려면 actor에 이벤트를 전달해주어야한다.

```javascript
const Component = (props) => {
  const actor = useActor(machine, {
    input: {
      userId: props.userId,
      defaultRting: props.defaultRating,
    },
  });

  useEffect(() => {
    actor.send({
      type: "userId.change",
      userId: props.userId,
    });
  }, [props.userId]);
};
```
