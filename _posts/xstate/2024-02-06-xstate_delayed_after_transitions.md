---
layout: post
title: "[XState] Delayed (after) transitions"
date: 2024-02-06
tags: [xstate]
comments: true
---

# Delayed (after) transitions

Delayed transitions은 설정된 시간 후에만 발생하는 트랜지션이다. 지연된 트랜지션은 앱 로직에 타임아웃과 intervals를 설정하는 데 유용하다. 타이머가 끝나기 전에 다른 이벤트가 발생하면 트랜지션이 완료되지 않는다.

지연된 트랜지션은 after 속성에 밀리초 단위로 정의된다. after 트랜지션이라고도 한다.

```typescript
import { createMachine } from "xstate";

const pushTheButtonGame = createMachine({
  initial: "waitingForButtonPush",
  states: {
    waitingForButtonPush: {
      after: {
        5000: {
          target: "timedOut",
          actions: "logThatYouGotTimedOut",
        },
      },
      on: {
        PUSH_BUTTON: {
          actions: "logSuccess",
          target: "success",
        },
      },
    },
    success: {},
    timedOut: {},
  },
});
```

## Delays

인라인, 참조, 표현식 등 몇 가지 방법으로 delays를 정의할 수 있다.

### Inlined delays

```typescript
const machine = createMachine({
  initial: "idle",
  states: {
    idle: {
      after: {
        1000: { target: "nextState" },
      },
    },
    nextState: {},
  },
});
```

1000밀리초 이후에 nextState 상태로 트랜지션된다.

### Referenced delays

문자열 delay 키를 지정하고 실제 지연 시간을 별도로 제공하여 참조 delay를 정의할 수도 있다.

```typescript
import { setup } from "xstate";

const machine = setup({
  delays: {
    timeout: 1000,
  },
}).createMachine({
  initial: "idle",
  states: {
    idle: {
      after: {
        timeout: { target: "nextState" },
      },
    },
    nextState: {},
  },
});
```

### Lifecycle

delayed transition 타이머는 state에서 exited되면 취소된다.
