---
layout: post
title: "Apollo Server - Resolvers"
date: 2023-05-31
tags: [Graphql]
comments: true
---

## Resolvers

apollo server의 graphql operation 프로세스

apollo server는 스키마의 모든 field를 데이터 채우는 방법을 알아야 해당 데이터에 대한 요청과 응답을 할 수 있다. 그럴려면 resolvers를 써야한다.

resolver는 스키마에서 단일 field의 데이터를 채우는 역할을 담당하는 함수이다. 백엔드 데이터베이스 또는 타사 API에서 데이터를 가져오는 등 사용자가 정의한 모든 방식으로 데이터를 채울 수 있다.

특정 필드에 대한 resolver를 정의하지 않으면, apollo server에서 자동으로 해당 field에 대한 **default resolver**를 정의한다.

### Defining a resolver

#### Base Syntax

아래와 같은 간단한 스키마가 있다고 가정해보자.

```graphql
type Query {
  numberSix: Int! # Should always return the number 6 when queried
  numberSeven: Int! # Should always return 7
}
```

우리는 루트 쿼리에 있는 항상 6을 리턴해야하는 `numberSix`, 항상 7을 리턴해야하는 `numberSeven` 필드에 대한 리졸버를 정의하여야한다.

```typescript
const resolvers = {
  Query: {
    numberSix() {
      return 6;
    },
    numberSeven() {
      return 7;
    },
  },
};
```

위 예시에서 의미하는 것은

- 단일 Javascript 객체에서 server의 모든 resolver를 정의하여야한다. 이 객체를 `resolver map` 이라고 한다.
- `resolver map`에는 스키마의 타입에 해당하는 최상위 field가 있다. (`Query` 같은..)
- 각 resolver 함수는 각 상응하는 field에 속하는 타입에 속한다.

#### Handling arguments

다음은 server에서 정의한 스키마이다.

```graphql
type User {
  id: ID!
  name: String
}

type Query {
  user(id: ID!): User
}
```

id로 하여금 user를 쿼리하여 가져오려고 한다.

```typescript
const users = [
  {
    id: "1",
    name: "Elizabeth Bennet",
  },
  {
    id: "2",
    name: "Fitzwilliam Darcy",
  },
];

const resolvers = {
  Query: {
    user(parent, args, contextValue, info) {
      return users.find((user) => user.id === args.id);
    },
  },
};
```

해당 resolver를 작성하였다.

위 예제에서 의미하는 것은

- resolver는 4개의 인자를 받아올 수 있다. (parent, args, contextValue, info)
- `args` 인수는 Graphql operation에 의해 field에 제공된 모든 인수를 포함하는 객체이다.

  이 예시에서는 User field에 대한 resolver를 정의하지 않는다. 그 이유는 apollo server가 생성하는 default resolver가 잘 작동하고 있기 떄문이다. User resolver가 반환한 객체에서 직접 값을 갖고 온다.

### Passing resolvers to Apollo Server

아래 예시에서는 최상위 await calls를 해서 비동기적으로 서버를 시작한다.

모든 resolvers를 정의한 후에는 typeDefs와 함께 ApolloServer 생성자에 전달해주어야한다. 모든 파일과 객체를 하나의 resolver map으로 merge해서 ApolloServer 생성자에 전달하기만 하면 된다. 그러면 수많은 다양한 파일 및 객체에 resolver를 정의할 수 있다.

### Resolver Chains

쿼리는 항상 `bottoms out` 형식으로 field를 반환한다. 이 규칙으로 인해 Apollo Server는 객체 타입을 반환하는 field를 확인할 때마다 항상 해당 객체의 하나 이상의 field를 확인한다. 이러한 하위 필드에는 객체 타입도 포함될 수 있다. 스키마에 따라 이 객체 필드 패턴은 임의의 깊이까지 계속되어 `resolver chain`이라고 하는 것을 생성할 수 있다.

#### Example

```
# A library has a branch and books
type Library {
  branch: String!
  books: [Book!]
}

# A book has a title and author
type Book {
  title: String!
  author: Author!
}

# An author has a name
type Author {
  name: String!
}

type Query {
  libraries: [Library]
}
```

이러한 스키마가 있다고 치자.

```
query GetBooksByLibrary {
  libraries {
    books {
      author {
        name
      }
    }
  }
}
```

클라이언트에서는 이런식으로 호출한다고 하면,

<img width="745" alt="스크린샷 2023-06-01 오전 1 14 32" src="https://github.com/leesoo7595/leesoo7595.github.io/assets/39291812/8b444254-ee11-4ec4-a56f-4adbe3166b8b">

이 쿼리에 대한 resolver chain은 query의 계층 구조와 일치한다.

위에 표시된 순서로 resolver가 실행되는데, 각각 `parent`라는 인수를 통해서 다음 resolver chain에 리턴값을 전달한다.

```
query GetBooksByLibrary {
  libraries {
    books {
      title
      author {
        name
      }
    }
  }
}
Then the resolver
```

위 쿼리에서 title이 추가됐을 경우에 resolver chain은 병렬적으로 실행되게 된다.

<img width="761" alt="스크린샷 2023-06-01 오전 1 34 20" src="https://github.com/leesoo7595/leesoo7595.github.io/assets/39291812/94edb9c7-7f0d-4a1d-980f-acb1e9a412af">

### Resolver arguments

resolver 함수는 4개의 인자를 가지고 있다. `parent, args, contextValue, info`

- `parent`: 해당 field의 parent에 대한 resolver의 리턴 값이다. parent가 없는 최상위 필드 resolver의 경우, apolloServer 생성자에 전달된 rootValue 함수에서 가져온다.
- `args`: 해당 field에서 제공된 graphql 인수를 포함하는 객체이다.
- `contextValue`: 특정 작업에 대해 실행중인 모든 resolver에서 공유되는 객체이다. 데이터로더, 인증 등 작업을 공유할 수 있다.
- `info`: field 이름, root에서 field로의 경로 등 작업의 실행 상태에 대한 정보를 가지고 있다.

### The contextValue argument

resolver는 contextValue argument를 인위적으로 수정해서는 안된다. 그럼으로써 모든 resolver에서 일관성을 보장하고 에러를 방지할 수 있다.

resolver는 세번째 인자를 통해 contextValue에 접근할 수 있다. 특정 operation에 대해서 실행중인 모든 resolver는 contextValue에 접근할 수 있다.

```typescript
import { UserAPI } from "./datasources/users";

const resolvers = {
  Query: {
    // Our resolvers can access the fields in contextValue
    // from their third argument
    currentUser: (_, __, contextValue) => {
      return contextValue.dataSources.userApi.findUser(contextValue.token);
    },
  },
};

interface MyContext {
  // Context typing
  token?: String;
  dataSources: {
    userApi: UserAPI;
  };
}

const server = new ApolloServer<MyContext>({
  typeDefs,
  resolvers,
});

const { url } = await startStandaloneServer(server, {
  context: async ({ req }) => ({
    token: getToken(req.headers.authentication),
    dataSources: {
      userApi: new UserAPI(),
    },
  }),
});
```

### Return values

resolver 함수의 리턴값은 타입에 따라 apollo server에서 다르게 처리된다.

- `Scalar, object`: resolver는 단일 value나 객체를 반환할 수 있는데, 이 리턴값은 parent 인자를 통해서 중첩된 모든 resolver에 전달된다.
- `Array`: 스키마에서 리졸버의 연결된 field에 list가 포함되어있다고 표시하는 경우에만 배열을 반환한다.
- `null / undefined`
- `Promise`

### Default resolvers

특정 스키마 field에 대한 resolver를 정의해주지 않으면, apollo server는 default resolver를 정의한다.

Default resolver는 아래의 로직을 따른다.

<img width="660" alt="스크린샷 2023-06-01 오전 2 32 49" src="https://github.com/leesoo7595/leesoo7595.github.io/assets/39291812/39f70ec2-ccac-490d-801f-2dfdc3640899">

```
type Book {
  title: String
}

type Author {
  books: [Book]
}
```

만약 books field는 Book이라는 객체의 배열을 리턴하는데, 단순히 title field는 default resolver를 통해서 사용할 수 있다. default resolver는 부모 인자에 title이란 정확한 이름을 가진 프로퍼티가 있으므로 parent.title을 올바르게 리턴해준다.

### Resolving unions and interfaces

하나 또는 다수의 객체 타입이 가능한 field를 정의할 수 있는 graphql types가 있다. 이 field는 다른 객체 타입을 resolve할 수 있기 때문에 `__resolveType` 함수를 무조건 선언해줌으로써 apollo server에 리턴되는 객체 타입을 알려주어야한다.

### Monitoring resolver performance

resolver의 퍼포먼스는 로직에 의존한다. 스키마의 어떤 field가 계산 비용이 많이 들거나, resolve 속도가 느린지 이해해서 필요할 때만 쿼리할 수 있도록 하는 것이 중요하다. 이를 apollo studio가 도와준다.

## Reference

- [Apollo Server - resolvers](https://www.apollographql.com/docs/apollo-server/data/resolvers)
