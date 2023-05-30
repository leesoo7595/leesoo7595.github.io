---
layout: post
title: "Apollo Server - Directives"
date: 2023-05-28
tags: [Graphql]
comments: true
---

## Directives

Directives (지시어)는 스키마나 operation의 추가적인 구성을 진행할 수 있다.

```graphql
type ExampleType {
  oldField: String @deprecated(reason: "Use `newField`.")
  newField: String
}
```

위 예시는 기본 지시어인 `@deprecated` 지시어를 보여준다.

지시어는 자체 인수를 받을 수 있습니다(reason).
지시어는 지시어가 장식하는 대상(이 경우 oldField 필드)의 선언 뒤에 나타납니다.

### Valid Location

Directives는 특정한 위치에서만 쓰일 수 있다. 이 위치들은 directives의 definition에 나열되어있다.

```graphql
directive @deprecated(
  reason: String = "No longer supported"
) on FIELD_DEFINITION | ARGUMENT_DEFINITION | INPUT_FIELD_DEFINITION | ENUM_VALUE
```

`@deprecated`라는 directives의 정의에 대한 예시이다. `reason` 인수는 optional인걸 알 수 있고 기본값을 제공해준다고 표시되어있다. 또한 `FIELD_DEFINITION | ARGUMENT_DEFINITION | INPUT_FIELD_DEFINITION | ENUM_VALUE`에 위치할 수 있다고 제시되어있다.

```graphql
# ARGUMENT_DEFINITION
# Note: @deprecated arguments _must_ be optional.
directive @withDeprecatedArgs(
  deprecatedArg: String @deprecated(reason: "Use `newArg`")
  newArg: String
) on FIELD

type MyType {
  # ARGUMENT_DEFINITION (alternate example on a field's args)
  fieldWithDeprecatedArgs(name: String @deprecated): String
  # FIELD_DEFINITION
  deprecatedField: String @deprecated
}

enum MyEnum {
  # ENUM_VALUE
  OLD_VALUE @deprecated(reason: "Use `NEW_VALUE`.")
  NEW_VALUE
}

input SomeInputType {
  nonDeprecated: String
  # INPUT_FIELD_DEFINITION
  deprecated: String @deprecated
}
```

위 예시는 각 위치별로 예시를 작성한 것이다. 다른 곳에 `@deprecated`를 사용하면 오류가 나타난다.

위와 같이 사용자 custom directives를 만들 수도 있다.

### Schema directives vs operation directives

일반적으로 Schema에서 쓰이는 directives와 operation에서 쓰이는 directives가 있다.

예를 들어 기본 지시어 중 `@deprecated`는 schema 전용 directives이고 `@skip`, `@include`는 operation 전용 directives이다.

GraphQL 사양에는 가능한 모든 directives 위치가 나열되어 있다. 스키마 위치는 `TypeSystemDirectiveLocation` 아래에 나열되고, 연산 위치는 `ExecutableDirectiveLocation` 아래에 나열된다.

## Default directives

- `@deprecated(reason: String)`: field 또는 enum형 값의 schema definition를 optional reason과 함께 더 이상 사용되지 않는 것으로 표시한다.
- `@skip(if: Boolean!)`: true이면 operation에서 쓰인 field 또는 fragment가 GraphQL 서버에서 확인되지 않는다.
- `@include(if: Boolean!)`: false인 경우, operation에서 쓰인 field 또는 fragment가 GraphQL 서버에서 확인되지 않는다.

## Custom directives

apollo server는 스키마를 변환하는 custom directives에 대한 기본 지원을 제공하지 않는다.

```
# Definition
directive @uppercase on FIELD_DEFINITION

type Query {
  # Usage
  hello: String @uppercase
}
```

직접 정의한 custom directives를 사용하여 스키마 동작을 변환한 후 apollo server에 제공하려면, `@graphql-tools` 라이브러리를 사용하는 것이 좋다.
예시는 [여기 uppercase custom directives](https://github.com/apollographql/docs-examples/blob/main/apollo-server/v4/custom-directives/upper-case-directive/src/index.ts)를 참고하면 좋을듯.

### In subgraphs

federated graph에서 directives를 사용하기전에, 다음 사항을 고려하여야한다.

- custom directives는 graph에 구성된 supergraphs schema에 포함되지 않는다. 복잡한 graphs는 모든 하위 directives를 제거한다. 주어진 subgraphs만 자신의 directives를 인식한다.
- directives는 각각의 subgraphs에 특정해서 고유하기 때문에, 서로 다른 subgraphs가 동일한 directives이지만 내용이 다른 것으로 정의하는 것이 가능하다. Composition은 이러한 불일치에 대해서 경고나 에러를 주지 않는다.
- multiple subgraphs가 특정 field를 resolve하는 경우, 각 subgraph들은 동일한 로직으로 그 field를 resolve하게 구현하여야한다. 그렇지 않으면 어떤 subgraph가 해당 field를 resolve하느냐에 따라 달라질 수 있다.

apollo server에서는 subgraph 스키마의 custom directives에 대한 transform 함수를 정의할 수 있다. 실행 가능한 subgraph의 스키마에서 transform 함수를 적용하려면, 먼저 `buildSubgraphSchema`를 사용하여 우선 subgraph schema를 생성한다.

그리고 결과를 ApolloServer 생성자에 직접 전달하는 대신 먼저 transform 함수를 적용한다. 모든 transform 함수를 적용한 후 하던대로 최종 subgraphs 스키마를 ApolloServer 생성자에 제공한다.

```typescript
let subgraphSchema = buildSubgraphSchema({ typeDefs, resolvers });
// Transformer function for an @upper directive
subgraphSchema = upperDirectiveTransformer(subgraphSchema, "upper");

const server = new ApolloServer({
  schema: subgraphSchema,
  // ...other options...
});
```

## Reference

- [Apollo Server - directives](https://www.apollographql.com/docs/apollo-server/schema/directives/)
