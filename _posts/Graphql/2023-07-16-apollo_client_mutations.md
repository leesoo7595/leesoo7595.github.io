---
layout: post
title: "Apollo Client - Mutations"
date: 2023-07-16
tags: [Graphql]
comments: true
---

이 글은 아폴로 클라이언트 [문서](https://www.apollographql.com/docs/react/data/mutations/)를 읽고 정리한 글이다.

# Mutations in Aollo Client

이제 서버에서 아폴로 클라이언트를 통해 데이터를 어떻게 쿼리하는지 알아보았고, 다음 스텝은 mutations을 활용해서 백엔드 데이터를 어떻게 수정하는지를 알아보자

이 글은 그랲큐엘 서버에서 useMutation를 사용하여 업데이트를 보내는 법을 알려준다. 또한 아폴로 클라이언트에서 mutation을 실행했을 때, 캐시를 어떻게 업데이트하는지 그리고 에러 상태와 로딩은 어떻게 트래킹하는지 알 수 있다.

## Executing a mutation

useMutation 훅은 아폴로 어플리케이션에서 뮤테이션을 실행하는데 기본적인 API이다.

뮤테이션을 실행하기 위해서는 컴포넌트 안에서 useMutation을 실행해서 실행하고 싶은 뮤테이션을 넘겨주면 된다.

```typescript
import { gql, useMutation } from "@apollo/client";

// Define mutation
const INCREMENT_COUNTER = gql`
  # Increments a back-end counter and gets its resulting value
  mutation IncrementCounter {
    currentValue
  }
`;

function MyComponent() {
  // Pass mutation to useMutation
  const [mutateFunction, { data, loading, error }] =
    useMutation(INCREMENT_COUNTER);
}
```

gql 함수로 뮤테이션 문자열을 Graphql document로 해석해서 useMutation에 넣어주어 전달한다.

컴포넌트가 렌더링될 때, useMutation은 다음을 포함한 튜플을 리턴한다.

- 언제든지 실행할 수 있는 mutation 함수
  - useQuery과는 달리 useMutation는 렌더링시 자동으로 실행되지 않는다. 대신 직접 뮤테이션 함수를 호출해야한다.
- 뮤테이션 실행에 관련해서 현재 상태를 나타내는 필드를 담은 객체 (data, loading 등등)
  - 이 객체는 useQuery 훅에서 리턴하는 객체와 비슷하다.

```jsx
function AddTodo() {
  let input;
  const [addTodo, { data, loading, error }] = useMutation(ADD_TODO);

  if (loading) return "Submitting...";
  if (error) return `Submission error! ${error.message}`;

  return (
    <div>
      <form
        onSubmit={(e) => {
          e.preventDefault();
          addTodo({ variables: { type: input.value } });
          input.value = "";
        }}
      >
        <input
          ref={(node) => {
            input = node;
          }}
        />
        <button type="submit">Add Todo</button>
      </form>
    </div>
  );
}
```

이 예제에서는 폼의 onSubmit이라는 핸들러에서 addTodo라는 뮤테이션 함수를 호출한다. addTodo는 useMutation 훅에서 리턴되는 아이이다. 이렇게 하면 아폴로 클라이언트가 그래프큐엘 서버로 전송하여 뮤테이션을 실행하도록 한다.

컴포넌트가 렌더링되자마자 동작이 실행되는 useQuery와 차이점을 기억하자. 뮤테이션은 사용자의 액션에 따라 실행되는 것이 일반적이기 떄문이다.

### Providing options

useMutation 훅은 두번째 인자로 options 객체를 받는다.

```typescript
// variables를 디폴트로 넣어줄 수 있다.
const [addTodo, { data, loading, error }] = useMutation(ADD_TODO, {
  variables: {
    type: "placeholder",
    someOtherVariable: 1234,
  },
});

// 뮤테이션 함수에 직접적으로 옵션을 넣을 수도 있다.
addTodo({
  variables: {
    type: input.value,
  },
});
```

#### option precedence

useMutation과 뮤테이션 함수 둘다 옵션을 넣어주면, 뮤테이션 함수의 값이 우선순위가 된다. variables 옵션의 경우, 두 객체가 얕은 복사로 병합되어 useMutation에만 제공된 변수는 유지되어 실행된다. 이것은 variables 옵션의 디폴트 값을 셋업하는데 유용하다.

### Tracking mutation status

뮤테이션함수는 추가적으로 useMutation 훅에서 뮤테이션 실행에 대한 현재 상태를 나타내는 객체로 리턴한다. 이 객체의 필드들은 뮤테이션함수가 호출되었는지 여부, 그리고 결과가 현재 loading 상태인지 여부를 나타내는 booleans을 포함하고 있다.

useMutation 훅은 onCompleted과 onError 옵션 또한 제공한다.

### Resetting mutation status

useMutation에서 리컨되는 뮤테이션 결과 객체는 reset 함수도 포함한다.

```javascript
const [login, { data, loading, error, reset }] = useMutation(LOGIN_MUTATION);
```

reset 함수를 호출하면 뮤테이션의 결과 값을 초기 상태(뮤테이션함수가 실행되기 전 상태로)로 리셋할 수 있다. 이를 사용하여 사용자가 뮤테이션 결과 데이터나 UI 에러를 무시할 수 있도록 할 수 있다.

reset을 호출하는 것이 뮤테이션 실행에 의해 리턴된 데이터 캐시를 지우지는 않는다. reset 함수는 useMuatation 훅에 관련된 상태만 영향을 준다. 그래서 해당 컴포넌트를 다시 렌더링하게끔 해준다.

```jsx
function LoginPage() {
  const [login, { error, reset }] = useMutation(LOGIN_MUTATION);

  return (
    <>
      <form>
        <input class="login" />
        <input class="password" />
        <button onclick={login}>Login</button>
      </form>
      {error && (
        <LoginFailedMessageWindow
          message={error.message}
          onDismiss={() => reset()}
        />
      )}
    </>
  );
}
```

## Updating local data

뮤테이션을 실행하면 백엔드 데이터를 수정한다. 어쩔때는 백엔드 데이터가 수정된거에 따라서 바로 로컬 캐시 데이터를 업데이트하고 싶을 수 있다. 예를 들면, 뮤테이션으로 투두리스트에 아이템이 추가되면, 해당 리스트를 바로 캐시된 데이터로 나타내고 싶을 것이다.

### Supported methods

로컬 데이터를 업데이트하는 가장 간단한 방법은 뮤테이션 영향을 받을 수 있는 모든 쿼리를 다시 가져오는 것이다. 하지만 이건 네트워크 요청을 추가적으로 해야한다.

만약 뮤테이션이 변경된 필드를 리턴하도록 한다면, 다른 추가적인 네트워크 요청 필요없이 캐시를 바로 업데이트할 수 있다. 하지만 이 방법은 뮤테이션이 더 복잡해져서 복잡성을 증가시킬 수 있다.

아폴로 클라이언트를 막 시작했다면, 캐시된 데이터를 업데이트하는데 쿼리를 refetch하는 방법을 추천한다. 이후 캐시를 직접 업데이트하여 앱의 응답성을 개선시킬 수 있다.

## Refetching queries

특정 뮤테이션을 실행한 후 쿼리를 리패치하는 일들이 자주 있기 때문에, 뮤테이션 옵션으로 refetchQueries 배열을 포함하는 방식을 사용할 수 있다.

```typescript
// Refetches two queries after mutation completes
const [addTodo, { data, loading, error }] = useMutation(ADD_TODO, {
  refetchQueries: [
    GET_POST, // DocumentNode object parsed with gql
    "GetComments", // Query name
  ],
});
```

`refetchQueries` 배열에 들어갈 수 있는 요소는 다음과 같다.

- gql 함수로 파싱된 `DocumentNode` 객체
- 전에 실행했던 문자열로 된 쿼리의 이름
  - 쿼리들을 이름으로 참조하려면 각 쿼리에 고유한 이름이 있는지 확인이 필요하다.

포함된 쿼리의 경우, 가장 최근에 제공된 variables로 실행된다.

refetchQueries 옵션을 useMutation과 뮤테이션 함수에 제공할 수 있다. 이것 또한 옵션 우선순위에 따라 동작한다.

앱에는 수십개 또는 수백개의 다른 쿼리들이 있는 경우 어떤 쿼리를 다시 리페치해야하는지 결정하기 어려울 수 있는 것을 기억해라

## Updating the cache directly

### Include modified objects in mutation responses

대부분의 경우 뮤테이션 응답은 뮤테이션으로 인해 수정된 객체 값을 포함해야한다. 이렇게 하면, 기본적으로 아폴로 클라이언트가 해당 객체들을 \_\_typename 및 id 필드를 활용해서 정규화 및 캐싱을 할 수 있다.

아폴로 클라이언트는 자동적으로 쿼리와 뮤테이션의 모든 객체에 `__typename` 필드를 추가한다.

```json
{
  "__typename": "Todo",
  "id": "5",
  "type": "groceries"
}
```

위 응답객체를 받으면, 아폴로 클라이언트는 `Todo:5`라는 키 값으로 캐시한다. 만약 캐시된 객체가 이미 해당 키로 존재한다면 아폴로 클라이언트는 뮤테이션 응답에 포함된 모든 기존 필드를 덮어씌운다.(다른 기존 필드는 유지된다.)

수정된 객체가 반환되면 이런식으로 백엔드와 캐시 데이터를 맞추는데 유용하다. 하지만 이걸로는 충분하지 않을 수 있다. 예를 들면 새롭게 캐시된 객체는 해당 객체를 포함해야하는 리스트 필드에 자동으로 추가되지 않는다. 이걸 해결하려면, update 함수를 정의할 수 있다.

### The update function

뮤테이션 응답으로 캐시에 수정된 필드를 업데이트하는 것이 불충분할 때 (특정 리스트 필드와 같이) update 함수를 선언해서 뮤테이션 후 바뀐 캐시 데이터를 수동적으로 적용시킬 수 있다.

```jsx
const GET_TODOS = gql`
  query GetTodos {
    todos {
      id
    }
  }
`;

function AddTodo() {
  let input;
  const [addTodo] = useMutation(ADD_TODO, {
    update(cache, { data: { addTodo } }) {
      cache.modify({
        fields: {
          todos(existingTodos = []) {
            const newTodoRef = cache.writeFragment({
              data: addTodo,
              fragment: gql`
                fragment NewTodo on Todo {
                  id
                  type
                }
              `,
            });
            return [...existingTodos, newTodoRef];
          },
        },
      });
    },
  });

  return (
    <div>
      <form
        onSubmit={(e) => {
          e.preventDefault();
          addTodo({ variables: { type: input.value } });
          input.value = "";
        }}
      >
        <input
          ref={(node) => {
            input = node;
          }}
        />
        <button type="submit">Add Todo</button>
      </form>
    </div>
  );
}
```

useMutation 함수를 통해서 update 함수를 전달할 수 있다.

update 함수에는 아폴로 클라이언트 캐시를 나타내는 cache 객체를 전달된다. 이 객체는 `readQuery/writeQuery`, `readFragment/writeFragment`, `modify`와 `evict` 같은 cache API에 접근할 수 있도록 한다. 이 메소드들은 그래프큐엘 서버와 상호작용하는 것처럼 캐시에서 동작을 실행할 수 있다.

제공하고 있는 캐시 함수를 알고 싶으면 [여기](https://www.apollographql.com/docs/react/caching/cache-interaction/)

update 함수는 또한 mutation의 결과가 포함된 data 프로퍼티를 담은 객체도 전달된다. 이 값을 사용해서 `cache.writeQuery`, `cache.writeFragment`, `cache.modify`를 사용해서 캐시를 업데이트할 수 있다.

mutation이 optimistic response를 지정하는 경우, update 함수는 optimistic 결과를 한번, mutation의 실제 리턴값이 올때 한번, 총 두번 호출된다.

위 예제에서 `ADD_TODO` 뮤테이션이 실행될 때, update 함수가 실행되기전 새롭게 추가되어 리턴된 `addTodo` 객체가 자동으로 캐시에 저장된다. 그러나 `ROOT_QUERY.todos`의 캐시 리스트(`GET_TODOS` 쿼리를 보는)는 자동으로 업데이트되지 않는다. 이것은 `GET_TODOS` 쿼리가 새로운 Todo 객체에 대한 알림을 받지 못하기 때문이다.

이걸 알려주려면, 우리는 modifier 함수를 실행시킴으로써 `cache.modify`를 사용하여 캐시에서 아이템을 직접 삽입하거나 삭제한다. 위 예제에서 우리는 `GET_TODOS`의 결과를 `ROOT_QUERY.todos` 캐시 배열에 저장되어있다는 것을 알고 있다. 그래서 우리는 `todos`라는 modifier 함수를 사용하여 Todo에 새롭게 추가된 아이를 캐시에 업데이트해줄 수 있다. `cache.writeFragment`로는 추가된 Todo에 대한 내부 참조를 얻어서 해당 참조를 ROOT_Query.todos의 배열에 추가한다.

update 함수 내에서 캐시된 데이터를 변경하면 해당 데이터의 변경사항을 수신중인 쿼리에 자동으로 브로드캐스트된다. 그래서 애플리케이션의 UI가 캐시된 값을 반영하도록 업데이트된다.

### Refetching after update

update 함수는 클라이언트의 로컬 캐시에서 뮤테이션의 백엔드 수정을 복제하는 것을 시도한다. 이 캐시 수정사항들은 모든 액티브한 쿼리들에게 알려주고, UI가 자동으로 업데이트된다. 만약 `update` 함수가 올바르게 동작하면 사용자는 다른 네트워크가 돌아가는 것을 기다릴 필요없이 최신 데이터를 즉시 보게된다.

그러나 `update` 함수는 캐시된 값을 잘못 설정하여 이 복제를 잘못할 수 있다. 영향을 받는 액티브한 쿼리를 다시 가져와서 `update` 함수의 수정 사항을 더블체크할 수 있다. 그렇게 하려면, `onQueryUpdated` 콜백함수를 mutation 함수 내에서 사용한다.

```typescript
addTodo({
  variables: { type: input.value },
  update(cache, result) {
    // Update the cache as an approximation of server-side mutation effects
  },
  onQueryUpdated(observableQuery) {
    // Define any custom logic for determining whether to refetch
    if (shouldRefetchQuery(observableQuery)) {
      return observableQuery.refetch();
    }
  },
});
```

update 함수가 실행된 후, 아폴로 클라이언트는 `onQueryUpdated`를 해당 업데이트되어 캐시된 필드를 가진 각 활동 쿼리에서 한번씩 호출한다. `onQueryUpdated`를 사용하면 커스텀 로직을 선언해서 관련 쿼리 refetch 여부를 결정할 수 있다.

`onQueryUpdated`를 사용하여 쿼리를 refetch하려면, 위처럼 `observableQuery.refetch()`를 반환한다. 아니면 아무런 값도 요구되지않는다. refetch된 쿼리의 응답이 update 함수의 수정사항과 다른 경우, 캐시와 UI가 모두 자동으로 업데이트된다. 아니면 사용자에게 변경사항이 표시되지 않는다.

가끔, 관련된 모든 쿼리를 수정하는 update 함수를 호출하는게 어려울 것이다. 모든 뮤테이션들이 update 함수를 통해 효율적으로 수정하여 적절한 정보를 리턴하진 않는다. 특정 쿼리가 반드시 포함되도록 하려면, onQueryUpdated와 refetchQueries를 같이 사용한다.

```typescript
addTodo({
  variables: { type: input.value },
  update(cache, result) {
    // Update the cache as an approximation of server-side mutation effects.
  },
  // Force ReallyImportantQuery to be passed to onQueryUpdated.
  refetchQueries: ["ReallyImportantQuery"],
  onQueryUpdated(observableQuery) {
    // If ReallyImportantQuery is active, it will be passed to onQueryUpdated.
    // If no query with that name is active, a warning will be logged.
  },
});
```

업데이트 함수로 인해 ReallyImportantQuery가 이미 onQueryUpdated로 전달될 예정이었다면, 이 함수는 한 번만 전달된다. refetchQueries를 사용하면 해당 쿼리가 반드시 포함되도록 보장한다.

예상했던 것보다 많은 쿼리가 포함되었다면, `ObservableQuery`를 검사해서 다시 가져올 필요가 없는지 확인 후 `onQueryUpdated`에서 false를 리턴하여 쿼리를 건너뛰거나 무시할 수 있다. `onQueryUpdated`에서 프로미스를 리턴하면 뮤테이션의 최종 값이 모든 프로미스를 대기하게 되어서 레거시인 `awaitRefetchQueries: true` 옵션이 필요가 없어진다.

뮤테이션을 수행하지 않고 `onQueryUpdated` API를 사용하려면 `client.refetchQueries` 메서드를 사용할 수 있다. 독립실행형인 `client.refetchQueries` API에서 `refetchQueries` 뮤테이션 옵션은 `include:...`에서 호출되고, update 함수는 명확성을 위해 `updateCache`로 호출된다.
아니면 같은 내부 시스템으로 `client.refetchQueries`와 뮤테이션 후 쿼리 리페칭을 모두 구동한다.

## useMutation API

### options

- mutation
- variables
- errorPolicy
- onCompleted
- onError
- onQueryUpdated
- refetchQueries
- awaitRefetchQueries
- ignoreResults
