---
layout: post
title: "[ES6] Module"
date: 2020-01-19
tags: [JavaScript]
comments: true
---

# module

javascript module이란? 주요 기능이나 새로운 기능 단위로 모아놓은 것을 말한다. javascript를 사용할 때, 내부적으로 사용하는 자바스크립트 모듈 시스템을 사용하였을 것이다. 그것은 Node.js에 있는 CommonJS가 될 수도 있고, AMD나, 아니면 그 외의 것들이 될 수 있다. 이런 모든 모듈 시스템의 공통점은 다른 것들을 import하거나, export할 수 있다는 것이다.

Javascript는 이제 그런 것들에 대한 표준된 방안을 가지고 있다. 모듈은, 특정 키워드로 내보내서 외부적으로 사용할 수 있게하는 것이다. const나, function, 그리고 많은 다른 변수 등으로 바인딩하거나 선언해서 export를 할 수 있다. 

```javascript
// lib.mjs
export const speak = (name) => `${name} is speaking`;
export function shout(name) {
    return `${name.toUpperCase()}!;`
}

// another export declaration
export default function(string) {
    return `this is export default ${string} function`;
}
// this default exports can be imported  by any name
```

위의 예시처럼, 변수 등으로 export하여 사용할 수 있다. 외부적으로 내보낼 수 있다는 것은, 다른 말로 하면 다른 모듈에서 import할 수도 있다는 뜻이다. 

```javascript
import defaultExport, {speak, shout} from 'lib.mjs';
speak('leesoo');    // leesoo is speaking
shout('leesoo is coming');  // LEESOO IS COMING!
defaultExport('function');  // this is export default function function 
```

## 클래식과 모듈의 차이점

* 모듈은 strict mode가 default로 적용된다.
* html-style의 문법은 모듈에서는 지원되지 않는다. 일반적인 script에서는 동작하던 것도 마찬가지.
* 모듈은 독립적인 스코프를 가지고 있다. 모듈을 쓸 때, 전역 스코프를 가지고 생성되지않는다는 것이다. 즉, 모듈 내에서는 var로 변수 선언을 한다해도 이는 전역변수가 될 수 없다. 그래서 모듈 내에서 선언한 변수는 모듈 외부에서 참조할 수 없다.
* 클래식 스크립트 내에서는 import, export 문법을 사용할 수 없다.

브라우저에서 JS 모듈을 사용할 땐, `script` 태그에 `type="module"`을 추가하여 사용하면 된다.

### 브라우저에서 모듈 & 클래식의 차이점

* 브라우저에서 모듈을 사용할 때, 똑같은 script 모듈이 두 번 입력되어 있어도 단 한번만 실행지만, 클래식은 DOM에 추가된 만큼 실행된다.
* 클래식은 CORS 헤더와 관계없이 항상 로드된다. 모듈은 CORS에 맞는 헤더가 필요하다. -> 정확히 어떤 뜻일까
* 클래식에서는 async attribute가 붙은 스크립트의 다운로드를 막을 수 없지만, 모듈은 가능하다.(defer attribute)

여기서 defer과 async 태그를 몰라서 mdn에서 가져왓다

<img width="711" alt="Screen Shot 2020-01-19 at 7 14 43 PM" src="https://user-images.githubusercontent.com/39291812/72679188-21b08500-3af0-11ea-91f8-ae7108371ddf.png">

<img width="713" alt="Screen Shot 2020-01-19 at 7 15 08 PM" src="https://user-images.githubusercontent.com/39291812/72679189-25dca280-3af0-11ea-86cc-70239f994c24.png">

### 파일 확장자

모듈 자바스크립트를 사용하려면 .mjs 확장자를 사용하는데, 웹에서는 이 확장자를 구분하지 않는다. .js로 제공된다.

* 브라우저는 type attribute를 이용하여 모듈인지 구분한다.
* 구글에서는 .mjs를 사용하길 권장한다.
    - 개발할 때, 모듈 파일인지 여부를 알 수 있다.
    - nodejs의 모듈 지원은 .mjs 파일에서만 동작한다. (?)

클래식은 HTML 파서의 속도를 늦춘다. 반면 모듈은 `defer` 어트리뷰트를 추가함으로써 스크립트 다운로드 및 파싱을 병행하도록 할 수 있다. 이는 모듈간의 의존관계로 인한 문제가 생기지 않음을 의미한다.

### Dynamic import (동적 import)

`static import`는 모든 모듈이 다운로드된 후에 코드가 실행된다. (아 그래서 import문에 특정 모듈이 존재하지않으면 바로 에러가 떠버렸구나) 특정 모듈만 실행하고 싶은 컴포넌트가 있다면, 해당 컴포넌트가 실행될 때 모듈을 받도록 하는 것이 전체 로드 시간을 줄일 수 있을 것이다.

```html
<script type="module">
    (async () => {
        const moduleJS = '.lib.mjs';
        const {speak, shout} = await import(moduleJS);
        speak('leesoo');    // leesoo is speaking
    })
</script>
```

#### import.meta

`import.meta`라는 것은, 현재 모듈에 대한 메타데이터를 제공하는 기능이다. ECMAScript에서 정의되지 않은 호스트 환경에 종속적인 기능을 사용하는 데에 정보를 제공해주는 기능이라고 한다........잘모르겠다.

#### bundling

* 모듈로 나눠지더라도, 수많은 모듈 환경들을 테스트할 때, 병목현상이 발생한 경우가 있다. 그래서 번들링을 권장한다.
* 모듈을 배포하기전에 번들러를 돌려서 모듈과 코드 수를 줄이는 것이 성능이 좋다.
* 번들링하는 것과, 하지 않는 것은 trade-off 관계이므로 잘 생각해서 결정하는 것이 사실 제일 좋음
* 모듈을 세분화해서 사용하는 것
* 모듈 preload하기 -> 모듈 전달 최적화를 위해, 의존성을 미리 로드해서 컴파일할 수 있도록 : 모듈의 종속성이 크면, 브라우저는 이를 확인하려고 많은 http 요청을 보내게된다. 이를 preload하여 한번에 처리할 수 있다.
* http/2 사용 -> 이쪽 부분은 공부가 더 필요할듯........

주의할 점은 여기서 소개한 module은 es6에서 새로 나온 기능이기 때문에, 모든 브라우저가 지원하지 못한다. 그래서 꼭 babel, webpack 같은 번들링을 이용하여 브라우저에서 호환할 수 있는 작업이 먼저 진행되어야 사용할 수 있을 것이다.

## 참고

- [웹에서 자바스크립트 모듈 사용하기](https://velog.io/@widian/%EC%9B%B9%EC%97%90%EC%84%9C-%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EB%AA%A8%EB%93%88-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)
- [Dynamic Import](https://v8.dev/features/dynamic-import)
- [Javascript MDN](https://developer.mozilla.org/ko/docs/Web/HTML/Element/script)