---
layout: post
title: "Apollo Client - Reading and Writing data to the cache"
date: 2023-08-10
tags: [Graphql]
comments: true
---

이 글은 [아폴로 공식문서](https://www.apollographql.com/docs/react/caching/cache-interaction/)를 읽고 정리한 글 이다.

# 캐시를 통해 데이터 읽고 쓰기

Graphql 서버를 통하지 않고도 아폴로 클라이언트 캐시를 통해 데이터를 직접적으로 읽고 쓸 수 있다. 서버에 패칭하기 전에 데이터와 상호작용이 가능하고 로컬로만 사용할 수 있는 데이터로 작업이 가능하다.

아폴로 클라이언트는 캐시된 데이터와 상호작용하는 몇가지 기술을 지원한다:

- [그랲큐엘 쿼리 사용하기](https://www.apollographql.com/docs/react/caching/cache-interaction/#using-graphql-queries)

  - `readQuery`/`writeQuery`/`updateQuery`
  - 원격 및 로컬 데이터를 모두 관리하기 위해 표준 그랲큐엘 쿼리를 사용한다.

- [그랲큐엘 프래그먼트 사용하기](https://www.apollographql.com/docs/react/caching/cache-interaction/#using-graphql-fragments)

  - `readFragment` / `writeFragment` / `updateFragment` / `useFragment`
  - 해당 객체를 가져올 때 전체 쿼리를 작성할 필요없이 캐시된 객체의 필드에 접근할 수 있다.

- [캐시필드를 직접적으로 수정하기](https://www.apollographql.com/docs/react/caching/cache-interaction/#using-cachemodify)
  - `cache.modify`
  - 그랲큐엘을 사용하지 않고 캐시된 데이터를 조작한다.

케이스에 맞춰 도움이되는 전략과 방법을 조합하여 사용할 수 있다.

## 그랲큐엘 쿼리 사용하기

서버에서 실행하는 쿼리와 유사하거나 동일한 그랲큐엘 쿼리를 사용하여 캐시 데이터를 읽고 쓸 수 있다:

### `readQuery`

`readQuery` 메소드는 다음과 같이 캐시에서 직접 그랲큐엘 쿼리를 가져올 수 있다:

```javascript
const READ_TODO = gql`
  query ReadTodo($id: ID!) {
    todo(id: $id) {
      id
      text
      completed
    }
  }
`;

// ID가 5인 캐시된 todo 아이템을 갖고옴
const { todo } = client.readQuery({
  query: READ_TODO,
  // 필요한 variables를 넣어준다.
  // 맞지않는 variables 타입일 경우 null을 리턴한다.
  variables: {
    id: 5,
  },
});
```

만약 캐시가 쿼리의 모든 필드에 대한 데이터가 포함되어있는 경우에 `readQuery`는 쿼리의 형태와 일치하는 객체를 반환한다.

```javascript
{
  todo: {
    __typename: 'Todo', // __typename은 자동으로 추가됨
    id: 5,
    text: 'Buy oranges 🍊',
    completed: true
  }
}
```

> 쿼리 문자열에 `__typename` 필드를 포함하지 않더라도 Apollo 클라이언트는 기본적으로 모든 개체의 `__typename`을 자동으로 쿼리한다.

**리턴된 객체를 직접 수정하지 말자.** 동일한 객체가 여러개의 컴포넌트에서 리턴될 수 있다. 캐시된 데이터를 안전하게 업데이트하기 위해서는 [읽기와 쓰기의 조합](https://www.apollographql.com/docs/react/caching/cache-interaction/#combining-reads-and-writes)을 보기.

캐시에 쿼리 필드 중 어떤 데이터가 누락된 경우, `readQuery`는 null을 리턴한다. `readQuery`에서 값이 없다고 서버에서 갖고 오지는 않는다.

> 아폴로 클라이언트 3.3 이전까지는, `readQuery`는 `MissingFieldError`라는 예외를 없는 필드와 함께 던졌었다. 3.3부터는 `readQuery`는 특정 필드가 없는 경우 항상 null을 리턴한다.

### `writeQuery`

`writeQuery` 메소드는 그랲큐엘 쿼리와 일치하는 형태로 데이터를 캐시에 입력할 수 있다. `readQuery`와 비슷하지만, `data` 옵션을 추가해주어야한다:

```javascript
client.writeQuery({
  query: gql`
    query WriteTodo($id: Int!) {
      todo(id: $id) {
        id
        text
        completed
      }
    }
  `,
  data: {
    // write를 하려면 데이터를 넣어주어야한다.
    todo: {
      __typename: "Todo",
      id: 5,
      text: "Buy grapes 🍇",
      completed: false,
    },
  },
  variables: {
    id: 5,
  },
});
```

이 예시는 ID가 5인 Todo 객체의 캐시를 만드는 방법이다.

`writeQuery`를 사용할 경우 다음을 기억하자:

- `writeQuery`로 캐시된 데이터에 대한 변경 사항은 Graphql 서버에 푸시되지 않는다. 즉, 환경을 다시 로드하면 변경사항이 사라진다.
- 쿼리의 형태는 그랲큐엘 서버의 스키마에 의해 강제되지 않는다.
  - 쿼리는 스키마에 없는 필드를 포함할 수 있다.
  - 스키마에서 유효하지 않은 스키마 필드를 제공할 수 있지만 일반적으로는 하지 말아야한다.

#### 존재하는 데이터를 수정하기

Todo 객체의 ID가 5인 캐시가 이미 존재할 때, `writeQuery`는 포함되는 데이터의 필드를 덮어쓴다. (다른 필드는 보존된다.):

```javascript
client.writeQuery({
  query: gql`
    query WriteTodo($id: Int!) {
      todo(id: $id) {
        id
        text
        completed
      }
    }`,
  data: { // Contains the data to write
    todo: {
      __typename: 'Todo',
      id: 5,
      text: 'Buy grapes 🍇',
      completed: false
    },
  },
  variables: {
    id: 5
  }
});

// BEFORE
{
  'Todo:5': {
    __typename: 'Todo',
    id: 5,
    text: 'Buy oranges 🍊',
    completed: true,
    dueDate: '2022-07-02'
  }
}

// AFTER
{
  'Todo:5': {
    __typename: 'Todo',
    id: 5,
    text: 'Buy grapes 🍇',
    completed: false,
    dueDate: '2022-07-02'
  }
}
```

> 쿼리에 필드를 포함하지만 데이터에 해당 필드에 대한 값을 포함하지 않은 경우, 필드의 현재 캐시된 값이 유지된다.

## 그랲큐엘 프래그먼트 사용하기

정규화된 캐시 객체에서 그래프큐엘 프래그먼트를 사용하여 캐시 데이터를 읽고 쓸 수 있다. 이렇게하면 완전히 유효한 쿼리가 필요한 `readQuery`/`writeQuery`보다 캐시된 데이터에 더 많은 'random-access'가 가능하다.

### `readFragment`

이 예시는 `readFragment`를 대신 사용하여 `readQuery`의 예시와 동일한 데이터를 가져온다:

```javascript
const todo = client.readFragment({
  id: "Todo:5", // Todo 아이템의 캐시 ID에 해당하는 값
  fragment: gql`
    fragment MyTodo on Todo {
      id
      text
      completed
    }
  `,
});
```

`readQuery`와는 달리, `readFragment`는 `id` 옵션을 필수로 주어야한다. 이 옵션은 캐시에 있는 객체의 캐시ID를 지정한다. 기본적으로 캐시ID는 `<__typename>:<id>` 포맷을 가지고 있다.(위 예제에서 캐시ID로 Todo:5를 주고 있다.) ID를 커스터마이징할 수 있다.

위 예제에서, `readFragment`는 다음 두 경우 중 하나에 해당하는 경우 null을 리턴한다.

- ID가 5인 Todo 객체가 캐시되어있지 않을때
- ID가 5인 Todo 객체가 캐시되어있지만, `text`나 `completed` 값 중 하나가 없을때

### `writeFragment`

`readFragment`를 사용하여 아폴로 클라이언트 캐시에서 'random-access' 데이터를 읽을 수 있을 뿐 아니라, `writeFragment` 메서드를 사용하여 데이터를 쓰는 것도 가능하다.

> `writeFragment`를 사용한 캐시된 데이터에 대한 변경 사항은 그래프큐엘 서버에 푸시되지 않는다. 환경을 리로드하는 경우, 변경 사항이 사라진다.

`writeFragment` 메서드는 `data` variable만 추가적으로 요구하는 것 빼고는 `readFragment`와 비슷하다. 예를 들면, 다음과 같은 `writeFragment` 호출은 id가 5인 Todo 객체에 대한 `completed` 플래그를 업데이트한다.

```javascript
client.writeFragment({
  id: "Todo:5",
  fragment: gql`
    fragment MyTodo on Todo {
      completed
    }
  `,
  data: {
    completed: true,
  },
});
```

아폴로 클라이언트 캐시를 보고 있는 것들은 (액티브 쿼리 포함) 이 변경 사항을 확인하고 그에 따라 애플리케이션 UI를 업데이트한다.

### `useFragment`

`useFragment` 훅을 사용하여 캐시에서 직접 특정 프래그먼트의 데이터를 읽을 수 있다. 이 훅은 캐시가 현재 특정 프래그먼트에 대해 포함하고 있는 모든 데이터에 대한 always-up-to-date를 리턴한다. [API를 참고](https://www.apollographql.com/docs/react/api/react/hooks/#usefragment)

## 읽기와 쓰기 조합하기

`readQuery`와 `writeQuery`(또는 `readFragment`와 `writeFragment`)를 조합해서 현재 캐시된 데이터를 가져와서 선택적으로 수정할 수 있다. 다음 예제는 새로운 Todo 아이템을 만들어서 캐시된 todo리스트에 추가한다. 이 추가 작업은 서버에는 보내지지 않는다는 것을 기억해라..

```javascript
// 현재 존재하는 모든 to-do 아이템을 쿼리
const query = gql`
  query MyTodoAppQuery {
    todos {
      id
      text
      completed
    }
  }
`;

// 현재 to-do 리스트를 가져온다.
const data = client.readQuery({ query });

// 새로운 to-do 아이템 생성
const myNewTodo = {
  id: "6",
  text: "Start using Apollo Client.",
  completed: false,
  __typename: "Todo",
};

// todo 리스트를 다시 작성하여 새로운 아이템 추가
client.writeQuery({
  query,
  data: {
    todos: [...data.todos, myNewTodo],
  },
});
```

### `updateQuery`와 `updateFragment` 사용하기

편의를 위해 캐시를 읽고 쓰는데 싱글메소드 호출로 조합해서 `cache.updateQuery` 또는 `cache.updateFragment`를 사용할 수 있다.

```javascript
// Query to fetch all todo items
const query = gql`
  query MyTodoAppQuery {
    todos {
      id
      text
      completed
    }
  }
`;

// 캐시에 있는 모든 todo 리스트를 완벽하게 세팅하기
cache.updateQuery({ query }, (data) => ({
  todos: data.todos.map((todo) => ({ ...todo, completed: true })),
}));
```

이 메소드는 두 개의 인자를 받는다:

- `read`(`query`, `fragment`) 메소드를 포함한 options
- `update` 함수

두 메서드 모두 캐시에서 데이터를 가져온 후 `update` 함수를 호출하여 캐시된 데이터를 전달한다. `update` 함수는 캐시 안에 있는 data를 대체할 값을 리턴할 수 있다. 위 예제를 보면, 모든 캐시된 Todo 객체는 `completed` 필드가 true로 설정되어있다. (다른 필드는 변경되지 않은 상태르 유지된다.)

대체값의 경우 불변의 방식으로 계산해야한다는 걸 기억하자. immutable updates에 대한 자세한 내용은 [리액트 문서](https://react.dev/learn/updating-objects-in-state) 참고

`update` 함수가 캐시된 데이터를 변경하지 않아야하는 경우, `undefined`를 리턴할 수 있다.

`update` 함수의 리턴값은 writeQuery 또는 writeFragment로 전달되고, 이 함수는 캐시된 데이터를 수정한다.

## `cache.modify` 사용하기

`InMemoryCache`의 `modify` 메소드는 각각의 캐시된 필드들을 직접 수정하거나 완전이 삭제하는 것이 가능하다.

- `writeQuery`와 `writeFragment`처럼, `modify`는 수정된 필드에 의존하는 모든 액티브 쿼리의 새로고침이 트리거된다. ()`broadcast: false`를 전달하는 경우 이 동작을 재정의할 수 있음)
- `writeQuery`와 `writeFragment`와 달리:

  - `modify`는 정의한 merge 함수를 우회한다. 즉, 필드가 항상 지정한 값으로 정확하게 덮어쓴다.
  - `modify`는 캐시에 아직 존재하지 않는 필드를 쓸 수 없다.

- 워치하고 있는 쿼리는 `client.watchQuery` 또는 `useQuery` 훅에 `fetchPolicy` 및 `nextFetchPolicy`와 같은 옵션을 전달해서 캐시 업데이트에 의해 유효하지 않을때 발생하는 상황을 컨트롤할 수 있다.

### 파라미터

`modify` 메소드에는 다음 매개변수가 사용된다:

- 수정할 캐시된 객체의 ID(`cache.identify`로 가져오는 것을 권장)
- 실행할 `modifier` 함수 맵(수정할 필드마다 하나씩)
- 선택적 `broadcast`와 `optimistic` boolean 값으로 동작을 커스터마이징할 수 있다.

modifier 함수는 단일 필드에 적용된다. 이 함수는 관련된 필드의 캐시된 값을 파라미터로 받아서 이를 대체할 값을 리턴한다.

다음 예제는 name 필드를 수정해서 값을 대문자로 변환하는 modify 호출의 예시이다:

```javascript
cache.modify({
  id: cache.identify(myObject),
  fields: {
    name(cachedName) {
      return cachedName.toUpperCase();
    },
  },
  /* broadcast: false // 자동 쿼리 새로고침을 방지하려면 이 내용을 포함하기 */
});
```

특정 필드에 `modifier` 함수를 제공하지 않으면 해당 필드의 캐시된 값은 변경되지 않은 상태로 유지된다.

#### 값 vs 참조

필드에 대한 modifier 함수를 정의할 떄 스칼라, 열거형 또는 기본 타입의 리스트가 포함된 경우 modifier 함수에는 필드의 정확한 기존 값이 전달된다. 예를 들면, 5라는 값을 가지고 있는 한 객체의 `quantity` 필드에 대한 modifier 함수를 정의할 때, modifier 함수에 5를 전달된다.

그러나, 객체 타입 또는 객체 리스트에서 포함된 필드의 modifier 함수를 정의할 때, 해당 객체들은 참조되어 나타낸다. 각 참조 포인트는 캐시ID에 대응한다. 만약 modifier 함수에서 다른 참조값을 리턴하게 되면, 이 필드가 포함된 캐시된 다른 객체를 바꾸게 된다. 기존 캐시된 객체의 데이터를 수정하면 안된다.

### modifier 함수 유틸리티

modifier 함수는 두번째 파라미터를 선택적으로 받을 수 있는데, 여기엔 몇개의 유틸리티를 포함하고 있는 객체가 있다.

이 유틸리티들은(`readField`와 `DELETE` 센티널 객체) 아래 예제에서 사용되고 있다. 사용가능한 모든 유틸리티에 대한 설명은 [API reference 참고](https://www.apollographql.com/docs/react/api/cache/InMemoryCache/#modifier-function-api)

### 예시

#### 리스트에서 하나의 아이템 제거

블로그 어플리케이션이 있고, 각 Post는 Comment 리스트를 가지고 있다. 페이지네이팅된 Post.comments 배열의 특정 Comment를 지우는 방법이다:

```javascript
const idToRemove = "abc123";

cache.modify({
  id: cache.identify(myPost),
  fields: {
    comments(existingCommentRefs, { readField }) {
      return existingCommentRefs.filter(
        (commentRef) => idToRemove !== readField("id", commentRef)
      );
    },
  },
});
```

파헤쳐보자.

- `id` 필드는 `cache.identify`를 사용하여 제거하려는 comment가 있는 Post 객체의 캐시ID를 얻어온다.
- `fields` 필드에는 modifier 함수들을 넣을 수 있는 객체를 제공한다. 이 경우 단일 modifier 함수를 정의한다.(comments 필드의)
- `comments` modifier 함수는 `existingCommentRefs`로 기존 캐시된 comments 배열을 받을 수 있다. 또한 캐시된 필드의 값을 읽는데 유용한 `readField` 유틸리티 함수를 사용한다.
- modifier 함수는 `idToRemove`와 일치하는 ID를 가진 모든 댓글을 필터링하는 배열을 리턴한다. 리턴된 배열을 캐시에 있는 기존 배열을 대체한다.

#### 리스트에서 하나의 아이템 추가

Post에서 Comment를 추가하기:

```javascript
const newComment = {
  __typename: "Comment",
  id: "abc123",
  text: "Great blog post!",
};

cache.modify({
  id: cache.identify(myPost),
  fields: {
    comments(existingCommentRefs = [], { readField }) {
      const newCommentRef = cache.writeFragment({
        data: newComment,
        fragment: gql`
          fragment NewComment on Comment {
            id
            text
          }
        `,
      });

      // 빠르고 안전한 체크 - 새로운 comment가 이미 캐시에 존재하면, 또 넣을 필요는 없다.
      if (
        existingCommentRefs.some(
          (ref) => readField("id", ref) === newComment.id
        )
      ) {
        return existingCommentRefs;
      }

      return [...existingCommentRefs, newCommentRef];
    },
  },
});
```

comments 필드의 modifier 함수가 호출되면, `writeFragment`를 호출해서 newComment 데이터를 캐시에 저장한다. `writeFragment` 함수는 캐시된 댓글을 참조하는 `newCommentRef`를 리턴한다.

안전하게 체크하는 방법으로 기존 comment 참조 배열인 existingCommentRefs을 스캔하여 새 comment가 리스트에 없는지 확인한다.

#### mutation 이후 캐시 업데이트

캐시가 식별이 가능한 `options.data` 객체(`__typename` 및 ID 필드에 기반)와 같이 `writeFragment`를 호출하면 `options.id`를 `writeFragment`에 전달하지 않아도 된다.

`options.id`를 명시적으로 넘기거나, `options.data`를 사용해서 `writeFragment`가 options를 알아내도록 하거나 상관없이 `writeFragment`는 식별된 객체에 대한 참조를 리턴한다.

이런 동작은 `useMutation`의 update 함수를 사용할 때 더욱 쉽게 `writeFragment`로 캐시에 존재하는 객체의 참조값을 얻을 수 있도록 한다:

```javascript
const [addComment] = useMutation(ADD_COMMENT, {
  update(cache, { data: { addComment } }) {
    cache.modify({
      id: cache.identify(myPost),
      fields: {
        comments(existingCommentRefs = [], { readField }) {
          const newCommentRef = cache.writeFragment({
            data: addComment,
            fragment: gql`
              fragment NewComment on Comment {
                id
                text
              }
            `,
          });
          return [...existingCommentRefs, newCommentRef];
        },
      },
    });
  },
});
```

이 예제에서는 `useMutation`으로 자동으로 Comment를 생성하고 캐시에 추가해주지만 Comment가 해당 Post의 comments리스트에 맞게 자동으로 추가하지는 않는다. 이 뜻은 이 Post의 comments 리스트를 보고 있는 모든 쿼리에서는 업데이트 되지않는다.

이 문제를 해결하기 위해서는 useMutation의 update 콜백을 사용해서 `cache.modify`를 호출한다. 그리고 위 예제 전의 예제와 같이 새 comment를 리스트에 추가한다. 이전 예제와는 다르게, comment는 이 useMutation에 의해 캐시에 추가되었다. 그래서 `cache.writeFragment`는 기존 객체에 대한 참조를 리턴한다.

#### 캐시된 객체에서 필드 삭제하기

modifier 함수는 옵셔널하게 두번째 인자에 `canRead`와 `isReference` 함수 같은 몇가지의 유용한 유틸리티를 포함한 객체를 줄 수 있다. 또한 DELETE라는 센티널 객체도 포함되어 있다.

특정 캐시된 객체에서 필드를 삭제하려면, 특정 필드의 modifier 함수에서 DELETE 센티널 객체를 리턴한다:

```javascript
cache.modify({
  id: cache.identify(myPost),
  fields: {
    comments(existingCommentRefs, { DELETE }) {
      return DELETE;
    },
  },
});
```

#### 캐시된 객체에서 필드 무효화

기본적으로 필드 값을 변경하거나 삭제하면 필드도 유효하지 않게 되므로 이전에 필드를 사용한 적이 있는 쿼리가 해당 필드를 다시 읽게된다.

`cache.modify`를 사용하면, 값을 변경하거나 삭제하지 않고 필드를 무효화할 수도 있고,`INVALIDATE` 센티널을 리턴할 수도 있다:

```javascript
cache.modify({
  id: cache.identify(myPost),
  fields: {
    comments(existingCommentRefs, { INVALIDATE }) {
      return INVALIDATE;
    },
  },
});
```

만약 주어진 객체의 모든 필드를 무효화하고 싶으면, modifier 함수에 fields 옵셔을 넘길 수 있다:

```javascript
cache.modify({
  id: cache.identify(myPost),
  fields(fieldValue, details) {
    return details.INVALIDATE;
  },
});
```

`cache.modify`를 사용할 땐, 각각 필드를 `details.fieldName`을 사용해서 정의할 수 있다. 이것은 `INVALIDATE`를 반환하는 함수뿐만 아니라, 모든 modifier 함수에서 작동한다.

## 객체의 캐시ID 얻기

커스텀 캐시ID를 사용하는 캐시의 경우, `cache.identify` 메소드를 통해 캐시ID를 해당 객체의 타입에서 얻어올 수 있다. 이 메소드는 객체를 가져와서 해당 객체의 `__typename`과 식별자 필드를 기반으로 ID를 계산한다.
즉, 각 타입의 캐시ID를 구성하는 필드를 추적할 필요가 없다.

### 예제

다음과 같은 그래프큐엘 객체가 캐시되어있다고 하자:

```javascript
const invisibleManBook = {
  __typename: "Book",
  isbn: "9780679601395", // 이 타입의 캐시ID
  title: "Invisible Man",
  author: {
    __typename: "Author",
    name: "Ralph Ellison",
  },
};
```

이 객체를 `writeFragment` 또는 `cache.modify`와 같은 메소드를 사용하여 캐시에서 상호작용하려면 캐시ID가 필요하다. id 필드가 없기 때문에 Book 타입의 캐시ID는 커스텀되었다.

Book 타입이 캐시ID에 `isbn` 필드를 사용하는지 조회할 필요없이 다음과 같이 `cache.identify` 메서드를 사용할 수 있다:

```javascript
const bookYearFragment = gql`
  fragment BookYear on Book {
    publicationYear
  }
`;

const fragmentResult = cache.writeFragment({
  id: cache.identify(invisibleManBook),
  fragment: bookYearFragment,
  data: {
    publicationYear: "1952",
  },
});
```

캐시는 Book 타입이 isbn 필드를 캐시ID로 사용한다는 것을 안다. 그래서 `cache.identify`를 사용해서 id 필드를 올바르게 채울 수 있다.

이 예시는 캐시ID가 싱글 필드로 isbn을 사용하기 때문에 간단하다. 하지만, 커스텀 캐시ID는 여러 필드로 구성될 수도 있다. 그래서 `cache.identify`를 사용하지 않고 객체의 캐시ID를 지정하는 것이 훨신 어렵고 반복적이다.
