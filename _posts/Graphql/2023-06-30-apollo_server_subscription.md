---
layout: post
title: "Apollo Server - Subscription"
date: 2023-06-30
tags: [Graphql]
comments: true
---

이 글은 [아폴로 공식문서](https://www.apollographql.com/docs/apollo-server/data/subscriptions)를 번역한 글 입니다.

# Subscriptions in Apollo Server

아폴로서버는 Subscription을 기본적으로 지원하지 않는다. Subscription를 사용하려면 아래처럼 하면 된다.

아폴로 페더레이션에서는 제공하지 않는다.

`graphql-ws` 라이브러리를 사용하여 Apollo Server 4에서 Subscription을 지원한다. `subscriptions-transport-ws` 라이브러리를 사용하는 것은 권장하지 않는다. 이유는 이 라이브러리는 활발하게 관리되고 있지 않기 때문이다.

Subscription은 서버사이드 이벤트가 발생할 때마다 결과를 업데이트할 수 있도록 하는 long-lasting GraphQL read operations이다. 일반적으로 업데이트된 결과는 서버에서 subscriptions(이하 구독) 중인 클라이언트로 푸시된다. 예를 들면, 채팅 어플리케이션의 서버는 구독을 사용하여 특정 대화방의 모든 클라이언트에게 새로 받은 메세지를 푸시할 수 있다.

구독 업데이트는 일반적으로 서버에서 푸시하기 때문에(클라이언트가 폴링하는 대신) HTTP 대신 WebSocket 프로토콜을 사용한다.

쿼리, 뮤테이션과 비교해서 구독은 구현하기 훨신 복잡하다. 구현하기 전에 구독이 정말로 필요한 경우인지 체크하기

## Schema definition

스키마에서 상위필드에 클라이언트가 구독할 수 있도록 Subscription type을 정의한다.

```graphql
type Subscription {
  postCreated: Post
}
```

`postCreated` 필드는 백엔드에서 `Post`를 생성했을때 업데이트될 것이고, 구독중인 클라이언트에게 `Post`를 밀어줄 것 이다.

클라이언트는 `postCreated` 필드를 Graphql 문자열로 구독할 수 있다.

```graphql
subscription PostFeed {
  postCreated {
    author
    comment
  }
}
```

각 subscription operation은 Subscription 타입인 하나의 필드로만 구독이 가능하다.

## Enabling subscriptions

구독은 Apollo Server 4의 `startStandaloneServer` 함수로 제공되지 않는다. 구독 기능을 사용하기 위해서는 `expressMiddleware` 함수를 사용하도록 전환하여야한다.(또는 구독을 지원하는 또 다른 아폴로서버 통합 패키지를 받는다.)

Express 앱과 분리된 구독을 사용하기 위한 웹소켓 서버를 모두 실행하려면, 이 두 가지를 효율적으로 래핑하고 새로운 listener가 되는 http.Server 인스턴스를 생성해야한다.

```bash
npm install graphql-ws ws @graphql-tools/schema
```

ApolloServer 인스턴스를 초기화한 파일에 임포트한다.

```typescript
import { createServer } from "http";
import { ApolloServerPluginDrainHttpServer } from "@apollo/server/plugin/drainHttpServer";
import { makeExecutableSchema } from "@graphql-tools/schema";
import { WebSocketServer } from "ws";
import { useServer } from "graphql-ws/lib/use/ws";
```

HTTP와 구독 서버 둘다 세팅하기 위해서 우리는 http.Server를 생성할 것이다. Express앱을 http 모듈에서 임포트한 `createServer` 함수에 패싱시켜주어야한다.

```typescript
// This `app` is the returned value from `express()`.
const httpServer = createServer(app);
```

`GraphQLSchema` 인스턴스를 생성한다.

구독 서버는 typeDefs 와 resolvers 옵션을 가지고 있지 않다. 대신 실행가능한 GraphQLSchema를 받는다. 이 스키마 객체를 구독 서버와 ApolloServer 모두에 전달할 수 있다.

```graphql
const schema = makeExecutableSchema({ typeDefs, resolvers });
// ...
const server = new ApolloServer({
  schema,
});
```

그 다음, 구독서버를 사용하기 위한 웹소켓서버를 생성한다.

```typescript
// Creating the WebSocket server
const wsServer = new WebSocketServer({
  // This is the `httpServer` we created in a previous step.
  server: httpServer,
  // Pass a different path here if app.use
  // serves expressMiddleware at a different path
  path: "/graphql",
});

// Hand in the schema we just created and have the
// WebSocketServer start listening.
const serverCleanup = useServer({ schema }, wsServer);
```

HTTP 서버와 웹소켓 서버를 둘다 셧다운시킬 수 있도록 ApolloServer 생성자에 플러그인을 추가해준다.

```typescript
const server = new ApolloServer({
  schema,
  plugins: [
    // Proper shutdown for the HTTP server.
    ApolloServerPluginDrainHttpServer({ httpServer }),

    // Proper shutdown for the WebSocket server.
    {
      async serverWillStart() {
        return {
          async drainServer() {
            await serverCleanup.dispose();
          },
        };
      },
    },
  ],
});
```

```typescript
await server.start();
app.use(
  "/graphql",
  cors<cors.CorsRequest>(),
  bodyParser.json(),
  expressMiddleware(server)
);

const PORT = 4000;
// Now that our HTTP server is fully set up, we can listen to it.
httpServer.listen(PORT, () => {
  console.log(`Server is now running on http://localhost:${PORT}/graphql`);
});
```

## Resolving a subscription

`Subscription` 필드를 위한 리졸버는 다른 타입들의 리졸버와 다르다. 특히, `Subscription` 필드 리졸버는 subscribe 함수를 정의한 객체이다.

```typescript
const resolvers = {
  Subscription: {
    hello: {
      // Example using an async generator
      subscribe: async function* () {
        for await (const word of ["Hello", "Bonjour", "Ciao"]) {
          yield { hello: word };
        }
      },
    },
    postCreated: {
      // More on pubsub below
      subscribe: () => pubsub.asyncIterator(["POST_CREATED"]),
    },
  },
  // ...other resolvers...
};
```

`subscribe` 함수는 비동기적인 결과를 이터레이팅하는 표준 인터페이스인 `AsyncIterator` 타입의 객체로 리턴하여야한다. `postCreated.subscribe` 필드에서는 `pubsub.asyncIterator`에 의해 `AsyncIterator`가 생성된다.

### The PubSub class

PubSub 클래스는 프로덕션 환경에서 사용되는 것은 권장되지 않는다. 왜냐하면 단일 서버 인스턴스만 지원하는 인메모리 이벤트 시스템이기 때문이다. 개발환경에서 구독이 동작하도록 설정한 후엔, 우리는 추상클래스인 PubSubEngine 클래스의 다른 서브클래스로 바꾸는 것을 강력히 추천한다. 권장 서브클래스는 프로덕션 PubSub 라이브러리에 나열되어 있다.

`publish-subscribe (pub/sub) model`을 사용하여 액티브한 구독을 업데이트하는 이벤트를 추적할 수 있다. `graphql-subscriptions` 라이브러리는 시작하는 데 좋은 기본 인메모리 이벤트 버스인 `PubSub` 클래스를 제공한다.

```bash
npm install graphql-subscriptions
```

`PubSub` 인스턴스를 사용하면, 서버 코드에서 특정 레이블에 이벤트를 게시하고, 특정 레이블과 관련된 이벤트를 수신할 수 있다.

```typescript
import { PubSub } from "graphql-subscriptions";

const pubsub = new PubSub();
```

### Publishing an event

`PubSub` 인스턴스의 `publish` 메소드를 사용하여 이벤트를 퍼블리싱할 수 있다.

```typescript
pubsub.publish("POST_CREATED", {
  postCreated: {
    author: "Ali Baba",
    comment: "Open sesame",
  },
});
```

- 첫번째 인자로는 퍼블리싱할 이벤트 레이블 이름을 문자열로 넣는다.
  - 퍼블리싱 하기전에 레이블 이름을 등록할 필요는 없다.
- 두번째 인자로는 이벤트와 연관된 페이로드를 넣을 수 있다.
  - 페이로드는 Subscription 필드와 서브필드와 연관된 리졸버를 필수적으로 갖고있는 데이터는 포함해야한다.(?)

GraphQL 구독이 동작할때, 너는 구독의 리턴값이 업데이트되어야하면 이벤트를 발생시킨다. 그런 업데이트는 일반적으로 뮤테이션이지만, 어떤 백엔드 로직에서 변화가 생겼을때 발생되어야한다.(?)

GraphQL API에서 제공하는 `createPost` 뮤테이션을 예시로 보면

```graphql
type Mutation {
  createPost(author: String, comment: String): Post
}
```

`createPost`의 기본적인 리졸버는 이렇게 생겼을 것이다.

```typescript
const resolvers = {
  Mutation: {
    createPost(parent, args, { postController }) {
      // Datastore logic lives in postController
      return postController.createPost(args);
    },
  },
  // ...other resolvers...
};
```

새로운 post의 데이터가 데이터스토어에 추가되기전에, 우리는 그런 데이터를 포함하여 이벤트를 발생시킬 수 있다.

```typescript
const resolvers = {
  Mutation: {
    createPost(parent, args, { postController }) {
      pubsub.publish("POST_CREATED", { postCreated: args });
      return postController.createPost(args);
    },
  },
  // ...other resolvers...
};
```

그러면, Subscription 필드의 리졸버에서 이 이벤트를 listen할 수 있다.

### Listening for events

`AsyncIterator` 객체는 특정 레이블에 해달하는 이벤트를 듣고 있고, 그들을 큐에 추가한다.

`PubSub`의 메소드인 `asyncIterator`을 호출하여 `AysncIterator`을 생성할 수 있다. 그리고 `AsyncIterator`가 들어야할 이벤트 레이블 이름이 담긴 배열을 패싱해야한다.

```typescript
pubsub.asyncIterator(["POST_CREATED"]);
```

모든 `Subscription` 필드의 리졸버에서 `subscribe` 함수는 `AsyncIterator` 객체를 반환해주어야한다.

```typescript
const resolvers = {
  Subscription: {
    postCreated: {
      subscribe: () => pubsub.asyncIterator(["POST_CREATED"]),
    },
  },
  // ...other resolvers...
};
```

이 `subscribe` 함수 세팅이 되면, 아폴로서버는 `POST_CREATED` 이벤트의 페이로드를 사용해서 `postCreated` 필드의 업데이트된 값을 푸시한다.

### Filtering events

어쩔때엔 클라이언트가 특정 기준에 대해 필터링된 구독 데이터를 받아야할 때도 있다. 이런 경우 `withFilter` 헬퍼함수를 `Subscription` 필드의 리졸버에 호출하여 사용할 수 있다.

```graphql
subscription ($repoName: String!) {
  commentAdded(repoFullName: $repoName) {
    id
    content
  }
}
```

위 예시는 repoFullName이라는 변수를 받아서 해당되는 특정 리파지토리의 코멘트가 추가되었을때 알려주는 클라이언트 실행 로직이다.

여기에는 문제가 발생할 수 있다: 우리 서버는 `COMMENT_ADDED` 이벤트를 리파지토리 **상관없이** 코멘트가 추가되었을때 `publishes`할 것이다. 이것은 `commentAdded` 리졸버는 어떤 리파지토리에 추가되었는지 상관없이 새로운 코멘트가 생성될 때마다 실행된다는 의미이다. 결과적으로, 구독하고 있는 클라이언트는 그들이 원하지 않는 데이터까지 받게 되어버린다.

이걸 고치려면, 우리는 `withFilter` 헬퍼함수를 사용해서 클라이언트별로 업데이트 여부를 제어할 수 있다.

```typescript
import { withFilter } from "graphql-subscriptions";

const resolvers = {
  Subscription: {
    commentAdded: {
      subscribe: withFilter(
        () => pubsub.asyncIterator("COMMENT_ADDED"),
        (payload, variables) => {
          // Only push an update if the comment is on
          // the correct repository for this operation
          return (
            payload.commentAdded.repository_name === variables.repoFullName
          );
        }
      ),
    },
  },
  // ...other resolvers...
};
```

`withFilter` 함수는 두 개의 파라미터를 받는다:

- 첫번째 인자는 함수를 받는데, 필터링을 적용하지 않을 경우의 `subscribe` 함수이다.
- 두번째 인자는 Filter 함수를 받는데, 만약 구독이 업데이트되어 특정 클라이언트한테 보내주어야할 때, return `true`일 경우 보내주고, `false`인 경우 보내주지 않는다. (프로미스 또한 허용됨) 또, 이 Filter 함수는 두 가지 파라미터를 받는다.
  - `payload`는 published된 이벤트의 페이로드이다.
  - `variables`는 객체인데, 클라이언트가 구독을 시작할때 제공하는 모든 arguments가 포함된 객체이다.

`withFilter`를 사용해서 클라이언트가 원하는 업데이트된 구독을 받을 수 있도록 할 수 있다.

### Operation context

쿼리나 뮤테이션에서 context를 초기화할땐, HTTP 헤더와 req 객체에 있는 다른 request 메타데이터를 추출해서 `context` 함수에다가 제공해주었었다.

subscriptions에서는, useServer 함수의 첫번째 인자에서 options를 추가하는 식으로 클라이언트 요청에서 정보를 추출할 수 있다.

예를 들면,

```typescript
// ...
useServer(
  {
    // Our GraphQL schema.
    schema,
    // Adding a context property lets you add data to your GraphQL operation contextValue
    context: async (ctx, msg, args) => {
      // You can define your own function for setting a dynamic context
      // or provide a static value
      return getDynamicContext(ctx, msg, args);
    },
  },
  wsServer
);
```

`context` 함수에 넘기고 있는 첫번째 파라밑터가 ctx인데, 이 객체는 리졸버에 전달되는 contextValue가 아닌 Subscription 서버의 컨텍스트를 의미한다.

`ctx.connectionParams` 프로퍼티를 통해 웹소켓 서버에 대한 클라이언트의 subscription 요청 파라미터에 접근할 수 있다.

```typescript
const getDynamicContext = async (ctx, msg, args) => {
  // ctx is the graphql-ws Context where connectionParams live
  if (ctx.connectionParams.authentication) {
    const currentUser = await findUser(ctx.connectionParams.authentication);
    return { currentUser };
  }
  // Otherwise let our resolvers know we don't have a current user
  return { currentUser: null };
};

useServer(
  {
    schema,
    context: async (ctx, msg, args) => {
      // Returning an object will add that information to
      // contextValue, which all of our resolvers have access to.
      return getDynamicContext(ctx, msg, args);
    },
  },
  wsServer
);
```

요약하면, `useServer.context` 함수는 리졸버에서 사용할 수 있는 `contextValue` 객체를 리턴한다.

`context` 옵션은 `subscription` 요청마다 호출된다. 이벤트가 발생할떄마다 실행되는 것이 아니다. 이것은 subscription operation을 클라이언트가 매번 보낼 때마다, 우리는 위 예시에서 권한 토큰을 체크한다는 것을 의미한다.

### onConnect and onDisconnect

구독 서버를 연결하거나 연결해제할 때 어떤 설정을 할 수 있다.

`onConnect` 함수를 정의해서 `false`를 리턴하거나 예외를 던지는 식으로 특정 연결을 거절할 수 있다. 클라이언트가 첫번째로 구독서버에 연결할 때 권한체크를 하는 데에 유용하다.

`useServer`의 첫번째 인자에 이 함수를 넣어서 제공할 수 있다.

```typescript
useServer(
  {
    schema,
    // As before, ctx is the graphql-ws Context where connectionParams live.
    onConnect: async (ctx) => {
      // Check authentication every time a client connects.
      if (tokenIsNotValid(ctx.connectionParams)) {
        // You can return false to close the connection  or throw an explicit error
        throw new Error("Auth token missing!");
      }
    },
    onDisconnect(ctx, code, reason) {
      console.log("Disconnected!");
    },
  },
  wsServer
);
```

### Example: Authentication with Apollo Client

쓰고있는 subscription 라이브러리에 대해 클라이언트와 서버의 subscription 프로토콜이 모두 일관성이 있는지 확인하여야한다.

아폴로클라이언트에서는 `GraphQLWsLink` 생성자를 지원한다. 여기서 `connectionParams`를 통해 정보를 추가할 수 있다. 이것은 연결되었을때 서버에 보내져서 `Context.connectionParams`으로 접근하여 클라이언트 요청에 대한 정보를 추출할 수 있다.

클라이언트쪽 로직은 이러함:

```typescript
import { GraphQLWsLink } from "@apollo/client/link/subscriptions";
import { createClient } from "graphql-ws";

const wsLink = new GraphQLWsLink(
  createClient({
    url: "ws://localhost:4000/subscriptions",
    connectionParams: {
      authentication: user.authToken,
    },
  })
);
```

`connectionParams` 인자는 서버로 전달되어 사용자의 자격증명의 유효성을 검증할 수 있다.

`useServer.context` 프로퍼티를 사용해서 사용자의 권한을 체크하고 실행중에 `context` 인자로 리졸버에 전달된 객체를 리턴할 수 있다.

```typescript
const findUser = async (authToken) => {
  // Find a user by their auth token
};

const getDynamicContext = async (ctx, msg, args) => {
  if (ctx.connectionParams.authentication) {
    const currentUser = await findUser(ctx.connectionParams.authentication);
    return { currentUser };
  }
  // Let the resolvers know we don't have a current user so they can
  // throw the appropriate error
  return { currentUser: null };
};

// ...
useServer(
  {
    // Our GraphQL schema.
    schema,
    context: async (ctx, msg, args) => {
      // This will be run every time the client sends a subscription request
      return getDynamicContext(ctx, msg, args);
    },
  },
  wsServer
);
```

위 예제를 정리하면, 각 구독 요청과 함께 전송된 인증토큰을 기반으로 사용자를 조회한 후 리졸버가 사용할 사용자 객체를 리턴하는 예이다. 사용자가 존재하지않거나 조회에 실패하면 리졸버에서 오류가 발생하고 해당 구독작업이 실행되지않게 된다.

### Production `PubSub` libraries

앞서 언급했던 `PubSub` 클래스는 event-publishing system이 인메모리이므로, 프로덕션 환경에서는 권장되지 않는다. 즉, 그랲큐엘 서버의 한 인스턴스에서 published한 이벤트는 다른 인스턴스에서 처리하는 subscriptions에서는 받을 수 없다.

대신, 레디스 또는 카프카와 같은 외부 데이터스토어로 백업할 수 있는 `PubSubEngine`이라는 추상클래스를 사용해야한다.

다음은 많이쓰이는 event-publishing systems을 위한 커뮤니티가 만든 PubSub 라이브러리들이다.

- [redis](https://github.com/davidyaha/graphql-redis-subscriptions)
- [Google PubSub](https://github.com/axelspringer/graphql-google-pubsub)

....
... 직접생성할수도있음

### Switching from `subscription-transport-ws`

아폴로에서는~ `graphql-ws`를 사용하도록 강력히 권고하고 있다. 이 라이브러리는 활발하게 유지관리되지 않는다.

그래도 이걸 쓸 경우에는 [아폴로 서버3 도큐먼트](https://www.apollographql.com/docs/apollo-server/v3/data/subscriptions/#switching-to-graphql-ws)를 봐주시길~

### Updating subscription clients

`subscriptios-transport-ws`에서 `graphql-ws`로 바꿀 의도라면 아래 클라이언트들을 업데이트해주어야한다.

<img width="813" alt="스크린샷 2023-07-05 오후 11 06 23" src="https://github.com/leesoo7595/leesoo7595.github.io/assets/39291812/a67af8ee-8fdd-46d8-b06f-30c0f67f41d5">

## Reference

- [Apollo Server - Generating types from a Graphql schema](https://www.apollographql.com/docs/apollo-server/workflow/generate-types)
