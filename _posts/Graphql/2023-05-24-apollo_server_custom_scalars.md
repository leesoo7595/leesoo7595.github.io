---
layout: post
title: "Apollo Server - Custom scalars"
date: 2023-05-24
tags: [Graphql]
comments: true
---

이 글은 Apollo docs를 읽고 번역한 글 입니다.

## Custom scalars

graphql에서 scalar 타입으로는 `Int`, `Float`, `String`, `Boolean`, `ID`를 표준으로 가지고 있다. 이 scalars로 대부분의 사용 케이스에 쓰임에도 불구하고, 어떤 환경에서는 다른 atomic 데이터타입을 필요로 한다. (`Date` 같은, 아니면 존재하는 타입을 validation을 할 수 있는)

이것이 가능하려면, custom scalar type을 정의해야한다.

### Defining a custom scalar

scalar를 커스텀 정의하려면, schema에 아래와 같이 추가해주어야함

```
scalar MyCustomScalar
```

이렇게하면 MyCustomScalar 타입을 스키마 내에서 기본 scalar 타입으로써 사용할 수 있다.

하지만 Apollo Server에서는 새로운 scalar type으로 인터렉트하는 방법을 알려주어야한다.

### Defining custom scalar logic

scalar 타입 커스텀을 정의한 후에는, Apollo Server에서 이 타입을 주고받을 수 있도록 추가 정의가 필요하다.

- 백엔드에서 scalar의 값이 표시되는 방식(백엔드 데이터베이스에 저장되는 표현)
- 이 값이 백엔드에서 JSON 호환 타입으로 직렬화하는 방법
- JSON 호환된 타입이 백엔드 타입으로 역직렬화되는 방법

위 작업은 `GraphQLScalarType` 클래스의 인스턴스에서 정의한다.

### Example: The `Date` scalar

`GraphQLScalarType` 객체에서 Date를 나타내는 custom scalar를 정의한 예시이다.

```typescript
import { GraphQLScalarType, Kind } from "graphql";

const dateScalar = new GraphQLScalarType({
  name: "Date",
  description: "Date custom scalar type",
  serialize(value) {
    if (value instanceof Date) {
      return value.getTime(); // Convert outgoing Date to integer for JSON
    }
    throw Error("GraphQL Date Scalar serializer expected a `Date` object");
  },
  parseValue(value) {
    if (typeof value === "number") {
      return new Date(value); // Convert incoming integer to Date
    }
    throw new Error("GraphQL Date Scalar parser expected a `number`");
  },
  parseLiteral(ast) {
    if (ast.kind === Kind.INT) {
      // Convert hard-coded AST string to integer and then to Date
      return new Date(parseInt(ast.value, 10));
    }
    // Invalid hard-coded value (not an integer)
    return null;
  },
});
```

위 이니셜라이징 정의는 아래 메소드를 가지고 있다.

- serialize
- parseValue
- parseLiteral

위 방식을 사용하면 Apollo Server가 scalar와 상호작용이 가능하다.

#### serialize

`serialize` 메소드는 scalar의 백엔드 형식을 JSON 호환 포맷으로 컨버팅해서 Apollo Server의 operation response에 포함될 수 있도록한다.

위의 예에서 Date 스칼라는 백엔드에서 Date JavaScript 객체로 표시된다. GraphQL 응답으로 Date 스칼라를 보낼때, JavaScript Date 객체의 getTime 함수가 반환하는 정수 값으로 직렬화한다.

#### parseValue

스칼라의 JSON 값을 리졸버의 인수로 추가하기전에 백엔드 형식으로 변환하도록 해준다.

아폴로 서버는 클라이언트가 인수의 graphql 변수로 스칼라를 제공할 때, 이 메서드를 호출한다. 스칼라가 하드코딩된 인수로 넘어올경우에는 parseLiteral 대신 호출된다.

#### parseLiteral

하드코딩된 인자의 값으로 해당 스칼라가 들어오게되면, 그 value는 쿼리 도큐먼트의 ast의 일부이다. apollo Server는 그런 경우에 `parseLiteral` 메소드를 통해서 해당 값의 ast 표현식을 백엔드 표현 형태로 컨버팅한다.

예시를 보면, `parseLiteral`는 AST 값을 string에서 integer로 바꾸고, integer를 Date 타입으로 `parseValue`의 결과와 매치해서 바꾸게된다.

### custom scalars를 apollo server에 제공하기

`GraphQLScalarType` 인스턴스를 정의한 이후에는, resolver map에 포함시켜서 스키마의 다른 타입과 필드들의 리졸버에 포함시키도록 한다.

### 또 다른 예시: integers를 odd values로 제한시키기

`Odd`라는 커스텀 스칼라를 만들어보자

```typescript
// Basic schema
const typeDefs = `#graphql
  scalar Odd

  type Query {
    # Echoes the provided odd integer
    echoOdd(odd: Odd!): Odd!
  }
`;

// Validation function for checking "oddness"
function oddValue(value: number) {
  if (typeof value === "number" && Number.isInteger(value) && value % 2 !== 0) {
    return value;
  }
  throw new GraphQLError("Provided value is not an odd integer", {
    extensions: { code: "BAD_USER_INPUT" },
  });
}

const resolvers = {
  Odd: new GraphQLScalarType({
    name: "Odd",
    description: "Odd custom scalar type",
    parseValue: oddValue,
    serialize: oddValue,
    parseLiteral(ast) {
      if (ast.kind === Kind.INT) {
        return oddValue(parseInt(ast.value, 10));
      }
      throw new GraphQLError("Provided value is not an odd integer", {
        extensions: { code: "BAD_USER_INPUT" },
      });
    },
  }),
  Query: {
    echoOdd(_, { odd }) {
      return odd;
    },
  },
};

const server = new ApolloServer({
  typeDefs,
  resolvers,
});
```

### 서드파티 커스텀 스칼라 임포트하기

다른 라이브러리에서 정의한 커스텀 스칼라라면, 임포트해서 사용할 수 있다.

예시: `graphql-type-json` 패키지 같은 경우 `GraphQLScalarType` 인스턴스로 되어있는 `GraphQLJSON` 라는 객체가 있다. 이 객체를 통해 JSON 스칼라를 사용할 수 있다.

```typescript
import { ApolloServer } from "@apollo/server";
import { startStandaloneServer } from "@apollo/server/standalone";
import GraphQLJSON from "graphql-type-json";

const typeDefs = `#graphql
  scalar JSON

  type MyObject {
    myField: JSON
  }

  type Query {
    objects: [MyObject]
  }
`;

const resolvers = {
  JSON: GraphQLJSON,
  // ...other resolvers...
};

const server = new ApolloServer({
  typeDefs,
  resolvers,
});

const { url } = await startStandaloneServer(server);

console.log(`🚀 Server listening at: ${url}`);
```

## Reference

- [Apollo Server - schema : custom-scalars](https://www.apollographql.com/docs/apollo-server/schema/custom-scalars)
