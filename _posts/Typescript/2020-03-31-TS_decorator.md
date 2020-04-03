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

## Emit

자식에서 부모로 전달하는 이벤트 핸들러이다. 주로 자식이 필요할 때 이 핸들러를 트리거함으로써 부모와 소통을 할 수 있다.

```javascript
export default {
    data() {
        return {
            count: 0
        }
    },
    methods: {
        addToCount(n) {
            this.count += n;
            this.$emit('add-to-count', n);
        },
        resetCount() {
            this.count = 0;
            this.$emit('reset');
        },
        returnValue() {
            this.$emit('return-value', 10)
        },
        promise() {
            const promise = new Promise(resolve => {
                setTimeout(() => {
                    resolve(20)
                }, 0)
            });

            promise.then(value => {
                this.$emit('promise', value);
            });
        }
    }
}
```

위와 같이 사용하는 방법은 타입스크립트를 사용하지 않은 평범한 뷰의 `emit` 이벤트 핸들러를 사용하는 방식이다. $emit을 사용함으로써 첫 번째 파라미터로는 부모가 실행할 메소드 이름을 받고, 두 번째 파라미터로는 부모에게 넘겨줄 값을 받는다.

```javascript
@Component
export default class MyComponent extends Vue {
    count = 0;

    @Emit()
    addToCount(n: number) {
        this.count += n;
    }

    @Emit('reset')
    resetCount() {
        this.count = 0;
    }

    @Emit()
    returnValue() {
        return 10;
    }

    @Emit()
    promise() {
        return new Promise(resolve => {
            setTimeout(() => {
                resolve(20);
            }, 0);
        })
    }
}
```

뷰에 타입스크립트를 사용하면, 부모의 핸들러 함수명과 동일한 함수명을 사용할 경우, 함수 위에 `@Emit` 데코레이터를 얹고 파라미터로 아무것도 전달하지 않아도 되지만, 부모의 핸들러 함수명과 `@Emit` 데코레이터를 사용하는 함수명과 매치가 되지 않으면 해당 데코레이터 파라미터로 부모의 핸들러 함수명을 넘겨주면 된다.

## Provide & Inject

뷰에서 `inject`는 자식 컴포넌트에서 제공한 속성을 의미한다. `provide`는 자식 컴포넌트가 `inject`를 사용함으로써 제공할 수 있는 속성들을 의미한다.

```javascript
// parent component providing 'foo'
var Provider = {
  provide: {
    foo: 'bar'
  },
  // ...
}

// child component injecting 'foo'
var Child = {
  inject: ['foo'],
  created () {
    console.log(this.foo) // => "bar"
  }
  // ...
}
```

위의 코드는 뷰에서 `provide`와 `inject`를 사용할 때 쓰이는 메소드이다. 타입스크립트를 사용하여 작성하였을 때는 아래와 같이 사용한다.

```javascript
export default class MyComponent extends Vue {
    @Provide() foo: string = 'bar';
    @Inject() readonly foo!: string;
}
```

`provide`와 `inject`는 기본적으로 프로퍼티 데코레이터의 종류 중 하나인데, 프로퍼티 데코레이터는 초기화를 하지 않으면 오류를 뱉는다. 이에 대한 해결책은 프로퍼티를 선언할 때 변수명 뒤에 optional하게 주거나, 반대로 `essential`하게 주면 오류가 사라진다. `optional`은 타입스크립트에서 ?를 붙이고, `essential`은 !를 붙인다. 

## Model

v-model 바인딩과 헷갈릴 수 있는 기능이다. v-model을 사용할 때 일어날 수 있는 충돌을 피하기 위한 옵션이다. 잘 사용하지 않으니 참고용으로만 알아둘 것!

```javascript
export default {
    model: {
        prop: 'checked',
        event: 'change'
    },
    props: {
        checked: {
            type: Boolean
        },
    },
}
```

model 컴포넌트를 사용하는 방법은 위와 같다. vue의 객체에 model을 키 값으로 객체에서 사용하는 식이다. 이 기능을 타입스크립트로 사용할 경우에는 다음과 같다.

```javascript
@Component
export default class MyComponent extends Vue {
    @Model('change', {type: Boolean}) readonly checked!: boolean
}
```

model 바인딩을 해줄 프로퍼티의 이름을 Model 데코레이터의 첫 번째 파라미터에 넣어주고, 타입 지정 등은 두 번째 파라미터에 넣어주는 식으로 사용한다.

## Mixin

es6와 타입스크립트에는 다중상속을 가능하게 하는 `Mixin` 기능이 있다. 즉, 중복되는 공통 로직을 컴포넌트로 빼서 `Mixin`으로 사용하기 쉽게 활용할 수 있다.

다음 로직을 보면, 보통 뷰에서는 객체 타입으로 구현을 하게되는데, 타입스크립트로 하게 될 경우 Vue라는 내부 컴포넌트를 상속받아 구현하는 것이 기본이 된다. 여기서 `Vue` 컴포넌트 대신 `Mixins`을 사용하여 `Mixins`의 인자에 공통된 로직을 컴포넌트로 만든 컴포넌트 명을 받아 다중상속을 하여 사용할 수 있다.

```javascript
@Component({
    components: {
        Toggle,
    }
})
export default class Dropdown extends Mixins(Toggle) {
    mounted() {
        console.log(this);
    }
}
```

### Reference

* [인프런 - Typescript_Vue](https://www.inflearn.com/course/Typescript_Vue)