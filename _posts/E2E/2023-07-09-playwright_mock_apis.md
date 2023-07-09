---
layout: post
title: "Playwright - Mock APIs"
date: 2023-07-09
tags: [E2E, playwright]
comments: true
---

이 글은 [playwright 공식문서 - Mock APIs](https://playwright.dev/docs/mock#mock-api-requests)를 읽고 정리한 글이다.

# Mock APIs

playwright는 네트워크 트래픽을 mock하고 수정할 수 있는 API를 http, https 모두 제공한다.

## Mock API requests

특정 api로의 모든 호출을 인터럽트하고 대신 테스트 데이터를 리턴한다. 해당 api의 엔드포인트에 대한 요청은 이루어지지 않는다.

```typescript
await page.route("https://dog.ceo/api/breeds/list/all", async (route) => {
  const json = {
    message: { test_breed: [] },
  };
  await route.fulfill({ json });
});
```

## Modify API responses

API 요청을 해야하지만, 재현 가능한 테스트를 위해 응답을 수정해야하는 경우가 있다. 이 경우에는 요청을 모킹하는게 아니라 요청을 수행하고 수정된 응답으로 요청을 이행할 수 있다.

```typescript
await page.route("https://dog.ceo/api/breeds/list/all", async (route) => {
  const response = await route.fetch();
  const json = await response.json();
  json.message["big_red_dog"] = [];
  // Fulfill using the original response, while patching the response body
  // with the given JSON object.
  await route.fulfill({ response, json });
});
```
