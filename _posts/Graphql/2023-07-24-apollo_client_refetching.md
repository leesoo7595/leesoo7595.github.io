---
layout: post
title: "Apollo Client - Refetching"
date: 2023-07-24
tags: [Graphql]
comments: true
---

이 글은 아폴로 [공식문서](https://www.apollographql.com/docs/react/data/refetching)를 읽고 정리한 글이다.

# Refetching queries in Apollo Client

아폴로 클라이언트는 캐시를 업데이트해서 GraphQL 데이터를 로컬로 수정할 수 있도록 해준다. 하지만 가끔은 서버에서 쿼리를 리페칭함으로써 클라이언트사이드의 GraphQL 데이터를 업데이트하는 것이 더 간단할 수 있다.

이론적으로는 클라이언트에서 업데이트한 뒤에 모든 액티브 쿼리를 리패칭해야하지만, 쿼리를 선택적으로 리페칭하게되면 시간과 네트쿼으 대역폭을 절약할 수가 있다. `InMemoryCache`를 사용하면 최근 캐시 업데이트로 인해 무효화되었을 수 있는 액티브 쿼리를 확인할 수 있다.

로컬 캐시 업데이트와 리패칭을 함께 사용하면 더욱 좋은 조합이 된다: 어플리케이션에서 로컬 캐시 수정 결과를 즉시 표시하는 동시에, 백그라운드에서 리페칭하여 서버에서 최신 데이터를 가져올 수 있다. UI는 로컬 데이터와 리패치된 데이터의 차이점을 비교하여 리렌더링한다.

리패칭은 특히 mutation 이후에 일반적으로 실행해서, mutate 함수는 리패치되어야하는 쿼리를 특정하고 어떻게 리패치할 것인지와 같은 `refetchQueries`와 `onQueryUpdated` 옵션을 받을 수 있다.

뮤테이션 밖에서 선택적으로 쿼리를 리패치하려면, ApolloClient의 메소드인 `refetchQueries`를 대신 사용할 수 있다.

## `client.refetchQueries`

### Refetch options

`client.refetchQueries` 메소드는 다음과 같은 타입스크립트 인터페이스의 형태인 `options` 객체를 가진다.

```typescript
interface RefetchQueriesOptions<
  TCache extends ApolloCache<any>,
  TResult = Promise<ApolloQueryResult<any>>
> {
  updateCache?: (cache: TCache) => void;
  include?: Array<string | DocumentNode> | "all" | "active";
  onQueryUpdated?: (
    observableQuery: ObservableQuery<any>,
    diff: Cache.DiffResult<any>,
    lastDiff: Cache.DiffResult<any> | undefined
  ) => boolean | TResult;
  optimistic?: boolean;
}
```

#### updateCache

캐시된 필드를 리패치가 트리거 됨으로써 해당하는 필드들을 업데이트할 수 있는 함수

#### include

리패치할 특정 쿼리들이 들어갈 배열. 각 요소는 쿼리의 문자열 이름 또는 DocumentNode 객체가 들어갈 수 있다. 유사한 예로 뮤테이션에 있는 `options.refetchQueries`가 있다. `"active"` 또는 `"all"`을 넘겨주면 모든 액티브 쿼리들을 리패칭한다.

#### onQueryUpdated

`options.updateCache`의 영향을 받거나 `options.include`에 리스트되어있는 `ObservableQuery`가 한번씩 호출되는 콜백함수이다.
만약 `onQueryUpdated`가 제공되지 않으면 기본적으로 `observableQuery.refetch()`(ObservableQuery가 리패칭된)의 결과를 리턴한다. `onQueryUpdated`가 제공되면, 동적으로 각 쿼리가(observableQuery 또는 include, updateCache에 명시된 쿼리) 리페치되어야하는지를 여부를 (또는 어떻게) 결정할 수 있다.
`false`를 리턴하면 연관된 쿼리가 리패치되는 것을 막는다.

#### optimistic

`true`인 경우, `InMemoryCache`의 임시적인 optimistic 레이어에서 `options.updateCache` 함수가 실행되고, 어떤 필드가 무효화되었는지 관찰한 후에 수정 사항을 캐시에서 삭제할 수 있다.
기본적으로는 `false`이다. `options.updateCache`가 캐시를 꾸준히 업데이트한다는 뜻.

### Refetch results

`client.refetchQueries` 메소드는 `onQueryUpdated`가 리턴한 TResult 결과를 수집하고, `onQueryUpdated`가 제공되지 않으면 기본적으로는 `TResult = Promise<ApolloQueryResult<any>>`로 설정된다.
`Promise.all(results)`를 사용하여 이러한 결과를 단일 `Promise<TResolved[]>`로 결합한다.

`Promise.all`의 프로미스 언래핑 동작 덕분에, 이 `TResolved` 타입은 `TResult`가 `PromiseLike<TResolved>` 또는 `boolean`인 경우를 제외하고는 `TResult`과 종종 동일한 타입이다.

리턴된 `Promise` 객체는 두 가지 유용한 프로퍼티를 가지고 있다.

#### queries

리패치된 `ObservableQuery` 객체의 배열

#### results

pending 상태의 프로미스를 포함해서 `onQueryUpdated`에서 리턴된 결과의 배열 또는 `onQueryUpdated`가 없을 때 기본적으로 제공되는 결과 배열이다.
`onQueryUpadted`가 특정 쿼리에 대해 `false`를 리턴하는 경우, 해당 쿼리에 대한 결과가 제공되지 않는다.
`true`인 경우, `results`를 포함한 `Promise<ApolloQueryResult<any>>`의 결과가 리턴된다.

이 두 가지 배열들은 각자가 패러랠하다: 같은 length를 가지고 있고, `results[i]`는 `ObservableQuery`에서 호출된 `queries[i]`에서 i에 해당하는 `onQueryUpdated`에서 생산된 result이다.

### Refetch recipes

#### Refetching a specific query

특정 쿼리 이름으로 리패치를 하려면, `include` 옵션을 사용한다. `include` 옵션은 `DocumentNode`로도 해당하는 쿼리를 리패치할 수 있다.

```typescript
await client.refetchQueries({
  include: ["SomeQueryName"],
});

await client.refetchQueries({
  include: [SOME_QUERY],
});
```

#### Refetching all queries

액티브 쿼리를 모두 리패치하려면 `'active'`를 전달한다. 아폴로클라이언트에서 관리하는 모든 쿼리를 리패치하려면 (심지어 observers도 없고, 어떤 마운트되지 않은 컴포넌트의 쿼리여도) `'all'`을 전달한다.

```typescript
await client.refetchQueries({
  include: "active",
});

await client.refetchQueries({
  include: "all", // Consider using "active" instead!
});
```

#### Refetching queries affected by cache updates

`updateCache` 콜백에서 수행된 캐시 업데이트의 영향을 받는 쿼리를 리패치할 수 있다.

```typescript
await client.refetchQueries({
  updateCache(cache) {
    cache.evict({ fieldName: "someRootField" });
  },
});
```

이렇게 하게되면, `Query.someRootField`에 종속되어있는 모든 쿼리를 리패치한다. 어떤 쿼리가 포함되어있는지 미리 알 필요가 없다.
`updateCache` 안에서는 모든 캐시 operations의 조합이 허용된다. (`writeQuery`, `writeFragment`, `modify`, `evict`...)

`updateCache`에 의해 수행된 업데이트는 기본적으로 캐시에 유지된다. `client.refetchQueries`가 관찰을 완료한 후 캐시를 변경하지 않고 즉시 삭제하는 기능을 사용하고 싶다면 temporary optimistic layer에서 대신 수행할 수 있다.

```typescript
await client.refetchQueries({
  updateCache(cache) {
    cache.evict({ fieldName: "someRootField" });
  },

  // 아래처럼 설정함으로써 Query.someRootField가 temporary optimistic layer에서만 evict한다.
  optimistic: true,
});
```

캐시 데이터를 실제로 변경하지 않고 캐시를 업데이트하는 또 다른 방법은 `cache.modify`와 `INVALIDATE` 센티널 객체를 사용하는 것이다.

```typescript
await client.refetchQueries({
  updateCache(cache) {
    cache.modify({
      fields: {
        someRootField(value, { INVALIDATE }) {
          // Query.someRootField를 포함하는 쿼리를 업데이트한다.
          // 실제로 캐시에서 해당 값을 변경하지 않고 업데이트한다.
          return INVALIDATE;
        },
      },
    });
  },
});
```

`client.refetchQueries`가 소개되기 전에는, `INVALIDATE` 센티넬이 그렇게 유용하지 않았다. 왜냐하면 `fetchPolicy: "cache-first"`를 가지고 있는(캐시 우선인) 무효화된(유효하지 않은?) 쿼리들은 일반적으로 변경되지 않은 결과를 다시 읽어와서 네트워크 요청을 수행하지 않도록한다. `client.refetchQueries` 메소드는 애플리케이션 코드에서 이 무효화 시스템을 더욱 쉽게 접근할 수 있어 무효화된 쿼리의 리패치 동작을 제어할 수 있다...

위 모든 예제에서 `include`나, `updateCache`를 사용하는 것과는 상관없이 `client.refetchQueries`는 네트워크에서 영향을 받는 쿼리를 리패치하고, 그 결과인 `Promise<ApolloQueryResult<any>>`를 `Promise<TResolved[]>`에 포함시켜 리턴한다.

특정 쿼리가 `include`와 `updateCache`에 둘다 포함되어 있다면, 그 쿼리는 한번 리패치 된다. 다시 이야기하면, `include` 옵션을 사용하면 `updateCache`에 포함된 쿼리에 상관없이 특정 쿼리가 항상 포함되도록 하는 좋은 방법이다.(두번 호출할까봐 신경쓰지 않아도됨!)

#### Refetching selectively

개발환경에서는 아마도 적절한 쿼리가 리패치되는 것을 정확히 확인하고 싶을 것이다. 각 쿼리가 리패칭되기 전에 인터셉트하려면, `onQueryUpdated` 함수를 명시하는 방법을 쓸 수 있다.

```typescript
const results = await client.refetchQueries({
  updateCache(cache) {
    cache.evict({ fieldName: "someRootField" });
  },

  onQueryUpdated(observableQuery) {
    // Logging 이나 debugger breakpoints를 개발환경에서 사용하면
    // client.refetchQueries가 어떤 작업을 하는지 이해하기에 유용하다.
    console.log(`Examining ObservableQuery ${observableQuery.queryName}`);
    debugger;

    // onQueryUpdated가 제공되지 않은 경우, 기본적인 리패칭 동작을 이행하도록 한다.
    return true;
  },
});

results.forEach((result) => {
  // 이 리턴값들은 ApolloQueryResult<any> 객체들이고, 네트워크에서 리페칭하여 갖고온 값들이다.
});
```

이 예제에서 어떻게 `client.refetchQueries`의 리패칭 동작을 바꾸지 않고 `onQueryUpdated` 함수를 추가하는지 보여준다. 이런 방식으로 순수하게 진단 또는 디버깅 목적으로만 `onQueryUpdated`를 사용할 수 있다.

만약 포함될 수 있는 특정 쿼리들을 건너뛰려면 `onQueryUpdated`에서 `false`를 리턴한다.

```typescript
await client.refetchQueries({
  updateCache(cache) {
    cache.evict({ fieldName: "someRootField" });
  },

  onQueryUpdated(observableQuery, { complete, result, missing }) {
    console.log(
      `Examining ObservableQuery ${
        observableQuery.queryName
      } whose latest result is ${JSON.stringify(result)} which is ${
        complete ? "complete" : "incomplete"
      }`
    );

    if (shouldIgnoreQuery(observableQuery)) {
      return false;
    }

    // 네트워크에서는 쿼리를 조건없이 리패칭한다.
    return true;
  },
});
```

`ObservableQuery`가 충분한 정보로 제공되지 않는 경우에는, `ObservableQuery`의 두번째 인자로 전달된 `Cache.DiffResult` 객체를 사용해서 쿼리의 최신 결과와 쿼리의 완성 및 누락된 필드를 테스트할 수 있다.

```typescript
await client.refetchQueries({
  updateCache(cache) {
    cache.evict({ fieldName: "someRootField" });
  },

  onQueryUpdated(observableQuery, { complete, result, missing }) {
    if (shouldIgnoreQuery(observableQuery)) {
      return false;
    }

    if (complete) {
      // 네트워크에서 무조건 리패치하는 것이 아닌 선택한 fetchPolicy에 따라 쿼리를 업데이트한다.
      return observableQuery.reobserve();
    }

    // 네트워크에서 무조건 리패치한다.
    return true;
  },
});
```

`onQueryUpdated`는 쿼리를 동적으로 필터할 수 있기 때문에, 위에서 언급했듯이 `include` 옵션을 사용해서 벌크로 사용하는 것도 잘 어울린다.

```typescript
await client.refetchQueries({
  // 모든 액티브 쿼리를 디폴트로 포함시키는 옵션이다.
  // onQueryUpdated를 사용하여 해당 쿼리를 필터링하는 것이 아니면 권장하지 않는다.
  include: "active";

  // 동적 필터링을 허용하여 모든 액티브 쿼리를 한번씩 호출한다.
  onQueryUpdated(observableQuery) {
    return !shouldIngoreQuery(observableQuery);
  },
});
```

#### Handling refetch errors

위 예제에서 우리는 `await client.refetchQueries(...)`를 통해 모든 리패칭된 쿼리를 최종 `ApolloQueryResult<any>` 결과들로 알아낼 수 있다. 이 조합된 프로미스는 `Promise.all`로 만들어져서, 하나의 실패가 일어나면 모든 `Promise<TResolved[]>`를 rejects하여 다른 성공한 결과들을 숨길 수 있다. 이것이 문제가 된다면 `client.refetchQueries`에서 리턴해주는 `queries`와 `results` 배열을 사용할 수 있다.

```typescript
const { queries, results } = client.refetchQueries({
  // ...
});

const finalResults = await Promise.all(
  results.map((result, i) => {
    return Promise.resolve(result).catch(error => {
      console.error(`Error refetching query ${queries[i].queryName}: ${error}`);
      return null; // 프로미스 reject가 일어난 경우 무시하도록 한다.
    });
  })
});
```

미래에는, `client.refetchQueries` 메소드에 추가 입력 옵션이 추가될 수 있다. 결과 객체에 프로퍼티를 추가하여 프로미스 관련 프로퍼티와 `queries` 및 `results` 배열을 보완할 수 있을 것이다.

만약 어떤 새로운 `client.refetchQueries` 입력 옵션 또는 결과 프로퍼티들이 유용할 것 같으면 언제든지 이슈를 개설하거나 사용 사례를 설명하는 토론을 시작해주세요.

## Corresponding `client.mutate` options

뮤테이션 이후 리패칭을 하려면, `client.refetchQueries`를 사용하는 것 대신 `client.mutate`에서 `client.refetchQueries`와 비슷한 옵션을 제공한다. 왜냐하면 뮤테이션이 진행하는 중 어떤 특정 시간에 리패칭이 실행되는 것이 중요하기 때문이다.

역사적인 이유로, `client.mutate` 옵션은 새로운 `client.refetchQueries` 옵션과 조금 다르지만, 내부적인 구현은 충분히 동일하다. 다음과 같은 옵션을 보고 해석하여 사용할 수 있다.

#### options.refetchQueries

`client.refetchQueries`의 `options.include`와 같이 구현

#### options.update

`client.refetchQueries`의 `options.updateCache`와 같이 구현

#### options.onQueryUpdated

`client.refetchQueries`의 `options.onQueryUpdated`와 같이 구현

#### options.awaitRefetchQueries

`client.refetchQueries`의 `options.onQueryUpdated`가 프로미스로 리턴됨
