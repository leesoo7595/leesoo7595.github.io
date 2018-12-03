---
layout: post
title: "[CSS] StyleSheet, Selector"
date: 2018-12-03
tags: [CSS]
comments: true
---

## Selector

```css
selector {
  property: value;
}
```

selector : 규칙을 어디에 적용할 지 결정
property : 어떤 규칙을
value : 어떤 값만큼 결정

### Chain Selector

한 요소에 아이디와 클래스들, 또는 여러 클래스가 적용되어 있을 경우에 사용

```css
.body-text.description {
  color: #999;
}
#index-title.header-title {
  front-weight: bold;
}
```

### Group Selector

여러 선택자에 같은 스타일을 적용하는 경우 사용, 쉼표로 구분한다.

```css
#index-title, #index-description {
  text-align: center;
}
```
### Combinator Selector

자식 선택자라고 하며, 포함 관계를 가지는 태그들 사이에서 포함하는 요소는 부모 요소, 포함되는 요소는 자식 요소라고 한다.

하위 선택자는 space로 구분되어 지정하며, 부모 요소에 포함된 모든 하위 요소를 지정한다.

자식 선택자는 부모요소의 바로 아래 자식 요소만을 지정한다.

```css
section ul { /* 하위 선택자 */
  border: 1px solid black;
}
section > ul { /* 자식 선택자 */
  border: 1px solid black;
}
```

같은 부모 요소를 가지는 요소들은 형제 관계라고 부른다. 이 때, 먼저 나오는 요소를 형 요소, 나중에 나오는 요소는 동생 요소라고 부른다.

```css
h1 + ul { /* 인접 형제 선택자 */
  background: lightBlue;
  color: darkBlue;
}
h1 ~ ul { /* 일반 형제 선택자 */
  background: lightBlue;
  color: darkBlue;
}
```

인접 형제 선택자란, h1 + ul을 했을 때 h1바로 다음에 인접해있는 ul을 가리킨다. 일반 형제 선택자는, h1 ~ ul 이런식으로 쓰이는데, h1 다음부터 있는 모든 ul을 가리킨다.
