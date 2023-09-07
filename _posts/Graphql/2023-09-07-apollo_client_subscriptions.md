---
layout: post
title: "Apollo Client - Subscriptions"
date: 2023-09-07
tags: [Graphql]
comments: true
---

# Subscriptions

그래프큐엘 서버에서 실시간으로 업데이트 한 것을 가져오기

queries, mutations과 추가적으로 subscriptions라는 operation 타입을 지원한다.

queries처럼, subscriptions로 데이터를 페칭하는 것이 가능하다. 반대로 queries와 달리, 시간이 지나면 결과가 변경될 수 있는 long-lasting operations이다. 그들은 그래프큐엘서버에서 연결을 유지할 수 있다.(일반적으로 웹소켓을 통해서), 서버에서 subscription의 결과가 업데이트한 것을 푸시하는 것이 가능하다.

구독은 클라이언트에 실시간으로 백엔드 데이터의 변화를 알리는데 유용하다. 예를 들면 새로운 객체 생성이나 중요한 필드의 수정 같은 경우이다.

## 구독을 언제 사용할까

일반적인 경우에서 클라이언트는 백엔드를 최신 상태로 유지하기 위해(?) 구독을 사용하면 안된다. 대신, 사용자가 관련 액션을(버튼을 클릭하는 등) 수행할 때 query들을 간헐적으로 폴링하거나, 필요에 따라 다시 실행시킨다.

다음의 경우 구독을 사용해야한다:

- 조금씩, 차차 바뀌는 커다란 객체들의 경우이다. 특히 대부분의 객체 필드가 종종 바뀌는 경우에, 거대한 객체에 대한 반복적인 폴링은 비용이 많이 든다. 대신 query로 객체의 초기상태를 패칭할 수 있고, 서버에서 선제적으로 각각의 필드 업데이트 부분을 푸시하여 일으킬 수 있다.
- 짧은 지연시간, 실시간 업데이트의 경우이다. 예를 들어 채팅 어플리케이션의 클라이언트는 새로운 메세지를 가능하면 빨리 받도록 원한다.

> 구독은 캐시의 변경사항이나 로컬 클라이언트 이벤트를 듣는 것에 대해서는 사용할 수 없다. 구독은 외부의 데이터 변화를 구독하는데 의도되었고, 바뀐 부분을 받아서 캐시에 저장하도록 한다. 그런 다음 아폴로 클라이언트의 가시성 모델을 활용하여 watchQuery 또는 useQuery를 사용하여 캐시의 변경사항을 관찰할 수 있다.

## 지원되는 구독 프로토콜

그래프큐엘 스펙으로는 구독 요청을 보내는 것에 대한 특정 프로토콜을 정의하지 않았다. 아폴로 클라이언트는 구독에 대해 다음 프로토콜을 지원한다:

- WebSocket
  - graphql-ws
  - subscriptions-transport-ws (유지되고 있지 않음)
- HTTP (chunked multipart)

> 주고받는 같은 endpoint를 사용하도록 한다.

### 웹소켓 서브프로토콜

`graphql-ws`를 사용해야함

### HTTP

multipart subscriptions over HTTP를 사용하려면 아폴로 클라이언트가 3.7.11 이후 버전이어야 한다.

아폴로 클라이언트는 terminating HTTPLink에 구독 operation이 전달된 경우, 요구되는 헤더를 요청에 자동으로 전송한다.

## 구독 정의

### Server side

그랲큐엘 스키마에서 Subscription 타입의 구독을 정의할 수 있다. 다음은 `commentAdded` 구독을 정의하여 새로운 comment가 추가된 경우 클라이언트에게 알리는 아이이다.

```graphql
type Subscription {
  commentAdded(postID: ID!): Comment
}
```

### Client side

클라이언트에서는 아폴로 클라이언트에서 실행하고 싶은 각 구독의 생김새를 정의한다:

```javascript
const COMMENTS_SUBSCRIPTION = gql`
  subscription OnCommentAdded($postID: ID!) {
    commentAdded(postID: $postID) {
      id
      content
    }
  }
`;
```

OnCommentAdded 구독을 실행하면, 그랲큐엘 서버와 연결하고 데이터 응답을 들을 수 있도록 한다. query와 달리, 서버에서 곧바로 실행되어 응답을 리턴하지 않는다. 대신 서버는 백엔드에서 특정 이벤트가 일어났을 때 클라이언트로 데이터를 푸시해준다.

그랲큐엘 서버가 구독 클라이언트에 데이터를 푸시해줄 때, 데이터는 다음과 같은 구조로 내려온다. 이건 쿼리와 같다:

```json
{
  "data": {
    "commentAdded": {
      "id": "123",
      "content": "What a thoughtful and well written post!"
    }
  }
}
```

## 웹소켓 셋업

### 1. 요구되는 라이브러리 설치

Apollo Link는 아폴로 클라이언트의 네트워크 커뮤니케이션을 커스터마이징하도록 도와준다. 이를 사용하여 작업을 수정하고 적절한 대상으로 라우팅하는 링크 체인을 정의할 수 있다.

웹소켓을 이용하여 구독을 실행하면, GraphQLWsLink를 링크 체인에 추가할 수 있다. 이 링크는 graphql-ws 라이브러리에서 요구된다. 다음과 같이 설치:

```bash
npm install graphql-ws
```

### 2. `GraphQLWsLink` 초기화

`GraphQLWsLink` 객체를 같은 프로젝트 파일에 ApolloClient를 초기화한 곳에서 임포트하고 초기화한다:

```javascript
import { GraphQLWsLink } from "@apollo/client/link/subscriptions";
import { createClient } from "graphql-ws";

const wsLink = new GraphQLWsLink(
  createClient({
    url: "ws://localhost:4000/subscriptions",
  })
);
```

url 옵션의 값을 그랲큐엘 서버의 구독 관련 웹소켓 엔드포인트로 바꾼다. 아폴로서버를 사용하는 경우 구독 엔드포인트 설정 참조

### 3. operation을 통한 커뮤니케이션 분리 (권장)

아폴로 클라이언트가 모든 operation 타입을 실행하는데 GraphQLWsLink를 사용할 수 있음에도 불구하고, 대부분의 경우 HTTP를 사용하여야한다. 이것은 쿼리와 뮤테이션이 stateful하거나 long-lasting connection이 요구되지 않기 때문이다. 웹소켓 연결이 아직 없는 경우, HTTP를 만드는 것이 더 효율적이고 스케일러블하다.

이것을 지원해주기 위해서는 아폴로 클라이언트 라이브러리에서 split 함수를 제공해준다. 이 함수는 두 개의 다른 Link를 사용할 수 있도록 한다.

다음 예시는 `GraphQLWsLink`와 `HttpLink`를 모두 초기화하여 이전 예제를 확장한 것이다.
그런 다음 split 함수를 사용하여 두 링크를 실행중인 operation 타입에 따라 둘 중 하나를 사용하는 단일 Link로 결합한다.

```javascript
import { split, HttpLink } from "@apollo/client";
import { getMainDefinition } from "@apollo/client/utilities";
import { GraphQLWsLink } from "@apollo/client/link/subscriptions";
import { createClient } from "graphql-ws";

const httpLink = new HttpLink({
  uri: "http://localhost:4000/graphql",
});

const wsLink = new GraphQLWsLink(
  createClient({
    url: "ws://localhost:4000/subscriptions",
  })
);

// split 함수는 3개의 인자를 받는다:
//
// * 실행할 각 operation 함수
// * "truthy" 값인 링크
// * "falsy" 값인 링크
const splitLink = split(
  ({ query }) => {
    const definition = getMainDefinition(query);
    return (
      definition.kind === "OperationDefinition" &&
      definition.operation === "subscription"
    );
  },
  wsLink,
  httpLink
);
```

이 로직을 사용하면, 쿼리와 뮤테이션은 HTTP로 사용하게 되고, 구독의 경우 웹소켓을 사용한다.

### 4. 아폴로 클라이언트에서 link chain 제공

link chain을 정의한 후, 아폴로 클라이언트에 link 생성자 옵션을 넘겨준다:

```javascript
import { ApolloClient, InMemoryCache } from "@apollo/client";

// ...code from the above example goes here...

const client = new ApolloClient({
  link: splitLink,
  cache: new InMemoryCache(),
});
```

link 옵션을 넣어주면, uri 옵션보다 이 옵션이 우선하게 된다.(제공된 url을 사용하여 기본 HTTP 링크 체인을 설정한다.)

### 5. 웹소켓에서 인증하기 (선택)

클라이언트가 구독 결과를 받도록 허용하기 전에 클라이언트를 인증해야하는 경우가 많다. 이것을 하려면, `connectionParams` 옵션을 GraphQLWsLink 생성자에 넣어준다:

```javascript
import { GraphQLWsLink } from "@apollo/client/link/subscriptions";
import { createClient } from "graphql-ws";

const wsLink = new GraphQLWsLink(
  createClient({
    url: "ws://localhost:4000/subscriptions",
    connectionParams: {
      authToken: user.authToken,
    },
  })
);
```

## multipart HTTP를 통한 구독

추가 라이브러리나 구성이 필요하지 않다. Apollo 클라이언트는 링크 또는 Apollo 클라이언트 인스턴스를 초기화할 때 지정한 URL에서 기본 종료 HTTPLink가 구독 작업을 수신할 때 요청에 필요한 헤더를 추가합니다.

## 구독 실행하기

리액트에서 구독을 실행시키기 위해 `useSubscription` 훅을 사용할 수 있다. useQuery처럼, useSubscription은 loading, error, data 프로퍼티를 담은 객체를 리턴해서 UI를 렌더링하는 데에 사용할 수 있다.

다음 예제를 통해 컴포넌트가 구독을 사용하여 지정된 블로그 게시물에 추가된 최근 comment를 렌더링한다. 그랲큐엘 서버가 새로운 comment를 클라이언트에 푸시하면, 컴포넌트는 새로운 comment로 리렌더한다.

```javascript
const COMMENTS_SUBSCRIPTION = gql`
  subscription OnCommentAdded($postID: ID!) {
    commentAdded(postID: $postID) {
      id
      content
    }
  }
`;

function LatestComment({ postID }) {
  const { data, loading } = useSubscription(COMMENTS_SUBSCRIPTION, {
    variables: { postID },
  });
  return <h4>New comment: {!loading && data.commentAdded.content}</h4>;
}
```

## 구독으로 쿼리 업데이트하기

아폴로클라이언트에서 쿼리가 결과를 리턴할때마다 해당 결과에는 subscribeToMore 함수를 포함한다. 이 함수를 사용하여 쿼리의 원래 결과에 대한 업데이트를 푸시하는 팔로업 구독을 실행할 수 있다.

> subscribeToMore 함수는 페이지네이션에서 일반적으로 쓰이는 fetchMore 함수와 구조가 비슷하다. 주요 차이점으로는 fetchMore은 후속 쿼리로 실행되고, 반면 subscribeToMore은 구독을 실행한다.

주어진 블로그 포스팅의 comment를 패칭한 표준 쿼리를 예시로 시작해보자:

```jsx
const COMMENTS_QUERY = gql`
  query CommentsForPost($postID: ID!) {
    post(postID: $postID) {
      comments {
        id
        content
      }
    }
  }
`;

function CommentsPageWithData({ params }) {
  const result = useQuery(COMMENTS_QUERY, {
    variables: { postID: params.postID },
  });
  return <CommentsPage {...result} />;
}
```

그랲큐엘 서버에서 새로운 comment를 포스트에 추가하자마자 클라이언트에 이 업데이트사항을 푸시하게 하고 싶다. 첫 번째로 우리는 아폴로 클라이언트에서 `COMMENTS_QUERY`가 실행한 결과에 대한 구독을 정의해야한다:

```javascript
const COMMENTS_SUBSCRIPTION = gql`
  subscription OnCommentAdded($postID: ID!) {
    commentAdded(postID: $postID) {
      id
      content
    }
  }
`;
```

그 다음에는, 우리는 `CommentsPageWithData` 함수를 수정해서 이 함수가 반환하는 CommentsPage 컴포넌트에 `subscribeToNewComments` 프로퍼티를 추가한다. 이 프로퍼티는 컴포넌트가 마운트 된 후, subscribeToMore을 호출하는 함수를 담당한다.

```jsx
function CommentsPageWithData({ params }) {
  const { subscribeToMore, ...result } = useQuery(COMMENTS_QUERY, {
    variables: { postID: params.postID },
  });

  return (
    <CommentsPage
      {...result}
      subscribeToNewComments={() =>
        subscribeToMore({
          document: COMMENTS_SUBSCRIPTION,
          variables: { postID: params.postID },
          updateQuery: (prev, { subscriptionData }) => {
            if (!subscriptionData.data) return prev;
            const newFeedItem = subscriptionData.data.commentAdded;

            return Object.assign({}, prev, {
              post: {
                comments: [newFeedItem, ...prev.post.comments],
              },
            });
          },
        })
      }
    />
  );
}
```

위 예제에서, 우리는 3가지 옵션을 subscribeToMore에 넘겼다:

- 구독이 실행할 document
- 구독이 실행될 때 포함해야하는 variables
- 아폴로 클라이언트에서 그랲큐엘 서버에 푸시된 `subscriptionData`와 캐시된 이전 결과와 어떻게 조합할지 알려주는 함수인 updateQuery. 이 함수의 리턴값은 쿼리에 대한 현재 캐시된 결과를 완전히 대체한다.

마지막으로, `CommentsPage`의 정의에서 컴포넌트가 마운트될 때 `subscribeToNewComments`를 호출한다:

```javascript
export function CommentsPage({ subscribeToNewComments }) {
  useEffect(() => subscribeToNewComments(), []);
  return <>...</>;
}
```
