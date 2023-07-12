---
layout: post
title: "Apollo Client - Queries"
date: 2023-07-11
tags: [Graphql]
comments: true
---

이 글은 [아폴로 클라이언트 공식문서](https://www.apollographql.com/docs/react/data/queries/#prerequisites)를 보고 정리한 글 입니다.

# Queries

이 글은 useQuery hook을 사용해서 GraphQL 데이터를 어떻게 fetch하는지, 그리고 UI에 결과를 어떻게 붙이는지를 보여주고, loading, error를 어떻게 관리하는지를 보여준다.

## Executing a query

useQuery 리액트 훅은 아폴로 어플리케이션에서 쿼리들을 실행시키는데 중요한 API 이다. 리액트 컴포넌트에서 쿼리를 실행할때, useQuery를 호출하고 GraphQL 쿼리스트링을 패스한다. 컴포넌트가 렌더링될때, useQuery는 아폴로 클라이언트에서 loading, error, data 프로퍼티를 포함한 객체를 리턴한다. 이 프로퍼티들을 사용하여 UI에 렌더링할 수 있다.

```typescript
import { gql, useQuery } from "@apollo/client";

const GET_DOGS = gql`
  query GetDogs {
    dogs {
      id
      breed
    }
  }
`;
```

`GET_DOGS`라는 이름의 쿼리를 생성했다. query documents로 파싱하려면 gql 함수를 사용하여 감싸주어야한다. 그리고 Dogs라는 이름의 컴포넌트를 생성한다. 여기에 `GET_DOGS` 쿼리를 useQuery 훅에 넘겨준다.

```typescript
function Dogs({ onDogSelected }) {
  const { loading, error, data } = useQuery(GET_DOGS);

  if (loading) return "Loading...";
  if (error) return `Error! ${error.message}`;

  return (
    <select name="dog" onChange={onDogSelected}>
      {data.dogs.map((dog) => (
        <option key={dog.id} value={dog.breed}>
          {dog.breed}
        </option>
      ))}
    </select>
  );
}
```

쿼리가 실행하하면 loading, error, data 값이 바뀌고, Dogs 컴포넌트는 해당 쿼리의 상태에 따라 다른 UI를 렌더할 수 있다.

- `loading`이 true인 경우 Loading...이 보여지게 된다.
- `loading`이 false이고 `error`가 없는 경우, 쿼리 실행이 완료된 것이다. 컴포넌트는 서버에서 받아온 품종 리스트가 채워진 드롭다운 메뉴를 렌더링한다.

사용자가 드롭다운에 있는 품종를 선택하면, 선택항목은 `onDogSelected` 함수를 통해서 부모 컴포넌트로 전달된다.

## Caching query results

아폴로 클라이언트가 서버를 통해 쿼리를 패치해서 결과를 가져오면, 자동으로 해당 결과들을 로컬에 캐싱한다. 이것은 추후 같은 쿼리가 실행되었을때 매우 빠르게 실행될 수 있도록 한다.

```typescript
const GET_DOG_PHOTO = gql`
  query Dog($breed: String!) {
    dog(breed: $breed) {
      id
      displayImage
    }
  }
`;

function DogPhoto({ breed }) {
  const { loading, error, data } = useQuery(GET_DOG_PHOTO, {
    variables: { breed },
  });

  if (loading) return null;
  if (error) return `Error! ${error}`;

  return (
    <img src={data.dog.displayImage} style={{ height: 100, width: 100 }} />
  );
}
```

이번엔 useQuery 훅에 쿼리에 넘겨줄 값들을 variables 옵션을 통해 넘겨주었다. 이렇게 하면, 우리가 현재 드롭다운 메뉴에서 선택한 품종을 넘길 수 있다.

드롭다운에서 `bulldog`을 선택하면 불독 사진이 나타난다. 그런 다음 다른 품종을 선택하고 다시 `bulldog`을 선택한다. 그렇게하면 두번째 불독 사진은 즉시 로드되는 것을 확인할 수 있다. 이것이 캐시가 동작하는 것

그 다음은 캐시된 데이터가 최신 상태로 유지하기 위한 몇 가지 기술을 알아보자.

## Updating cached query results

가끔은 쿼리의 캐시된 데이터가 서버의 데이터와 비교해서 최신상태인지 확인하고 싶을 때가 있다. 아폴로 클라이언트는 `polling`과 `refetching`을 지원한다.

### Polling

Polling은 쿼리에 특정 간격을 설정해서 주기적으로 실행하여 서버와 실시간에 가까운 동기화를 제공한다. polling을 가능하게 하려면, useQuery 훅에서 milliseconds 단위로 `pollInterval` 값을 전달하면 된다.

```jsx
function DogPhoto({ breed }) {
  const { loading, error, data } = useQuery(GET_DOG_PHOTO, {
    variables: { breed },
    pollInterval: 500,
  });

  if (loading) return null;
  if (error) return `Error! ${error}`;

  return (
    <img src={data.dog.displayImage} style={{ height: 100, width: 100 }} />
  );
}
```

`pollInterval`을 500으로 세팅하게되면, 0.5초마다 현재 품종의 이미지를 서버에서 불러오게된다. 만약 `pollInterval`이 0이면, 쿼리는 폴링되지 않는다.

`startPolling`과 `stopPolling` 함수를 useQuery 훅에서 사용해서 폴링을 동적으로 시작하고 멈추게 할 수 있다. 이 함수들을 사용할때 `pollInterval`을 `startPolling` 함수의 인자를 통해 가져와서 설정할 수 있다.

### Refetching

Refetching은 고정된 반복을 사용하는 대신, 특정 사용자의 액션에 따른 응답으로 쿼리 결과를 다시 가져올 수 있다.

`refetch` 함수에서는 새로운 `variables` 객체를 선택적으로 넘겨줄 수 있다. 만약 `variables` 객체를 넘기지 않고 `refetch()`만 사용하는 경우, 쿼리는 이전 실행에서 사용한 `variables`와 동일한 것을 사용한다.

```jsx
function DogPhoto({ breed }) {
  const { loading, error, data, refetch } = useQuery(GET_DOG_PHOTO, {
    variables: { breed },
  });

  if (loading) return null;
  if (error) return `Error! ${error}`;

  return (
    <div>
      <img src={data.dog.displayImage} style={{ height: 100, width: 100 }} />
      <button onClick={() => refetch({ breed: "new_dog_breed" })}>
        Refetch new breed!
      </button>
    </div>
  );
}
```

버튼을 누르게되면 새로운 강아지 사진으로 UI가 업데이트된다. Refetching은 새로운 데이터를 가져오는 것을 보장하지만, 복잡한 loading 상태를 가지고 있다.

#### Providing new variables to refetch

`refetch`를 새로운 variables와 사용하는 경우

```jsx
<button
  onClick={() =>
    refetch({
      breed: "dalmatian", // Always refetches a dalmatian instead of original breed
    })
  }
>
  Refetch!
</button>
```

기존 쿼리의 variables 중 일부에만 새 값을 전달한 경우, 다른 생략된 variables는 기존 값들을 사용한다.

## Inspecting loading states

useQuery 훅에서 loading 상태를 어떻게 가져오는지 이미 알고 있다. 이건 쿼리가 처음 로드될 때는 매우 유용하지만, refetching하거나 polling할 때의 loading 상태는 어떤 일이 일어날까?

이전에 refetching 예시로 돌아가보면, refetch 버튼을 누르면 컴포넌트가 새로운 데이터를 받아올 때까지 리렌더링이 되지않는 것을 알 수 있다. 사진을 refetch하는 중임을 표시하려면 어떻게 해야할까?

useQuery 훅의 result 객체는 networkStatus 속성을 통해 쿼리 상태에 대한 세분화된 정보(fine-grained information)를 제공한다. 이 정보를 활용해서, `notifyOnNetworkStatusChange` 옵션을 true로 설정하여 refetch가 되는 동안 쿼리 컴포넌트가 리렌더링이 될 수 있도록 하였다.

```tsx
import { NetworkStatus } from "@apollo/client";

function DogPhoto({ breed }) {
  const { loading, error, data, refetch, networkStatus } = useQuery(
    GET_DOG_PHOTO,
    {
      variables: { breed },
      notifyOnNetworkStatusChange: true,
    }
  );

  if (networkStatus === NetworkStatus.refetch) return "Refetching!";
  if (loading) return null;
  if (error) return `Error! ${error}`;

  return (
    <div>
      <img src={data.dog.displayImage} style={{ height: 100, width: 100 }} />
      <button onClick={() => refetch({ breed: "new_dog_breed" })}>
        Refetch!
      </button>
    </div>
  );
}
```

이 옵션을 활성화하게 되면 networkStatus 속성에서 제공하는 세분화된 정보(fine-grained information)를 사용하지 않는 경우에도 loading이 업데이트 되는 것을 보장할 수 있다.

networkStatus 속성은 다양한 loading 상태를 나타내는 `NetworkStatus` enum으로 되어있다. Refetch는 `NetworkStatus.refetch`를 나타내고, polling과 pagination의 값들 또한 가지고 있다. 가능한 loading 상태 리스트를 모두 보려면 [여기](https://github.com/apollographql/apollo-client/blob/main/src/core/networkStatus.ts)를 보자.

## Inspecting error states

`useQuery` 훅의 `errorPolicy` 설정을 통해 쿼리의 에러핸들링을 커스터마이징할 수 있다. 기본값은 `none`으로 설정되어있고, 이 뜻은 아폴로 클라이언트가 모든 GraphQL 오류를 런타임 오류로 처리하도록 한다. 이 경우 아폴로 클라이언트는 서버에서 리턴한 모든 쿼리 응답 데이터를 삭제하고 useQuery의 데이터 객체에 `error` 속성을 설정한다.

만약 `errorPolicy`를 `all`로 설정하면, useQuery는 응답데이터를 버리지않아서 부분적으로 결과를 렌더링할 수 있게된다.

## Manual execution with useLazyQuery

리액트에서 컴포넌트를 렌더링할 때 useQuery를 사용하면, 아폴로 클라이언트는 자동으로 쿼리를 실행한다. 만약 쿼리를 다른 이벤트에 따라 실행하게 하고 싶으면(버튼을 클릭한다던가) `useLazyQuery` 훅을 사용하면 된다.

useLazyQuery 훅은 이벤트에 대한 응답에 따라서 컴포넌트 렌더링을 시키고 싶을 때 쿼리를 실행시키는 퍼풱한 아이이다. useQuery랑은 달리 useLazyQuery를 호출하면 해당 쿼리를 바로 실행시키지 않는다. 대신, 튜플로 결과를 리턴하는데 그 결과에 쿼리 함수를 호출하여 해당 쿼리를 실행하고 싶을때 실행할 수 있도록 한다.

```jsx
import React from "react";
import { useLazyQuery } from "@apollo/client";

function DelayedQuery() {
  const [getDog, { loading, error, data }] = useLazyQuery(GET_DOG_PHOTO);

  if (loading) return <p>Loading ...</p>;
  if (error) return `Error! ${error}`;

  return (
    <div>
      {data?.dog && <img src={data.dog.displayImage} />}
      <button onClick={() => getDog({ variables: { breed: "bulldog" } })}>
        Click me!
      </button>
    </div>
  );
}
```

useLazyQuery의 리턴값인 튜플에서 첫번째 아이는 쿼리함수이고, 두번째 아이는 useQuery와 같은 result 객체이다.

위 코드와 같이 useLazyQuery 자체에 옵션을 전달할 수 있지만, 쿼리함수에도 옵션을 전달할 수 있다. 두 옵션 모두에 특정 옵션을 전달할 경우, 쿼리함수에 전달한 값이 우선으로 실행된다. 이것은 default 옵션을 useLazyQuery에 설정하고 쿼리함수에 커스터마이징 옵션을 설정할 수 있게해서 더욱 편리하다.

지원하는 모든 옵션을 보려면 [여기](https://www.apollographql.com/docs/react/api/react/hooks/#uselazyquery)~

## Setting a fetch policy

기본적으로, useQuery 훅은 아폴로 클라이언트의 캐시를 체크해서 요구된 데이터가 로컬에 존재하는지 알 수 있다. 캐시가 있는 경우, useQuery는 데이터를 리턴하지만 GraphQL 서버에 쿼리하지 않는다. 이것은 아폴로 클라이언트에서 fetch policy의 디폴트로 설정되어있는 `cache-first` 정책이다.

다른 fetch policy로 설정할 수 있다. 그렇게 하려면 호출할 useQuery에 `fetchPolicy` 옵션을 포함한다.

```js
const { loading, error, data } = useQuery(GET_DOGS, {
  fetchPolicy: "network-only", // Doesn't check cache before making a network request
});
```

### nextFetchPolicy

`nextFetchPolicy`를 쿼리에 설정할 수 있다. `fetchPolicy`는 쿼리의 첫번째 실행에 쓰이는 아이였다면, `nextFetchPolicy`는 쿼리가 이후 캐시 업데이트에 응답하는 방식을 결정하는데 사용한다.

```js
const { loading, error, data } = useQuery(GET_DOGS, {
  fetchPolicy: "network-only", // Used for first execution
  nextFetchPolicy: "cache-first", // Used for subsequent executions
});
```

예를 들면, 위 코드의 경우 초기 호출은 네트워크 요청을 항상 하되, 그 후부턴 캐시를 읽어오도록 한다.

#### nextFetchPolicy functions

위 코드의 경우 각 쿼리들에게 수동적으로 적용해주어야하기 때문에, `nextFetchPolicy`를 디폴트로 적용하는 경우에는 `ApolloClient` 인스턴스에 `defaultOptions.watchQuery.nextFetchPolicy`를 설정하는 식으로 가능하다.

```js
new ApolloClient({
  link,
  client,
  defaultOptions: {
    watchQuery: {
      nextFetchPolicy: "cache-only",
    },
  },
});
```

이 설정은 `nextFetchPolicy`를 다르게 적용하지 않은 모든 `client.watchQuery` 호출 및 useQuery 호출에 적용된다.

만약 `nextFetchPolicy`의 동작 방식을 더 자세히 다루고 싶은 경우에는 `WatchQueryFetchPolicy` 문자열 대신 함수를 넘겨줄 수도 있다.

```js
new ApolloClient({
  link,
  client,
  defaultOptions: {
    watchQuery: {
      nextFetchPolicy(currentFetchPolicy) {
        if (
          currentFetchPolicy === "network-only" ||
          currentFetchPolicy === "cache-and-network"
        ) {
          // Demote the network policies (except "no-cache") to "cache-first"
          // after the first request.
          return "cache-first";
        }
        // Leave all other fetch policies unchanged.
        return currentFetchPolicy;
      },
    },
  },
});
```

이 `nextFetchPolicy` 함수는 각각 요청 후에 호출되며, `currentFetchPolicy` 파라미터를 사용해서 fetch policy를 어떻게 수정할 것인지 결정할 수 있다.

각 요청 후에 호출되는 것에 더해서 `nextFetchPolicy` 함수는 variables가 바뀌었을때에도 호출된다. 기본적으로 fetchPolicy를 초기값으로 재설정하는데 `cache-and-network`나 `network-only` 정책으로 시작된 쿼리에 대해 새로운 네트워크 요청을 하는데 중요한 역할을 해준다.

variables가 바뀐 경우를 인터럽트하거나 핸들하려면, `NextFetchPolicyContext` 객체를 `nextFetchPolicy` 함수의 두번째 인자에 넘겨주는 식으로 할 수 있다.

```js
new ApolloClient({
  link,
  client,
  defaultOptions: {
    watchQuery: {
      nextFetchPolicy(
        currentFetchPolicy,
        {
          // "after-fetch" 또는 "variables-changed" 인 경우 -> 왜 이 함수가 실행되었는지 알려주는 인자
          reason,
          // 다른 옵션들 (currentFetchPolicy === options.fetchPolicy).
          options,
          // 기존 옵션인 options.fetchPolicy 값을 가져온다. nextFetchPolicy가 적용되기 전인 첫번째 호출때.
          initialPolicy,
          // The ObservableQuery associated with this client.watchQuery call.
          observable,
        }
      ) {
        // variables가 바뀌었을 때, 기본 정책을 options.fetchPolicy가 context.initialPolicy로 리셋한다.
        // 이 로직을 생략하는 경우, nextFetchPolicy 함수가 오버라이딩되어 기본 동작을 재정의할 수 있다.
        // 이렇게 하면 options.fetchPolicy가 변경되지 않도록 할 수 있다.
        if (reason === "variables-changed") {
          return initialPolicy;
        }

        if (
          currentFetchPolicy === "network-only" ||
          currentFetchPolicy === "cache-and-network"
        ) {
          // Demote the network policies (except "no-cache") to "cache-first"
          // after the first request.
          return "cache-first";
        }

        // Leave all other fetch policies unchanged.
        return currentFetchPolicy;
      },
    },
  },
});
```

## Supported fetch policies

- cache-first

첫번째 쿼리가 시행되었을때 캐시가 있는지를 먼저 본다. 만약 모든 요청된 데이터가 캐시가 존재하면 해당 데이터가 리턴된다. 아니면 아폴로 클라이언트는 GraphQL 서버 갔다와서 데이터를 주고 캐시한다.

네트워크 요청을 최소화하는게 우선인 경우에 쓰인다. 기본적인 fetch policy이다.

- cache-only

쿼리를 실행할 때 캐시만을 가져오는 경우에 쓰인다. 서버쪽 쿼리를 하지 않는다.

모든 요청된 필드가 캐시에 포함되어 있지 않은 경우 에러를 준다.

- cache-and-network

캐시와 Graphql 서버에 둘다 쿼리를 실행한다. 서버측 쿼리 결과가 캐시된 필드를 수정하는 경우 쿼리가 자동으로 업데이트된다.

빠른 응답을 제공함과 동시에 캐시된 데이터를 서버 데이터와 일관되게 유지하는데 유용하다.

- network-only

캐시 체킹 없이 GraphQL 서버에서 쿼리를 할때 쓰인다. 쿼리 결과는 캐시에 저장된다.

서버 데이터와의 일관성을 우선시하지만 캐시된 데이터를 사용할 수 있는 경우에는 즉각적인 응답을 제공하지 못한다.

- no-cache

쿼리 결과가 캐시에 저장되지 않는 것 빼고 `network-only`와 비슷하다.

- standby

이 쿼리는 기본 필드값이 변경될 때 자동으로 업데이트되지 않는 것을 빼고, `cache-first`와 동일하다. `refetch`나 `updateQueries`를 사용하여 수동으로 쿼리를 업데이트할 수 있다.

## useQuery API

useQuery를 호출할때 대부분 생략할 수 있지만, 있다는 것을 알면 좋음

### Options

**operation options**

- query
- variables
- errorPolicy
- onCompleted
- onError
- skip

**networking options**

- pollInterval
- notifyOnNetworkStatusChange
- context
- ssr
- client

**caching options**

- fetchPolicy
- nextFetchPolicy
- returnPartialData

### Result

**operation data**

- data
- previousData
- error
- variables

**network info**

- loading
- networkStatus
- client
- called

**helper functions**

- refetch
- fetchMore
- startPolling
- stopPolling
- subscribeToMore
- updateQuery
