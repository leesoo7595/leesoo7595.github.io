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

## Reference

- [Apollo Server - schema](https://www.apollographql.com/docs/apollo-server/schema/schema/#the-schema-definition-language)
