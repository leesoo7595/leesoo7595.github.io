---
layout: post
title: "[XState] Events and transitions"
date: 2023-12-12
tags: [xstate]
comments: true
---

이 글은 [XState의 공식문서](https://stately.ai/docs/transitions)를 읽고 정리한 글이다.

# Events and transitions

트랜지션은 하나의 유한한 상태가 트리거된 이벤트로 인해 다르게 바뀌는 것을 말한다.

이벤트란 트랜지션을 유발하는 신호, 트리거, 또는 메세지이다. actor가 이벤트를 수신하면 머신은 현재 상태에서 해당 이벤트에 활성화된 트랜지션(enabled transitions)이 있는지 확인한다. 있다면, 머신은 해당 트랜지션을 가져와서 작업을 실행한다.

트랜지션은 **deterministic(결정론적)**이다; 상태와 이벤트의 각 조합이 항상 동일한 다음 상태를 가리키고 있다. 상태 머신이 이벤트를 받으면, 활성화된 유한 상태만 확인하여 해당 이벤트에 대한 트랜지션이 있는지 확인한다. 이러한 트랜지션을 `enabled transitions`이라고 한다. 활성화된 트랜지션이 있으면 상태 머신이 트랜지션의 작업을 실행한 다음, 다음 대상 상태(target state)로 트랜지션한다.

트랜지션은 `state:` 안의 `on:`으로 나타낸다:

```javascript
import { createMachine } from 'xstate';
const feedbackMachine = createMachine({
  id: 'feedback',
  initial: 'question',
  states: {
    question: {
      on: {
        'feedback.good': {
          target: 'thanks'
        }
      }
    }
    thanks: {}
  },
});
```

## Event objects

XState에서 이벤트는 type 프로퍼티와 선택적인 payload를 가진 이벤트 객체를 나타낸다:

- type 프로퍼티는 이벤트 타입을 의미하는 string이다.
- payload는 이벤트에 대해 추가적인 데이터를 포함하는 객체이다.

```javascript
feedbackActor.send({
  // The event type
  type: "feedback.update",
  // Additional payload
  feedback: "This is great!",
  rating: 5,
});
```

## 트랜지션 선택하기

트랜지션은 가장 깊은 아래 상태를 먼저 확인해서 선택된다. 트랜지션이 활성화되어있다면(guard가 통과된 등등..), 트랜지션이 수행된다. 그렇지 않은 경우, 부모 상태가 확인되는 식으로 진행된다.

1. 가장 깊은 활성 상태 노드(atomic state node)에서 시작한다.
2. 트랜지션이 활성화된 경우, (guard가 없거나 해당 guard가 true로 평가된 경우) 해당 트랜지션이 선택된다.
3. 활성화된 트랜지션이 없는 경우, 부모 상태 노드로 올라가서 1번을 반복.
4. 마지막으로 활성화된 트랜지션이 없는 경우, 트랜지션이 선택되지 않으며, 상태는 바뀌지 않는다.

## Self-transitions

상태는 스스로 트랜지션이 가능하다. 이것을 self-transition이라고 하는데, 유한한 상태를 바꾸지 않고 context를 바꾸거나 작업을 실행하는 데에 유용하다. 상태를 재시작할때 self-transition을 쓸 수도 있다.

### Root self-transitions:

```javascript
import { createMachine, assign } from "xstate";

const machine = createMachine({
  context: { count: 0 },
  on: {
    someEvent: {
      // No target
      actions: assign({
        count: ({ context }) => context.count + 1,
      }),
    },
  },
});
```

<img width="512" alt="스크린샷 2023-12-09 오후 3 33 41" src="https://github.com/leesoo7595/leesoo7595.github.io/assets/39291812/02eb1594-8ab5-4d33-ab1d-5450884f9067">

### Self-transitions on states:

```javascript
import { createMachine, assign } from "xstate";

const machine = createMachine({
  context: { count: 0 },
  initial: "inactive",
  states: {
    inactive: {
      on: { activate: { target: "active" } },
    },
    active: {
      on: {
        someEvent: {
          // No target
          actions: assign({
            count: ({ context }) => context.count + 1,
          }),
        },
      },
    },
  },
});
```

<img width="597" alt="스크린샷 2023-12-09 오후 3 39 42" src="https://github.com/leesoo7595/leesoo7595.github.io/assets/39291812/785adb74-7b3c-4626-946a-071f33c1917a">

## 상태 사이의 트랜지션

트랜지션은 보통 두 개의 상태 사이에 존재한다. 이 트랜지션은 target을 상태 키로 설정하여 정의한다.

```javascript
const feedbackMachine = createMachine({
  // ...
  states: {
    form: {
      on: {
        submit: {
          // Target is the key of the sibling state
          target: "submitting",
        },
      },
    },
    submitting: {
      // ...
    },
  },
});
```

## 부모에서 자식으로 가는 트랜지션

상태 머신이 이벤트를 받을 때, 가장 깊은 상태부터 확인하여 활성화된 트랜지션이 있는지 본다. 그리고 활성화된 트랜지션이 없다면, 부모 상태르르 확인하는 식으로 가게되어 root 상태에 도달할 때까지 진행된다.

어떤 상태가 활성화되어있는지 상관없이 이벤트가 상태로 트랜지션되도록 하려는 경우 유용한 패턴은 부모 상태에서 자식 상태로 트랜지션하는 것이다.

## Re-entering

기본적으로 상태 머신이 어떤 상태에서 같은 상태로 트랜지션되거나 부모 상태에서 해당 부모 상태의 자식으로 트랜지션할 때 상태 머신은 다시 상태로 들어가지 않는다; 즉, 부모 상태의 종료, 진입 동작을 실행하지 않는다. 기존에 호출된 actor를 중지하거나 새로 호출된 actor를 시작하지 않는다.

이것을 바꿀 수 있도록 하는게 `reenter` 프로퍼티이다: 만약에 부모 상태에서 다시 re-enter하려면, `reenter: true`로 설정할 수 있다. 이것은 상태가 자체 또는 하위 상태로 전환할 때 다시 들어가서 상태의 종료 및 진입 동작을 실행한다.

### Self-transitions with `reenter: true`:

```javascript
import { createMachine } from "xstate";

const machine = createMachine({
  initial: "someState",
  states: {
    someState: {
      entry: () => console.log("someState entered"),
      exit: () => console.log("someState exited"),
      on: {
        "event.normal": {
          target: "someState", // or no target
        },
        "event.thatReenters": {
          target: "someState", // or no target
          reenter: true,
        },
      },
    },
  },
});

const actor = createActor(machine);
actor.start();

actor.send({ type: "event.normal" });
// Does not log anything

actor.send({ type: "event.thatReenters" });
// Logs:
// "someState exited"
// "someState entered"
```

위 예시를 통해 reenter 옵션에 해당하는 경우, 액터가 exit로 나가고, entry로 재진입하는 것을 콘솔로 확인할 수 있다.

### Parent-child (or descendent) transitions with `reenter: true`:

```javascript
const machine = createMachine({
  initial: "parentState",
  states: {
    parentState: {
      entry: () => console.log("parentState entered"),
      exit: () => console.log("parentState exited"),
      on: {
        "event.normal": {
          target: ".someChildState",
        },
        "event.thatReenters": {
          target: ".otherChildState",
          reenter: true,
        },
      },
      initial: "someChildState",
      states: {
        someChildState: {
          entry: () => console.log("someChildState entered"),
          exit: () => console.log("someChildState exited"),
        },
        otherChildState: {
          entry: () => console.log("otherChildState entered"),
          exit: () => console.log("otherChildState exited"),
        },
      },
    },
  },
});

const actor1 = createActor(machine);
actor1.start();
actor1.send({ type: "event.normal" });
// Logs:
// "someChildState exited"
// "someChildState entered"

const actor2 = createActor(machine);
actor2.start();
console.log("---");
actor2.send({ type: "event.thatReenters" });
// Logs:
// "someChildState exited"
// "parentState exited"
// "parentState entered"
// "otherChildState entered"
```

## Transitions to any state

아무 state로 트랜지션하기

- `{ target: 'sibling.child.grandchild' }`
- `{ target: '.child.grandchild' }`
- `{ target: '#specificState' }`

## Forbidden transitions

트랜지션 막기

- `{ on: { forbidden: {} } }`

트랜지션을 생략하는 것과 다르게, 트랜지션 선택 알고리즘이 검색을 중지한다.

- `{ on: { forbidden: { target: undefined } } }` 이렇게도 사용할 수 있다.

## Wildcard transitions

와일드카드 트랜지션은 모든 이벤트와 일치하는 트랜지션이다. 와일드 카드 문자인 \*를 이벤트 타입으로 사용하여 정의된다:

```typescript
import { createMachine } from "xstate";

const feedbackMachine = createMachine({
  initial: "asleep",
  states: {
    asleep: {
      on: {
        // This transition will match any event
        "*": { target: "awake" },
      },
    },
    awake: {},
  },
});
```

와일트카드 트랜지션은 이럴때 유용하다:

- 다른 트랜지션으로 처리되지 않는 이벤트를 처리
- 모든 이벤트를 처리하는 포괄적인 트랜지션으로 사용할 수 있다.

와일드카드 트랜지션은 우선순위가 가장 낮다; 다른 트랜지션이 활성화되어 있지 않은 경우에만 사용된다.

## Partial wildcard transitions

부분적인 와일드카드 트랜지션은 특정 프리픽스(접두사)로 시작하는 모든 이벤트와 일치하는 트랜지션이다. 이벤트 설명자는 dot 뒤에 와일드카드 문자인 \*를 이벤트 타입으로 사용하여 정의한다.

```typescript
import { createMachine } from "xstate";

const feedbackMachine = createMachine({
  initial: "prompt",
  states: {
    prompt: {
      on: {
        // This will match any event that starts with 'feedback.':
        // 'feedback.good', 'feedback.bad', etc.
        "feedback.*": { target: "form" },
      },
    },
    form: {},
    // ...
  },
});
```

와일드 카드 문자는(\*) 점(.) 다음에 오는 이벤트 설명자의 접미사에만 사용할 수 있다.

### valid wildcard examples

- ✅ `mouse.*`: matches mouse.click, mouse.move, etc.
- ✅ `mouse.click.*`: matches mouse.click.left, mouse.click.right, etc.

### invalid wildcard

- 🚫 `mouse*`: invalid; does not match any event.
- 🚫 `mouse.*.click`: invalid; `*` cannot be used in the middle of an event descriptor.
- 🚫 `*.click`: invalid; `*` cannot be used in the prefix of an event descriptor.
- 🚫 `mouse.click*`: invalid; does not match any event.
- 🚫 `mouse.*.*`: invalid; `*` cannot be used in the middle of an event descriptor.

## Multiple transitions in parallel states

parallel states는 동시에 활성화할 수 있는 여러 영역이 있으므로, 여러 트랜지션이 동시에 활성화할 수 있다. 이 경우 해당 영역에 활성화된 모든 트랜지션이 적용된다.

이는 여러 타겟들이 문자열 배열로 지정된다.
