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

`readQuery` 메소드는 다음과 같이 캐시에서 직접 그랲큐엘 쿼리를 실행할 수 있다:

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
