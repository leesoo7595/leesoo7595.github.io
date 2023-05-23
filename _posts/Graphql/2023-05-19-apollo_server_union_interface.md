---
layout: post
title: "Apollo Server - Union & Interface"
date: 2023-05-19
tags: [Graphql]
comments: true
---

이 글은 Apollo docs를 읽고 번역한 글 입니다.

## Union and Interface

추상 스키마 타입!

Union, Interface 여러 object 타입으로 리턴할 수 있는 추상 Graphql 스키마 타입이다.

### Union

```
union Media = Book | Movie
```

field는 이 union 타입으로 리턴을 할 수 있다.

```
type Query {
  allMedia: [Media] # This list can include both Book and Movie objects
}
```

### Querying a Union

클라이언트는 어떤 object 타입의 필드가 union으로 리턴된다는 사실을 알 수 없다. 이를 고려하기 위해 쿼리에는 가능한 여러 타입의 하위 field가 포함될 수 있다.

```
query GetSearchResults {
  search(contains: "Shakespeare") {
    # Querying for __typename is almost always recommended,
    # but it's even more important when querying a field that
    # might return one of multiple types.
    __typename
    ... on Book {
      title
    }
    ... on Author {
      name
    }
  }
}

// 서버에서 가져오는 데이터
{
  "data": {
    "search": [
      {
        "__typename": "Book",
        "title": "The Complete Works of William Shakespeare"
      },
      {
        "__typename": "Author",
        "name": "William Shakespeare"
      }
    ]
  }
}
```

인라인 fragments를 사용하여 Book 타입의 title이나, Author 타입의 name을 가져올 수 있다.

### Resolving a union

Union을 리졸브하려면, Apollo Server에서 어떤 Union 타입을 리턴해야할지 특정해주어야한다. 이를 위해서는 `__resolveType` 함수를 정의해주어야한다.

`__resolveType` 함수는 객체의 타입을 결정하고, 해당 타입의 이름을 문자열로 리턴하는 역할을 한다. 이런식으로 사용하기 위해서는 다음의 로직을 사용할 수 있는데, 예를들면

- union에서 특정 타입에 고유한 field 유무 확인
- instanceof를 사용해서 자바스크립트 객체의 타입이 Graphql 객체 타입과 관련되어 있을 때

아래는 SearchResult union에 대한 `__resolveType` 함수이다.

```typescript
const resolvers = {
  SearchResult: {
    __resolveType(obj, contextValue, info){
      // Only Author has a name field
      if(obj.name){
        return 'Author';
      }
      // Only Book has a title field
      if(obj.title){
        return 'Book';
      }
      return null; // GraphQLError is thrown
    },
  },
  Query: {
    search: () => { ... }
  },
};
```

`__resolveType` 함수가 유효한 타입의 이름이 아닌 값을 반환하는 경우는 Graphql error를 생성한다.

### Interface

인터페이스는 여러 객체 타입이 포함할 수 있는 field의 집합을 지정할 수 있다.

만약 객체 타입이 인터페이스를 `implements`한다면, 필드에 해당하는 모든 인터페이스를 포함해야한다.

```graphql
interface Book {
  title: String!
  author: Author!
}

type Textbook implements Book {
  title: String! # Must be present
  author: Author! # Must be present
  courses: [Course!]!
}

type Query {
  books: [Book!]! # Textbook 타입 객체도 포함할 수 있다.
}
```

하나의 field는 하나의 인터페이스(또는 하나의 인터페이스 리스트)를 리턴 타입으로 가질 수 있다. 인터페이스 타입으로 field 지정이 되는 경우, 해당 인터페이스를 가지고 있는 모든 객체 타입을 반환할 수 있다.

### Querying an interface

field 리턴 타입이 인터페이스인 경우, 클라이언트에서는 해당 field에서 인터페이스에 포함된 모든 하위 필드를 쿼리할 수 있다.

### Resolving an interface

인터페이스 또한 `__resolveType` 함수를 정의해야한다.

## Reference

- [Apollo Server - schema : unions-interfaces](https://www.apollographql.com/docs/apollo-server/schema/unions-interfaces/)
