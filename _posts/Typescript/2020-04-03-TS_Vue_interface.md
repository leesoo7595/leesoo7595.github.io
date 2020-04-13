---
layout: post
title: "[Typescript] Vue와 Interface"
date: 2020-04-03
tags: [Typescript, Vue]
comments: true
---

## Interface

```javascript
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
    let newSquare = {color: "white", area: 100};
    if (config.color) {
        newSquare.color = config.color;
    }
    if (config.width) {
        newSquare.area = config.width * config.width
    }
    return newSquare;
}

let mySquare = createSquare({color: "black"});
```

### Generic

```javascript
function identity(arg: number): number {
    return arg;
}

function identity<T>(arg: T): T {
    return arg;
}
```

#### Vuex의 Store

```javascript
import Vuex, {StoreOptions} from 'vuex';
Vue.use(Vuex);

interface State {
    // 상태값에 대한 인터페이스
}

const Store: StoreOptions<State> {
    state: {

    },
    mutations: {

    },
    actions: {

    },
    getters: {

    },
};

export default Vuex.Store(Store)
```