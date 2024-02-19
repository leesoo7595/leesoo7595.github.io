---
layout: post
title: "[XState] Initial State"
date: 2024-02-19
tags: [xstate]
comments: true
---

# Initial States

state machine이 시ㄱ되면, initial state로 먼저 진입한다. 머신은 최상위에 오직 하나의 initial state를 가질 수 있다; 만약 여러개의 초기 상태들이 있다면, 머신은 어디서 시작해야할지 알 수 없을 것이다.

XState에서는 초기 상태는 `initial` 프로퍼티로 인해 정의된다.

```typescript
const feedbackMachine = createMachine({
  id: "feedback",

  // Initial state
  initial: "prompt",

  // Finite states
  states: {
    prompt: {
      /* ... */
    },
    // ...
  },
});
```

## Specifying an initial state

전형적으로 상태머신은 다수의 유한한 상태들(finite states)이 있을 수 있다. 머신의 `initial` 프로퍼티는 머신이 시작해야하는 초기 상태를 지정한다.

부모 상태 또한 `initial` 프로퍼티에 초기 상태를 지정해야한다.

# Finite states

유한한 상태는 상태머신이 주어진 시간안에 있을 수 있는 가능한 상태 중 하나이다. 상태머신은 가능한 상태의 수가 제한되어 있기 때문에 이를 "finite" 유한이라고 한다. 상태는 머신이 해당 상태에 있을 때 어떻게 "동작"하는지를 나타낸다; 이것은 그것의 status(상태) 또는 모드를 의미한다.

예를 들어 피드백 양식의 경우 양식을 작성 중인 상태이거나 양식을 제출 중인 상태일 수 있다. 양식을 작성하는 동시에 제출할 수는 없으며, 이는 "불가능한 상태"이다.

상태머신은 항상 초기 상태에서 시작하고, final state(마지막 상태)에서 끝난다. 상태머신은 항상 유한한 상태에 있다.

```typescript
const feedbackMachine = createMachine({
  id: "feedback",

  // Initial state
  initial: "prompt",

  // Finite states
  states: {
    prompt: {
      /* ... */
    },
    form: {
      /* ... */
    },
    thanks: {
      /* ... */
    },
    closed: {
      /* ... */
    },
  },
});
```

유한 상태와 컨텍스트를 결합하여 머신의 전체 상태를 구성할 수 있따.

```typescript
const feedbackMachine = createMachine({
  id: "feedback",
  context: {
    name: "",
    email: "",
    feedback: "",
  },

  initial: "prompt",
  states: {
    prompt: {
      /* ... */
    },
  },
});

const feedbackActor = createActor(feedbackMachine).start();

// Finite state
console.log(feedbackActor.getSnapshot().value);
// logs 'prompt'

// Context ("extended state")
console.log(feedbackActor.getSnapshot().context);
// logs { name: '', email: '', feedback: '' }
```

## Initial State

초기 상태는 머신이 시작할 떄의 상태이다. 이것은 initial 프로퍼티를 통해 정의된다.

## State nodes

XState에서 **상태 노드**는 전체 statechart 트리를 구성하는 유한 상태 "노드"이다. 상태노드는 루트 머신 config에 포함한 다른 상태 노드의 states 프로퍼티에 정의된다.

```typescript
// The machine is the root state node
const feedbackMachine = createMachine({
  id: "feedback",
  initial: "prompt",

  // State nodes
  states: {
    // State node
    prompt: {
      /* ... */
    },
    // State node
    form: {
      /* ... */
    },
    // State node
    thanks: {
      /* ... */
    },
    // State node
    closed: {
      /* ... */
    },
  },
});
```

## Tags

상태노드는 tags를 가질 수 있다. tags는 상태노드를 그룹화하거나 카테고라이징하는데 도움이 되는 문자열 용어이다.
예를 들면, loading 태그를 사용하여 데이터가 로드 중인 상태를 나타내는 상태노드를 표시하고, `state.hasTag(tag)`를 사용하여 해당 태그가 지정된 상태 노드가 상태에 포함되어 있는지 확인할 수 있다.

```typescript
const feedbackMachine = createMachine({
  id: 'feedback',
  initial: 'prompt',
  states: {
    prompt: {
      tags: ['visible'],
      // ...
    },
    form: {
      tags: ['visible'],
      // ...
    },
    thanks: {
      tags: ['visible', 'confetti'],
      // ...
    },
    closed: {
      tags: ['hidden'],
    },
  },
});

const feedbackActor = createActor(feedbackMachine).start();

console.log(feedbackActor..getSnapshot().hasTag('visible'));
// logs true
```

## Meta

메타데이터는 상태노드의 관련 속성을 설명하는 스태틱 데이터이다. 상태노드의 `.meta` 프로퍼티에서 메타데이터를 지정할 수 있다. 이는 UI에서 상태노드에 대한 정보를 표시하거나 문서화를 하는데 유용할 수 있다.

`state.meta` 프로퍼티는 모든 활성 상태노드에서 `.meta` 데이터를 수집하여 상태노드의 ID를 key로, 메타데이터를 값으로 사용하여 객체로 배치한다.

```typescript
const feedbackMachine = createMachine({
  id: "feedback",
  initial: "prompt",
  meta: {
    title: "Feedback",
  },
  states: {
    prompt: {
      meta: {
        content: "How was your experience?",
      },
    },
    form: {
      meta: {
        content: "Please fill out the form below.",
      },
    },
    thanks: {
      meta: {
        content: "Thank you for your feedback!",
      },
    },
    closed: {},
  },
});

const feedbackActor = createActor(feedbackMachine).start();

console.log(feedbackActor.getSnapshot().meta);
// logs the object:
// {
//   feedback: {
//     title: 'Feedback',
//   },
//   'feedback.prompt': {
//     content: 'How was your experience?',
//   }
// }
```

## Transitions

트랜지션은 유한 상태에서 다른 상태로 어떻게 이동하는지를 의미한다. on 프로퍼티로 정의할 수 있다.

## Targets

트랜지션의 target 프로퍼티는 머신이 트랜지션이 일어났을 때 어디로 가야할지 지정해주는 곳이다.

## Identifying state nodes

상태는 유니크한 ID로 식별할 수 있다: `id: 'myState'` 이는 부모 상태가 다른 경우에도 다른 상태의 상태를 타겟팅하는게 유용하다:

```typescript
import { createMachine, createActor } from "xstate";

const feedbackMachine = createMachine({
  initial: "prompt",
  states: {
    // ...
    closed: {
      id: "finished",
      type: "final",
    },
    // ...
  },
  on: {
    "feedback.close": {
      // Target the `.closed` state by its ID
      target: "#finished",
    },
  },
});
```
