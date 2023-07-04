---
layout: post
title: "Apollo Server - Generating types from a GraphQL schema"
date: 2023-06-29
tags: [Graphql]
comments: true
---

이 글은 [아폴로 공식문서](https://www.apollographql.com/docs/apollo-server/workflow/generate-types)를 번역한 글 입니다.

# Generating types from a GraphQL schema

Graphql은 타입 시스템을 사용해서 Graphql 스키마에서 각 타입과 필드에 사용 가능한 데이터를 명확하게 정의한다. 타입 생성 라이브러리는 강력하게 타입화된 Graphql 스키마의 특성을 활용해서 해당 스키마를 기반으로 타입스크립트 타입을 자동으로 생성할 수 있다.

이렇게 생성된 TS 타입을 리졸버에서 사용하여 리졸버의 리턴 값이 스키마에서 지정한 필드 타입과 일치하는지 타입 검사를 수행할 수 있다. 리졸버의 타입 검사를 통해 에러를 빠르게 발견할 수 있고, 타입 안전이 보장되어 안심할 수 있다.

## Setting up your project

우리는 Graphql Code Generator 라이브러리를 사용해서 Graphql 스키마를 기반으로 타입을 생성할 것이다. Graphql Code Generator에 스키마를 제공하는 방법은 여러가지가 있다. 아래에서는 스키마가 .graphql 파일에 있는 가장 일반적인 방법이다.

```graphql
type Query {
  books: [Book]
}

type Book {
  title: String
  author: String
}

type AddBookMutationResponse {
  code: String!
  success: Boolean!
  message: String!
  book: Book
}

type Mutation {
  addBook(title: String, author: String): AddBookMutationResponse
}
```

스키마를 .graphql 파일로 옮긴 경우 가져오기를 업데이트하여 스키마가 서버에 제대로 전달되는지 확인. 서버를 생성하는 파일에서 fs 패키지의 readFileSync를 사용하여 스키마를 읽어올 수 있다:

```typescript
// ...other imports
import { readFileSync } from "fs";

// Note: this uses a path relative to the project's
// root directory, which is the current working directory
// if the server is executed using `npm run`.
const typeDefs = readFileSync("./schema.graphql", { encoding: "utf-8" });

const server = new ApolloServer<MyContext>({
  typeDefs,
  resolvers,
});

// ... start our server
```

서버를 재시작해서 확인하고, 스키마를 기반으로 타입을 자동으로 생성하는 데 필요한 패키지를 설치한다.

## Installing and configuring dependencies

```bash
npm install -D @graphql-codegen/cli @graphql-codegen/typescript @graphql-codegen/typescript-resolvers
```

설치 후, GraphQL Code Generator에 타입을 생성할 위치와 방법을 알려주는 configuring file을 설정한다. 이 작업은 수동으로 codegen.yml 파일을 생성하거나, 다음 명령어를 사용하여 진행할 수 있다.

```yaml
# This configuration file tells GraphQL Code Generator how
# to generate types based on our schema.
schema: "./schema.graphql"
generates:
  # Specify where our generated types should live.
  ./src/__generated__/resolvers-types.ts:
    plugins:
      - "typescript"
      - "typescript-resolvers"
    config:
      useIndexSignature: true
      # More on this below!
      contextType: "../index#MyContext"
```

위는 codegen.yml 파일의 예시이다.

마지막으로 TS 타입이 정기적으로 생성되도록 package.json 파일에 유용한 스크립트를 추가하는 것이 좋다.

```json
{
  // ...
  "scripts": {
    "generate": "graphql-codegen --config codegen.yml",
    "compile": "npm run generate && tsc",
    "start": "npm run compile && node ./dist/index.js"
  }
  // ...
}
```

코드를 watch하는 스크립트 추가해서 타입을 재생성하고 작업시 백그라운드에서 타입스크립트 파일을 다시 컴파일 할 수 있도록 구성하는 것이 좋다.

위에서 npm start 명령을 실행하면 GraphQL 스키마에 따라 타입이 생성되고 TypeScript 코드가 컴파일된다. graphql-codegen 명령을 처음 실행하면 codegen.yml 파일에 지정한 경로에 생성된 타입으로 가득 찬 파일이 표시된다.

## Adding types to resolvers

`typescript-resolvers` 플러그인은 리졸버 맵에 타입을 추가하는 데 사용할 수 있는 리졸버 타입을 생성하여 리졸버가 스키마에 지정된 필드 타입과 일치하는 값을 리턴하도록 한다.

리졸버 타입을 리졸버를 정의하는 파일에 임포트한다.

```typescript
// This is the file where our generated types live
// (specified in our `codegen.yml` file)
import { Resolvers } from "./__generated__/resolvers-types";
```

이제 리졸버 맵에 리졸버 타입을 직접 추가할 수 있다.

```typescript
export const resolvers: Resolvers = {};
```

이제 리졸버는 각 리졸버의 인수와 리턴 값이 스키마와 일치하는지 타입 검사를 할 수 있다.

```typescript
export const resolvers: Resolvers = {
  Query: {
    // TypeScript now complains about the below resolver because
    // the data returned by this resolver doesn't match the schema type
    // (i.e., type Query { books: [Book] })
    books: () => {
      return "apple";
    },
  },
};
```

리졸버가 여러 파일에 있는 경우, 리졸버에 대해 생성된 해당 타입을 해당 파일로 가져올 수 있다.

```typescript
// queries.ts
import { QueryResolvers } from "__generated__/resolvers-types";

// Use the generated `QueryResolvers`
// type to type check our queries!
const queries: QueryResolvers = {
  Query: {
    // ...queries
  },
};

export default queries;

// mutations.ts
import { MutationResolvers } from "__generated__/resolvers-types";

// Use the generated `MutationResolvers` type
// to type check our mutations!
const mutations: MutationResolvers = {
  Mutation: {
    // ...mutations
  },
};

export default mutations;
```

## Context typing for resolvers

Graphql code generator를 컨피규링해서 리졸버가 공유하는 컨텍스트에 대한 타입을 추가하여 존재하지 않는 값을 사용할 때, 타입스크립트에서 경고 표시를 하도록 할 수 있다.

이렇게 하려면 먼저 컨텍스트 값을 입력하기 위한 일반 유형 매개변수로 전달한 인터페이스를 Apollo Server로 내보내야 합니다:

```typescript
export interface MyContext {
  dataSources: {
    books: Book[];
  };
}

const server = new ApolloServer<MyContext>({
  typeDefs,
  resolvers,
});
```

내보낸 컨텍스트 인터페이스를 다음과 같이 contextType 구성 옵션에 전달할 수 있다:

```yaml
# ...
config:
  useIndexSignature: true
  # Providing our context's interface ensures our
  # context's type is set for all of our resolvers.

  # Note, this file path starts from the location of the
  # file where you generate types.
  # (i.e., `/src/__generated__/resolvers-types.ts` above)
  contextType: "../index#MyContext"
```

타입을 다시 생성하면 이제 컨텍스트 값이 모든 리졸버에 자동으로 입력된다.

```typescript
const resolvers: Resolvers = {
  Query: {
    // Our third argument (`contextValue`) has a type here, so we
    // can check the properties within our resolver's shared context value.
    books: (_, __, contextValue) => {
      return contextValue.dataSources.books;
    },
  },
};
```

## Reference

- [Apollo Server - Generating types from a Graphql schema](https://www.apollographql.com/docs/apollo-server/workflow/generate-types)
