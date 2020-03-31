---
layout: post
title: "[Typescript] Decorator"
date: 2020-03-31
tags: [Typescript]
comments: true
---

# Decorator

Typescript에서 데코레이터란, 간단히 말해서 함수이다.

## Component

```javascript
Vue.component('App', {
    // options...
});
```

뷰 컴포넌트를 작성할 때, 보통 위처럼 작성하지만, 타입스크립트의 데코레이터형을 사용할 수 있게 해주는 `vue-property-decorator`을 사용할 때에는 클래스형으로 사용할 수 있게 해준다.

```javascript
@Component
export default class App extends Vue {}

@Component({
    components: {
        Home,
    }
})
export default class App extends Vue {}
```

위와 같은 방법으로 동일하게 컴포넌트 작성이 가능하다.

반대로 상위 컴포넌트에서 특정 컴포넌트를 불러와서 사용할 때엔 위 코드의 두 번째 컴포넌트처럼 component 데코레이터 내부에서 불러올 컴포넌트를 선언해주면 된다.

## Props

```javascript
Vue.component('child', {
    props: ['message'],
});
```

기존의 vue에서 props를 불러올 때 코드 작성은 위와 같다. 이는 마찬가지로 Props 데코레이터를 써서 똑같은 결과를 낼 수 있다.

```javascript
@Component
export default class PropExample extends Vue {
    @Prop() message: string;
}
```

Prop 데코레이터를 사용하여 위와 같이 동일하게 작성할 수 있다.