---
layout: post
title: "React Native - Render, Commit, Mount"
date: 2025-07-05
tags: [React Native]
comments: true
---

# Render, Commit, and Mount

React Native 렌더러는 Host Platform에 React 로직을 렌더링하기 위해 일련의 작업을 거친다. 이 작업을 Render pipeline이라고 하며, 초기 렌더링과 UI 상태 업데이트를 할 때 이 작업이 진행된다.

render pipeline은 일반적으로 3단계로 나눌 수 있다.

### 1. Render

React는 js에서 React element tree를 생성하는 로직을 실행한다. 이 tree에서 renderer는 C++로 React Shadow Tree를 생성한다.

### 2. Commit

React Shadow Tree가 완전히 생성되면 렌더러가 commit 단계를 트리거한다. 그러면 React Element Tree와 새로 생성된 React Shadow Tree가 모두 마운트될 다음 Tree로 승격된다. 또한 레이아웃 정보의 계산도 스케쥴링한다.

### 3. Mount

레이아웃 계산의 결과까지 가지고 있는 React Shadow Tree가 Host View Tree로 변환된다.

## Intial Render

```jsx
function MyComponent() {
  return (
    <View>
      <Text>Hello, World</Text>
    </View>
  );
}
```

위를 렌더링한다고 상상해보자.

위의 예시에서 `<MyComponent />`는 React Element이다. React는 재귀적으로 React Element가 더 이상 reduces될 수 없을 때까지 React Element를 호출하여 터미널 React Host Component(div, text 등)로 만든다. 이제 너는 React Element Tree의 React Host Component를 가지고 있다.

### Phase 1. Render

이 element를 감소시키는 과정에서, 각각의 React Element가 일어나고, renderer는 동시에 React Shadow Node도 만든다. 이 일은 React Host Components인 경우에 해당하며, React Composite Components에서는 일어나지 않는다.

예를 들면 <View>는 ViewShadowNode 객체를 만들도록 주도하고, <Text>는 TextShadowNode 객체를 만들도록 주도한다. <MyComponent>를 직접적으로 나타내는 React Shadow Node가 없다는 것에 주목하자.

React는 두개의 React Element Nodes 사이에 부모-자식 관계를 형성할때, renderer는 같은 관계를 React Shadow Nodes에도 나타날 수 있도록 한다. 이게 React Shadow Tree가 조립되는 방식이다.

**Additional Details**

- (React Shadow Node 생성, 두 React Shadow Node 간의 부모-자식 관계 생성) 작업은 동기식이고 스레드-세이프한 작업으로, 일반적으로 자바스크립트 스레드 위에서 React(JS)에서 C++ 렌더러로 실행된다.
- React Element Tree(및 이를 구성하는 React Element Node)는 무한정 존재하지 않는다. React에서 fibers로 구체화된 시간적 표현이다. Host Component를 나타내는 각 fiber는 JSI로 인해 가능해진 React Shadow Node에 대한 C++ 포인터를 저장한다.
- React Shadow Tree는 불변이다. React Shadow Node를 업데이트하기 위해서는 렌더러가 새로운 React Shadow Tree를 생성한다. 그러나 렌더러는 상태 업데이트의 성능을 높이기 위해 cloning 작업을 제공한다.

React Shadow Tree가 완성되면, 렌더러는 React Element Tree의 commit 단계를 트리거한다.

### Phase 2. Commit
