---
layout: post
title: "Apollo Client - Cache Configuration"
date: 2023-08-03
tags: [Graphql]
comments: true
---

이 글은 [아폴로 클라이언트 공식문서](https://www.apollographql.com/docs/react/caching/cache-configuration)를 읽고 정리한 글이다.

# 아폴로 클라이언트 캐시 설정하기

## 초기세팅

`InMemoryCache` 객체를 생성하고 ApolloClient 생성자에 넣어준다:

```typescript
import { InMemoryCache, ApolloClient } from "@apollo/client";

const client = new ApolloClient({
  // ...other arguments...
  cache: new InMemoryCache(options),
});
```

`InMemoryCache` 생성자는 많은 설정옵션들을 받는다.

## 설정 옵션

애플리케이션에 더 적합한 캐시의 동작을 설정할 수 있다. 예를 들면:

- 특정 타입의 캐시ID 형태를 커스터마이징
- 각각 필드의 저장 및 검색 커스터마이징
- 프래그먼트 매칭을 위한 다형성 타입 관계 정의
- 페이지네이션 패턴 디자인
- 클라이언트 사이드의 로컬 상태 관리하기

캐시 동작을 커스터마이징하기 위해서는, 설정 객체를 `InMemoryCache` 생성자에 넘겨주어야한다. 이 객체는 다음 필드를 제공한다:

- `addTypeName`: boolean 타입이고, true인 경우 캐시는 자동으로 `__typename` 필드를 모든 객체에서 요청하므로, operation 정의에서 `__typename`을 생략할 수 있다. 기본적으로 캐시는 캐시된 모든 객체에 대한 캐시ID의 일부로 \_\_typename 필드를 사용하므로, 이 필드를 항상 가져오도록 하는 것이 좋다. 기본값은 true이다.
- `resultCaching`: boolean 타입이고, true인 경우 캐시는 기초데이터가 변경되지 않는 한 동일한 쿼리를 실행할 때마다 동일한 응답 객체를 반환한다. 이를 통해 쿼리 결과의 변경사항을 감지할 수 있따. 기본 값은 true이다.
- `resultCacheMaxSixe`: number 타입이고, 캐시에 대한 반복 읽기 속도를 높이기 위해 메모리에 유지되는 result 객체수의 제한이다. 기본 값은 2의 16제곱이다.
- `possibleTypes`: 이 객체를 포함하면 스키마 유형 간의 다형성 관계를 정의할 수 있다. 이걸 사용하면 인터페이스나 유니언 별로 캐시된 데이터를 조회할 수 있다. 객체의 각 키값은 인터페이스나 유니언의 `__typename`이고, 해당 값은 해당 유니언이나 인터페이스를 구현하는 타입의 `__typename`의 배열이다.
- `typePolicies`: 이 객체를 포함하면 타입별로 캐시의 동작을 커스텀할 수 있다. 이 객체의 각 키는 커스텀할 타입의 `__typename`이고, 해당 값은 TypePolicy 객체이다.
- `dataIdFromObject`: 이 함수는 응답 객체를 받아서 스토어에서 데이터를 정규화할 떄 사용할 고유 식별자를 리턴하는 함수이다. 자세한건 [여기](https://www.apollographql.com/docs/react/caching/cache-configuration/#customizing-identifier-generation-globally)를 보세용

## 캐시 ID 커스터마이징하기

스키마에서 각각 타입에 대한 `InMemoryCache`에서 캐시ID를 생성하는 방법을 커스터마이징할 수 있다. 이것은 특히 타입이 `id` 또는 `_id` 외에 다른 필드를 고유 식별자로 사용하는 경우에 유용하다.

이것을 하려면, 커스터마이징하고 싶은 각 타입의 `TypePolicy`를 정의해야한다. `InMemoryCache` 생성자에 제공하는 옵션 객체에서 캐시의 모든 `typePolicies`를 지정한다.

다음과 같이 관련된 `TypePolicy` 객체에 `keyFields` 필드를 포함하자:

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Product: {
      // In an inventory management system, products might be identified
      // by their UPC.
      keyFields: ["upc"],
    },
    Person: {
      // In a user account system, the combination of a person's name AND email
      // address might uniquely identify them.
      keyFields: ["name", "email"],
    },
    Book: {
      // If one of the keyFields is an object with fields of its own, you can
      // include those nested keyFields by using a nested array of strings:
      keyFields: ["title", "author", ["name"]],
    },
    AllProducts: {
      // Singleton types that have no identifying field can use an empty
      // array for their keyFields.
      keyFields: [],
    },
  },
});
```

이 예제는 다른 `keyFields`를 가진 다양한 `typePolices`를 보여준다.

- Product 타입은 `upc` 필드를 식별필드로 사용하였다.
- Person 타입은 `name`과 `email` 필드의 조합을 사용하였다.
- Book 타입은 캐시Id에 부분적으로 서브필드를 포함시켰다.

  - `["name"]` 항목은 배열의 이전 필드(`author`)의 `name` 필드가 캐시ID의 일부임을 나타낸다. 이 항목이 유효하려면 책의 `author` 필드가 `name` 필드를 포함하는 객체여야 한다.
  - Book 타입의 유효한 캐시ID는 다음 구조를 갖는다.

  ```
  Book:{"title":"Fahrenheit 451","author":{"name":"Ray Bradbury"}}
  ```

- `AllProducts` 타입은 싱글톤 타입에 대한 특별한 전략을 보여준다. 캐시에 하나의 `AllProducts` 객체만 포함되고 해당 객체에 식별필드가 없는 경우 해당 객체의 keyFields에 빈 배열을 제공할 수 있다.

하나의 객체가 여러개의 `keyFields`를 가진다면, 캐시ID는 항상 고유성을 보장하기 위해 동일한 순서로 해당 필드들을 나열한다.

이 `keyFields` 문자열은 항상 스키마에 정의된 표준 필드 이름을 참조한다는 것을 매모하자. 즉, ID 계산은 필드 별칭에 민감하지 않다.

### 객체의 캐시ID 계산

만약 여러 필드를 사용하여 커스텀 캐시ID를 정의했다면, 해당 ID를 계산하여 이를 필요로 하는 메소드를 제공하는 것이 어려울 수 있다.

이것을 돕기 위해 `cache.identify` 메소드를 사용하여 캐시에서 가져오는 정규화된 객체의 캐시ID를 계산할 수 있다. [객체의 커스텀 ID 가져오기](https://www.apollographql.com/docs/react/caching/cache-interaction/#obtaining-an-objects-cache-id)를 참고.

### 전역으로 커스텀 식별자 생성

`__typename`에 특정되지 않는 단일 `keyFields` 함수를 정의하는 경우, 아폴로 클라이언트 2.X에 도입된 `dataIdFromObject` 함수를 사용할 수 있다.

```typescript
import { defaultDataIdFromObject } from "@apollo/client";

const cache = new InMemoryCache({
  dataIdFromObject(responseObject) {
    switch (responseObject.__typename) {
      case "Product":
        return `Product:${responseObject.upc}`;
      case "Person":
        return `Person:${responseObject.name}:${responseObject.email}`;
      default:
        return defaultDataIdFromObject(responseObject);
    }
  },
});
```

`dataIdFromObject` API는 2.x에서 쉽게 전환할 수 있도록 아폴로 클라이언트 3에도 포함되어 있다.

위 함수는 다른 로직을 사용해서 객체의 `__typename`에 따라 키를 생성한다. 이 같은 경우, `typePolicies`를 통해서 Product 및 Person 타입에 대한 `keyFields` 배열을 정의해야한다.

이 코드는 다음과 같은 단점이 있다:

- 별칭 실수에 민감
- 정의되지 않은 객체 속성에 대한 보호는 수행하지 않는다.
- 우연히 다른 시간에 다른 키 필드를 사용하면 캐시에서 불일치가 발생할 수 있다.

### 정규화 비활성화

특정 타입에 대해 객체들을 정규화하지 않도록 `InMemoryCache`에 가르쳐줄 수 있다. 이는 타임스탬프로 식별되거나 업데이트를 받지 않는 일시적인 데이터에 유용할 수 있다.

타입에 대해 정규화를 비활성화하려면, 해당 타입에 대한 `TypePolicy`를 정의하고 keyFields 필드를 false로 세팅한다.

정규화되지 않은 객체는 대신 캐시의 부모 객체 내에 포함되게 된다. 이러한 객체에 직접 접근할 수는 없지만 부모 객체를 통해 접근이 가능하다.

## TypePolicy 필드

스키마에서 특정 타입과 캐시가 상호작용하는 것을 커스텀하려면, `__typename` 문자열을 `TypePolicy` 객체에 매핑하는 객체를 `InMemoryCache` 생성자에 전달하면 된다.

`TypePolicy` 객체는 다음 필드를 포함한다.

```typescript
type TypePolicy = {
  // 이 타입에 대한 기본 키 필드를 정의할 수 있다.
  // 필드 이름의 배열, 임의의 문자열을 반환하는 함수 또는 false를 설정하여 이 타입의 객체에 대한 정규화를 비활성화할 수 있다.
  keyFields?: KeySpecifier | KeyFieldsFunction | false;

  // 스키마에서 root 쿼리에 커스텀 __typename을 사용하는 경우이거나,
  // Mutation 또는 Subscription 타입을 사용할 때 그에 해당하는 필드를 true로 설정하여
  // 이 타입이 해당 타입으로 사용됨을 나타낸다.
  queryType?: true;
  mutationType?: true;
  subscriptionType?: true;

  fields?: {
    [fieldName: string]:
      | FieldPolicy<StoreValue>
      | FieldReadFunction<StoreValue>;
  };
};

// 재귀적 타입 별칭은 TypeScript 3.7에서 제공된다.
// 이건 실제 타입은 아니지만 이렇게 해주어야한다.
type KeySpecifier = (string | KeySpecifier)[];

type KeyFieldsFunction = (
  object: Readonly<StoreObject>,
  context: {
    typename: string;
    selectionSet?: SelectionSetNode;
    fragmentMap?: FragmentMap;
  }
) => string | null | void;
```

### root 동작 타입 오버라이딩하기 (일반적이지 않음)

`keyFields` 외에도 TypePolicy는 쿼리, 뮤테이션, 구독 타입을 true로 설정하여 해당 타입이 루트쿼리, 뮤테이션, 구독 타입임을 나타낼 수 있다.

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    UnconventionalRootQuery: {
      // 캐시가 __typename을 알고 있는 경우에만 RootQueryFragment가 일치할 수 있다.
      queryType: true,
    },
  },
});

const result = cache.readQuery({
  query: gql`
    query MyQuery {
      ...RootQueryFragment
    }
    fragment RootQueryFragment on UnconventionalRootQuery {
      field1
      field2 {
        subfield
      }
    }
  `,
});

const equivalentResult = cache.readQuery({
  query: gql`
    query MyQuery {
      field1
      field2 {
        subfield
      }
    }
  `,
});
```

캐시는 주로 **typename 정보를 포함하여 서버에서 쿼리를 받아올때 **typename 정보를 얻는다. 기술적으로 모든 operation에서 가장 바깥쪽 선택 집합에 대해 동일한 방법을 사용할 수 있지만, \_\_typename은 거의 항상 쿼리와 뮤테이션이므로 캐시는 TypePolicy에서 따로 지정하지 않는 한 이러한 일반적인 기본값을 가정한다.

대부분 객체들은 **typename 필드가 적절한 정규화와 식별화에 필수적이다. root 쿼리와 뮤테이션 타입에서는 **typename이 그렇게 유용하거나 중요하지 않다. 왜냐하면 그 타입들은 각 클라이언트마다 하나의 인스턴스만 가지는 싱클톤이기 때문이다.

### fields 프로퍼티

TypePolicy의 마지막 프로퍼티는 fields 프로퍼티이다. 이 프로퍼티는 각각 캐시필드의 동작을 커스텀할 수 있도록 해준다.
