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

## Reference

- [Apollo Server - directives](https://www.apollographql.com/docs/apollo-server/schema/directives/)
