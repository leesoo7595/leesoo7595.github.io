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

commit 단계는 두 가지의 작업으로 구성된다: Layout Calculation과 Tree Promotion

- Layout Calculation: 이 작업은 각각의 React Shadow Node의 사이즈와 position을 계산한다. RN에서는, Yoga를 실행시켜서 각 React Shadow Node의 layout을 계산하도록 한다. 이 실제 계산을 하기 위해서는 js의 React Element에 있던 각 React Shadow Node의 스타일을 요구한다. 또한 노드로 변환될때 차지할 수 있는 사용 공간을 결정하는 React Shadow Tree의 root에 대한 layout 제약도 요구된다. (이게 뭐지)
- Tree Promotion (New Tree -> Next Tree): 이 작업은 새로운 React Shadow Tree를 마운트 될 다음 tree로 승격하는 작업이다. 이 프로모션(승격)은 새로운 React Shadow Tree가 모든 마운트되기 위한 정보와 React Element Tree의 최신 상태를 나타내고 있음을 가리킨다. 다음 트리의 마운트는 UI Thread의 다음 tick에서 일어난다.

**Additional Details**

- 이 작업들은 백그라운드의 스레드에서 비동기적으로 실행된다.
- 대부분의 layout 계산은 전적으로 C++로 실행된다. 그러나 어떤 컴포넌트들의 layout 계산은 host platform(Text, TextInput)을 의존한다. 텍스트의 size와 position은 각 host platform에 따라 다르며 host platform 계층에서 계산해야한다. 이를 위해 yoga는 host platform에 정의된 함수를 호출하여 컴포넌트 레이아웃을 계산한다.

### Phase 3. Mount

mount 단계에서는 React Shadow Tree(layout 계산으로 인해 데이터가 포함된)를 화면에 렌더될 픽셀을 가지고 있는 Host View Tree로 변환하는 작업을 한다. 리마인드하면, React Element Tree는 이렇게 생겼다.

```jsx
<View>
  <Text>Hello, World</Text>
</View>
```

높은 수준에서 RN 렌더러는 각 React Shadow Node에 해당하는 Host View를 생성하고 이를 화면에 마운트한다. 위를 예로 들면, renderer는 <View>를 위한 android.view.ViewGroup 인스턴스를 생성하고 <Text>를 위한 android.widget.TextView을 생성한다. 그리고 Hello World를 채운다.
iOS에서도 비슷하게 UIView를 생성하고 NSLayoutManager를 호출하여 text를 채운다. 각 host view는 React Shadow Node를 통한 props를 사용하여 설정하고, size, position을 각 계산된 layout 정보에 설정한다.

자세하게는 mounting 단계에서는 3가지의 스텝으로 이루어진다:

- Tree Diffing
- Tree Promotion
- View Mounting
