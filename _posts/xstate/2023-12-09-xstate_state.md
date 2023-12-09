---
layout: post
title: "[XState] state"
date: 2023-12-09
tags: [xstate]
comments: true
---

이 글은 [XState 공식문서](https://stately.ai/docs/states)를 읽고 정리한 글이다.

# State

state는 간단한 Paused와 Playing이 가능한 머신의 상태 또는 모드를 나타낸다. state machine은 같은 시간대에서 하나의 state만 존재할 수 있다.

이 상태들을 **finite(유한한)**라고 한다; 머신은 사용자가 미리 정의한 상태에서만 이동할 수 있다.

<img width="598" alt="스크린샷 2023-12-08 오후 8 50 28" src="https://github.com/leesoo7595/leesoo7595.github.io/assets/39291812/c76d1ed7-bf1b-4fbb-9d09-06ca6cd64438">

## State object

상태 객체는 실행중인 머신(actor)의 현재 상태를 나타내고, 다음 프로퍼티들을 포함한다:

- value : 현재 state 값
  - a string
  - an object nested states
- context : 현재 context (extended state)
- meta : state node meta data를 포함한 객체

```javascript
const feedbackMachine = createMachine({
  id: "feedback",
  initial: "question",
  context: {
    feedback: "",
  },
  states: {
    question: {
      meta: {
        question: "How was your experience?",
      },
    },
  },
});

const actor = createActor(feedbackMachine);
actor.start();

console.log(actor.getSnapshot());
// Logs an object containing:
// {
//   value: 'question',
//   context: {
//     feedback: ''
//   },
//   meta: {
//     'feedback.question': {
//       question: 'How was your experience?'
//     }
//   }
// }
```

## state snapshots 접근하기

`.getSnapshot()` 메서드를 구독하거나 읽어서 actor가 만든 방출된 상태(snapshot)에 접근할 수 있다.

```javascript
const actor = createActor(feedbackMachine);

actor.subscribe((snapshot) => {
  console.log(snapshot);
  // logs the current snapshot state, e.g.:
  // { value: 'question', ... }
  // { value: 'thanks', ... }
});

actor.start();

console.log(actor.getSnapshot());
// logs { value: 'question', ... }
```

## State value

상태가 중첩된 상태 머신은(또는 statechart) 각 노드가 상태 노드인 유사 트리구조이다. root 상태 노드는 전체 머신을 나타내는 최상위 상태 노드이다. root 노드는 자식 상태 노드들을 가지고 있고, 각 자식 상태노드들은 또 자식 상태노드들을 가지고 있고.. 등등등으로 되어있다.

state value는 머신 안의 모든 활성화된 상태 노드들을 나타내는 객체이다. 자식 상태노드가 없는 상태노드를 가진 상태 머신의 경우, state value는 string이다:

- state.value === 'question'
- state.value === 'thanks'
- state.value === 'closed'

부모 상태노드를 가진 상태머신은 state value가 객체이다:

- state.value === { form: 'invalid' }

이것은 상태머신이 액티브한 form이란 키를 가진 자식 노드를 가지고 있고, 그 키는 `'invalid'`를 가지고 있다는 것을 나타낸다.

상태 머신이 `parallel state nodes`이면, state value는 각 상태 노드 영역에 대해 여러 키들을 포함하는 객체이다.

```javascript
state.value ===
  {
    monitor: "on",
    mmode: "dark",
  };
```

상태 머신은 root 상태 노드 외에 다른 상태 노드가 없을 수도 있다. 이런 상태 머신의 경우, state value는 null이다.

## State context

상태 머신은 context를 가질 수 있다. context는 머신의 state에서 더 연장된 개념을 나타내는 객체이다. context는 불변성을 가지고 있고, `assign`이라는 action에 의해서만 수정할 수 있다. `state.context` 프로퍼티를 읽어서 현재 context를 가져올 수 있다.

```javascript
const currentState = feedbackActor.getSnapshot();

console.log(currentState.context);
// logs { feedback: '' }
```

- context가 machine config에 명시되어있지 않다면 객체는 기본적으로 비어있다.
- 이 객체를 임의로 수정하면 안됨; immutable/read-only로 관리되어야 한다.

## State children

`state.children` 프로퍼티는 현재 상태에서 모든 현재 생성/호출한 actors를 나타낸다.
이것은 actor ID들의 key들과 ActorRef 인스턴스들을 나타내는 value들이 있는 객체이다.

- 생성/호출된 actors를 ID로 접근이 가능하다.
- 그래서 생성/호출된 actors ID를 넘겨줘야한다.
- 멈춘 actors의 경우 보이지 않는다.

## `state.can(eventType)`

이 메소드는 event 객체가 머신 actor로 전송될 때 상태 변경을 일으킬지 여부를 결정한다. 이 메소드는 상태가 전송된 이벤트로 인해 상태가 변할 것이라면 true를 반환한다. 아니면 false를 반환한다.

```javascript
const feedbackMachine = createMachine({
  //...
  states: {
    form: {
      //...
      on: {
        "feedback.submit": {
          guard: "isValid",
          target: "thanks",
        },
      },
    },
  },
});

const feedbackActor = createActor(feedbackMachine).start();

const currentState = feedbackActor.getSnapshot();

console.log(currentState.can({ type: "feedback.submit" }));
// logs `true` if the 'feedback.submit' event will cause a transition, which will occur if:
// - the current state is 'form'
// - the 'isValid' guard evaluates to `true`
```

주어진 상태 및 이벤트 객체에 대해 트랜지션이 활성화되어있으면 state는 'changed(변경된)' 것으로 간주된다.

## `state.hasTag(tag)`

`state.hasTag(tag)` 메소드는 현재 상태 값에 지정된 태그가 있는 상태 노드가 있는지 확인한다. 이것은 상태가 특정 상태인지 또는 상태가 특정 상태 그룹의 구성원인지 여부를 판단하는 데 유용하다.

```javascript
const feedbackMachine = createMachine({
  // ...
  states: {
    submitting: {
      tags: ["loading"],
      // ...
    },
  },
});

const feedbackActor = createActor(feedbackMachine).start();

const currentState = feedbackActor.getSnapshot();

const showLoadingSpinner = currentState.hasTag("loading");
```

`state.hasTag(tag)`가 머신의 변경에 더 resilient(탄력적)이므로 state.matches(stateValue)보다 더 선호된다.

## `state.matches(stateValue)`

`state.matches(stateValue)` 메소드는 현재 `state.value`가 주어진 stateValue와 일치하는지 여부를 확인한다. 만약 현재 `state.value`가 제공된 stateValue의 하위집합인 경우 메서드는 true를 반환한다.

```javascript
// state.value === 'question'
state.matches("question"); // true

// state.value === { form: 'invalid' }
state.matches("form"); // true
state.matches("question"); // false
state.matches({ form: "invalid" }); // true
state.matches({ form: "valid" }); // false
```

## `state.output`

`state.output` 프로퍼티는 상태머신이 최상위 final 상태에 있을 때의 출력 데이터를 나타낸다. 다시 말하면 `state.status === 'done'`일 때를 나타낸다.

```javascript
const state = actor.getSnapshot();

if (state.status === "done") {
  console.log(state.output);
} else {
  // Output does not exist
  state.output === undefined;
}
```

## `state.getMeta()`

```javascript
const feedbackMachine = createMachine({
  id: "feedback",
  // ...
  states: {
    form: {
      meta: {
        view: "shortForm",
      },
    },
  },
});

const feedbackActor = createActor(feedbackMachine).start();

// Assume the current state is 'form'
const currentState = feedbackActor.getSnapshot();
console.log(currentState.getMeta());
// logs { 'feedback.form': { view:'shortForm' } }
```
