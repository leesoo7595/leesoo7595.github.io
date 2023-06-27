---
layout: post
title: "Apollo Server - Build and run queries against Apollo Server"
date: 2023-06-27
tags: [Graphql]
comments: true
---

이 글은 [아폴로 공식문서](https://www.apollographql.com/docs/apollo-server/workflow/build-run-queries/)를 번역한 글 입니다.

# Build and run queries against Apollo Server

프로덕션이 아닌 환경에서는 Apollo Server 4의 랜딩 페이지는 Apollo Sandbox가 내장되어있다. `http://localhost:4000`로 접근할 수 있다.

<img width="763" alt="스크린샷 2023-06-27 오후 9 57 21" src="https://github.com/leesoo7595/leesoo7595.github.io/assets/39291812/07b333b5-9198-4b76-917f-44c6653d32e7">

아폴로 샌드박스는 로컬 개발에서 사용되는 것으로, 계정이나 그런게 필요가 없다. 샌드박스에서는 서버를 대상으로 작업을 빌드하고 테스트할 수 있는 강력한 웹 IDE인 Apollo Studio Explorer를 포함하고 있다.

## Production vs non-production

`NODE_ENV`가 `production`일떄, 아폴로 서버는 다른 랜딩 페이지를 제공한다.

<img width="398" alt="스크린샷 2023-06-27 오후 9 57 54" src="https://github.com/leesoo7595/leesoo7595.github.io/assets/39291812/a234adbd-0f86-4558-9cac-58894714ad6e">

이는 아폴로서버가 프로덕션 환경에서 기본적으로 인트로스펙션을 비활성화하기 때문이다. 이는 프로덕션환경에서는 샌드박스와 같은 도구가 작동하지 않는다는 것을 의미한다.

그래서 apollo server 랜딩페이지를 변경하려는 경우, 프로덕션 환경과 비프로덕션 환경에 따라 다르게 설정하는 것이 좋다. (프로덕션 환경에서는 비활성도 가능)

### Changing the landing page

아폴로서버의 기본 URL에서 제공되는 랜딩페이지를 변경할 수 있다. 기본 랜딩페이지를 구성하거나, GraphQL Playground를 제공하거나, 커스텀 랜딩페이지를 만들거나, 완전히 비활성화하는 등의 선택이 가능하다.

이 방법은 아폴로서버 플러그인 중 하나를 설치하면 된다.

### Configuring the default landing Page

아폴로서버는 환경에 따른 기본적인 랜딩페이지를 보여주기 위해 플러그인들 중 하나를 사용한다.

- `ApolloServerPluginLandingPageLocalDefault` (non-production environments)
- `ApolloServerPluginLandingPageProductionDefault` (production)

이러한 플러그인 중 수동으로 설치해서 동작을 구성할 수 있다.

```typescript
import { ApolloServer } from "@apollo/server";
import {
  ApolloServerPluginLandingPageLocalDefault,
  ApolloServerPluginLandingPageProductionDefault,
} from "@apollo/server/plugin/landingPage/default";

const server = new ApolloServer({
  typeDefs,
  resolvers,
  plugins: [
    process.env.NODE_ENV === "production"
      ? ApolloServerPluginLandingPageProductionDefault()
      : ApolloServerPluginLandingPageLocalDefault({ embed: false }),
  ],
});
```

위는 아폴로 샌드박스로 연결되는 링크가 포함된 페이지를 제공하도록 로컬 랜딩페이지를 설정하였다.

### Custom landing page

아폴로서버의 기본 url에서 커스텀 HTML 랜딩페이지를 제공하는 것도 가능하다. 그럴려면, 너만의 `renderLandingPage` 메소드를 호출해서 쓰는 custom plugin을 정의한다.

```typescript
const server = new ApolloServer({
  typeDefs,
  resolvers,
  plugins: [
    {
      async serverWillStart() {
        return {
          async renderLandingPage() {
            const html = `
              <!DOCTYPE html>
              <html>
                <head>
                </head>
                <body>
                  <h1>Hello world!</h1>
                </body>
              </html>`;
            return { html };
          },
        };
      },
    },
  ],
});
```

### Disabling the landing page

아폴로서버의 랜딩페이지를 비활성화하기 위해서는 ApolloServer 생성자에 다음 플러그인을 넣어주면 된다.

```typescript
import { ApolloServerPluginLandingPageDisabled } from "@apollo/server/plugin/disabled";

const server = new ApolloServer({
  typeDefs,
  resolvers,
  plugins: [ApolloServerPluginLandingPageDisabled()],
});
```

## Reference

- [Apollo Server - build run queries](https://www.apollographql.com/docs/apollo-server/workflow/build-run-queries/)
