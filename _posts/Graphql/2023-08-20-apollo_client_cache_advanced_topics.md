---
layout: post
title: "Apollo Client - Advanced topics on caching in Apollo Client"
date: 2023-08-20
tags: [Graphql]
comments: true
---

이 글은 아폴로 클라이언트 [공식문서](https://www.apollographql.com/docs/react/caching/advanced-topics/)를 읽고 정리한 글이다.

이 문서에서는 아폴로 클라이언트 캐시 사용시 특별한 경우와 고려 사항에 대해 설명한다.

## 캐시 우회하기

가끔은 특정 그래프큐엘 연산에 캐시를 사용해서는 안되는 경우가 있다. 예를 들어 쿼리의 응답이 한 번만 사용되는 토큰의 경우가 그렇다. 이런 경우에는 `no-cache fetch policy`를 사용하자:

```javascript
const { loading, error, data } = useQuery(GET_DOGS, {
  fetchPolicy: "no-cache",
});
```

이 fetch policy를 사용하는 operation의 결과는 캐시에 기록하지 않고, 서버에 요청을 보내기 전에 캐시에서 데이터를 확인하지도 않는다.

## 캐시 유지하기

AsyncStorage 또는 localStorage와 같은 스토리지에서 `InMemoryCache`를 유지 및 리하이드레이트할 수 있다. 그렇게 하기 위해서는 [apollo3-cache-persist](https://github.com/apollographql/apollo-cache-persist) 라이브러리를 사용해야한다. 이 라이브러리는 여러 storage providers와 함께 동작한다.

시작하기 앞서, `persisCache`에 storage provider와 캐시를 전달한다. 기본적으로 캐시의 내용은 즉시 비동기식으로 복원되고, 구성 가능한 짧은 간격을 두고 캐시에 쓸 때마다 유지된다.

> `persisCache` 메소드는 Promise를 리턴하는 비동기이다.

```javascript
import AsyncStorage from "@react-native-async-storage/async-storage";
import { InMemoryCache } from "@apollo/client";
import { persistCache } from "apollo3-cache-persist";

const cache = new InMemoryCache();

persistCache({
  cache,
  storage: AsyncStorage,
}).then(() => {
  // 평소와 같이 아폴로 클라이언트를 설정한다.
});
```

더 고급사용법과 추가적인 설정 옵션을 위해서는 [apollo3-cache-persis README](https://github.com/apollographql/apollo-cache-persist)를 읽어보자.

## 캐시 재설정

어떤 경우에는, 캐시를 전체적으로 재설정을 해야할 때가 있다. 예를 들면 사용자가 로그아웃을 하는 경우이다. 이걸 해내기 위해서는, `client.resetStore`를 호출한다. 이 메소드는 비동기인데, 그 이유는 액티브 쿼리를 모두 새로 고치기 때문이다.

```javascript
import { useQuery } from "@apollo/client";
function Profile() {
  const { data, client } = useQuery(PROFILE_QUERY);
  return (
    <Fragment>
      <p>Current user: {data?.currentUser}</p>
      <button onClick={async () => client.resetStore()}>Log out</button>
    </Fragment>
  );
}
```

> 액티브 쿼리 리페칭 없이 캐시를 재설정하려면, `client.clearStore()`를 대신 사용한다.

### 캐시 재설정에 대한 응답

`client.resetStore`가 호출될 때 실행되는 콜백 함수를 등록할 수 있다. `client.onResetStore`를 호출하고 콜백함수를 넘겨주는 것이다. 여러 콜백함수를 등록하려면, `client.onResetStore`를 여러번 호출한다. 모든 콜백함수들은 배열에 추가되며 캐시가 재설정될 때마다 동시에 실행된다.

예를 들면, `client.onResetStore`를 사용하여 기본값을 캐시에 기록한다. 이 기능은 아폴로 클라이언트의 local state management를 사용하고 애플리케이션의 어느 곳에서나 `client.resetStore` 호출할 때 유용하다.

```javascript
import { ApolloClient, InMemoryCache } from "@apollo/client";
import { withClientState } from "apollo-link-state";

import { resolvers, defaults } from "./resolvers";

const cache = new InMemoryCache();
const stateLink = withClientState({ cache, resolvers, defaults });

const client = new ApolloClient({
  cache,
  link: stateLink,
});

client.onResetStore(stateLink.writeDefaults);
```

리액트 컴포넌트에서도 `client.onResetStore`를 호출할 수 있다. 이것은 케시를 초기화한 후 UI를 강제로 리렌더링할 때 유용하다.

`client.onResetStore` 메서드는의 리턴 값은 콜백함수 등록을 취소하기 위해 호출할 수 있는 함수이다:

```javascript
import { useApolloClient } from "@apollo/client";

function Foo() {
  const [reset, setReset] = useState(0);
  const client = useApolloClient();

  useEffect(() => {
    const unsubscribe = client.onResetStore(
      () => new Promise(() => setReset(reset + 1))
    );

    return () => {
      unsubscribe();
    };
  });

  return reset ? <div /> : <span />;
}

export default Foo;
```

## TypePolicy 상속

자바스크립트 개발자는 클래스 선언의 extends 절에서 상속이라는 개념에 익숙할 것이고, `Object.create`로 생성된 프로토타입 체인을 다뤄본 경험이 있을 것이다.

상속은 강력한 code-sharing 도구이고, 여러 이유로 인해 아폴로 클라이언트와 잘 동작한다:

- `InMemoryCache`는 `possibleTypes` 덕분에 상위타입-하위타입 관계(interface와 union 같은)에 대해 이미 알고 있으므로, 해당 정보를 제공하기 위해 추가 설정이 필요하지 않다.
- 상속을 통해 상위타입은 `keyFields`와 개별 필드 정책을 포함한 모든 하위타입에 대한 기본설정 값을 제공할 수 있고, 하위타입에서 선택적으로 원하는 부분을 재정의할 수 있다.
- Graphql 스키마에서 하나의 하위타입은 여러개의 상위타입을 가질 수 있고, 이는 클래스 또느느 프로토타입의 하나의 상속 모델을 사용하여 모델링하는 것이 어렵다. 다른 말로 하면, 자바스크립트에서 여러개룰 상속하는 것을 지원하는데 요구되는 것은 내장된 언어 기능을 재사용하는 것이 아닌, 시스템을 구축해야한다.
- 개발자들은 스키마가 상위타입에 대해 전혀 알지 못하더라도, 타입 간에 동작을 재사용하는 방법으로 `possibleTypes` 맵에 자체 클라이언트 전용 상위타입을 추가할 수 있다.
- `possibleTypes` 맵은 현재 fragment 매칭 목적으로만 사용되고, 이는 클라이언트가 수행하는 작업에서 중요하지만 상당히 사소한 부분이다. 상속은 `possibleTypes`에 대한 또 다른 강력한 용도를 추가ㅎ하고, 효과적으로 사용하는 경우 typePolicies의 반복을 크게 줄일 수 있다.

`InMemoryCache`에서 typePolicy 상속이 어떻게 동작하는지 아래 예시를 통해 생각할 수 있다:

```typescript
const cache = new InMemoryCache({
  possibleTypes: {
    Reptile: ["Snake", "Turtle"],
    Snake: ["Python", "Viper", "Cobra"],
    Viper: ["Cottonmouth", "DeathAdder"],
  },

  typePolicies: {
    Reptile: {
      // 모든 Reptile이 포획되어있고, ID가 있는 태그가 있다고 가정하자.
      keyFields: ["tagId"],
      fields: {
        // 학명 관련 오직은 Reptile 하위타입 간에 공유가 가능
        scientificName: {
          merge(_, incoming) {
            // 모든 학명을 소문자로 정규화한다.
            return incoming.toLowerCase();
          },
        },
      },
    },

    Snake: {
      fields: {
        // 이 뱀이 독사인지 모를때 (venomous) truthy non-boolean 값으로 기본값을 설정
        venomous(status = "unknown") {
          return status;
        },
      },
    },
  },
});
```

## 뮤테이션 이후 쿼리 리패칭하기

특정 경우에는, 뮤테이션 이후 캐시를 업데이트하기 위해 `update` 함수를 작성하는 것이 복잡하거나, 변경된 필드를 리턴하지 않는 경우 불가능할 수도 있다.

이런 경우에는 `refetchQueries` 옵션을 `useMutation` 훅에 제공하여 자동적으로 뮤테이션이 완료된 이후에 특정 쿼리를 재실행할 수 있다.

세부사항은 [여기](https://www.apollographql.com/docs/react/data/mutations/#refetching-queries)를 보시길

> `refetchQueries`는 `update` 함수보다 구현속도가 빠를 수 있지만, 일반적으로는 바람직하지 않는 추가 네트워크 요청이라는 사실을 기억하자. 자세한 내용은 [블로그](https://www.apollographql.com/blog/apollo-client/caching/when-to-use-refetch-queries/) 참고하기.

## 캐시 리다이렉트

어떤 경우에는 쿼리가 다른 참조로 캐시에 이미 존재하는 데이터를 요청하는 경우도 있다.
예를 들면, UI에 리스트뷰와 디테일뷰가 있고, 둘 다 동일한 데이터를 사용할 수 있다.

리스트뷰는 다음 쿼리를 통해 실행된다:

```graphql
query Books {
  books {
    id
    title
    abstract
  }
}
```

특정 book이 선택되면, 디테일뷰는 이 쿼리를 사용할 때 각각의 아이템이 보여지게 된다:

```graphql
query Book($id: ID!) {
  book(id: $id) {
    id
    title
    abstract
  }
}
```

이와 같은 경우, 두 번쨰 쿼리의 데이터가 이미 캐시에 있을 수 있지만 해당 데이터가 다른 쿼리에 의해 가져온 것이기 때문에 아폴로 클라이언트는 이를 알지 못한다. 캐시된 Book 객체를 찾을 위치를 아폴로 클라이언트에 알려주려면, Book 필드에 대한 필드 정책인 `read` 함수를 정의하면 된다:

```javascript
import { ApolloClient, InMemoryCache } from "@apollo/client";

const client = new ApolloClient({
  cache: new InMemoryCache({
    typePolicies: {
      Query: {
        fields: {
          book: {
            read(_, { args, toReference }) {
              return toReference({
                __typename: "Book",
                id: args.id,
              });
            },
          },
        },
      },
    },
  }),
});
```

이 `read` 함수는 `toReference` helper 유틸리티를 사용하여 `__typename` 및 `id`를 기반으로 Book 객체에 대한 캐시 참조를 생성하고 리턴한다.

이제 쿼리에 Book 필드가 포함될 때마다 위의 `read` 함수가 실행되어 Book 객체에 대한 참조를 리턴한다. 아폴로 클라이언트는 이 참조를 사용하여 캐시에서 객체를 조회하고 객체가 있는 경우 리턴한다. 객체가 존재하지 않으면 아폴로 클라이언트는 네트워크를 통해 쿼리를 실행해야한다는 것을 알 수 있다.

> 네트워크 요청을 피하려면 쿼리에서 요청된 모든 필드가 이미 캐시에 있어야 한다. 디테일뷰의 쿼리가 리스트뷰의 쿼리가 가져오지 않은 Book 필드를 가져오는 경우 아폴로 클라이언트는 cache hit이 불완전한 것으로 간주하고 네트워크를 통해 전체 쿼리를 실행한다.

## 페이지네이션 유틸리티

### 증분(Incremental) 로딩: `fetchMore`

`fetchMore` 함수를 사용하여 후속 쿼리에서 리턴된 데이터로 쿼리의 캐시된 결과를 업데이트 할 수 있다. 대부분의 경우. `fetchMore`는 무한 스크롤 페이지네이션과 같은 이미 데이터가 있는데 더 많은 데이터를 로드하는 기타 상황 등을 처리하는 데에 사용된다.

fetchMore 함수는 [여기](https://www.apollographql.com/docs/react/pagination/core-api/#the-fetchmore-function) 참고

### `@connection` 디렉티브

기본적으로 페이지네이션 쿼리는 다른 쿼리와 동일하지만 `fetchMore` 호출이 동일한 캐시 키를 업데이트한다는 점을 제외하면 다르다. 이러한 쿼리는 초기 쿼리와 해당 매개변수 모두에 의해 캐시되기 때문에, 나중에 캐시에서 페이지네이션된 쿼리를 검색하거나 업데이트할 때 문제가 발생한다. limits, offsets 또는 cursors와 같은 페이지 인자들은 `fetchMore`에서 필요한 것 외에는 신경쓰지 않고, 단순히 캐시된 데이터에 접근하기 위해 제공하고자 하는 것도 아니다.

이 문제를 해결하려면 `@connection` 디렉티브를 사용하여 결과에 대한 커스터마이징 캐시키를 지정할 수 있다. `connection`을 사용하면 필드의 캐시 키를 설정하고 어떤 인수가 실제로 쿼리를 변경하는지 필터링할 수 있다.

`@connection` 디렉티브를 사용하려면, 커스텀 스토어 키를 원하는 쿼리 세크먼트에 추가하고 스토어 키를 지정하는 `key` 매개변수를 넘겨준다. `key` 매개변수는 추가적으로 생성된 커스텀 스토어키에 포함할 쿼리 인수 이름의 배열을 받는 `filter` 매개변수를 포함할 수도 있다.

```javascript
const query = gql`
  query Feed($type: FeedType!, $offset: Int, $limit: Int) {
    feed(type: $type, offset: $offset, limit: $limit)
      @connection(key: "feed", filter: ["type"]) {
      ...FeedEntry
    }
  }
`;
```

위 쿼리처럼, 여러 `fetchMore`를 사용하더라도 각 feed 업데이트 결과는 항상 스토어의 feed 키가 최신 누적 값으로 업데이트된다. 이 예시에서는 `@connection` 디렉티브의 선택적 `filter` 인수를 사용하여 스토어 키에 `type` 쿼리 인수를 포함시킴으로써 각 feed 타입의 쿼리를 누적하는 여러 스토어 값을 생성한다.

이제 안정적인 스토어키를 얻었으니 `writeQuery`를 사용하여 스토어 업데이트를 쉽게 수행할 수 있다.

```javascript
client.writeQuery({
  query: gql`
    query Feed($type: FeedType!) {
      feed(type: $type) @connection(key: "feed", filter: ["type"]) {
        id
      }
    }
  `,
  variables: {
    type: "top",
  },
  data: {
    feed: [],
  },
});
```

`type` 인수만 스토어키로 사용하고 있기 때문에, 오프셋이나 limits를 제공할 필요가 없다.
