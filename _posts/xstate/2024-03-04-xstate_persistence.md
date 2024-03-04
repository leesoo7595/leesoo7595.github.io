---
layout: post
title: "[XState] Persistence"
date: 2024-03-04
tags: [xstate]
comments: true
---

# Persistence

액터는 내부 상태를 보존하고 나중에 복원할 수 있다. 지속성(Persistence)은 액터의 상태를 로컬스토리지나 데이터베이스 같은 영구 저장소에 저장하는 것을 말한다. 복원(Restoration)은 영구 저장소에서 액터의 상태를 복원하는 것을 말한다.

지속성은 브라우저를 다시 로드하는 동안 상태를 유지하는 데 유용하다. 백엔드 어플리케이션에서 지속성은 워크플로우가 여러 요청에 걸쳐있고, 서비스 재시작에도 살아남고, 내결함성, 장기실행프로세스를 할 수 있고, autiable과 traceable하게 해준다.

XState에서는 `actor.getPersistedSnapshot()`를 통해서 스냅샷을 가져와 `createActor(behavior, { snapshot: restoredState }).start()`를 통해 복원할 수 있다.

```typescript
const feedbackActor = createActor(feedbackMachine).start();

// Get state to be persisted
const persistedState = feedbackActor.getPersistedSnapshot();

// Persist state
localStorage.setItem("feedback", JSON.stringify(persistedState));

// Restore state
const restoredState = JSON.parse(localStorage.getItem("feedback"));

const restoredFeedbackActor = createActor(feedbackMachine, {
  snapshot: restoredState,
}).start();
```

## Persisting state

`actor.getPersistedSnapshot()`를 사용해서 보존되는 상태를 얻을 수 있다.

```typescript
const feedbackActor = createActor(feedbackMachine).start();

// Get state to be persisted
const persistedState = feedbackActor.getPersistedSnapshot();
```

내부 상태는 머신뿐만 아니라 모든 액터에서 지속될 수 있다. 지속 상태는 액터의 내부 상태를 나타내는 반면, 스냅샷은 액터가 마지막으로 방출한 값을 나타내므로 `actor.getSnapshot()`의 스냅샷과 동일하지 않다.

```typescript
const promiseActor = fromPromise(() => Promise.resolve(42));

// Get the last emitted value
const snapshot = promiseActor.getSnapshot();
console.log(snapshot);
// logs 42

// Get the persisted state
const persistedState = promiseActor.getPersistedSnapshot();
console.log(persistedState);
// logs { status: 'done', data: 42 }
```

## Restoring state

`createActor(logic, { snapshot: restoredState })`의 두번째 인수의 state 옵션에 지속 상태를 전달하여 지속 상태로 복원할 수 있다.

```typescript
// Get persisted state
const restoredState = JSON.parse(localStorage.getItem("feedback"));

// Restore state
const feedbackActor = createActor(feedbackMachine, {
  snapshot: restoredState,
});

feedbackActor.start();
```

머신 액터의 액션의 이미 실행된 것으로 간주되므로 다시 실행되지 않는다. 그러나 호출은 다시 시작되고 스폰된 액터는 재귀적으로 복원된다.

## Deep persistence

머신 액터에서 state 지속 & 복원은 깊어서, 모든 호출, 스폰된 액터는 재귀적으로 지속, 복원된다.

```typescript
const feedbackMachine = createMachine({
  // ...
  states: {
    form: {
      invoke: {
        id: "form",
        src: formMachine,
      },
    },
  },
});

const feedbackActor = createActor(feedbackMachine).start();

// Persist state
const persistedState = feedbackActor.getPersistedSnapshot();
localStorage.setItem("feedback", JSON.stringify(persistedState));

//  ...

// Restore state
const restoredState = JSON.parse(localStorage.getItem("feedback"));

const restoredFeedbackActor = createActor(feedbackMachine, {
  snapshot: restoredState,
}).start();
// Will restore both the feedbackActor and the invoked form actor at
// their persisted states
```

## Persisting state machine values

상태머신 액터의 유한 상태 값만 유지하려면 `machine.resolveState(...)` 메서드를 사용할 수 있다.

```typescript
import { someMachine } from "./someMachine";

const restoredStateValue = localStorage.getItem("someState");
// Assume that this is "pending"

const resolvedState = someMachine.resolveState({
  value: restoredStateValue,
  // context: { ... }
});

// Restore the actor
const restoredActor = createActor(someMachine, {
  snapshot: resolvedState,
});

restoredActor.start();
```

## Event sourcing

상태 지속의 대안으로 이벤트 소싱이란 것이 있는데, 이는 해당 상태를 일으킨 이벤트를 재생하여 액터의 상태를 복원하는 방법이다. 이벤트 소싱은 호환되지 않는 상태가 발생할 가능성이 적고, 동작을 재생할 수 있기 때문에 상태 지속보다 더 안정적일 수 있다.

이벤트 소싱을 구현하는 한 가지 방법은 검사 API를 사용하여 이벤트가 발생할 때 이벤트를 지속한 다음, 이를 재생하여 액터의 상태를 복원하는 것이다.

```typescript
const events = [];

const someActor = createActor(someMachine, {
  // Inspect and persist events
  inspect: (inspectionEvent) => {
    if (inspectionEvent.type === "@xstate.event") {
      const event = inspectionEvent.event;

      // Only listen for events sent to the root actor
      if (inspectionEvent.actorRef !== someActor) {
        return;
      }

      events.push(event);
    }
  },
});

someActor.start();

// ...

// Assuming the events are stored somewhere, e.g. in localStorage,
// you can replay them to restore the state of the actor

const restoredActor = createActor(someMachine);
restoredActor.start();

for (const event of events) {
  // Replay events
  restoredActor.send(event);
}
```

## Caveats

상태를 유지하고 복원할 때 주의해야할 몇 가지 주의 사항이 있다:

- 호환되지않는 상태 : 머신이나 액터로직이 변경되면, 복원된 상태가 새 로직과 호환되지 않을 수 있다.
- 동작 재생 : 이미 실행된 동작은 다시 실행되지 않는다. 이 사용 사례에는 이벤트 소싱이 선호 된다.
- 직렬화 : 상태는 직렬화가 가능해야하며, 이는 JSON 직렬화가 가능해야함을 의미한다. 즉, 함수, 클래스 또는 기타 직렬화할 수 없는 값은 지속할 수 없다.
