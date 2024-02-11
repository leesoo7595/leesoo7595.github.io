---
layout: post
title: "[XState] Actions"
date: 2024-02-06
tags: [xstate]
comments: true
---

# Actions

액션은 실행 후 잊어버리는 효과(fire-and-forget effect)이다. 상태머신이 트랜지션될 때, 액션을 실행할 수 있다. 액션은 이벤트에 대한 응답으로 발생하며, 일반적으로 `actions: [...]` 프로퍼티로 정의된다. state에서 기재될 수 있는 모든 트랜지션에 대해 액션을 정의할 수 있다. 시작, 종료 프로퍼티에도 가능하다.

```typescript
import { setup } from 'xstate';

const feedbackMachine = setup({
  actions: {
    track: (_, params: unknown) => {
      track(params);
      // Tracks { response: 'good' }
    },
    showConfetti: () => {
      // ...
    }
  }
}).createMachine({
  // ...
  states: {
    // ...
    question: {
      on: {
        'feedback.good': {
          actions: [
            { type: 'track', params: { response: 'good' } }
          ]
        }
      },
      exit: ['exitAction']
    }
    thanks: {
      entry: ['showConfetti'],
    }
  }
});
```

actions 예시:

- 메세지 로깅
- 다른 actor로 메세지 전달
- context 업데이트

## Action objects

액션 객체는 type과 선택적인 params를 가진다:

- params 프로퍼티는 작업과 관련된 매개변수 값이 저장된다.

```typescript
import { setup } from "xstate";

const feedbackMachine = setup({
  actions: {
    track: (_, params: unknown) => {
      /* ... */
    },
  },
}).createMachine({
  // ...
  states: {
    // ...
    question: {
      on: {
        "feedback.good": {
          actions: [
            {
              // Action type
              type: "track",
              // Action params
              params: { response: "good" },
            },
          ],
        },
      },
    },
  },
});
```

## Dynamic action parameters

params를 리턴하는 함수를 사용하여 동적으로 파라미터값을 `params` 프로퍼티에 전달할 수 있다. 이 함수는 context와 event를 가지고 있는 객체를 인자로 가진다.

```typescript
import { setup } from "xstate";

const feedbackMachine = setup({
  actions: {
    logInitialRating: (_, params: { initialRating: number }) => {
      // ...
    },
  },
}).createMachine({
  context: {
    initialRating: 3,
  },
  entry: [
    {
      type: "logInitialRating",
      params: ({ context }) => ({
        initialRating: context.initialRating,
      }),
    },
  ],
});
```

아래 방식은 재사용이 가능하고 머신의 context나 event 타입에 의존하지 않는 정의방법으로 권장된다.

```typescript
import { setup } from "xstate";

function logInitialRating(_, params: { initialRating: number }) {
  console.log(`Initial rating: ${params.initialRating}`);
}

const feedbackMachine = setup({
  actions: { logInitialRating },
}).createMachine({
  context: { initialRating: 3 },
  entry: [
    {
      type: "logInitialRating",
      params: ({ context }) => ({
        initialRating: context.initialRating,
      }),
    },
  ],
});
```

## Inline actions

```typescript
import { createMachine } from "xstate";

const feedbackMachine = createMachine({
  entry: [
    // Inline action
    ({ context, event }) => {
      console.log(/* ... */);
    },
  ],
});
```

액션 객체를 사용하는 것이 권장된다.

## Implementing actions

```typescript
// 방법 1
import { setup } from "xstate";

const feedbackMachine = setup({
  actions: {
    track: ({ context, event }, params) => {
      // Action implementation
      // ...
    },
  },
}).createMachine({
  // Machine config
  entry: [{ type: "track", params: { msg: "entered" } }],
});

// 방법 2
const feedbackActor = createActor(
  feedbackMachine.provide({
    actions: {
      track: ({ context, event }, params) => {
        // Different action implementation
        // (overrides previous implementation)
        // ...
      },
    },
  })
);
```

## Assign action

`assign()` 액션은 context에 데이터를 할당하는 특수한 액션이다. `assign(assignment)`의 assignments 인수는 context에 대한 할당이 지정되는 곳이다.

```typescript
import { setup } from "xstate";

const countMachine = setup({
  types: {
    events: {} as { type: "increment"; value: number },
  },
}).createMachine({
  context: {
    count: 0,
  },
  on: {
    increment: {
      actions: assign({
        count: ({ context, event }) => context.count + event.value,
      }),
    },
  },
});

const countActor = createActor(countMachine);
countActor.subscribe((state) => {
  console.log(state.context.count);
});
countActor.start();
// logs 0

countActor.send({ type: "increment", value: 3 });
// logs 3

countActor.send({ type: "increment", value: 2 });
// logs 5
```

## Raise action

raise 액션은 동일한 머신에서 받아온 이벤트를 발생하는 특수한 액션이다. 이벤트 발생은 머신이 스스로에게 이벤트를 'send'할 수 있는 방법이다:

```typescript
import { createMachine, raise } from 'xstate';

const machine = createMachine({
  // ...
  entry: raise({ type: 'someEvent', data: 'someData' });
});
```

내부적으로 이벤트다 raise되면, **internal event queue** 내부 이벤트 대기열에 배치된다. 현재 트랜지션이 완료된 이후에 이 이벤트들은 FIFO 순서대로 처리된다. 외부 이벤트는 내부 이벤트 큐의 모든 이벤트가 처리된 이후에만 처리된다.

raised 이벤트는 동적일 수도 있다:

```typescript
import { createMachine, raise } from "xstate";

const machine = createMachine({
  // ...
  entry: raise(({ context, event }) => ({
    type: "dynamicEvent",
    data: context.someValue,
  })),
});
```

delay로도 사용될 수 있으며, 내부 이벤트 대기열에 넣지 않고 delay시켜서 발생시킬 수도 있다.

## Send-to action

`sendTo(...)` 액션은 특정 액터에게 이벤트를 보내는 특수한 액션이다.

```typescript
const machine = createMachine({
  on: {
    transmit: {
      actions: sendTo("someActor", { type: "someEvent" }),
    },
  },
});

// dynamic
const machine = createMachine({
  on: {
    transmit: {
      actions: sendTo("someActor", ({ context, event }) => {
        return { type: "someEvent", data: context.someData };
      }),
    },
  },
});

// 도착지 actor는 actor ID 또는 스스로를 참조하는 actor가 될 수 있다.
const machine = createMachine({
  context: ({ spawn }) => ({
    someActorRef: spawn(fromPromise(/* ... */)),
  }),
  on: {
    transmit: {
      actions: sendTo(({ context }) => context.someActorRef, {
        type: "someEvent",
      }),
    },
  },
});

// delay와 id 같은 다른 옵션을 세번째 인자에 전달할 수 있다.
const machine = createMachine({
  on: {
    transmit: {
      actions: sendTo(
        "someActor",
        { type: "someEvent" },
        {
          id: "transmission",
          delay: 1000,
        }
      ),
    },
  },
});
```

## Enqueue actions

`enqueueActions(...)` 액션 생성자는 액션을 실제로 실행하지 않고 순차적으로 실행할 액션을 대기열에 추가하는 상위 레벨 액션이다. 이 함수는 context, event를 콜백으로 받을 수 있고, 더해서 enqueue, check 함수를 수신할 수 있다:

- `enqueue(...)` 함수는 액션을 대기열에 추가하는 데에 사용된다. 이 함수는 액션 객체 또는 액션 함수를 받는다:

```typescript
actions: enqueueActions(({ enqueue }) => {
  // Enqueue an action object
  enqueue({ type: "greet", params: { message: "hi" } });

  // Enqueue an action function
  enqueue(() => console.log("Hello"));

  // Enqueue a simple action with no params
  enqueue("doSomething");
});
```

- `check(...)` 함수는 조건적으로 액션을 대기열에 추가하는 데에 사용된다. guard 객체 또는 guard 함수를 받아서 이 가드가 true인지 여부를 나타내는 boolean을 리턴한다.

```typescript
actions: enqueueActions(({ enqueue, check }) => {
  if (check({ type: "everythingLooksGood" })) {
    enqueue("doSomething");
  }
});
```

- 기본 빌트인 액션으로 헬퍼메소드가 `enqueue`에 있다:
- `enqueue.assign()`
- `enqueue.sendTo()`
- `enqueue.raise()`
- `enqueue.spawnChild()`
- `enqueue.stopChild()`
- `enqueue.cancel()`

대기열에 추가된 액션은 조건부로 호출할 수 있지만, 비동기적으로 대기열에 추가할 수 없다.

```typescript
const machine = createMachine({
  // ...
  entry: enqueueActions(({ context, event, enqueue, check }) => {
    // assign action
    enqueue.assign({
      count: context.count + 1,
    });

    // Conditional actions (replaces choose(...))
    if (event.someOption) {
      enqueue.sendTo("someActor", { type: "blah", thing: context.thing });

      // other actions
      enqueue("namedAction");
      // with params
      enqueue({ type: "greet", params: { message: "hello" } });
    } else {
      // inline
      enqueue(() => console.log("hello"));

      // even built-in actions
    }

    // Use check(...) to conditionally enqueue actions based on a guard
    if (check({ type: "someGuard" })) {
      // ...
    }

    // no return
  }),
});
```

## Log action

`log(...)` 액션은 콘솔에 로그메세지를 간단하게 남길 수 있다.

```typescript
import { createMachine, log } from "xstate";

const machine = createMachine({
  on: {
    someEvent: {
      actions: log("some message"),
    },
  },
});
```

## Cancel action

`cancel(...)` 액션은 지연된 `sentTo(...)` 또는 `raise(...)` 액션을 그들의 IDs로 취소한다.

```typescript
import { createMachine, sendTo, cancel } from "xstate";

const machine = createMachine({
  on: {
    event: {
      actions: sendTo(
        "someActor",
        { type: "someEvent" },
        {
          id: "someId",
          delay: 1000,
        }
      ),
    },
    cancelEvent: {
      actions: cancel("someId"),
    },
  },
});
```

## Stop child action

`stopChild(...)` 액션은 자식 액터를 멈춘다. 액터는 그들의 부모 액터를 통해서 멈출 수 있다:

```typescript
import { createMachine, stopChild } from "xstate";

const machine = createMachine({
  context: ({ spawn }) => ({
    spawnedRef: spawn(fromPromise(/* ... */), { id: "spawnedId" }),
  }),
  on: {
    stopById: {
      actions: stopChild("spawnedId"),
    },
    stopByRef: {
      actions: stopChild(({ context }) => context.spawnedRef),
    },
  },
});
```

## Modeling

이벤트에 대한 응답으로 액션이 실행해야하는 경우, 액션만 있는 self-transition을 만들 수 있다.

```typescript
import { createMachine } from "xstate";

const countMachine = createMachine({
  context: {
    count: 0,
  },
  on: {
    increment: {
      actions: assign({
        count: ({ context, event }) => context.count + event.value,
      }),
    },
    decrement: {
      actions: assign({
        count: ({ context, event }) => context.count - event.value,
      }),
    },
  },
});
```
