---
layout: post
title: "Apollo Server - Error handling"
date: 2023-06-06
tags: [Graphql]
comments: true
---

## Error handling

클라이언트와 서버에서 오류 실행가능하도록하기

Apollo Server v4에서는 유효하지 않은 변수를 주면 400이 아닌 200 코드가 리턴되는 기능이 소개되었다. 이 기능을 다시 400으로 주도록 하려면 `status400ForVariableCoercionErrors: true` 옵션을 주도록하면 됨

Apollo Server는 GraphQl operation 동안 에러를 마주했을때, 클라이언트 응답으로 발생한 오류가 포함된 배열을 가지고 있다. 각 배열 안의 에러는 에러 code와 stacktrace를 포함하는 유용한 정보를 제공해주는 `extensions` 필드를 가지고 있다.

Apollo Server는 디버깅을 더 편하게 하기 위해 `ApolloServerErrorCode` enum을 제공한다. 그걸 사용해서 발생한 에러가 Apollo Server에서 제공해주는 다양한 타입들 중에 있는지 체크, 사용할 수 있다.

error code를 체크해서 왜 에러가 발생했는지 체크할 수 있고, 또한 다른 타입의 에러에서 로직을 추가할 수도 있다.

```typescript
import { ApolloServerErrorCode } from "@apollo/server/errors";

if (error.extensions?.code === ApolloServerErrorCode.GRAPHQL_PARSE_FAILED) {
  // respond to the syntax error
} else if (error.extensions?.code === "MY_CUSTOM_CODE") {
  // do something else
}
```

클라이언트가 다양한 오류 타입들에게 다르게 응답할 수 있도록 할 수 있다. 그리고 커스텀 에러와 코드를 만들 수도 있다.

### Built-in error codes

- `GRAPHQL_PARSE_FAILED`

### Custom errors

graphql 패키지의 `GraphQLError` 클래스를 사용해서 커스텀 에러와 코드를 만들 수 있다.

```typescript
import { GraphQLError } from "graphql";

throw new GraphQLError(message, {
  extensions: { code: "YOUR_ERROR_CODE", myCustomExtensions },
});
```

커스텀 에러는 왜 에러가 발생했는지 클라이언트가 이해할 수 있도록 추가적인 context를 제공할 수 있다. 우리는 일반적인 케이스에서의 에러를 만들도록 추천한다. 예를 들면, user가 로그인하지 않았을때, 또는 누군가가 특정 액션을 수행하는 것이 금지될때와 같은 경우의 일반적인 케이스를 의미한다.

```typescript
import { GraphQLError } from "graphql";

throw new GraphQLError("You are not authorized to perform this action.", {
  extensions: {
    code: "FORBIDDEN",
  },
});
```

### Throwing errors

apollo server는 경우에 해당할 때 에러를 자동적으로 던진다. apollo server에서 자동으로 하지 않을 때, resolvers에서도 오류를 발생시키도록 할 수도 있다.

```typescript
const resolvers = {
  Query: {
    userWithID: (_, args) => {
      if (args.id < 1) {
        throw new GraphQLError("Invalid argument value", {
          extensions: {
            code: "BAD_USER_INPUT",
          },
        });
      }
      // ...fetch correct user...
    },
  },
};
```

resolver에서 GraphQLError 인스턴스가 아닌 일반 에러를 던지면 그 에러는 stacktrace와 code를 포함한 extensions 필드를 던져 준다. (특히 `INTERNAL_SERVER_ERROR`)

### Including custom error details

`GraphqlError`를 발생시킬 때, 에러의 extensions 객체에 임의 필트를 추가하여 클라이언트에게 추가적인 context를 줄 수 있다. 에러 constructor에서 해당 필드를 정의할 수 있다.

```typescript
const resolvers = {
  Query: {
    userWithID: (_, args) => {
      if (args.id < 1) {
        throw new GraphQLError("Invalid argument value", {
          extensions: {
            code: "BAD_USER_INPUT",
            argumentName: "id",
          },
        });
      }
      // ...fetch correct user...
    },
  },
};
```

```json
{
  "errors": [
    {
      "message": "Invalid argument value",
      "locations": [
        {
          "line": 2,
          "column": 3
        }
      ],
      "path": ["userWithID"],
      "extensions": {
        "code": "BAD_USER_INPUT",
        "argumentName": "id",
        "stacktrace": [
          "GraphQLError: Invalid argument value",
          "    at userWithID (/my-project/index.js:25:13)",
          "    ...more lines..."
        ]
      }
    }
  ]
}
```

### Omitting or including stacktrace

`stacktrace` 에러필드는 개발하거나 디버깅할때 유용하지만, 프로덕션에서는 노출되면 안될 것이다.

기본적으로 Apollo Server는 `NODE_ENV` 환경에서 `production`, `test`라는 값은 `stacktrace`가 생략된다.

이 기본값을 `ApolloServer` 생성자의 `includeStacktraceInErrorResponses` 옵션을 통해서 오버라이딩할 수 있다. 이게 true이면 stacktrace는 항상 포함된다. false일 경우 항상 생략된다.

stacktrace가 생략되는 경우, 어플리케이션에서도 사용할 수 없다. error stacktrace를 클라이언트에 포함하지 않고 로깅하려면...

### Masking and logging errors

클라리언트에게 패스 또는 apollo studio에 리포트하기전에 apollo server 에러 세부정보를 수정할 수 있다. 이를 통해 민감정보를 생략할 수 있도록 해준다.

#### For client responses

`formatError` hook 함수가 클라이언트에 전달되기 전에 각 에러에서 실행되는데, 이 함수를 사용해서 특정 에러를 로그하거나 마스킹할 수 있다.

`formatError` hook은 두 인자를 받는다. 하나는 JSON 객체로 포맷된 에러, 그리고 다른 하나는 GraphQLError로 래핑된 표준 에러이다.

## Refernce

- [Apollo Server - Errors](https://www.apollographql.com/docs/apollo-server/data/errors)
