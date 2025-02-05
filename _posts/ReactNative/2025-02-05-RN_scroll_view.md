---
layout: post
title: "React Native - ScrollView"
date: 2025-02-05
tags: [React Native]
comments: true
---

# ScrollView

ScrollView를 감싸는 동시에 responder 시스템과의 통합을 제공하는 컴포넌트이다.

ScrollView는 높이가 제한되지 않은 자식을 높이가 제한된 컨테이너에 포함하므로 작동하려면 높이가 제한되어야 한다. ScrollView의 높이를 바인딩하려면 height를 직접 설정하거나, 모든 Parent view의 높이가 바인딩되어있는지 확인해야한다. view stack 아래로 `{ flex: 1 }`을 넘기지 않으면 오류가 발생할 수 있다.

아직까지는 다른 responder가 이 ScrollView의 responder를 차단하는 것을 지원하지는 않는다.

`<ScrollView>` vs `<FlatList>` 둘 중 어느 것을 사용해야하나?

`ScrollView`는 모든 react child를 한 번에 렌더링하지만 성능이 좀 떨어진다.

보여주려는 아이템이 매우 길 때, 즉 여러 화면에 걸쳐진 컨텐츠일 때 모든 js 컴포넌트와 native views를 한 번에 생성하면 렌더링 속도가 느려지고 메모리 사용량이 증가한다.

이 때 `FlatList`가 쓰인다. `FlatList`는 아이템들이 보여질 때 렌더링되며, 화면 밖으로 스크롤되는 항목은 제거하여 메모리와 처리 시간을 절약할 수 있다.

`FlatList`는 항목 사이의 구분선이나 여러 열, 무한 스크롤 로딩 또는 기본적으로 지원하는 여러 기능을 렌더링하려는 경우에도 유용하다.

그러면 ScrollView 말고 FlatList를 안 쓸 이유가 뭐가 있지..!

ScrollView 내부에 FlatList를 사용하는 실수를 많이 하는 것으로 보인다. ScrollView 안에 FlatList를 또 사용하면 ScrollView는 자식을의 height를 다 계산해서 한꺼번에 보여줘야하지만, FlatList의 경우 내부 항목들의 height를 그때그때 처리해서 효율적으로 보여주기 때문에 ScrollView 내부에 FlatList를 감싸는 행위 자체가 옳지 못한 것으로 보인다.

https://velog.io/@mywonhyuni/RN-ScrollView-%EC%95%88%EC%97%90-FlatList-%EC%A4%91%EC%B2%A9-%EC%82%AC%EC%9A%A9

## Reference

### Props

View의 Props를 상속한다.

### StickyHeaderComponent

sticky header를 렌더링할 때 사용되는 React 컴포넌트이고, stickyHeaderIndices와 함께 사용되어야한다.

예를 들어 리스트에 애니메이션이 적용되고 숨길 수 있는 hindable header를 원하는 경우와 같이 custom transform을 사용하는 경우 이 컴포넌트를 설정할 수 있다. 컴포넌트를 제공하지 않는 경우 기본 ScrollViewStickyHeader 컴포넌트가 사용된다.
