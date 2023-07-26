---
layout: post
title: "Apollo Client - Fragments"
date: 2023-07-27
tags: [Graphql]
comments: true
---

이 글은 아폴로 [공식문서](https://www.apollographql.com/docs/react/data/fragments/)를 읽고 정리한 글이다.

# Fragments

operation들 사이에서 필드 공유

GraphQL Fragment는 여러 쿼리와 뮤테이션에서 공유될 수 있는 코드 조각이다.

아래 예시는 Person 객체에서 사용될 수 있는 NameParts 프래그먼트를 선언한 것이다.

```graphql
fragment NameParts on Person {
  firstName
  lastName
}
```

모든 프래그먼트는 관련된 타입에 속하는 하위 필드들을 포함한다. 위 예제에서 Person 타입은 NameParts 프래그먼트가 유효하려면 firstName과 lastName 필드를 꼭 선언해야한다.

우리는 다음과 같이 Person 객체를 참조하는 쿼리나 뮤테이션에 NameParts 프래그먼트를 원하는대로 포함할 수 있다.

```graphql
query GetPerson {
  people(id: "7") {
    ...NameParts
    avatar(size: LARGE)
  }
}
```

프래그먼트를 포함할 때 프래그먼트 앞에 자바스크립트의 스프레드 연산자처럼 `...`을 사용해주어야한다.

NameParts 정의에 따르면 위 쿼리는 아래와 같다.

```graphql
query GetPerson {
  people(id: "7") {
    firstName
    lastName
    avatar(size: LARGE)
  }
}
```

만약 나중에 NameParts 프래그먼트 안에 포함되어있는 필드가 바뀌면, 우리는 자동으로 바뀐 필드들이 포함된 프래그먼트를 사용하여 operation에 반영될 것이다. 이것은 operation들이 일관된 필드를 가질 수 있도록 유지하는 노력을 줄여준다.

## Example usage

comments 관련 Graphql operation을 몇개 실행시키는 블로그 어플리케이션이 있다고 하자(코멘트를 제출하기, 포스트의 코멘트를 가져오기 등). 이 operation들은 아마 Comment 타입의 특정 필드들을 모두 포함할 것이다.

이런 핵심 필드들을 명시하려면, 우리는 Comment 타입의 프래그먼트를 선언할 수 있다:

```javascript
import { gql } from "@apollo/client";

export const CORE_COMMENT_FIELDS = gql`
  fragment CoreCommentFields on Comment {
    id
    postedBy {
      username
      displayName
    }
    createdAt
    content
  }
`;
```

프래그먼트는 어플리케이션 어디에서나 선언이 가능하다. 위 예제의 경우 fragments.js 파일에서 프래그먼트를 exports해온다.

우리는 이제 CoreCommentFields 프래그먼트를 Graphql operation에 포함할 수 있따:

```javascript
import { gql } from "@apollo/client";
import { CORE_COMMENT_FIELDS } from "./fragments";

export const GET_POST_DETAILS = gql`
  ${CORE_COMMENT_FIELDS}
  query CommentsForPost($postId: ID!) {
    post(postId: $postId) {
      title
      body
      author
      comments {
        ...CoreCommentFields
      }
    }
  }
`;

// ...PostDetails component definition...
```

- 우리는 `CORE_COMMENT_FIELDS`를 임포트했다
- `GET_POST_DETAILS`를 gql로 정의해서 프래그먼트를 넣어주었다. (`${CORE_COMMENT_FIELDS}` placeholder를 통해서)
- `CoreCommentFields` 프래그먼트를 쿼리 안에서 `...`를 붙여서 포함시켰다.

## Registering named fragments using `createFragmentRegistry`

아폴로 클라이언트 3.7에서 프래그먼트는 `InMemoryCache`에 등록될 수 있다. 선언 사이에 프래그먼트들을 끼워넣을(?) 필요없이 쿼리나 `InMemoryCache` 오퍼레이션에 이름으로 사용될 수 있다. (`cache.readFragment`, `cache.readQuery`, `cache.watch`)

리액트 예제를 보자

```javascript
import { ApolloClient, gql, InMemoryCache } from "@apollo/client";
import { createFragmentRegistry } from "@apollo/client/cache";

const client = new ApolloClient({
  uri: "http://localhost:4000/graphql"
  cache: new InMemoryCache({
    fragments: createFragmentRegistry(gql`
      fragment ItemFragment on Item {
        id
        text
      }
    `)
  })
});
```

ItemFragment가 InMemoryCache에 등록되었기 때문에, 이름을 참고해서 GetItemList 쿼리 안에 프래그먼트를 바로 넣을 수 있다.

```javascript
const listQuery = gql`
  query GetItemList {
    list {
      ...ItemFragment
    }
  }
`;
function ToDoList() {
  const { data } = useQuery(listQuery);
  return (
    <ol>
      {data?.list.map((item) => (
        <Item key={item.id} text={item.text} />
      ))}
    </ol>
  );
}
```

### Overriding registered fragments with local versions

쿼리는 쿼리 내부에 선언된 프래그먼트가 다른 이미 등록된 프래그먼트에 의해 간접적으로 참조되는 경우에도 `createFragmentRegistry`를 통해 등록된 프래그먼트보다 우선하는 프래그먼트들의 로컬 버전(쿼리 내부에서 바로 선언하여 사용되는)을 선언할 수 있다. 로컬 프래그먼트가 간접적으로 다른 등록 프래그먼트에 있어도 가능하다.

```javascript
import { ApolloClient, gql, InMemoryCache } from "@apollo/client";
import { createFragmentRegistry } from "@apollo/client/cache";

const client = new ApolloClient({
  uri: "http://localhost:4000/graphql"
  cache: new InMemoryCache({
    fragments: createFragmentRegistry(gql`
      fragment ItemFragment on Item {
        id
        text
        ...ExtraFields
      }

      fragment ExtraFields on Item {
        isCompleted
      }
    `)
  })
});
```

ItemList.jsx에 내부에서 선언된 ExtraFields 프래그먼트가 기존의 InMemoryCache에 등록된 ExtraFields보다 우선한다. 명시적으로 정의가 등록된 프래그먼트보다 우선시되기 때문에 ExtraFields 프래그먼트가 등록된 ItemFragment 내부에서만 해당 정의가 사용되며, GetItemList 쿼리가 실행될 때만 사용된다.

```javascript
const listQuery = gql`
  query GetItemList {
    list {
      ...ItemFragment
    }
  }

  fragment ExtraFields on Item {
    createdBy
  }
`;
function ToDoList() {
  const { data } = useQuery(listQuery);
  return (
    <ol>
      {data?.list.map((item) => (
        {/* `createdBy`는 items에 존재하고, `isCompleted` 는 없다. */}
        <Item key={item.id} text={item.text} author={item.createdBy} />
      ))}
    </ol>
  );
}
```

## Colocating framents

트리처럼 구성되는 구조인 Graphql 응답은 프론트엔드의 랜더되는 컴포넌트의 계층 구조와 유사하다. 이 유사함 때문에, 프래그먼트를 사용하여 쿼리 로직을 컴포넌트 간에 분할할 수 있고, 그래서 각 컴포넌트가 사용하는 필드를 정확하게 요청하도록 한다. 이를 통해서 컴포넌트 로직을 더욱 간결하게 만들 수 있다.

앱에서 아래 계층 구조를 가지고 있다면,

```
FeedPage
└── Feed
    └── FeedEntry
        ├── EntryInfo
        └── VoteButtons
```

이 앱에서는 FeedPage 컴포넌트에서 쿼리가 실행되어 FeedEntry 객체 리스트를 가져온다. EntryInfo와 VoteButtons는 하위 컴포넌트이고 FeedEntry 객체의 특정 필드만을 필요로 한다.

### Creating colocated fragments

배치(?) 프래그먼트는 해당 프래그먼트의 필드를 사용하는 특정 컴포넌트에 연결된다는 것만 제외하면 다른 프래그먼트와 같다. 예를 들면, VoteButtons는 FeedPage의 자식 컴포넌트인데, FeedEntry 객체에서 score와 vote의 choice만 사용한다.

```javascript
VoteButtons.fragments = {
  entry: gql`
    fragment VoteButtonsFragment on FeedEntry {
      score
      vote {
        choice
      }
    }
  `,
};
```

자식 컴포넌트 안에 프래그먼트를 선언한 후, 부모 컴포넌트는 다음과 같이 선언된 배치된 프래그먼트를 참조할 수 있다.

```javascript
FeedEntry.fragments = {
  entry: gql`
    fragment FeedEntryFragment on FeedEntry {
      commentCount
      repository {
        full_name
        html_url
        owner {
          avatar_url
        }
      }
      ...VoteButtonsFragment
      ...EntryInfoFragment
    }
    ${VoteButtons.fragments.entry}
    ${EntryInfo.fragments.entry}
  `,
};
```

`VoteButtons.fragments.entry`에 대한 이름말고는 딱히 다를게 없다. 컴포넌트가 주어졌을 때 해당 컴포넌트의 프래그먼트를 찾을 수 있다면 컨벤션은 자유롭게 사용가능하다.

### Importing fragments when using Webpack

.graphql 파일이 `graphql-tag/loader`로 로드될때, 우리는 import 구문을 통해 프래그먼트를 포함할 수 있다. 예를 들면:

```graphql
#import "./someFragment.graphql"
```

이렇게 하면 현재 파일에서 `someFragment.graphql`의 내용을 사용할 수 있다. 자세한 내용은 [여기 - 웹팩 프래그먼트](https://www.apollographql.com/docs/react/integrations/webpack/#fragments) 참고

## Using fragments with unions and interfaces

union과 interface에서 프래그먼트를 정의할 수 있다.

다음 예시는 쿼리 안에 3개의 프래그먼트가 인라인으로 포함되어있다.

```graphql
query AllCharacters {
  all_characters {
    ... on Character {
      name
    }

    ... on Jedi {
      side
    }

    ... on Droid {
      model
    }
  }
}
```

all_characters 쿼리는 Character 객체 리스트를 리턴한다. Character 타입은 Jedi와 Driod 타입이 둘 다 구현되어있는 인터페이스이다. 각 리스트 안의 아이템은 Jedi 타입에 있는 side 필드를 포함하고, Droid 타입의 model 필드를 포함한다.

그러나 이 쿼리가 동작하려면, 클라이언트는 Character 인터페이스와 이를 구현하는 타입 간의 다형성 관계를 이해해야한다. 클라이언트에 이러한 관계를 알리기 위해 `InMemoryCache`를 초기화할 때 `possibleType` 옵션을 전달할 수 있다.

## Defining `possibleTypes` manually

`possibleTypes` 옵션은 아폴로 클라이언트 3.0 이상부터 사용가능하다.

possibleTypes 옵션을 InMemoryCache 생성자에 전달해서 스키마 안의 supertype-subtype 관계를 명시해줄 수 있다. 이 객체는 인터페이스 또는 유니온 타입의 이름을 구현하거나 하위 유형으로 매핑한다.

```typescript
const cache = new InMemoryCache({
  possibleTypes: {
    Character: ["Jedi", "Droid"],
    Test: ["PassingTest", "FailingTest", "SkippedTest"],
    Snake: ["Viper", "Python"],
  },
});
```

이 예제는 3개의 인터페이스인 (Character, Test, Snake) 리스트와 그들을 구현한 객체 타입이다.

만약 스키마에서 소수의 유니온과 인터페이스만을 가지고 있다면, `possibleTypes`에 수동적으로 명시하면 될 것이다. 하지만, 스키마가 점점 커지고 복잡성이 늘어나면, possibleTypes에 스키마에서 자동으로 가져와 생성해주는 것을 고려해야한다.

## Generating `possibleTypes` automatically

다음 예제는 Graphql 인트로스펙션(클라이언트와 도구가 Graphql 서버의 전체 스키마를 가져올 수 있도록 하는 특수 쿼리) 쿼리를 `possibleTypes`의 객체로 변환하는 스크립트이다.

```javascript
const fetch = require("cross-fetch");
const fs = require("fs");

fetch(`${YOUR_API_HOST}/graphql`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    variables: {},
    query: `
      {
        __schema {
          types {
            kind
            name
            possibleTypes {
              name
            }
          }
        }
      }
    `,
  }),
})
  .then((result) => result.json())
  .then((result) => {
    const possibleTypes = {};

    result.data.__schema.types.forEach((supertype) => {
      if (supertype.possibleTypes) {
        possibleTypes[supertype.name] = supertype.possibleTypes.map(
          (subtype) => subtype.name
        );
      }
    });

    fs.writeFile(
      "./possibleTypes.json",
      JSON.stringify(possibleTypes),
      (err) => {
        if (err) {
          console.error("Error writing possibleTypes.json", err);
        } else {
          console.log("Fragment types successfully extracted!");
        }
      }
    );
  });
```

그리고 나서 possibleTypes를 JSON 모듈로 InMemoryCache를 생성하는 파일로 가져올 수 있다.

```typescript
import possibleTypes from "./path/to/possibleTypes.json";

const cache = new InMemoryCache({
  possibleTypes,
});
```

## useFragment

[여기](https://www.apollographql.com/docs/react/api/react/hooks-experimental/)
