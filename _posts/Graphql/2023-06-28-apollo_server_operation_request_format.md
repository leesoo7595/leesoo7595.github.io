---
layout: post
title: "Apollo Server - Operation request format"
date: 2023-06-28
tags: [Graphql]
comments: true
---

이 글은 [아폴로 공식문서](https://www.apollographql.com/docs/apollo-server/workflow/requests)를 번역한 글 입니다.

# Operation request format

아폴로서버에 HTTP 요청을 보내는 방법

Apollo Server는 POST 요청으로 전송된 쿼리나 뮤테이션을 허용한다. GET 요청으로 전송된 쿼리도 허용한다.

## POST requests

Apollo Server는 JSON 바디로 들어오는 POST 요청을 허용한다. 유효한 요청은 query 필드를 포함하고, variables, extensions, operationName을 선택적으로 같이 들어올 수 있다.(요청이 가능한 여러 operations가 포핟된 경우, 실행할 쿼리를 명확히 하기위함이다.) `application/json` 타입으로 Content-Type을 지정해주어야한다.

```graphql
query GetBestSellers($category: ProductCategory) {
  bestSellers(category: $category) {
    title
  }
}
```

위 쿼리를 HTTP 요청을 통해 실행시키고 싶다면,

```json
{
  "query": "query GetBestSellers($category: ProductCategory){bestSellers(category: $category){title}}",
  "operationName": "GetBestSellers",
  "variables": { "category": "BOOKS" }
}
```

유효한 POST 요청을 body에다가 위처럼 구성하여 보낼 수 있다.

`operationName`의 경우 필수적으로 request body에 들어가야하는 것은 아니다. 왜냐면 `query`에 오직 하나의 정의된 operation이 들어가기 때문이다.

```sh
# 아래 curl을 활용해서 바로 http 요청으로 아폴러서버에 query를 실행시킬 수 있다.
curl --request POST \
  -H 'Content-Type: application/json' \
  --data '{"query":"query GetBestSellers($category: ProductCategory){bestSellers(category: $category){title}}", "operationName":"GetBestSellers", "variables":{"category":"BOOKS"}}' \
  https://rover.apollo.dev/quickstart/products/graphql
```

아폴로 서버의 디폴트 프로덕션 랜딩페이지에서도 curl 커맨드를 사용하여 query를 테스트할 수 있게끔 제공해준다.

## Batching

Apollo Server 4에서는 기본적으로 batching HTTP 요청을 제공하지 않는다. HTTP 배치가 가능하기 위해서는, `allowBatchedHttpRequests: true` 옵션을 `ApolloServer` 생성자에 넘겨주도록 명시해야한다.

HTTP 배치 처리를 활성화한 경우, 다음과 같이 쿼리 객체의 JSON 배열을 제공해서 단일 POST 요청으로 쿼리들을 일괄 처리할 수 있다.

```json
[
  {
    "query": "query { testString }"
  },
  {
    "query": "query AnotherQuery{ test(who: \"you\" ) }"
  }
]
```

배치 요청을 보내면, 아폴로서버는 GraphQL 배열 응답을 받게 된다.

요청에 포함된 여러 operations에 동일한 HTTP 응답 헤더를 세팅하려면, 헤더별로 나중 opeations의 헤더가 우선시된다.

## GET requests

아폴로서버는 또한 쿼리를 통해 GET 요청도 가능하다. (mutations는 안됨) GET 요청을 사용하면, 쿼리 세부정보가 URL 쿼리 매개변수로 제공된다. variables 옵션은 URL 이스케이프된 JSON 객체입니다.

쿼리를 GET 요청으로 보내면 CDN 캐싱에 도움이 될 수 있다.

```sh
# curl 요청을 활용해서 GET 요청 보내기
curl --request GET \
  https://rover.apollo.dev/quickstart/products/graphql?query=query%20GetBestSellers%28%24category%3A%20ProductCategory%29%7BbestSellers%28category%3A%20%24category%29%7Btitle%7D%7D&operationName=GetBestSellers&variables=%7B%22category%22%3A%22BOOKS%22%7D
```

POST 요청과는 달리 GET 요청에는 `Content-Type` 헤더가 필요하지 않다. 그러나 Apollo Server 4의 기본 CSRF 방지 기능을 사용하도록 설정한 경우 GET 요청에는 다음 중 하나가 포함되어야 한다.

- `X-Apollo-Operation-Name` 헤더가 비어있지않아야함
- `Apollo-Require-Preflight` 헤더가 비어있지 않아야함

## Incremental delivery (experimental)

아직 실험단계이지만, @defer, @stream 디렉티브를 추가한다.
이 디렉티브는 클라이언트가 operations의 일부를 초기 응답 후에 보낼 수 있도록 지정하여 느린 필드로 인한 모든 필드가 지연되지 않도록 할 수 있다. `graphql-js`라고도 불리는 라이브러리는 아직 출시되지않은 버전17에서만 Incremental delivery를 제공한다. 서버에 graphql@17 사전 릴리스가 설치되어 있는 경우 Apollo Server 4는 이러한 incremental delivery 디렉티브를 실행하고 스트리밍 `multipart/mixed` 응답을 제공할 수 있습니다.

이 지원은 옵트인 방식으로 제공되어 디렉티브가 기본적으로 정의되어있지 않다. 그래서 @defer, @stream을 사용하려면 SDL에 정의를 하여야한다. 아래 정의를 그대로 스키마에 붙여넣을 수 있다.

```graphql
directive @defer(
  if: Boolean
  label: String
) on FRAGMENT_SPREAD | INLINE_FRAGMENT
directive @stream(if: Boolean, label: String, initialCount: Int = 0) on FIELD
```

스키마 객체를 직접 생성하는 경우, 스키마 생성자에 적절한 디렉티브를 제공해야한다.

```typescript
import {
  GraphQLSchema,
  GraphQLDeferDirective,
  GraphQLStreamDirective,
  specifiedDirectives,
} from "graphql";

const schema = new GraphQLSchema({
  query,
  directives: [
    ...specifiedDirectives,
    GraphQLDeferDirective,
    GraphQLStreamDirective,
  ],
});
```

Incremental delivery 디렉티브를 사용해서 operations를 보내는 클라이언트는 accept 헤더에 `multipart/mixed` 응답을 받을 것이라는 것을 명시적으로 표시해야한다. 또한 Incremental delivery는 아직 그랲큐엘 스펙에서 확정되지 않았고, 최종버전이 나오기 전까지는 변경될 수 있다. 그래서 deferSpec 파라미터를 통해 지정해줌으로써 아폴로서버가 생성하는 특정 응답포맷을 지정해주어야한다.

Incremental delivery 응답을 수락할 준비가 된 클라이언트는 `multipart/mixed; deferSpec=20220824`와 같은 accept 헤더를 보내야 합니다. 이 헤더는 멀티파트 응답만 허용해야하고, 일반적으로 클라이언트는 멀티파트 또는 단일 파트 응답이 모두 허용됨을 나타내는 multipart/mixed, deferSpec=20220824, application/json과 같은 허용 헤더를 전송한다.

동일한 요청으로 배치 처리와 Incremental delivery를 결합할 수 없다.

## Reference

- [Apollo Server - operation request format](https://www.apollographql.com/docs/apollo-server/workflow/requests)
