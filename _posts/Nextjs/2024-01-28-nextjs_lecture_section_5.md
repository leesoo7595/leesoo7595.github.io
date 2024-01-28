---
layout: post
title: "[Nextjs] Nextjs app router 강의 섹션 5"
date: 2024-01-28
tags: [nextjs]
comments: true
---

제로초의 next app router 강의를 기반으로 docs 공부를 보충하였습니다.

# Request Memoization

리액트는 fetchAPI를 확장하여 URL과 옵션이 동일한 요청을 자동으로 메모화도록 한다. 즉, 리액트 컴포넌트 트리의 여러 위치에서 동일한 데이터에 대한 fetch 함수를 한 번만 실행하면서 호출할 수 있다.

```typescript
async function getItem() {
  // The `fetch` function is automatically memoized and the result
  // is cached
  const res = await fetch("https://.../item/1");
  return res.json();
}

// This function is called twice, but only executed the first time
const item = await getItem(); // cache MISS

// The second call could be anywhere in your route
const item = await getItem(); // cache HIT
```

## Duration

**캐시의 지속성**

캐시는 리액트 컴포넌트 트리가 렌더링을 완료할 때까지 서버 request 수명동안 지속된다. 현재 페이지에서 렌더링 되는 것까지만 request 캐시 지속성이 있다.

## Revalidating

**캐시의 유효성**

메모화가 서버 요청 간에 공유되지 않고 렌더링 중에만 적용되므로, 다시 유효성을 검사할 필요가 없다.

### Time-based Revalidation

어떤 시간대가 지나면 갱신하기

```ts
// Revalidate at most every hour
fetch("https://...", { next: { revalidate: 3600 } });
```

### On-demand Revalidation

직접 캐시 갱신하기, `revalidateTag` 활용하기

## Opting out

메모화를 선택 해제하려면 request에서 `AbortController` 신호를 전달한다.

```ts
// Opt out of caching for an individual `fetch` request
fetch(`https://...`, { cache: "no-store" });
```

# Full Route Cache

빌드 시점에 자동으로 라우트를 렌더링하고 캐시한다. 모든 요청에 대해 서버에서 렌더링하는 대신 캐시된 경로를 제공하여 페이지 로딩 속도를 높일 수 있도록한다.

서버에서 nextjs는 react api를 사용하여 렌더링을 오케스트레이션한다. 렌더링 작업은 개별 라우트 세그먼트와 suspense 바운더리에 따라 청크로 나뉘어진다.

풀라우트 캐시는 정적인 페이지인 경우에 유용하다. 정적인 페이지가 아닌경우, 즉 쿠키를 쓰거나, 헤더, 서치파람 같이 자주 바뀌는 경우, 그때마다 새로 그려서 캐시를 업데이트해줘야하기 때문에 의미가 없음.

# Router Cache

사용자 세션이 진행되는 동안, 개별 라우트 세그먼트로 분할된 리액트 서버 컴포넌트 페이로드를 저장하는 인메모리 클라이언트 캐시가 있다.

사용자가 라우트를 네비게이트할 때, nextjs는 방문한 라우트 세그먼트를 캐시하고 사용자가 네비게이팅 가능성이 높은 라우터를 미리 가져온다.

## duration

캐시는 브라우저의 임시 메모리에 저장된다. 라우터 캐시의 지속시간은 두 가지 요인에 의해 결정된다.

- Session: 새로고침시에는 삭제된다. 그 외는 지속된다.
- Automatic Invalidation Period: 개별 세그먼트의 (레이아웃) 캐시는 특정 시간이 지나면 자동으로 무효화된다. 기간은 라우트가 정적, 동적으로 렌더링되는지에 따라 다르다.
  - 동적 렌더링 : 30초
  - 정적 렌더링 : 5분

## Invalidation

라우터 캐시를 무효화하는 방법은 두 가지가 있다.

- Server Action

  - 라우트(`revalidatePath`), 또는 캐시 태그(`revalidateTag`)를 기준으로 수동으로 재검증하기
  - `cookies.set` 또는 `cookies.delete`를 사용하면 라우터 캐시가 무효화되어 쿠키를 사용하는 라우터가 캐시로 인해 잘못된 쿠키가 사용되지 않게끔 방지한다.

- `router.refresh`를 호출하면 라우터 캐시가 무효화되고 현재 라우터에 대한 새 요청이 서버에 전송된다.

## Opting out

라우터 캐시를 끄는 것은 불가능하다. 하지만 `router.refresh`, `revalidatePath`, `revalidateTag`를 호출하여 이를 무효화할 수 있다.(invalidate) 이렇게하면 라우터 캐시가 지워지고 서버에 새로 요청하여 최신 데이터가 표시되게 된다.

# 참고

- [nextjs - caching, request-memoization](https://nextjs.org/docs/app/building-your-application/caching#request-memoization)
