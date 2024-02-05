---
layout: post
title: "[XState] Eventless always transitions"
date: 2024-02-05
tags: [xstate]
comments: true
---

# Eventless (always) transitions

Eventless transitions(이벤트 없는 전환)은 명시적인 이벤트 없이 발생하는 전환이다. 이러한 트랜지션은 트랜지션이 활성화되어있을 때 항상 수행된다.

이벤트 없는 트랜지션은 `always` state 프로퍼티에 지정되며, 흔히 always 트랜지션이라고 한다.

```typescript
import { createMachine } from "xstate";
const machine = createMachine({
  states: {
    form: {
      initial: "valid",
      states: {
        valid: {},
        invalid: {},
      },
      always: {
        guard: "isValid",
        target: "valid",
      },
    },
  },
});
```

## Eventless transitions and guards

이벤트 없는 트랜지션은 일반 트랜지션이 수행된 직후에 수행된다. 예를 들어 guard가 참인 경우와 같이 활성화된 경우에만 수행된다. 이벤트 없는 트랜지션은 특정 조건이 참일 때 작업을 수행하는데에 더 유용하다.

## Avoid infinite loops

always 트랜지션은 항상 실행되므로 무한 루프를 만들지 않도록 주의해야 한다. XState는 대부분의 무한 루프 시나리오를 방지하는데 도움이 된다.

target이나 guard가 없는 이벤트 없는 트랜지션은 무한 루프를 유발한다. 가드와 액션을 사용하는 트랜지션은 가드가 계속 참으로 반환되면 무한 루프가 발생할 수 있다.

이벤트없는 트랜지션은 다음을 사용하여 정의해야한다:

- target
- guard + target
- guard + actions
- guard + target + actions

target가 선언된 경우 값은 현재 상태 노드와 달라야한다.

## When to use

state 변경이 필요하지만 특정 트리거가 없는 경우에 이벤트 없는 전환이 유용할 수 있다.

```typescript
import { createMachine } from "xstate";

const machine = createMachine({
  id: "kettle",
  initial: "lukewarm",
  context: {
    temperature: 80,
  },
  states: {
    lukewarm: {
      on: {
        boil: { target: "heating" },
      },
    },
    heating: {
      always: {
        guard: ({ context }) => context.temperature > 100,
        target: "boiling",
      },
    },
    boiling: {
      entry: ["turnOffLight"],
      always: {
        guard: ({ context }) => context.temperature <= 100,
        target: "heating",
      },
    },
  },
  on: {
    "temp.update": {
      actions: ["updateTemperature"],
    },
  },
});
```

## Eventless transitions and Typescript

이벤트 없는 트랜지션은 모든 이벤트에 의해 잠재적으로 활성화될 수 있으므로 이벤트 타입은 모든 가능한 이벤트의 union이다.
