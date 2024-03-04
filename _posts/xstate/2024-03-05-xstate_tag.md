---
layout: post
title: "[XState] Tags"
date: 2024-03-05
tags: [xstate]
comments: true
---

# Tags

상태 노드에는 상태 노드를 그룹화하거나 분류하는 데 도움이 되는 문자열 용어인 태그를 가질 수 있다. 예를 들어 "loading" 태그를 사용하여 데이터가 로드 중인 상태를 나타내는 상태 노드를 표시하고 `state.hasTag(tag)`를 사용하여 해당 태그가 지정된 상태 노드가 상태에 포함되어 있는지 확인할 수 있습니다:

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

## Tags and Typescript

머신 config의 `types.tags` 프로퍼티에서 머신의 태그를 강력하게 입력할 수 있다.

```typescript
onst machine = createMachine({
  types: {} as {
    tags: 'pending' | 'success' | 'error';
  },
  // ...
  states: {
    loadingUser: {
      tags: ['pending'],
    },
  },
});

const actor = createActor(machine).start();

actor
  .getSnapshot()
  // Autocompleted
  .hasTag('pending');
```
