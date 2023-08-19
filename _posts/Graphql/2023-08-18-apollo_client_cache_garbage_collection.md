---
layout: post
title: "Apollo Client - Garbage collection and cache eviction"
date: 2023-08-18
tags: [Graphql]
comments: true
---

이 글은 아폴로 클라이언트 [공식문서](https://www.apollographql.com/docs/react/caching/garbage-collection/)를 읽고 정리한 글 이다.

아폴로 클라이언트 3을 사용하면 더 이상 유용하지 않은 캐시된 데이터를 선택적으로 제거할 수 있다. 기본 garbage collenction 전략인 `gc` 메소드는 대부분의 애플리케이션에 적합하지만, `evict` 메소드는 이를 필요로 하는 애플리케이션에 보다 세밀한 제어 기능을 제공한다.

> 이런 메소드들을 아폴로 클라이언트 객체가 아닌 InMemoryCache 객체에서 직접 호출한다.

## `cache.gc()`

`gc` 메소드는 정규화된 캐시에서 도달할 수 없는 모든 객체를 삭제한다:

```javascript
cache.gc();
```

객체에 도달할 수 있는지 여부를 결정하기 위해, 캐시는 알려진 모든 root 객체(주로 Query 또는 Mutation)에서 시작하여 tracing 전략을 사용하여 사용가능한 모든 하위 참조를 재귀적으로 방문한다. 이 과정에서 방문되지 않은 정규화된 객체들은 모두 제거된다. `cache.gc()` 메소드는 제거된 객체의 id 리스트를 리턴한다.

그래프큐엘 데이터를 가지치기하는 것 외에도 `cache.gc()`는 캐시가 이전 캐시 결과의 변경되지 않은 부분을 보존하는데 사용하는 메모리를 해제할 수도 있다:

```javascript
cache.gc({ resetResultCache: true });
```

이 메모리를 해제하면 캐시 읽기 속도가 일시적으로 느려지는데, 이는 해당 읽기가 이전 읽기 작업의 혜택을 받지 못하기 때문이다.

`canonizeResults: true` 옵션을 사용한다면, `cache.gc`는 canonical 결과 객체를 조회하는데 사용하는 메모리를 재설정할 수도 있다:

```javascript
cache.gc({
  resetResultCache: true,
  resetResultIdentities: true,
});
```

이러한 추가적인 `cache.gc` 옵션은 메모리 사용 패턴이나 누수를 조사하는데 유용할 수 있다. heap의 스냅샷을 찍거나 할당 타임라인을 기록하기전에, 브라우저의 개발자 도구를 사용하여 자바스크립트 가비지 컬렉션을 강제로 수행하여 캐시에 의해 해제된 메모리가 완전히 수집되어 heap으로 리턴되었는지 확인하는 것이 좋다.

### 가비지 컬렉션 구성하기

`retain` 메소드를 사용하면 객체에 도달할 수 없는 경우에도 객체(및 하위 객체)가 가비지 컬렉션에 포함되지 않도록 할 수 있다:

```javascript
cache.retain("my-object-id");
```

나중에 보존된 객체를 가비지 컬렉션으로 처리하려면 `release` 메서드를 사용하자:

```javascript
cache.release("my-object-id");
```

객체에 연결할 수 없는 경우 다음번 `gc` 호출시 가비지 컬렉션된다.

## `cache.evict()`

정규화된 객체를 캐시에서 제거하려면 `evict` 메서드를 사용하여 제거할 수 있다.

```javascript
cache.evict({ id: "my-object-id" });
```

제거할 필드의 이름을 넘겨주는 형태로 캐시된 객체에서 단일 필드를 제거할 수도 있다:

```javascript
cache.evict({ id: "my-object-id", fieldName: "yearOfFounding" });
```

객체를 evicting하는 것은 캐시된 다른 객체에 연결할 수 없게되는 경우가 많다. 따라서 캐시에서 하나 이상의 객체를 `evict`한 후, `cache.gc()`를 호출해야한다.

## 매달린 참조 (dangling references)

객체가 캐시에서 evict(제거)되면, 해당 객체에 대한 참조가 캐시된 다른 객체에 남아있을 수 있다. 아폴로 클라이언트는 이 참조된 객체가 나중에 다시 캐시에 기록될 수 있으므로 기본적으로 이 매달린 참조를 보존한다. 이는 참조가 여전히 유용할 수 있음을 의미한다.

매달린 참조를 포함할 수 있는 필드에 대해 커스터마이징 read 함수를 정의하여 매달린 참조의 동작을 커스터마이징할 수 있다. 이 함수는 필드의 참조된 객체가 누락되었을 때 필요한 모든 정리를 수행할 수 있다. 예를 들어 read 함수의 경우:

- 사용가능한 객체 리스트에서 참조된 객체를 필터링한다.
- 필드의 값을 null로 세팅한다.
- 특정 기본값 리턴

모든 read 함수는 필드가 현재 매달린 참조를 포함하는 경우를 감지하는데 도움을 주는 `canRead` 함수가 전달된다.

다음 코드는 두 개의 `read` 함수를 정의한다. (하나는 `Query.ruler`과 다른 하나는 `Deity.offspring`) 그리고 두 함수 모두 `canRead`를 사용한다:

```javascript
new InMemoryCache({
  typePolicies: {
    Query: {
      fields: {
        ruler(existingRuler, { canRead, toReference }) {
          // 기존 ruler가 없는 경우, 아폴로가 deity를 ruling한다.
          return canRead(existingRuler)
            ? existingRuler
            : toReference({
                __typename: "Deity",
                name: "Apollo",
              });
        },
      },
    },

    Deity: {
      keyFields: ["name"],
      fields: {
        offspring(existingOffspring: Reference[], { canRead }) {
          // offspring을 제거한 후 남은 매달린 참조를 필터링한다.
          // offspring이 없는 경우 빈 배열 제공
          return existingOffspring ? existingOffspring.filter(canRead) : [];
        },
      },
    },
  },
});
```

- `Query.ruler`의 `read` 함수는 `existingRuler`가 사라졌을 때 기본적인 ruler(Apollo)를 반환한다.
- `Deity.offspring`의 `read` 함수는 리스트를 필터링하여 캐시에서 살아있는 offspring만 리턴한다.

캐시된 리스트 필드에서 매달린 참조를 필터링하는 것은 매우 일반적이어서(위의 `Deity.offspring`처럼) 리스트 필드의 기본 `read` 함수가 이 필터링을 자동을 수행한다. `read` 함수를 커스터마이징하여 이 동작을 재정의할 수 있다.

위의 예제처럼 하나의 매달린 참조가 포함된 필드에 대한 일반적인 해결책은 없으므로, 커스터마이징 `read` 함수를 작성하는 것이 가장 유용하다.
