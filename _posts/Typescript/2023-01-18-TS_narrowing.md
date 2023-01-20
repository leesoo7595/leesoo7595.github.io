---
layout: post
title: "Typescript - Narrowing"
date: 2023-01-18
tags: [Typescript]
comments: true
---

## Narrowing

padLeft라는 함수가 있다고 생각해보자.

```typescript
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
  }
  return padding + input;
}
```

만약 위 함수에서 `padding`이 `number`라면, `input` 앞에 `padding` 횟수만큼 공백이 띄워지지만, `padding`이 `string`이라면, `input` 앞에 `padding`이 붙여져야한다.

그렇게 되면, 타입스크립트에서는 `padding`을 `number`나 `string`을 받을 때 우리가 `padding`이라는 값을 `number`나 `string`으로 Narrowing해줘야한다.
