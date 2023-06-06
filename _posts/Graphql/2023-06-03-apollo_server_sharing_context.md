---
layout: post
title: "Apollo Server - Context and contextValue"
date: 2023-06-03
tags: [Graphql]
comments: true
---

## Context and contextValue

operation을 진행하는 도중, contextValue라는 객체를 만들어서 서버에서 resolvers와 plugins 전반적으로 데이터를 공유할 수 있다.

contextValue를 통해서 해당하는 필요한 resolver에 유용하게 전달할 수 있다. 예를 들면 authentication 스코프 단위나, 데이터를 가져오는 소스라던지, 데이터베이스 커넥션, 그리고 커스텀 fetch 함수 등등을 할 수 있다. 너가 batch request를 하기 위해 dataloaders를 사용한다면, contextValue를 통해서 공유할 수 있다.

### context function

`apollo server 4`에서 context function 정의하는 형태가 변경되었다.

context function은 비동기여야하고 객체를 리턴해야한다. 이 객체는 서버의 resolvers와 plugins에 contextValue라는 이름으로 접근할 수 있다.

선택적으로 `expressMiddleware`나 `startStandaloneServer` 같은 함수를 context 함수로 넘길 수도 있다.

서버는 모든 request에 한번씩은 context 함수를 호출해서 contextValue를 커스터마이징해서 http 헤더같은 요청 세부사항을 커스텀할 수 있게 해준다.

```typescript
import { GraphQLError } from "graphql";

const resolvers = {
  Query: {
    // Example resolver
    adminExample: (parent, args, contextValue, info) => {
      if (contextValue.authScope !== ADMIN) {
        throw new GraphQLError("not admin!", {
          extensions: { code: "UNAUTHENTICATED" },
        });
      }
    },
  },
};

interface MyContext {
  // You can optionally create a TS interface to set up types
  // for your contextValue
  authScope?: String;
}

const server = new ApolloServer<MyContext>({
  typeDefs,
  resolvers,
});

const { url } = await startStandaloneServer(server, {
  // Your async context function should async and
  // return an object
  context: async ({ req, res }) => ({
    authScope: getScope(req.headers.authorization),
  }),
});
```

타입스크립트를 사용한다면 `context` 함수에 전달할 파라미터 타입을 `ApolloServer`에 전달해주어야한다.

`context` 초기화 함수가 비동기이기 때문에, 데이터베이스 연결설정이나 다른 작업이 완료될 때까지 기다리는게 가능하다.

```typescript
context: async () => ({
  db: await client.connect(),
})

// Resolver
(parent, args, contextValue, info) => {
  return contextValue.db.query('SELECT * FROM table_name');
}
```

### Throwing errors

기본적으로 context 함수는 에러를 던질 때, apollo server는 JSON응답으로 500 HTTP 코드를 반환한다. 만약 에러가 GraphQLError가 아니면, 에러 메세지 앞에 `Context creation failed: `가 붙게된다.

http 기반 GraphQLError의 HTTP 상태코드를 바꿀 수도 있다.

```typescript
context: async ({ req }) => {
  const user = await getUserFromReq(req);
  if (!user) {
    throw new GraphQLError('User is not authenticated', {
      extensions: {
        code: 'UNAUTHENTICATED',
        http: { status: 401 },
      }
    });
  }

  // If the below throws a non-GraphQLError, the server returns
  // `code: "INTERNAL_SERVER_ERROR"` with an HTTP status code 500, and
  // a message starting with "Context creation failed: ".
  const db = await getDatabaseConnection();

  return { user, db };
},
```

## The contextValue object

context 함수는 `contextValue`라는 객체를 리턴한다. 그 객체를 통해서 plugins와 resolvers에 접근할 수 있다.

### Resolvers

리졸버는 `contextValue` 인자를 수정해서는 안된다. 이 객체는 모든 resolvers에서 일관성이 있어야하고, 예상치못한 에러를 막을 수 있도록 보장되게 한다.

resolvers는 세번째 인자를 통해 `contextValue` 객체를 공유한다.모든 리졸버는 contextValue에 접근함으로써 특정한 operation을 실행시킬 수 있다.
