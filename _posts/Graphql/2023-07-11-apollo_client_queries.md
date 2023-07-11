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
