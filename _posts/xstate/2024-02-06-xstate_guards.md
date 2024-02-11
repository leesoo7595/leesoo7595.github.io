---
layout: post
title: "[XState] Guards"
date: 2024-02-06
tags: [xstate]
comments: true
---

# Guards

가드는 머신이 이벤트를 통과할 때 확인하는 조건 함수이다. true이면 다음 state로의 트랜지션을 진행하고, false이면 나머지 조건에 따라 다음 state로 넘어간다.

가드된 트랜지션은 가드가 true로 평가되는 경우에만 활성화되는 트랜지션이다. 가드는 트랜지션 활성화 여부를 결정한다. 모든 트랜지션이 가드된 트랜지션이 될 수 있다.

가드는 true 또는 false를 리턴하는 순수 동기함수여야한다.

```typescript
const feedbackMachine = createMachine(
  {
    // ...
    states: {
      form: {
        on: {
          "feedback.submit": {
            guard: "isValid",
            target: "submitting",
          },
        },
      },
      submitting: {
        // ...
      },
    },
  },
  {
    guards: {
      isValid: ({ context }) => {
        return context.feedback.length > 0;
      },
    },
  }
);
```

## Multiple guarded transitions

특정 상황에서 단일 이벤트가 다른 state로 트랜지션되도록 하려면, 가드된 트랜지셙 배열을 제공할 수 있다. 각 트랜지션은 순서대로 테스트되고, 가드가 true로 평가되는 첫 번째 트랜지션이 사용된다.

배열의 마지막 트랜지션으로 사용할 기본 트랜지션을 지정할 수 있다. 가드 중 어느 것도 true로 평가되지 않으면 기본 트랜지션이 사용된다.

```typescript
const feedbackMachine = createMachine({
  // ...
  prompt: {
    on: {
      "feedback.provide": [
        // Taken if 'sentimentGood' guard evaluates to `true`
        {
          guard: "sentimentGood",
          target: "thanks",
        },
        // Taken if none of the above guarded transitions are taken
        // and if 'sentimentBad' guard evaluates to `true`
        {
          guard: "sentimentBad",
          target: "form",
        },
        // Default transition
        { target: "form" },
      ],
    },
  },
});
```

## Inline guards

가드를 인라인 함수로 정의할 수 있다. 하지만 직렬화된 가드를 사용하는 것이 권장된다.

```typescript
on: {
  event: {
    guard: ({ context, event }) => true,
    target: 'someState'
  }
}
```

## Guard object

가드는 제공된 가드 구현을 참조하는 가드 타입인 type과 선택적 매개변수인 params를 가진 객체로 정의할 수 있다:

```typescript
const feedbackMachine = createMachine(
  {
    // ...
    states: {
      // ...
      form: {
        on: {
          submit: {
            guard: { type: "isValid", params: { maxLength: 50 } },
            target: "submitting",
          },
        },
      },
      // ...
    },
  },
  {
    guards: {
      isValid: ({ context }, params) => {
        return (
          context.feedback.length > 0 &&
          context.feedback.length <= params.maxLength
        );
      },
    },
  }
);

const feedbackActor = createActor(
  feedbackMachine.provide({
    guards: {
      isValid: ({ context }, params) => {
        return (
          context.feedback.length > 0 &&
          context.feedback.length <= params.maxLength &&
          isNotSpam(context.feedback)
        );
      },
    },
  })
).start();
```

## Higher-level guards

XState는 다른 가드를 구성하는 상위 레벨 가드를 제공한다. 상위 레벨 가드에는 3가지가 있다:

- `and([...])`
- `or([...])`
- `not(...)`

```typescript
on: {
  event: {
    guard: and(["isValid", "isAuthorized"]);
  }
}

on: {
  event: {
    guard: and(["isValid", or(["isAuthorized", "isGuest"])]);
  }
}
```

## In-state guards

`stateIn(stateValue)` 가드를 사용하여 현재 state가 제공된 stateValue와 일치하는지 확인할 수 있다. parallel states에 유용하다.

```typescript
on: {
  event: {
    guard: stateIn('#state1');
  },
  anotherEvent: {
    guard: stateIn({ form: 'submitting' })
  }
}
```

In-state 가드는 상태 노드가 아닌 전체 머신의 상태와 일치한다. 일반적으로는 일반 state에서 in-state 가드를 사용할 필요는 없다. 상태머신에서 트랜지션을 모델링하여 in-state 가드를 사용할 필요가 없도록 하는게 좋다.
