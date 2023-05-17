---
layout: post
title: "Apollo Server - Graphql schema basics"
date: 2023-05-17
tags: [Graphql]
comments: true
---

이 글은 Apollo docs를 읽고 번역한 글 입니다.

## Graphql schema basics

Graphql Server라 함은, 스키마를 사용하여 사용 가능한 데이터의 형태를 말한다. 이 스미카는 백엔드 데이터 저장소에서 채워지는 타입을 가진 필드 계층 구조를 정의한다. 또한 이 스키마는 클라이언트가 실행할 수 있는 Queries, Mutation을 명확하게 지정해준다.

여기서는 스키마의 기본 구성 요소와 Graphql Server용 스키마를 만드는 방법에 대해서 설명한다.

### Schema Definition Language

Graphql 사양은 스키마를 정의하고 이를 문자열로 저장하는 데 사용하는 사람이 읽을 수 있는 스키마 정의 언어를 정의한다.

스키마는 타입의 모음과 이런 카입의 관계들을 정의한다. 그리고 스키마는 데이터가 어디서 오는지, 어떻게 저장되는지는 역할을 하지 않는다. 이는 전적으로 implementation-agnostic(구현에 구애받지않는다.)이다.

### Field Definitions

대부분의 스키마 type들은 하나 또는 그 이상의 field를 정의한다.

```graphql
# This Book type has two fields: title and author
type Book {
  title: String # returns a String
  author: Author # returns an Author
}
```

각 field는 지정된 타입의 데이터를 반환한다. field의 리턴 타입은 `scalar`, `object`, `enum`, `union`, `interface`가 될 수 있다.

### List Fields

field는 특정 타입의 list를 포함할 수 있다. 대괄호로 list field를 표시할 수 있다.

```
type Author {
  name: String
  books: [Book] # A list of Books
}
```

### Field nullability

기본적으로 스키마의 모든 field가 지정된 타입 대신 null을 반환하는 것이 가능하다. 이는 `!`를 사용하여 특정 field에 대해 null을 허용하지 않도록 지정할 수 있다.

```
type Author {
  name: String! # Can't return null
  books: [Book]
}
```

이런 field들은 `non-nullable`하다. 만약 서버가 `non-nullable` field에서 `null`을 리턴하도록 시도한다면, error가 던져질 것이다.

#### Nullability and lists

list field일때, `!`로 두 위치에서 표시할 수 있다.

```
type Author {
  books: [Book!]! # list는 null이 될 수 없고, list내의 item또한 null이 될 수 없다.
}
```

- `!`가 대괄호 안에 표시되어있는 경우, list 내의 요소에서 null을 포함할 수 없다.
- `!`가 대괄호 밖에 표시되어있는 경우, list 자체가 null이 될 수 없다.
- 모든 케이스에서 list field가 빈 list를 리턴할 수 있다.

### Supported types

Graphql 스키마에 있는 모든 타입들은 다음 카테고리에 있다.

- Scalar
- Object (This includes the three special root operation types: Query, Mutation, and Subscription.)
- Input
- Enum
- Union
- Interface

### Scalar types

Scalar 타입은 primitive 타입과 유사하다. 이는 항상 구체적인 타입을 가지고 있다.

- Int: 부호가 있는 32‐bit integer
- Float: 부호가 있는 double-precision floating-point value
- String: A UTF‐8 character sequence
- Boolean
- ID (serialized as a String): 객체를 refetch하거나, 캐시의 키로 주로 사용되는 고유 식별자이다. 문자열로 직렬화되지만 ID 타입의 경우 human-readable을 의도하지는 않았다.

이러한 primitive type들은 대부분 쓰인다. 좀 더 구체적으로 사용하는 경우 custom scalar types를 만드는 것이 가능하다.

### Object types

아마 Graphql Schema에서 너가 정의하게될 대부분의 타입이 object 타입이다. object 타입은 각각 타입을 가지고 있는 fields들의 집합이다.

```
type Book {
  title: String
  author: Author
}

type Author {
  name: String
  books: [Book]
}
```

#### \_\_typename field

모든 schema 내의 object 타입은 자동으로 `__typename` field를 가지고 있다. (정의할 필요 없다.) 이 필드는 이 object 타입의 이름을 String으로 반환한다.

Graphql clients는 object의 `__typename`을 여러 목적으로 사용하는데, 예를 들면 여러 타입을 반환할 수 있는 field에서 어떤 타입을 반환하였는지 확인하는 등의 작업을 수행할 수 있기 때문이다. (union or interface)
Apollo Client는 결과를 캐싱할 때, `__typename`에 의존하기 때문에, 모든 query 객체에서는 `__typename`을 자동적으로 포함하고 있다.

`__typename`이 항상 존재하기 때문에 모든 Graphql 서버에 유효한 쿼리이다.

```
query UniversalQuery {
  __typename
}
```

### Query type

Query 타입은 특수한 Object 타입인데, client에서 server에 실행하는 query들의 최상위 레벨의 entry point를 정의하고 있기 때문이다.

query 타입의 각 field는 서로 다른 entry point와 리턴타입을 정의한다.

```
type Query {
  books: [Book]
  authors: [Author]
}
```

이 Query 타입은 `books`와 `authors`라는 두 가지의 field를 정의하고 있다. 각 field는 해당하는 타입의 리스트를 반환한다.

REST 기반의 API의 경우, `books`와 `authors`는 서로 다른 엔드포인트에서 반환될 가능성이 크다. 이런 Graphql의 유연성 덕분에 client는 한번의 요청으로 두 리소스를 모두 쿼리할 수 있다.

#### Structuring a query

client가 실행할 쿼리를 작성하면, 해당 쿼리는 스키마에서 정의한 타입의 생김새와 일치한다.

client가 모든 books의 title 리스트와 authors의 name 리스트를 요청하는 경우 다음 쿼리를 실행하면 된다.

```
query GetBooksAndAuthors {
  books {
    title
  }

  authors {
    name
  }
}

// 서버에서 주는 값
{
  "data": {
    "books": [
      {
        "title": "City of Glass"
      },
      ...
    ],
    "authors": [
      {
        "name": "Paul Auster"
      },
      ...
    ]
  }
}
```

사실 클라이언트는 위처럼 쿼리를 작성해서 두 가지의 리스트를 가져오는 것보다는 각 books의 author가 포함된 리스트를 가져오는 것을 선호할 것이다.
`Book` 타입에는 `Author` 타입의 field가 존재하므로 다음과 같이 쿼리할 수 있다.

```
query GetBooks {
  books {
    title
    author {
      name
    }
  }
}

// 서버에서 주는 값
{
  "data": {
    "books": [
      {
        "title": "City of Glass",
        "author": {
          "name": "Paul Auster"
        }
      },
      ...
    ]
  }
}
```

### Mutation type

Mutation 타입은 Query 타입과 구조와 목적이 비슷하다. Query 타입은 read operations를 위한 entry point라면, Mutation 타입은 write operations를 위한 entry point를 정의한다.

Mutation 타입의 각 field는 서로 다른 호출, 리턴 타입을 정의한다. Mutation 타입은 다음과 같이 스키마를 정의할 수 있다.

```
type Mutation {
  addBook(title: String, author: String): Book
}
```

이 Mutation 타입은 `addBook`이라는 하나의 Mutation을 정의한다. 이 Mutation은 title과 author를 인자로 받고, Book 객체를 만들어서 새롭게 리턴한다.예상했듯이, Book 객체는 우리가 스키마에 정의한 구조를 따른다.

#### Structuring a mutation

Mutation 또한 스키마의 타입 정의와 구조가 일치한다. 다음 예시의 Mutation은 `Book`을 만들고 생성된 객체의 특정 field를 리턴하도록 한다.

```
mutation CreateBook {
  addBook(title: "Fox in Socks", author: "Dr. Seuss") {
    title
    author {
      name
    }
  }
}

// 서버에서 주는 값
{
  "data": {
    "addBook": {
      "title": "Fox in Socks",
      "author": {
        "name": "Dr. Seuss"
      }
    }
  }
}
```

하나의 mutation의 작동방식은 다수의 최상위 fields의 Mutation 타입을 포함할 수 있다. 이것은 한번의 operation이 여러번 서버에서 writes할 수 있게끔 실행이 가능하다는 것이다. 동시에 실행되는 것을 방지하기 위해 최상위 Mutation field들은 나열된 순서대로 순차적으로 resolve된다.

### Input types

Input 타입은 field에 인자로 계층형 데이터를 제공할 수 있는 특수한 object 타입니다. (flat한 Scalar 인자만 제공하는 것과는 반대)

Input 타입의 정의는 Object 타입과 비슷하며, `input`이라는 키워드를 사용한다.

```
input BlogPostContent {
  title: String
  body: String
}
```

input 타입의 각 field는 Scalar, enum 또는 또다른 input 타입만 가능하다.

```
input BlogPostContent {
  title: String
  body: String
  media: [MediaDetails!]
}

input MediaDetails {
  format: MediaFormat!
  url: String!
}

enum MediaFormat {
  IMAGE
  VIDEO
}
```

input 타입을 정의한 후, 다양한 Object fields에서 이 타입을 인수로 사용할 수 있다.

```
type Mutation {
  createBlogPost(content: BlogPostContent!): Post
  updateBlogPost(id: ID!, content: BlogPostContent!): Post
}
```

input 타입은 동일한 정보의 한 세트가 요구되는 여러 operations에서 유용하게 쓰일 수 있다. 하지만 재사용의 경우 주의해야한다. operations(연산)은 필수적으로 요구되는 인수가 다를 수 있다.

Query와 Mutation에서 같은 input 타입을 쓸 때 주의하여야한다. 많은 경우에 인수들은 mutation에서는 필수적으로 요구되지만 query에서는 옵셔널인 경우가 많다. 그런 경우 각 operation 타입마다 input 타입을 분리해서 작성해야한다.

### Enum types

enum은 scalar 타입과 비슷하지만, 해당 값들은 스키마에 정의되어 있다.

```
enum AllowedColor {
  RED
  GREEN
  BLUE
}
```

enum은 사용자가 정해진 옵션 리스트에서 선택하는 상황일때 제일 유용하다. 또한 apollo studio explorer에서 enum은 자동완성이 된다는 장점이 있다.

enum은 string으로 직렬화되므로, field 인수를 포함해서 scalar가 유효한 모든 곳에 나타날 수 있다.

```
type Query {
  favoriteColor: AllowedColor # enum return value
  avatar(borderColor: AllowedColor): String # enum argument
}
```

```
query GetAvatar {
  avatar(borderColor: RED)
}
```

#### internal values

백엔드에서 public API와 다른 enum 값을 내부적으로 강제하는 경우가 있다. apollo server에서 resolver map 안의 내부 값에 해당하는 enum 값을 설정할 수 있다.... (무슨뜻?)

이 기능은 일반적으로 다른 형식의 enum 값을 기대하는게 아니라면 요구되는 사항은 아니다.

다음 예제는 AllowedColor의 내부 값이다.

```typescript
const resolvers = {
  AllowedColor: {
    RED: "#f00",
    GREEN: "#0f0",
    BLUE: "#00f",
  },
  // ...other resolver definitions...
};
```

이 내부 값들은 public API를 변경하지 않는다. apollo server 리졸버는 이런 값들을 schema 값 대신 허용한다.

```typescript
const resolvers = {
  AllowedColor: {
    RED: "#f00",
    GREEN: "#0f0",
    BLUE: "#00f",
  },
  Query: {
    favoriteColor: () => "#f00",
    avatar: (parent, args) => {
      // args.borderColor is '#f00', '#0f0', or '#00f'
    },
  },
};
```

## Growing with a schema

관리의 범위가 커지게 되면서 이러한 변경 사항을 추적하려면 버전 관리에서 스키마의 정의를 유지, 관리해야한다.

스키마에 대한 추가, 변경은 이전 버전과 호환되미나, 기존 동작을 제거, 변경하는 경우는 기존의 client에 대한 변경 사항을 깨트릴 수 있다.

- type이나 field를 삭제하는 것
- type이나 field를 renaming하는 것
- field에 널러블 유무를 추가하는 것
- field의 인수를 삭제하는 것

apollo studio와 같은 관리도구를 사용하면 client에 스키마 변경이 잠재적인 영향을 미치는지 여부를 파악하기 용이하다. 또한 studio는 field 레벨단의 스키마 기록 추적, 작업 세이프 리스팅 등의 고급 기능을 제공한다.(?)

## Descriptions

그랲큐엘 스키마 정의 언어는 마크다운 도큐먼트 작성을 지원해주는데, descriptions이라고 부른다. 이를 통해 그래프 소비자는 이를 통해 field를 보고 사용방법을 익힐 수 있다.

```graphql
"Description for the type"
type MyObjectType {
  """
  Description for field
  Supports **multi-line** description for your [API](http://example.com)!
  """
  myField: String!

  otherField(
    "Description for argument"
    arg: Int
  )
}
```

문서화가 잘 되어있는 스키마는 좋은 개발환경을 제공하는데 도움을 준다. 이는 apollo studio explorer 같은 개발 도구가 field 이름이 제공될 때 설명과 함께 field이름도 자동으로 완성해주기 때문이다.

## Naming conventions

graphql 사양은 유연하고 특정 네이밍 가이드라인을 강요하지는 않는다. 하지만 일관성 유지를 위해 규칙을 설정하는 것이 좋다.

- field 이름은 camelCase
- type 이름은 PascalCase
- enum 이름은 PascalCase
- enum 값들은 ALL_CAPS

## Query-driven schema design

graphql 스키마는 클라이언트의 요구에 맞게 설계될 때 가장 파워풀하다. 데이터가 저장되는 방식이 아닌, 사용하는 방식을 기준으로 스키마를 설계해야한다.

데이터 저장소에 클라이언트가 아직 필요로하지않는 field가 관계에 포함되어있는 경우, 스키마에서 해당 field를 생략할 수 있다. 클라이언트가 사용중인 field를 제거하는 것보다, 스키마에서 새 필드를 추가하는 것이 더 안전하다.

## Designing Mutations

graphql에서는 모든 mutations의 response에서 mutation에 의해 수정된 데이터를 포함하도록 권장한다. 이렇게하면 클라이언트에서 후속 query를 보낼 필요 없이 최신 데이터를 얻을 수 있다.

```
type Mutation {
  # This mutation takes id and email parameters and responds with a User
  updateUserEmail(id: ID!, email: String!): User
}

type User {
  id: ID!
  name: String!
  email: String!
}

# client
mutation updateMyUser {
  updateUserEmail(id: 1, email: "jane@example.com") {
    id
    name
    email
  }
}

// 서버에서 오는 데이터
{
  "data": {
    "updateUserEmail": {
      "id": "1",
      "name": "Jane Doe",
      "email": "jane@example.com"
    }
  }
}
```

mutation의 response에 수정된 객체를 포함하면 클라이언트 코드의 효율성이 크게 향상된다. query와 마찬가지로 mutation에서 클라이언트에게 유용한지를 고려해서 결정하는 경우 스키마 구조를 파악하는데 도움디 된다.

## Structuring mutation responses

하나의 mutation을 통해서 여러 타입이나 동일한 타입의 여러 인스턴스를 수정할 수 있다. 예를 들면, 사용자가 블로그 게시물에 좋아요를 누르는 경우이다. 이런 mutation은 게시물의 좋아요 수를 늘려야하고, 좋아요 게시물 목록을 업데이트할 수 있는데, 이런 경우에는 mutation의 response 구조가 어떤 모습이어야하는지 명확하지 않다.

mutation은 또한 데이터를 수정하기 때문에 Query보다 error가 일어날 가능성이 높다. mutation을 한 데이터는 성공적으로 수정하고 다른 데이터는 수정하지 못하는 부분 error를 일으킬 가능성도 있다. error 타입과는 관계없이 오류를 일관된 방식으로 클라이언트에게 전달하는 것이 중요하다.

위의 두 가지 문제를 해결하기 위해서는 스키마에 해당 인터페이스를 구현하는 객체 타입 모음과 함께 MutationResponse 인터페이스를 정의하는 것이 좋다.

```
interface MutationResponse {
  code: String!
  success: Boolean!
  message: String!
}
```

그리고 아래와 같이 활용할 수 있다.

```
type UpdateUserEmailMutationResponse implements MutationResponse {
  code: String!
  success: Boolean!
  message: String!
  user: User
}

// 서버에서 들어오는 데이터
{
  "data": {
    "updateUserEmail": {
      "code": "200",
      "success": true,
      "message": "User email was successfully updated",
      "user": {
        "id": "1",
        "name": "Jane Doe",
        "email": "jane@example.com"
      }
    }
  }
}
```

## Reference

- [Apollo Server - schema](https://www.apollographql.com/docs/apollo-server/schema/schema/#the-schema-definition-language)
