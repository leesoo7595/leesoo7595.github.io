---
layout: post
title: "Apollo Client - Customizing the behavior of cached fields"
date: 2023-08-15
tags: [Graphql]
comments: true
---

이 글은 [아폴로 클라이언트 공식문서](https://www.apollographql.com/docs/react/caching/cache-field-behavior)를 읽고 정리한 글이다.

# 캐시된 필드의 동작 커스터마이징하기

아폴로 클라이언트 캐시에서 특정 필드를 읽고 쓰는 동작을 커스터마이징할 수 있다.
그렇게 하기 위해서는, 해당 필드에 대한 `field policy(필드 정책)`를 정의해주어야 한다. 필드 정책은 다음을 포함할 수 있다:

- 해당 필드의 캐시된 값을 읽을때 어떤 일이 발생하는지 지정하는 `read` 함수
- 해당 필드의 캐시된 값을 쓸 때 어떤 일이 발생하는지 지정하는 `merge` 함수
- 캐시가 불필요한 중복된 데이터 저장하는 것을 피하도록 도와주는 [key arguments](https://www.apollographql.com/docs/react/caching/cache-field-behavior/#specifying-key-arguments)의 배열

필드 정책들을 `InMemoryCache` 생성자에 넘겨준다. 각 필드 정책은 필드의 상위 타입에 해당하는 `TypePolicy` 객체 내에 정의된다.

다음 예제는 `Person` 타입의 `name` 필드의 필드정책을 정의한다:

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Person: {
      fields: {
        name: {
          read(name) {
            // 캐시된 name 필드를 upper case로 바꿔서 리턴
            return name.toUpperCase();
          },
        },
      },
    },
  },
});
```

이 필드 정책은 `Person.name`이 쿼리될 때 리턴되는 캐시 값을 리턴하는 내용을 지정하는 `read` 함수를 정의했다.

## `read` 함수

필드의 `read` 함수를 정의하면, 클라이언트에서 해당 필드를 쿼리할 때 캐시는 그 함수를 호출한다. 쿼리 응답으로 필드는 캐시된 값 대신 `read` 함수의 리턴 값을 나타낸다.

모든 `read` 함수는 두 개의 파라미터를 전달한다:

- 첫번째 인자로는 필드의 현재 캐시된 값이다(존재했을때). 이 값을 통해 좀 더 유용하게 리턴값을 계산할 수 있다.
- 두번째 인자로는 필드에 전달되는 인수를 포함해서 여러 프로퍼티와 헬퍼함수에 대한 접근을 제공하는 객체이다.
  - `FieldFunctionOptions` 타입의 필드들을 확인하려면 `FieldPolicy`의 [API 문서](https://www.apollographql.com/docs/react/caching/cache-field-behavior/#fieldpolicy-api-reference)를 참고하기

다음 `read` 함수는 `Person` 타입의 `name` 필드가 캐시를 가지고 있지 않을 때, 기본 값으로 `UNKNOWN NAME`을 리턴한다. 캐시된 값이 있으면 수정되지 않은 상태로 리턴된다.

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Person: {
      fields: {
        name: {
          read(name = "UNKNOWN NAME") {
            return name;
          },
        },
      },
    },
  },
});
```

### 필드 인자들 핸들링하기

필드에서 인수를 허용하는 경우, `read` 함수의 두번째 파라미터로는 해당 인수에 제공된 값이 들어있는 `args` 객체가 포함된다.

예를 들면, 다음 `read` 함수는 `name` 필드에 `maxLength` 인자가 제공되었는지 유무를 체크한다. 만약 있다면, 함수는 `maxLength`의 첫번째 문자를 `Person`의 `name`으로 리턴한다. 그렇지 않으면 `Person`의 `name`이 리턴된다.

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Person: {
      fields: {
        // 필드의 TypePolicy가 read 함수만 포함하면,
        // 이전 예제에서 보았듯이 객체 안에 네스팅하는 대신
        // 함수를 옵셔널하게 정의할 수 있다.
        name(name: string, { args }) {
          if (args && typeof args.maxLength === "number") {
            return name.substring(0, args.maxLength);
          }
          return name;
        },
      },
    },
  },
});
```

필드에 여러 매개변수가 필요한 경우, 각 매개변수를 변수로 래핑한 다음 디스트럭쳐링되어 리턴되어야한다. 각 파라미터는 각각의 서브필드로 사용가능하다.

다음 `read` 함수는 `fullName`의 하위필드인 `firstName`의 기본 값으로 `UNKNOWN FIRST NAME`을 할당하고, `fullName`의 하위필드인 `lastName`의 기본 값으로 `UNKNOWN LAST NAME`를 할당한다.

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Person: {
      fields: {
        fullName: {
          read(
            fullName = {
              firstName: "UNKNOWN FIRST NAME",
              lastName: "UNKNOWN LAST NAME",
            }
          ) {
            return { ...fullName };
          },
        },
      },
    },
  },
});
```

다음 쿼리는 `fullName` 필드의 하위필드인 `firstName`과 `lastName`을 리턴한다:

```graphql
query personWithFullName {
  fullName {
    firstName
    lastName
  }
}
```

스키마에 정의되지않은 필드의 `read` 함수를 정의할 수도 있다. 예를 들면, 다음 `read` 함수를 통해 항상 로컬에 저장된 데이터로 채워지는 `userId` 필드를 쿼리할 수 있게 한다:

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Person: {
      fields: {
        userId() {
          return localStorage.getItem("loggedInUserId");
        },
      },
    },
  },
});
```

> 로컬에 정의된 값으로만 필드를 쿼리하는 경우, `@client` 디렉티브를 쿼리에 포함해서 서버에서 가져오는 값이 아니라는 것을 아폴로 클라이언트에게 표시하여야 한다.

`read` 함수는 다른 사용 케이스도 포함한다:

- 부동소수점 값을 가장 가까운 정수로 반올림하는 등의 클라이언트 요구에 맞게 캐시된 데이터를 변환한다.
- 동일한 객체에 있는 하나 이상의 스키마 필드에서 local-only 필드를 파생한다.(예. birthDate 필드에서 연령 필드를 파생하기)
- 여러 객체에서 하나 이상의 스키마 필드를 통해 local-only 필드를 파생한다.

`read` 함수의 모든 옵션 리스트를 보고 싶으면 [여기](https://www.apollographql.com/docs/react/caching/cache-field-behavior/#fieldpolicy-api-reference)를 참고하기. 이 옵션들이 거의다 필요하지 않을 것 같지만, 캐시에서 필드를 읽어올때 각각은 중요한 역할을 한다.

## `merge` 함수

필드의 `merge` 함수를 정의하면, 캐시에서 서버에서 들어오는 값이 기록될 때마다 해당 함수를 호출한다. 기록될 때, 필드는 새로운 값을 들어온 값 대신 `merge` 함수의 리턴된 값을 통해 기록된다.

### merging arrays

`merge` 함수의 일반적인 사용사례는 배열을 가지고 있는 필드를 어떻게 쓸 것인지를 정의할 때이다. 기본적으로, 해당 필드의 기존 배열은 들어온 필드로 완전히 대체된다. 대부분의 경우 다음과 같이 두 배열을 연결해서 사용하는 것이 좋다:

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Agenda: {
      fields: {
        tasks: {
          merge(existing = [], incoming: any[]) {
            return [...existing, ...incoming];
          },
        },
      },
    },
  },
});
```

> 이 패턴은 특히 페이징된 리스트일 떄 일반적으로 사용된다
