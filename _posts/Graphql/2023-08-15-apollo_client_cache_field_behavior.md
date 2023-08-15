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

### 배열 병합하기

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

캐시에 아직 필드에 대한 데이터가 포함되어있지 않기 때문에, 필드의 특정 인스턴스에 대해 이 함수를 처음 호출할 때는 `existing`이 undefined라는 것을 기억하자. `existing = []`을 지정함으로써 기본 파라미터 값을 더 편하게 핸들링할 수 있다.

> `merge` 함수는 incoming 배열을 existing 배열에 곧바로 바꿀 수는 없다. 잠재적인 에러를 피하기 위해 새로운 배열을 대신 리턴하여야한다. 개발모드에서는 아폴로 클라이언트는 `Object.freeze`를 사용하여 기존 데이터의 의도치않은 수정을 피한다.

### 정규화되지 않은 객체들을 병합하기

`merge` 함수를 사용하면, 캐시에서 정규화되지 않은 네스팅된 객체가 동일한 정규화된 상위 객체 내에 중첩되어 있다고 가정하여 지능적으로 중첩된 객체를 결합할 수 있다.

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Book: {
      fields: {
        author: {
          // 정규화되지 않은 Book 안의 Author 객체
          merge(existing, incoming, { mergeObjects }) {
            return mergeObjects(existing, incoming);
          },
        },
      },
    },
  },
});
```

#### 예제

그래프큐엘 스키마로 다음 타입들을 가지고 있다고하자:

```graphql
type Book {
  id: ID!
  title: String!
  author: Author!
}

type Author { # Has no key fields
  name: String!
  dateOfBirth: String!
}

type Query {
  favoriteBook: Book!
}
```

이 스키마에서 Book 객체는 `id` 필드를 가지고 있으므로 캐시를 정규화할 수 있다. 그러나, `Author` 객체는 `id` 필드를 가지고 있지 않고, 특정 인스턴스를 고유하게 식별할 수 있는 다른 필드도 없다. 그러므로, 캐시는 Author 객체의 캐시는 정규화될 수 없고, 두 개의 다른 Author 객체는 실제로 동일한 Author를 나타내는 경우를 구분할 수 없다.

이제, 클라이언트에서 두 개의 쿼리를 실행시켜보자:

```graphql
query BookWithAuthorName {
  favoriteBook {
    id
    author {
      name
    }
  }
}

query BookWithAuthorBirthdate {
  favoriteBook {
    id
    author {
      dateOfBirth
    }
  }
}
```

첫번째 쿼리가 리턴될 때, 아폴로 클라이언트는 캐시에 `Book` 객체를 다음과 같이 기록한다:

```json
{
  "__typename": "Book",
  "id": "abc123",
  "author": {
    "__typename": "Author",
    "name": "George Eliot"
  }
}
```

> Author 객체가 정규화되지 못하는 것을 기억하자. 그들은 상위 객체에서 직접 네스팅된다.

이제, 두번째 쿼리가 리턴될 때, 캐시된 Book 객체는 다음과 같이 업데이트된다:

```json
{
  "__typename": "Book",
  "id": "abc123",
  "author": {
    "__typename": "Author",
    "dateOfBirth": "1819-11-22"
  }
}
```

`Author`의 `name` 필드가 삭제되었다..! 이는 두 쿼리에서 리턴된 `Author` 객체가 실제로 동일한 Author를 참조하는지 아폴로 클라이언트에서 확인할 수 없기 때문이다. 그래서 두 객체의 필드를 병합하는 대신, 아폴로 클라이언트는 객체를 완전히 덮어씌우게 된다.(그리고 워닝을 기록)

그러나, 우리는 이런 두개의 객체를 같은 author로 나타낼 수 있다. 왜냐면 book의 author는 사실상 바뀌지 않기 때문이다. 그러므로, 우리는 동일한 `Book`에 속하는 한 `Book.author` 객체를 동일한 객체로 취급하도록 캐시에게 얘기해줄 수 있다. 이것은 두 개의 다른 것을 리턴하는 쿼리로도 `name`과 `dateOfBirth` 필드를 병합하는 것을 가능케한다.

이걸 할려면, 우리는 `Book`의 타입 정책 안의 `author` 필드의 `merge` 함수를 커스텀하여 정의해야한다:

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Book: {
      fields: {
        author: {
          merge(existing, incoming, { mergeObjects }) {
            return mergeObjects(existing, incoming);
          },
        },
      },
    },
  },
});
```

우리는 `mergeObjects`라는 헬퍼 함수를 사용하여 Author 객체의 `existing`과 `incoming`을 받아서 값들을 머지한다. 스프레드 연산자를 사용해서 객체들을 병합하는 것 대신, `mergeObjects`를 사용하는 것은 중요한데, 왜냐하면 `mergeObjects`는 `Book.author`의 하위 필드들에 대해 정의된 모든 `merge` 함수를 호출하기 때문이다.

`merge` 함수는 `Book` 또는 `Author` 관련 로직이 전혀 포함되어있지 않다는 것을 기억하자. 이것은 비정규화 객체 필드에 얼마든지 재사용할 수 있다는 것을 의미한다. 이 `merge` 함수의 정의는 매우 일반적이므로, 다음과 같은 약어로도 정의할 수 있다:

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Book: {
      fields: {
        author: {
          // options.mergeObjects(existing, incoming)와 같은 의미
          merge: true,
        },
      },
    },
  },
});
```

요약하면, 위의 `Book.author` 정책을 사용하면 캐시가 특정 정규화된 Book 객체와 연결된 모든 Author 객체를 지능적으로 병합할 수 있다.

> 정규화되지 않은 두 객체를 병합하려면 `merge: true`의 경우 다음 조건이 모두 true여야 한다:
>
> - 두 객체는 캐시에서 정확히 동일한 정규화된 객체의 정확히 동일한 필드를 차지해야한다.
> - 두 객체는 같은 `__typename`을 가지고 있어야한다.
>   - 이는 여러 객체 타입들 중 하나를 리턴할 수 있는 인터페이스 또는 유니온 리턴타입이 있는 필드가 있을때 중요하다.
>
> 이런 규칙을 위반하는 동작이 필요한 경우, `merge: true`를 사용하는 대신 커스터마이징한 `merge` 함수를 작성해야한다.

### 비정규화된 객체들에 대한 병합된 배열들

`Book`이 여러 `authors`를 가질 수 있는 상황을 생각해보자:

```graphql
query BookWithAuthorNames {
  favoriteBook {
    isbn
    title
    authors {
      name
    }
  }
}

query BookWithAuthorLanguages {
  favoriteBook {
    isbn
    title
    authors {
      language
    }
  }
}
```

`favoriteBook.authors` 필드에는 정규화되지 않은 Author 객체 목록이 포함되어 있다. 이 경우에는 위의 두 쿼리에서 반환된 `name`과 `language` 필드가 서로 올바르게 연결되도록 보다 정교한 `merge` 함수를 정의해야한다.

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Book: {
      fields: {
        authors: {
          merge(existing: any[], incoming: any[], { readField, mergeObjects }) {
            const merged: any[] = existing ? existing.slice(0) : [];
            const authorNameToIndex: Record<string, number> =
              Object.create(null);
            if (existing) {
              existing.forEach((author, index) => {
                authorNameToIndex[readField<string>("name", author)] = index;
              });
            }
            incoming.forEach((author) => {
              const name = readField<string>("name", author);
              const index = authorNameToIndex[name];
              if (typeof index === "number") {
                // 존재하는 author 데이터와 새로운 author 데이터와 병합하기
                merged[index] = mergeObjects(merged[index], author);
              } else {
                // 이 배열에서 이 author를 처음 볼 때
                authorNameToIndex[name] = merged.length;
                merged.push(author);
              }
            });
            return merged;
          },
        },
      },
    },
  },
});
```

기존 `authors` 배열을 새로운 배열로 대체하는 대신, 이 코드는 배열을 서로 연결하면서 중복된 author의 name도 확인한다. 중복된 name이 발견될 때마다 `Author` 객체의 필드가 병합된다.

`readField` 헬퍼함수는 `author`가 캐시의 다른 곳에 있는 데이터를 참조하는 참조 객체일 가능성을 허용하므로 `author.name`을 직접 사용하는 것보다 더 강력하다. `Author` 타입이 결국 `keyFields`를 정의하여 정규화되는 경우에 중요하다.

이 예제에서 알 수 있듯이 `merge` 함수는 매우 정교해질 수 있다. 이런 경우, 일반 로직을 재사용 가능한 헬퍼함수로 추출할 수 있다:

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Book: {
      fields: {
        authors: {
          merge: mergeArrayByField<AuthorType>("name"),
        },
      },
    },
  },
});
```

이제 재사용 가능한 추상화한 뒤에 세부 사항을 숨겼으므로, 구현이 얼마나 복잡해지는지는 더이상 중요하지 않다. 이렇게하면 클라이언트측 비지니스 로직을 개선하는 동시에 전체 애플리케이션에서 관련 로직을 일관되게 유지할 수 있으므로 자유롭다.

### 타입 레벨에서 `merge` 함수 정의하기

아폴로 클라이언트 3.3 이상부터는 정규화되지 않은 객체 타입에 대한 기본 `merge` 함수를 정의할 수 있다. 이렇게 하면, 필드별로 재정의하지 않는 한 해당 타입을 반환하는 모든 필드에서 기본 `merge` 함수를 사용하게 된다.

타입 정책상에서 비정규화 타입에 대한 기본 `merge` 함수를 선언한다. `비정규화된 객체 병합하기`에 따른 비정규화된 `Author` 타입에 대한 결과는 다음과 같다:

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Book: {
      fields: {
        // 더이상 필수가 아니다.
        // author: {
        //   merge: true,
        // },
      },
    },

    Author: {
      merge: true,
    },
  },
});
```

위에서 보여주듯이, `Book.author`을 위한 필드레벨의 `merge` 함수는 더이상 요구되지 않는다. 이 기본 예제의 최종 결과는 동일하지만, 이 경우 나중에 추가할 수 있는 다른 Author 리턴 필드에 기본 `merge` 함수를 자동으로 적용하게끔 한다.

### 페이지네이션 핸들링

필드가 배열을 가질 때, 배열의 결과를 페이지네이팅하는 것이 유용하다. 왜냐하면 모든 결과는 임의로 커질 수 있기 때문이다.

전형적으로, 쿼리는 명시된 페이지네이션 인자들을 포함한다:

- 숫자 오프셋 또는 시작 ID를 사용하여 배열에서 시작할 위치
- 단일 페이지에 리턴할 최대 요소 수

필드에 페이지네이션을 구현한 경우, 필드에 대한 `read`와 `merge` 함수를 구현할 때 페이지 관련 인수를 염두에 두는 것이 중요하다:

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Agenda: {
      fields: {
        tasks: {
          merge(existing: any[], incoming: any[], { args }) {
            const merged = existing ? existing.slice(0) : [];
            // 인수에 따라 들어오는 요소를 올바른 위치에 삽입
            const end = args.offset + Math.min(args.limit, incoming.length);
            for (let i = args.offset; i < end; ++i) {
              merged[i] = incoming[i - args.offset];
            }
            return merged;
          },

          read(existing: any[], { args }) {
            // 만약 캐시를 기록하기 전에 필드를 읽을 때, 함수는 undefined를 리턴할 것이고,
            // 이는 올바르게 필드가 누락되었음을 나타낸다.
            const page =
              existing && existing.slice(args.offset, args.offset + args.limit);
            // 기존 배열의 범위를 벗어난 페이지를 요청하는 경우, page.length는 0이 될 것이고,
            // 빈 배열 대신 undefined를 리턴해야한다.
            if (page && page.length > 0) {
              return page;
            }
          },
        },
      },
    },
  },
});
```

위 예제에서 보여주듯이, `read` 함수는 종종 동일한 인수를 역방향으로 처리하여 `merge` 함수와 협력해야한다.

주어진 Page가 `args.offset`에서 시작하는 대신 특정 엔티티 ID 다음에 시작되도록 하려면, 다음과 같이 `merge` 및 `read` 함수를 구현하고 `readField` 헬퍼 함수를 사용하여 기존 Task의 ID를 검사할 수 있다:

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Agenda: {
      fields: {
        tasks: {
          merge(existing: any[], incoming: any[], { args, readField }) {
            const merged = existing ? existing.slice(0) : [];
            // 모든 기존 task ID들을 Set으로 가져오기
            const existingIdSet = new Set(
              merged.map((task) => readField("id", task))
            );
            // 기존 데이터에 이미 존재하는 들어오는 task들은 지운다.
            incoming = incoming.filter(
              (task) => !existingIdSet.has(readField("id", task))
            );
            // 들어오는 task 페이지 바로 앞에 있는 task의 인덱스를 찾기
            const afterIndex = merged.findIndex(
              (task) => args.afterId === readField("id", task)
            );
            if (afterIndex >= 0) {
              // afterIndex를 찾으면 해당 인덱스 뒤에 들어온 값들을 삽입
              merged.splice(afterIndex + 1, 0, ...incoming);
            } else {
              // 그렇지 않으면 기존 데이터 끝에 들어온 데이터를 삽입
              merged.push(...incoming);
            }
            return merged;
          },

          read(existing: any[], { args, readField }) {
            if (existing) {
              const afterIndex = existing.findIndex(
                (task) => args.afterId === readField("id", task)
              );
              if (afterIndex >= 0) {
                const page = existing.slice(
                  afterIndex + 1,
                  afterIndex + 1 + args.limit
                );
                if (page && page.length > 0) {
                  return page;
                }
              }
            }
          },
        },
      },
    },
  },
});
```

`readField(fieldName)`를 호출하면 현재 객체에서 지정된 필드의 값을 리턴하는 것을 기억하자. 객체를 `readField`에 두 번째 인수로 전달하면(`readField('id', task)`), readField는 대신 지정된 객체에서 지정된 필드를 읽는다. 위 예제에서 `id` 필드를 기존 `Task` 객체에서 읽을 때 들어오는 task 데이터의 중복을 제거할 수 있다.

위 페이지네이션 코드는 복잡하지만, 페이지네이션 전략을 구현한 후에는 필드 타입에 관계없이 해당 전략을 사용하는 모든 필드에 재사용할 수 있다. 예를 들어:

```typescript
function afterIdLimitPaginatedFieldPolicy<T>() {
  return {
    merge(existing: T[], incoming: T[], { args, readField }): T[] {
      ...
    },
    read(existing: T[], { args, readField }): T[] {
      ...
    },
  };
}

const cache = new InMemoryCache({
  typePolicies: {
    Agenda: {
      fields: {
        tasks: afterIdLimitPaginatedFieldPolicy<Reference>(),
      },
    },
  },
});
```

### `merge` 함수 비활성화하기

어떤 경우에는, 특정 필드에 대한 `merge` 함수를 완전히 비활성화해야할 수도 있다. 그렇게 하기 위해서는 `merge: false`를 다음과 같이 전달한다:

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Book: {
      fields: {
        // 더이상 불필요하다!
        // author: {
        //   merge: true,
        // },
      },
    },

    Author: {
      merge: false,
    },
  },
});
```

### key 인수 지정

필드가 인수를 허용하는 경우, 필드의 `FieldPolicy`에 `keyArgs` 배열을 지정할 수 있다. 이 배열은 필드의 리턴 값에 영향을 주는 key arguments가 어떤 것인지 나타낸다. 이 배열을 지정하면 캐시의 중복 데이터 양을 줄이는데 도움이 될 수 있다.

#### 예제

스키마 Query 타입에 `monthForNumber` 필드가 포함되어 있다고 해보자. 이 필드는 제공된 number 인수가 주어지면 특정 month의 세부 정보를 리턴한다. `number` 인자는 이 필드의 핵심 인수이다:

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Query: {
      fields: {
        monthForNumber: {
          keyArgs: ["number"],
        },
      },
    },
  },
});
```

키가 아닌 인수의 예로는 쿼리를 승인하는 데는 사용되지만 결과를 계산하는데는 사용되지 않는 access token이 있다.`monthForNumber`가 `accessToken` 인수도 허용하는 경우, 해당 인수의 값은 어떤 월의 세부 정보가 반환되는지 영향을 주지 않는다.

기본적으로 필드의 모든 인수는 key arguments가 된다. 즉, 캐시는 특정 필드를 쿼리할 때 사용자가 제공하는 모든 고유한 인수의 값 조합에 대해 별도의 값을 지정한다.

필드의 key arguments를 지정하면, 캐시는 해당 필드의 나머지 인수가 key arguments가 아닌 것으로 이해한다. 이것은 key arguments가 아닌 인수가 변경될 때 캐시가 완전히 별도의 값을 지정할 필요가 없다는 것을 의미한다.

예를 들면, `number` 인수는 같지만 `accessToken` 인수는 다른 `monthForNumber` 필드를 사용하여 서로 다른 두 개의 쿼리를 실행한다고 하자. 이 경우 두 호출 모두 유일한 key argument에 동일한 값을 사용하기 때문에 두 번째 쿼리 응답이 첫 번째 쿼리 응답을 덮어씌우게 된다.

### `keyArgs` 함수 제공하기

만약 특정 필드의 `keyArgs`를 더 세밀하게 제어해야하는 경우 인수 이름 배열 대신 함수를 전달할 수 있다. 이 `keyArgs` 함수는 두 개의 매개변수를 받는다:

- 필드에 대한 모든 argument 값을 포함하는 `args` 객체
- 기타 세부 정보를 제공하는 `context` 객체

자세한건 `KeyArgsFunction`의 [문서](https://www.apollographql.com/docs/react/caching/cache-field-behavior/#fieldpolicy-api-reference) 참고~

## `FieldPolicy` API reference
