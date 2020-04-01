---
layout: post
title: "[Typescript] Vue와 Decorator"
date: 2020-03-31
tags: [Typescript, Vue]
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

## Method

```javascript
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }

    @enumerable(false)
    greet() {
        return "Hello, " + this.greeting;
    }
}

function enumerable(value: boolean) {
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        descriptor.enumerable = value;
    };
}
```

메소드 데코레이터는 위의 `Greeter` 클래스에서 `greet`이라는 메소드를 사용할 때, 데코레이터로 `enumerable`이란 함수를 먼저 돌고 나서 해당 메소드를 쓸 수 있도록 하고 있다.

여기서 `enumerable` 함수를 메소드의 데코레이터로 두면, 위에 객체로 정의되는 `Greeter`라는 객체가 정의되기 전에 해당 함수를 돌아서 해당 객체를 재정의할 수 있게 해준다. `enumerable`은 `es2015`에서 나온 `Object.defineProperty`라는 메소드 역할을 한다. `enumerable`에서 받는 3개의 프로퍼티가 있는데, 첫 번째로 `target`으로는 `Greeter`라는 객체를 받고, `propertyKey`로는 `Greeter`의 프로퍼티인 `greeting`을 받는다. 그리고 세 번째로 `descriptor`으로는 해당 객체를 기술하는 기본적인 요소들을 이야기한다.

여기서 `enumerable` 함수는 `descriptor`의 `enumberable` 속성을 `false`로 수정하는 역할을 해준다. 이 역할은 즉, 해당 객체가 열거할 수 없게끔 만드는 일을 한다. 그래서 `Greeter` 객체는 열거할 수 없는 객체가 된다.

### Vue의 @Watch

```javascript
const watchEx = new Vue({
    el: '#watch-example',
    data: {
        question: '',
        answer: '질문을 해주세요',
    },
    watch: {
        question: function (newQuestion) {
            this.answer = '입력을 기다리는 중...';
        }
    }
});
```

타입스크립트의 데코레이터를 사용하지 않은 뷰의 `watch` 사용 방법은 위와 같다. 마찬가지로 객체로 `watch` 프로퍼티를 통해서 해당 데이터의 변화를 감지하여 `question` 함수가 동작하도록 되어있다. 타입스크립트의 데코레이터인 `@Watch`를 사용할 경우엔 다음과 같다.

```javascript
@Component
export default class WatchEx extends Vue {
    question: string = '';
    answer: string = '질문을 해주세요.';

    @Watch('question')
    watcher() {
        this.answer = '입력을 기다리는 중...';
    }
}
```

타입스크립트의 데코레이터를 사용한 `@Watch` 사용방법은 다음과 같다. 데코레이터 `Watch`를 사용하면 파라미터로 해당 클래스(객체)의 변화를 감지할 프로퍼티 값을 넣는다. 그리고 그 다음에 함수를 입력하여 변화 감지시 취할 코드를 작성한다.