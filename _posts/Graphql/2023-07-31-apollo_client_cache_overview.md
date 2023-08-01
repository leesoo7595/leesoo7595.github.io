---
layout: post
title: "Apollo Client - Cache Overview"
date: 2023-08-02
tags: [Graphql]
comments: true
---

이 글은 [아폴로 클라이언트 공식문서](https://www.apollographql.com/docs/react/caching/overview)를 읽고 정리한 글이다.

# Caching in Apollo Client

아폴로 클라이언트는 Graphql 쿼리들의 결과를 정규화된 로컬 인메모리 캐시에 저장한다. 이 방식은 네트워크 요청을 보내지 않고도 아폴로 클라이언트가 이미 캐시된 데이터로 쿼리를 즉시 응답할 수 있도록 해준다.

예를 들면, 처음 앱에서 GetBook 쿼리를 통해 id가 5인 Book 객체를 가져오면, 흐름은 이렇게 된다.

<img width="762" alt="스크린샷 2023-07-30 오후 2 13 16" src="https://github.com/leesoo7595/leesoo7595.github.io/assets/39291812/1f08894a-1838-42e1-af0b-e7606d85cb06">

그리고 그 후에 앱에서 GetBook 쿼리를 통해 같은 객체를 불러오려면 흐름은 이런 식으로 이루어진다.

<img width="723" alt="스크린샷 2023-07-30 오후 2 15 20" src="https://github.com/leesoo7595/leesoo7595.github.io/assets/39291812/2c4e242c-29c3-41ce-a23d-8e0f088c0cf1">

아폴로 클라이언트 캐시는 고급 구성을 할 수 있다. 스키마에서 각각 필드와 타입에 대한 동작을 커스터마이징 할 수 있고, Graphql 서버에서 가져오지 않은 로컬 데이터를 저장하고 상호작용이 가능하다.

## 데이터는 어떻게 저장되는가?

아폴로 클라이언트의 `InMemoryCache`는 서로 참조할 수 있는 `flat lookup table`로 데이터를 저장한다. 이 객체들은 Graphql 쿼리에서 리턴되는 객체와 같다. 여러 쿼리가 동일한 객체의 서로 다른 필드을 가져오는 경우, 하나의 캐시된 객체는 여러 쿼리에서 리턴된 필드가 포함될 수 있다.

캐시는 평평하지만, 객체는 graphql 쿼리로 리턴돼서 평평하지 않은 경우가 많다! 실제로 네스팅은 깊게 설정될 수 있다. 다음 쿼리 예시를 보자

```json
{
  "data": {
    "person": {
      "__typename": "Person",
      "id": "cGVvcGxlOjE=",
      "name": "Luke Skywalker",
      "homeworld": {
        "__typename": "Planet",
        "id": "cGxhbmV0czox",
        "name": "Tatooine"
      }
    }
  }
}
```

이 응답은 Person 객체를 포함하고, 이 객체의 `homeworld` 필드에 Planet 객체가 포함되어 있다.

그래서 `InMemoryCache`는 이 네스팅된 데이터를 어떻게 `flat lookup table`로 바꾸는 것일까? 이 데이터를 저장하기 전에 캐시는 정규화를 먼저 해야한다.

## 데이터 정규화

아폴로 클라이언트 캐시는 쿼리로부터 데이터를 받아올 때, 아래를 따른다:

### 1. 객체 식별하기

먼저, 캐시는 쿼리 응답에 포함된 모든 고유 객체들을 식별한다. 위 예제를 보면, 두 개의 객체가 있다:

- id가 `cGVvcGxlOjE=`인 Person 객체
- id가 `cGxhbmV0czox`인 Planet 객체

### 2. 캐시 ID 생성하기

모든 객체를 식별하고 난 후, 캐시는 각각 캐시ID를 생성한다. 캐시 ID는 `InMemoryCache`에서 특정 객체들이 있는 동안 유니크하게 식별한다.

기본적으로, 객체의 캐시ID는 콜론(:)으로 구분된 객체의 `__typename` 및 `id(_id)` 필드를 조합한 것이다.

그래서 위 예제를 예로 들어서 객체의 기본적인 캐시ID는 아래와 같다:

- `Person:cGVvcGxlOjE=`
- `Person:cGxhbmV0czox`

특정 객체 타입의 캐시ID 포맷을 커스터마이징할 수도 있다. [여기](https://www.apollographql.com/docs/react/caching/cache-configuration/#customizing-cache-ids)보기

만약 캐시가 특정 객체에 대해서 캐시ID를 생성하지 못할 때(예를 들면, `id`, `_id` 필드가 존재하지 않을 때), 그 객체는 부모 객체 안에 바로 캐시되고, 그 객체는 무조건 부모 객체를 통해 참조하여야 한다.(이 것은 캐시가 완전히 평평하지 않음을 의미한다.)

### 3. 객체 필드를 참조로 교체하기

그 다음은, 캐시는 객체가 포함된 각 필드를 가져와 해당 값을 적절한 객체에 대한 참조로 바꾼다. 예를 들면, 참조 교체 전 위 예제의 Person 객체는 다음과 같다.

```json
{
  "__typename": "Person",
  "id": "cGVvcGxlOjE=",
  "name": "Luke Skywalker",
  "homeworld": {
    "__typename": "Planet",
    "id": "cGxhbmV0czox",
    "name": "Tatooine"
  }
}
```

그리고 여기는 같은 객체가 교체된 이후이다.

```json
{
  "__typename": "Person",
  "id": "cGVvcGxlOjE=",
  "name": "Luke Skywalker",
  "homeworld": {
    "__ref": "Planet:cGxhbmV0czox"
  }
}
```

`homeworld` 필드는 이제 Planet 객체를 적절하게 정규화한 참조값을 가지고 있다.

이 교체(replacement) 작업은 이전 단계에서 해당 객체에 대한 캐시ID를 생성하지 못한 경우의 특정 객체에는 수행되지 않는다. 대신 원래 객체는 그대로 유지된다.

그 후에, 만약 같은 `homeworld`를 가지고 있는 또 다른 Person 객체를 쿼리하게 된다면, 정규화된 Person 객체는 캐시된 같은 객체의 참조를 포함하게 될 것이다. 정규화는 극적으로 데이터 중복을 감소시킬 수 있고, 로컬 데이터를 서버와 같은 값인 최신 상태로 유지하는 데 도움을 준다.

### 4. 정규화된 객체들을 저장하기

마지막으로, 이 객체들을 캐시의 flat lookup table에 모두 저장된다.

들어오는 객체가 기존에 존재하는 캐시된 객체의 캐시ID를 가질 때마다 해당 객체의 필드가 병합된다:

- 만약 새롭게 들어오는 객체와 이미 존재하는 객체가 어떤 필드들을 공유하고 있다면, 들어온 객체로 해당 필드의 캐시된 값을 덮어씌운다.
- 기존 객체에만 있는 필드이거나, 들어오는 객체에만 존재하는 필드의 경우 보존된다.

정규화는 클라이언트에서 그래프의 일부 복사본을 구성한다. 앱의 상태가 변경될 때 읽고 업데이트하는데 최적화된 형식으로 구성된다.

## 캐시 시각화

캐시 데이터 구조를 이해하고 싶다면, [Apollo Client Devtools](https://www.apollographql.com/docs/react/development-testing/developer-tooling/#apollo-client-devtools) 설치를 강력히 추천한다.

이 브라우저 익스텐션은 캐시에 포함된 정규화된 객체들을 모두 볼 수 있는 인스펙터를 포함한다.

### 예시

SWAPI demo API의 쿼리를 아폴로 클라이언트에서 실행시켜보자.

```graphql
query {
  allPeople(first: 3) {
    # Return the first 3 items
    people {
      id
      name
      homeworld {
        id
        name
      }
    }
  }
}
```

이 쿼리는 각각 해당하는 homeworld가 있는(Planet 객체의) 3개의 Person 객체들을 리턴한다.

각 객체들이 `__typename` 필드를 결과에 포함하고 있음에도 불구하고 이 필드에는 포함되어있지 않다. 왜냐면 아폴로 클라이언트는 자동으로 모든 객체에는 \_\_typename을 쿼리하기 때문이다.

결과가 캐시된 이후, 우리는 아폴로 클라이언트 개발도구에서 우리의 캐시 상태를 볼 수 있다.

<img width="729" alt="스크린샷 2023-08-02 오전 1 23 50" src="https://github.com/leesoo7595/leesoo7595.github.io/assets/39291812/291ff4e3-5742-486c-a90d-e72cc353c455">

우리 캐시는 이제 (ROOT_QUERY 객체 외): Person 객체 3개와 Planet 객체 2개를 포함한 총 5개의 정규화된 객체를 가지고 있다.

**왜 우리는 2개의 `Planet` 객체들만 가지고 있는가?** 왜냐하면 3개의 Person 객체가 2개의 같은 `homeworld`를 가지고 있기 때문이다. 정규화한 데이터는 이렇게, 아폴로 클라이언트는 다수의 다른 객체들은 `__ref` 필드를 통해서 참조값을 가지는 식으로 한 객체당 하나의 복사본만을 캐시로 가질 수 있다.

## 다음 스텝

이제 기초적인 아폴로 클라이언트 캐시의 동작을 이해했으니, 설정하는 방법을 알아보자.

그리고, 서버에서 쿼리를 실행시키지 않고 캐시를 직접적으로 읽고 쓰는 법을 알아볼 것이다. 이것은 로컬 상태관리에서 매우 강력한 옵션이다.
