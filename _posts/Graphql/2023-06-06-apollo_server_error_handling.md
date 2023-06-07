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

```typescript
const server = new ApolloServer({
  typeDefs,
  resolvers,
  formatError: (formattedError, error) => {
    // Return a different error message
    if (
      formattedError.extensions.code ===
      ApolloServerErrorCode.GRAPHQL_VALIDATION_FAILED
    ) {
      return {
        ...formattedError,
        message: "Your query doesn't match the schema. Try double-checking it!",
      };
    }

    // Otherwise return the formatted error. This error can also
    // be manipulated in other ways, as long as it's returned.
    return formattedError;
  },
});
```

만약 일반 에러 인스턴스에 접근하고 싶다면 formatError의 두 번쨰 인자에 접근하면 된다.

```typescript
formatError: (formattedError, error) => {
    if (error instanceof CustomDBError) {
      // do something specific
    }
  },
```

resolver에서 에러를 던질때, `GraphQLError`로 처음에 던진 에러를 래핑한다. 이 `GraphQLError`는 깔끔하게 error를 포맷팅하고, `path`와 같은 에러가 일어난 곳을 알려주는 유용한 필드를 포함시켜준다.

만약 `GraphQLError`로 감싸지지 않게 하고 기존에 던진 에러에 접근하려면 `unwrapResolverError`를 사용하면 된다. 이 함수는 resolver 에러 또는 그 외의 에러의 `GraphQLError` 래핑을 바꿀 수 있도록 해준다.

```typescript
new ApolloServer({
  formatError: (formattedError, error) => {
    // unwrapResolverError removes the outer GraphQLError wrapping from
    // errors thrown in resolvers, enabling us to check the instance of
    // the original error
    if (unwrapResolverError(error) instanceof CustomDBError) {
      return { message: "Internal server error" };
    }
  },
});
```

`formatError`에서 반은 에러를 특정한 context에 맞게 조정하려면, `didEncounterErrors` 라이프사이클 이벤트를 사용하여 에러에 추가 프로펕티를 플러그인을 만들어서 조정할 수 있다.

#### For Apollo Studio reporting

Apollo Server 4의 새로운 기능: 오류 세부 정보는 기본적으로 추적에 포함되지 않습니다. 대신, 각 오류의 메시지는 <masked>로 대체되고, maskedBy 오류 확장은 다른 모든 확장을 대체합니다. maskedBy 확장자에는 마스킹을 수행한 플러그인의 이름(ApolloServerPluginUsageReporting 또는 ApolloServerPluginInlineTrace)이 포함됩니다.

서버의 에러 비율을 분석하기 위해 Apollo Studio르르 사용할 수 있다. 기본적으로 operations은 studio에 traces를 보내지만, error details는 포함하지 않는다. 만약 에러정보를 보내고 싶다면, 모든 에러를 보낼 수도 있고, 특정 에러가 전송되기 전에 수정할 수도 있다.

에러를 보내기 위해서는 아래처럼 할 수 있다.

```typescript
new ApolloServer({
  // etc.
  plugins: [
    ApolloServerPluginUsageReporting({
      // If you pass unmodified: true to the usage reporting
      // plugin, Apollo Studio receives ALL error details
      sendErrors: { unmodified: true },
    }),
  ],
});
```

`sendErrors.transform` 옵션에 함수를 전달하게되면, 특정 에러를 기록하거나 수정할 수 있다.

비밀번호가 틀려서 unauthenticated 에러가 났을 때, 에당 에러를 apollo studio에서 해당 에러를 피할 수 있도록 transform 함수를 정의할 수 있다.

```typescript
import { ApolloServer } from "@apollo/server";
import { ApolloServerPluginUsageReporting } from "@apollo/server/plugin/usageReporting";
const server = new ApolloServer({
  typeDefs,
  resolvers,
  plugins: [
    ApolloServerPluginUsageReporting({
      sendErrors: {
        transform: (err) => {
          // Return `null` to avoid reporting `UNAUTHENTICATED` errors
          if (err.extensions.code === "UNAUTHENTICATED") {
            return null;
          }

          // All other errors will be reported.
          return err;
        },
      },
    }),
  ],
});
```

#### redacting information from an error message

만약 개인 식별 정도가 에러 메세지에 있으면, transform 함수로 해당 정보를 apollo studio에 보내지 않도록 할 수 있다.

```typescript
import { GraphQLError } from "graphql";

throw new GraphQLError(
  "The x-api-key:12345 doesn't have sufficient privileges."
);
```

```typescript
import { ApolloServer } from "@apollo/server";
import { ApolloServerPluginUsageReporting } from "@apollo/server/plugin/usageReporting";

const server = new ApolloServer({
  typeDefs,
  resolvers,
  plugins: [
    ApolloServerPluginUsageReporting({
      sendErrors: {
        transform: (err) => {
          // Make sure that a specific pattern is removed from all error messages.
          err.message = err.message.replace(/x-api-key:[A-Z0-9-]+/, "REDACTED");
          return err;
        },
      },
    }),
  ],
});
```

위의 경우, apollo studio에서 에러는 아래와 같이 리포팅된다.

```
The REDACTED doesn't have sufficient privileges.
```

### Setting HTTP status code and headers

GraphQL은 HTTP 동사 및 상태 코드를 통해 통신할 때 REST와 동일한 규칙을 사용하지 않는다. 에러 일관성을 위해 포함된 오류 코드, 또는 커스텀 에러를 사용하는 것이 좋다.

아폴로 서버는 다양한 상황에서 HTTP 상태코드와 다르게 사용된다.

- 아폴로 서버는 올바르게 시작되지않았거나 종료되지 않았다면, 500을 준다.
- 스키마 관련해서 유효성을 검사할 수 없는 경우 400으로 응답한다.
- 잘못된 HTTP 메서드로 요청했을 때는 405로 응답한다.
- context 함수에서 에러가 발생하는 경우 500
- 요청 처리하는 동안 에러 발생시 500
- 그 외는 200을 준다. 기본적으로 서버가 operation을 수행할 수 있고, 실행이 완료되는 경우이다. (리졸버 관련 에러가 포함될 수 있다.)

HTTP 상태코드 또는 응답 header를 커스텀하는 3가지의 방법이 있다. resolver, context 함수, 또는 Plugin을 사용하는 방법이다.

operation가 실행될 때는 200을 전송하는 것을 권장한다. 그래서 리졸버를 통해 이 부분을 커스텀하는 것보다는 context 함수나 요청하기 전에 후킹되는 plugin을 사용하는 것이 좋다.

리졸버에서 수정하는 방법은 아래와 같음

```typescript
import { GraphQLError } from "graphql";

const resolvers = {
  Query: {
    someField() {
      throw new GraphQLError("the error message", {
        extensions: {
          code: "SOMETHING_BAD_HAPPENED",
          http: {
            status: 404,
            headers: new Map([
              ["some-header", "it was bad"],
              ["another-header", "seriously"],
            ]),
          },
        },
      });
    },
  },
};
```

아래는 plugin을 사용하여 수정

```typescript
const setHttpPlugin = {
  async requestDidStart() {
    return {
      async willSendResponse({ response }) {
        response.http.headers.set("custom-header", "hello");
        if (
          response.body.kind === "single" &&
          response.body.singleResult.errors?.[0]?.extensions?.code === "TEAPOT"
        ) {
          response.http.status = 418;
        }
      },
    };
  },
};

const server = new ApolloServer({
  typeDefs,
  resolvers,
  plugins: [setHttpPlugin],
});
```

## Refernce

- [Apollo Server - Errors](https://www.apollographql.com/docs/apollo-server/data/errors)
