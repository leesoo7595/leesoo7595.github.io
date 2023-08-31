---
layout: post
title: "Apollo Client - Suspense"
date: 2023-08-28
tags: [Graphql]
comments: true
---

이 글은 [Apollo Client 공식문서](https://www.apollographql.com/docs/react/data/suspense/)를 읽고 정리한 글이다.

# Suspense

> 아폴로 클라이언트와 리액트 18 Suspense 기능 사용하기

"Suspense"는 일반적으로 React18에 도입된 `concurrent rendering engine`동시성 렌더링 엔진을 사용하여 React 어플리케이션을 빌드하는 새로운 방식에 사용된다. 자식 로딩이 완료될 때까지의 fallback 상태를 표시할 수 있는 React API인 `<Suspense />` 컴포넌트도 있다.

이 가이드에서는 3.8에 도입된 아폴로 클라이언트의 데이터 fetching hooks를 살펴보고, React의 강력한 Suspense 기능을 활용한다.

## Suspense로 fetching하기

`useSuspenseQuery` hook은 네트워크 요청을 시작하고 요청이 이루어지는 동안 이를 호출하는 컴포넌트를 suspend(일시 중단)시킨다. 렌더링 중에 불러오는 동안 React의 Suspense 기능을 활용할 수 있도록 해주는 useQuery의 Suspense를 지원하는 아이라고 생각하면 된다.

예시를 보자:

```tsx
import { Suspense } from "react";
import { gql, TypedDocumentNode, useSuspenseQuery } from "@apollo/client";

interface Data {
  dog: {
    id: string;
    name: string;
  };
}

interface Variables {
  id: string;
}

interface DogProps {
  id: string;
}

const GET_DOG_QUERY: TypedDocumentNode<Data, Variables> = gql`
  query GetDog($id: String) {
    dog(id: $id) {
      # 기본적으로 객체의 캐시 키값은 __typename과 id 필드의 조합이어서,
      # id를 정확하게 만들어주어야한다. is in the response so our data can be
      # properly cached.
      id
      name
    }
  }
`;

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Dog id="3" />
    </Suspense>
  );
}

function Dog({ id }: DogProps) {
  const { data } = useSuspenseQuery(GET_DOG_QUERY, {
    variables: { id },
  });

  return <>Name: {data.dog.name}</>;
}
```

> 이 예제에서는 `Data` 및 `Variables`에 대한 타입스크립트 인터페이스와 `TypedDocumentNode`를 사용하여 `GET_DOG_QUERY`의 타입을 수동으로 정의한다. GraphQL Code Generator은 이런 타입 정의들을 자동으로 만들어주는 유명한 도구이다. GraphQl Code Generator를 아폴로 클라이언트와 통합하는 방법에 대한 자세한 내용은 [Using TypeScript](https://www.apollographql.com/docs/react/development-testing/static-typing/)를 참조하세요.

이 예제에서는 `App` 컴포넌트가 `useSuspenseQuery`를 통해 반려견 한 마리에 대한 기록을 가져오는 `Dog` 컴포넌트를 렌더링한다. React가 처음으로 `Dog`를 렌더링하려고 시도할 때, 캐시는 `GetDog` 쿼리 요청을 처리할 수 없으므로 `useSuspenseQuery`가 네트워크 요청을 시작한다. 네트워크 요청이 보류되는 동안, `Dog`가 일시중단되어 앱에서 일시중단된 컴포넌트 위에 있는 가장 가까운 `Suspense` boundary를 트리거하여 'Loading...' fallback을 렌더링하게된다. 네트워크 요청이 완료되면 `Dog`는 새로 캐시된 Mozzarella the Cogi의 이름으로 렌더링한다.

`useSuspenseQuery`가 `loading`의 boolean 값을 리턴하지 않는다는 것을 눈치챘을 것이다. 왜냐하면 데이터를 가져올 때 `useSuspenseQuery`를 호출하는 컴포넌트가 항상 일시중단되기 때문이다. 결론은 렌더링할 때 `data`가 항상 정의된다는 것이다! Suspense 패러다임에서는 일시중단된 컴포넌트의 외부에 존재하는 fallback이 이전에 렌더링을 담당했던 로딩 상태를 대체한다.

**여기서 Suspense 패러다임이란, 보통 loading 상태를 해당 컴포넌트가 담당하는게 일반적인데, 이 패러다임은 해당 컴포넌트의 밖에 있는 컴포넌트가 담당한다.**

> Typescript 사용자를 위한 참고: `GET_DOG_QUERY`는 `Data` 제네릭타입 인자를 통해 결과 타입을 명시한 TypedDocumentNode이므로, useSuspenseQuery가 리턴하는 데이터의 Typescript는 이 타입을 반영한다. 즉, Dog가 렌더링될 때 `data`가 정의되고, `data.dog`가 `{ id: string; name: string; breed: string; }` 형식을 갖도록 보장한다.

**그래서 Dog 컴포넌트 내부에서 useSuspenseQuery를 호출할 때, 처음에 데이터를 가져오는 중에는 useSuspenseQuery라인에서 렌더링이 스탑하고 Promise를 throw 한다. 그러면 상위 컴포넌트의 Suspense는 fallback을 보여준다. (참고: error를 가져오면 ErrorBoundary를 보여줌) 그리고 data가 들어오면 Dog 컴포넌트가 '마저' 렌더링된다.**

### variables 바꾸기

이전 예제에서는, `useSuspenseQuery`에 `id` 변수를 하드코딩하여 넘겨주었다. 이제, 동적인 값을 넘겨주어 그에 따른 다른 기록의 dog들을 패칭할 수 있게끔 해보자. 우리는 dogs 리스트에서 name과 id를 가져올 것이고, 사용자가 각각 dog을 선택하는 경우 breed(품종)을 포함한 자세한 정보를 가져올 것이다.

```tsx
export const GET_DOG_QUERY: TypedDocumentNode<DogData, Variables> = gql`
  query GetDog($id: String) {
    dog(id: $id) {
      id
      name
      breed
    }
  }
`;

export const GET_DOGS_QUERY: TypedDocumentNode<DogsData, Variables> = gql`
  query GetDogs {
    dogs {
      id
      name
    }
  }
`;

function App() {
  const { data } = useSuspenseQuery(GET_DOGS_QUERY);
  const [selectedDog, setSelectedDog] = useState(data.dogs[0].id);

  return (
    <>
      <select onChange={(e) => setSelectedDog(e.target.value)}>
        {data.dogs.map(({ id, name }) => (
          <option key={id} value={id}>
            {dog.name}
          </option>
        ))}
      </select>
      <Suspense fallback={<div>Loading...</div>}>
        <Dog id={selectedDog} />
      </Suspense>
    </>
  );
}

function Dog({ id }: DogProps) {
  const { data } = useSuspenseQuery(GET_DOG_QUERY, {
    variables: { id },
  });

  return (
    <>
      <div>Name: {data.dog.name}</div>
      <div>Breed: {data.dog.breed}</div>
    </>
  );
}
```

select의 캐시에 없는 Dog를 선택할 때마다 컴포넌트가 suspend(일시중단)되고 Suspense의 fallback ui가 실행된다. `cache-first` 정책에 따라 아폴로 클라이어트는 cache hit 이후 네트워크 요청을 하지 않기 때문에, 캐시에 특정 객체를 기록한 후 드랍다운에서 해당 객체를 다시 선택해도 컴포넌트는 suspend되지 않는다.

###

`startTransition`를 사용해서 UI 업데이트가 이루어지면 fallback UI를 사용하지 않고 UI가 렌더링되게 할 수 있다.

```tsx
import { useState, Suspense, startTransition } from "react";

function App() {
  const { data } = useSuspenseQuery(GET_DOGS_QUERY);
  const [selectedDog, setSelectedDog] = useState(data.dogs[0].id);

  // startTransition을 사용하는 경우, Dog를 select할 때 fallback UI가 사용되지 않는다.
  return (
    <>
      <select
        onChange={(e) => {
          startTransition(() => {
            setSelectedDog(e.target.value);
          });
        }}
      >
        {data.dogs.map(({ id, name }) => (
          <option key={id} value={id}>
            {name}
          </option>
        ))}
      </select>
      <Suspense fallback={<div>Loading...</div>}>
        <Dog id={selectedDog} />
      </Suspense>
    </>
  );
}
```

`startTransition`를 사용하는 경우, `useSuspenseQuery`의 fallback 상태로 들어오는 Promise throw하는 부분이 `startTransition` 안에서 진행되기 때문에 패스되게 된다. 그리고 startTransition 상태가 끝난 다음 큐에서 useSuspenseQuery가 마저 렌더링을 계속하여 UI를 보여주게된다.

### Showing pending UI during a transition

useTransition을 사용하여 데이터를 가져오는 pending 상태유무를 알 수 있다.

```tsx
import { useState, Suspense, useTransition } from "react";

function App() {
  const [isPending, startTransition] = useTransition();
  const { data } = useSuspenseQuery(GET_DOGS_QUERY);
  const [selectedDog, setSelectedDog] = useState(data.dogs[0].id);

  return (
    <>
      <select
        // useTransition의 isPending을 사용하여 스타일을 변경
        style={{ opacity: isPending ? 0.5 : 1 }}
        onChange={(e) => {
          startTransition(() => {
            setSelectedDog(e.target.value);
          });
        }}
      >
        {data.dogs.map(({ id, name }) => (
          <option key={id} value={id}>
            {name}
          </option>
        ))}
      </select>
      <Suspense fallback={<div>Loading...</div>}>
        <Dog id={selectedDog} />
      </Suspense>
    </>
  );
}
```

**페이지네이션할 때, fallback UI와 적당히 조합하여 useTransition의 isPending 상태를 잘 활용한다면 사용자 경험이 더 좋은 부드러운 느낌의 UI를 선택할 수 있다. fallback UI만을 사용한다고 해서 사용자경험이 무조건 좋을 수는 없음.**

Suspense를 사용하면서 좋은 것은 `Server Component`, Suspense를 사용하면 하위 컴포넌트가 렌더링이 되지 않아서 `Server Component`와의 조합이 좋다.

### 부분적인 데이터 렌더링

```tsx
interface PartialData {
  dog: {
    id: string;
    name: string;
  };
}

const PARTIAL_GET_DOG_QUERY: TypedDocumentNode<PartialData, Variables> = gql`
  query GetDog($id: String) {
    dog(id: $id) {
      id
      name
    }
  }
`;

// Write partial data for Buck to the cache
// so it is available when Dog renders
client.writeQuery({
  query: GET_DOG_QUERY_PARTIAL,
  variables: { id: "1" },
  data: { dog: { id: "1", name: "Buck" } },
});

function App() {
  const client = useApolloClient();

  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Dog id="1" />
    </Suspense>
  );
}

function Dog({ id }: DogProps) {
  const { data } = useSuspenseQuery(GET_DOG_QUERY, {
    variables: { id },
    returnPartialData: true,
  });

  return (
    <>
      <div>Name: {data?.dog?.name}</div>
      <div>Breed: {data?.dog?.breed}</div>
    </>
  );
}
```

data option에 특정 객체 데이터를 정의해주면, UI가 렌더링될 때 data에 지정해준 객체 데이터를 활용하여 부분적으로 데이터를 보여줄 수가 있다.

### 에러핸들링

Suspense 컴포넌트를 사용하는 것과 유사하게, ErrorBoundary 컴포넌트를 같은 방법으로 사용하면 하위 컴포넌트의 값에서 에러가 나타나는 경우 ErrorBoundary 컴포넌트의 fallback UI를 나타내게끔 활용할 수 있다.

```tsx
function App() {
  const { data } = useSuspenseQuery(GET_DOGS_QUERY);
  const [selectedDog, setSelectedDog] = useState(data.dogs[0].id);

  return (
    <>
      <select onChange={(e) => setSelectedDog(e.target.value)}>
        {data.dogs.map(({ id, name }) => (
          <option key={id} value={id}>
            {name}
          </option>
        ))}
      </select>
      <ErrorBoundary fallback={<div>Something went wrong</div>}>
        <Suspense fallback={<div>Loading...</div>}>
          <Dog id={selectedDog} />
        </Suspense>
      </ErrorBoundary>
    </>
  );
}
```

### 폭포수 요청 피하기 (request waterfalls)

`useBackgroundQuery`를 사용하여 request waterfalls 현상을 방지할 수 있다.

```tsx
import {
  useBackgroundQuery,
  useReadQuery,
  useSuspenseQuery,
} from "@apollo/client";

function App() {
  // We can start the request here, even though `Dog`
  // suspends and the data is read by a grandchild component.
  const [queryRef] = useBackgroundQuery(GET_BREEDS_QUERY);

  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Dog id="3" queryRef={queryRef} />
    </Suspense>
  );
}

function Dog({ id, queryRef }: DogProps) {
  const { data } = useSuspenseQuery(GET_DOG_QUERY, {
    variables: { id },
  });

  return (
    <>
      Name: {data.dog.name}
      <Suspense fallback={<div>Loading breeds...</div>}>
        <Breeds queryRef={queryRef} />
      </Suspense>
    </>
  );
}

interface BreedsProps {
  queryRef: QueryReference<BreedData>;
}

function Breeds({ queryRef }: BreedsProps) {
  const { data } = useReadQuery(queryRef);

  return data.breeds.map(({ characteristics }) =>
    characteristics.map((characteristic) => (
      <div key={characteristic}>{characteristic}</div>
    ))
  );
}
```

### queryKey를 사용해서 queries 구별하기

아폴로 클라이언트는 Suspense 훅을 사용하여 데이터를 가져올 때 쿼리와 변수의 조합을 사용하여 각 query를 고유하게 식별한다.

앱에서 동일한 `query`와 `variables`를 사용하는 여러 컴포넌트를 렌더링하는 경우, 여러 훅이 만든 쿼리들이 동일한 ID를 공유하여 특정 컴포넌트가 네트워크 요청을 하게되면 모든 컴포넌트가 동시에 suspend되는 문제가 생길 수 있다.

이런 현상을 막기위해 `queryKey`를 사용하여 각 훅의 유니크한 ID가 있는지 확인할 수 있다. `queryKey`가 제공되면 아폴로 클라이언트는 `query`와 `variables`와 추가적으로 훅의 고유성의 일부러 `queryKey`를 사용한다.

**하나가 바뀌면 전부다 Suspense로 빠지는 것을 막기위한 키**

### skipToken을 사용하여 suspense 훅을 스킵하기

`useSuspenseQuery`와 `useBackgroundQuery`에는 모두 skip 옵션이 존재하지만, 이 옵션은 적은 코드 변경으로 `useQuery`에서 쉽게 마이그레이션할 수 있도록 하기 위해서만 존재한다. 그래서 장기적으로 봤을 때 skip을 사용해서는 안된다.

대신, `skipToken`을 사용하자:

```javascript
import { skipToken, useSuspenseQuery } from "@apollo/client";
const { data } = useSuspenseQuery(
  query,
  id ? { variables: { id } } : skipToken
);
```

**skip 옵션보다 skipToken을 사용하는 것을 추천**

## React Server Component (RSC)

### Next.js 13 App Router와 같이 사용하기

Next.js 13에서 Next.js의 새로운 기능인 App Router는 React 커뮤니티의 React Server Component와 Streaming SSR을 완벽하게 지원하는 최초의 프레임워크로, 앱의 라우팅 레이어부터 아래까지 Suspense를 1급 개념으로 통합하였다.

이 기능들을 통합하기 위해서는 아폴로 클라이언트 팀은 실험적인 패키지인 `@apollo/experimental-nextjs-app-support`를 릴리즈했고, 이를 통해 data fetching 라이브러리 최초로 RSC와 스트리밍 SSR을 모두 지원하는 apollo client를 원활하게 사용할 수 있다. 자세한 내용은 [README](https://github.com/apollographql/apollo-client-nextjs#readme) 보기

#### streaming SSR 중 `useBackgroundQuery` 사용하여 페칭하는 동안 스트리밍하기

클라이언트가 렌더링하는 앱에서는 useBackgroundQuery를 사용하여 request waterfalls를 방지할 수 있지만, App Router처럼 Streaming SSR을 사용하는
