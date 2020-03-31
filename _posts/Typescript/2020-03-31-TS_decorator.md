---
layout: post
title: "[Typescript] Decorator"
date: 2020-03-31
tags: [Typescript]
comments: true
---

# Decorator

Typescript에서 데코레이터란, 간단히 말해서 함수이다.

## Class Component

```javascript
Vue.component('App', {
    // options...
});
```

뷰 컴포넌트를 작성할 때, 보통 위처럼 작성하지만, 타입스크립트의 데코레이터형을 사용할 수 있게 해주는 `vue-property-decorator`을 사용할 때에는 클래스형으로 사용할 수 있게 해준다.

```javascript
@Component
export default class App extends Vue {}
```

위와 같은 방법으로 동일하게 컴포넌트 작성이 가능하다.
