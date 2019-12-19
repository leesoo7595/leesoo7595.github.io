---
layout: post
title: "[JavaScript] Event"
date: 2019-12-15
tags: [JavaScript]
comments: true
---

## 이벤트

이벤트란 어떤 사건을 의미한다.

## 이벤트 루프와 동시성

자바스크립트는 단일 스레드이다. 그래서 하나의 task를 처리할 수 있다. 하지만 자바스크립트가 사용되는 환경(웹 어플리케이션)에서는 많은 task가 동시에 처리되고 있는 것을 확인 할 수 있다.

예를 들면, 웹 브라우저는 애니메이션 효과를 보여주면서 마우스 입력을 받아 처리한다. 또한 Node.js에서 동시에 여러 개의 HTTP 요청을 처리하기도 한다.

자바스크립트는 단일 스레드인데, 어떻게 이런 동시성을 지원해줄 수 있을까?

### 이벤트 루프

정답은 이벤트 루프이다. Node.js는 이벤트 루프 기반의 비동기 방식으로 동작한다고 소개한다. Non-Blocking IO를 지원한다고도 한다.(많이 들엇움 ㅎㅡㅎ)

이 말이 즉 자바스크립트는 이벤트 루프를 이용해서 비동기 방식으로 동시성을 지원한다는 의미이다. 여기서 주의해야할 것은, ECMAScript에서 동시성을 지원해주는 것이 아니라는 것이다. 이벤트 루프를 사용하는 것은 자바스크립트 엔진이 아닌 자바스크립트 엔진을 구동하는 환경, 브라우저나 Node.js이다.

* Javascript : 넷스케이프에서 브라우저를 동적으로 사용하고 싶었다. 그래서 만든 언어이다. 다른 브라우저도 Javascript를 사용하기 시작하였다. (ECMAScript + BOM + DOM)
* ECMAScript : 여러 브라우저가 사용하면서 표준을 세우게 되었는데, 이것이 ECMAScript이다.하지만 표준을 따르는 것은 브라우저마다 차이가 있다. -> 자바스크립트 엔진의 차이
* 자바스크립트 엔진 : V8, SpiderMonkey 같은 Javascript 코드를 실행하는 인터프리터. 자바스크립트 엔진은 메모리 힙(Memory Heap)과 호출 스택(Call Stack)을 가지고 있다.

잠시 기본기 좀 확인을 하였다......자바스크립트 엔진이 이벤트 루프 관련된 일을 하는 것이 아니다. 자바스크립트 엔진(V8, SpiderMonkey 등)은 작업 요청이 들어왔을 때, **호출 스택**에 쌓이고, 이를 통해 순차적으로 실행을 시키는 일만 한다.

즉, **자바스크립트의 동시성을 지원하기 위한 비동기 요청 처리는** 자바스크립트 엔진을 구동하는 환경인 브라우저나 Node.js가 담당하고 있다.

<img width="627" alt="Screen Shot 2019-12-15 at 8 48 47 PM" src="https://user-images.githubusercontent.com/39291812/70862182-5cb51b00-1f7c-11ea-9f56-3d1b416e5a79.png">

위 이미지는 브라우저 환경을 그림으로 표현한 것이다. 위 그림에서도 우리가 비동기 호출을 위해 사용하는 함수들은 자바스크립트 엔진 외부에 있는 `Web API` 영역에 정의되어 있는 것을 확인할 수 있다. 또한 이벤트 루프, 태스크 큐도 자바스크립트 엔진 외부에 구현되어 있다.

<img width="600" alt="Screen Shot 2019-12-15 at 9 02 43 PM" src="https://user-images.githubusercontent.com/39291812/70862323-43ad6980-1f7e-11ea-9119-0f3ee933b88a.png">

Node.js 또한 브라우저 환경과 비슷하다. Node.js는 비동기 IO를 지원하기 위해 `libuv` 라이브러리를 사용하여 이벤트 루프를 사용한다.(`libuv`가 이벤트 루프를 제공한다.) 자바스크립트 엔진은 비동기 작업을 수행하기 위해 Node.js API를 호출하는데, 이때 넘겨지는 콜백은 `libuv`의 이벤트 루프를 통해 스케쥴 및 실행된다.

즉, 여기서 명심해야할 것은, 자바스크립트는 단일 스레드 기반의 언어이다. 라는 말은, 자바스크립트 엔진이 단일 호출 스택을 사용한다라는 관점에서만 적용된다는 것이다. 

사실 자바스크립트가 구동되는 환경에서는 여러 개의 스레드가 사용된다. 이러한 환경이 단일 호출 스택을 사용하는 자바스크립트 엔진과 연동되기 위해서 사용하는 장치가 이벤트 루프라는 것이다.

### 단일 호출 스택

자바스크립트의 함수가 실행되는 방식은 Run to Completion이라고 말하는데, 이는 하나의 함수가 실행되면 이 함수의 실행이 끝날 때까지는 다른 작업이 중간에 끼어들지 못하는 것을 의미한다.

자바스크립트는 하나의 호출 스택을 사용하고, 현재 스택에 쌓여있는 함수들이 모두 실행이 끝나고 스택에서 제거될 때까지 다른 함수가 실행될 수 없다.

```javascript
function foo() {
    bar();
    console.log('foo');
}

function bar() {
    console.log('bar');
}

function baz() {
    console.log('baz');
}

setTimeout(baz, 10);
foo();

// bar -> foo -> baz
```

위 코드에서 foo() 함수가 먼저 호출되고 나서 baz()함수가 찍힌다는 것은 명백하다.

1. setTimeout 함수는 브라우저에게 타이머 이벤트를 요청한 후 바로 스택에서 제거된다. 
2. foo 함수가 스택에 추가된다.
3. foo 함수에서 실행되는 함수가 차례로 추가된다. (bar 함수, console.log)
4. bar 함수가 추가된 후, bar 함수에서 실행할 함수가 추가되고, 더 들어갈 함수가 없는 경우 실행시키고 제거된다.
5. 그렇게 foo 함수가 내부적으로 실행하는 함수가 제거되고, foo 함수도 제거된다.
6. 그렇게 호출 스택이 비워지게된 후, baz 함수가 스택에 추가되어 함수를 실행시키고 제거된다.

### Event Queue(Task Queue)

위 코드에서, setTimeout 함수는 왜 호출 스택이 모두 비워진 후 실행될 수 있을까?

**태스크 큐와 이벤트 루프이다. `Task Queue`는 콜백 함수들이 대기하는 큐 형태의 배열이다. 이벤트 루프는 호출 스택이 비워질 때마다 태스크 큐에서 첫 번째 콜백 함수(Task)를 가져와서 실행하는 역할을 한다.**

이벤트 루프는 호출 스택에 현재 실행중인 Task가 없는지, 태스크 큐에 Task가 있는지를 반복적으로 확인한다. 

정리하면, 

* 모든 비동기 API는 작업이 완료되면 콜백 함수(Task)를 태스크 큐에 추가한다.
* 이벤트 루프는 현재 실행중인 테스크가 없을 때(호출 스택이 비워졌을 때) 태스크 큐에서 첫 번째 태스크를 가져와서 실행한다.

## 이벤트 종류

* UI Event
* Keyboard Event
* Mouse Event
* Focus Event
* Form Event
* Clipboard Event

## 이벤트 핸들러 등록

* 인라인 이벤트 핸들러 방식 : HTML 요소의 이벤트 핸들러(onClick 같은) 메소드에 이벤트 핸들러를 등록하는 방법 -> 직접적으로 특정 프로퍼티에 함수가 할당되는 것이다. 여러 함수 할당이 가능하다. (권장X)
* 이벤트 핸들러 프로퍼티 방식 : 인라인처럼 HTML과 javascript 코드가 뒤섞이지는 않지만(단점 해결) 특정 이벤트 핸들러 프로퍼티(onclick)에 하나의 이벤트 핸들러(함수)만 바인딩할 수 있다.
* addEventListener 메소드 방식 : 해당 메소드를 이용하여 대상 DOM 요소에 이벤트를 바인딩하고, 발생했을 때 실행될 콜백 함수를 지정하는 방식이다.

`addEventListener` 함수 방식은 다른 방식과는 다른 장점이 있다.

### addEventListener 메소드 방식

- 하나의 이벤트에 하나 이상의 이벤트핸들러를 추가할 수 있다.
- 캡쳐링, 버블링을 지원한다.
- 모든 DOM 요소에 동작할 수 있다. (HTML, XML, SVG) 브라우저는 웹 문서를 로드 한 후에 파싱하여 DOM을 생성한다.
- 해당 메소드는 IE9 이상에서 동작한다. 이전은 attachEvent 메소드를 사용.

```html
<!DOCTYPE html>
<html>
<body>
  <label>User name <input type='text'></label>
  <script>
    const input = document.querySelector('input[type=text]');

    input.addEventListener('blur', function () {
      alert('blur event occurred!');
    });

    addEventListener('click', function () {
      alert('Clicked!');
    });
  </script>
</body>
</html>
```

`addEventListener` 메소드를 사용할 때 대상 DOM 요소를 지정하지 않으면, window(DOM문서를 포함하고 있다.)에서 발생하는 click 이벤트에 해당 콜백함수를 바인딩한다. 그래서 DOM 전체를 가리키고있는 콜백함수가 해당 이벤트에 묶여, DOM 전체에서 해당 이벤트가 발생했을 때, 콜백함수가 무조건 일어나게 된다.

그래서 보통 특정 DOM 요소에서 `addEventListener` 메소드를 사용하게 된다. 그러면 해당 요소에서 발생하는 특정이벤트에 콜백함수를 바인딩하게 할 수 있다. 그렇게 되면 해당 요소의 특정이벤트가 실행되었을 때 지정했던 콜백함수가 실행된다.

`elem.addEventListener('click', function () { alert('click event occurred') })`

addEventListener의 사용방법 (첫 번째 파라미터는 특정 이벤트명, 두 번째 파라미터는 해당 이벤트에 바인딩할 콜백함수)

여기서 콜백함수를 호출하는 형식이 아니라, 함수 자체를 지정하도록 한다. (호출하면, 이벤트 발생과 상관없이 실행되어버린다.) 이렇게 할 경우, 해당 콜백함수에 인자를 전달할 수 없는 문제가 나타나는데, 이는 함수를 밖에 하나 더 만들어주고, 안에서 인자를 전달할 함수를 써주면 되겠다.

```javascript
const number = 0;
input.addEventListener('click', function() {
    onClick(number);
})
```

## 이벤트 핸들러 함수의 this

### 인라인 이벤트 핸들러

인라인이벤트핸들러는, 일반 함수로서 호출되므로 **this는 전역객체인 window**를 가리킨다.

```html
<!DOCTYPE html>
<html>
<body>
  <button onclick="foo()">Button</button>
  <script>
    function foo () {
      console.log(this); // window
    }
  </script>
</body>
</html>

```

### 이벤트 핸들러 프로퍼티

이벤트핸들러 프로퍼티에서 이벤트핸들러는 메소드이다. 그래서 이의 **this는 해당 이벤트에 바인딩된 DOM 요소**에 해당한다.

`this === e.currentTarget`

```html
<!DOCTYPE html>
<html>
<body>
  <button class="btn">Button</button>
  <script>
    const btn = document.querySelector('.btn');

    btn.onclick = function (e) {
      console.log(this); // <button id="btn">Button</button>
      console.log(e.currentTarget); // <button id="btn">Button</button>
      console.log(this === e.currentTarget); // true
    };
  </script>
</body>
</html>
```

### addEventListener 메소드

`addEventListener` 메소드에서 콜백함수로 지정하는 방식의 이벤트 핸들러 내부의 this도 마찬가지로 이벤트리스너에 바인딩된 DOM 요소를 가리킨다.

그래서 해당 this는 이벤트 객체의 currentTarget 프로퍼티와 같다.

```html
<!DOCTYPE html>
<html>
<body>
  <button class="btn">Button</button>
  <script>
    const btn = document.querySelector('.btn');

    btn.addEventListener('click', function (e) {
      console.log(this); // <button id="btn">Button</button>
      console.log(e.currentTarget); // <button id="btn">Button</button>
      console.log(this === e.currentTarget); // true
    });
  </script>
</body>
</html>
```

## 이벤트의 흐름

DOM은 트리구조 형식으로 구현되어있다. 그래서 특정 HTML 요소에 이벤트가 발생하면 연쇄적인 반응이 일어나는데, 이에 관련하여 이벤트 전파 방향에 따라 **버블링**과 **캡처링**으로 나눌 수 있다.

### 이벤트 버블링

**이벤트 버블링**이란, DOM 트리 구조상에서 이벤트가 발생한 자식(하위) 요소부터 최상위 요소(body)까지 전달되어 가는 특성이다.

```html
<body>
	<div class="one">
		<div class="two">
			<div class="three">
			</div>
		</div>
	</div>
    <script>
        var divs = document.querySelectorAll('div');
        divs.forEach(function(div) {
            div.addEventListener('click', logEvent);
        });

        function logEvent(event) {
            console.log(event.currentTarget.className);
        }
    </script>  
</body>
```

위의 코드를 실행하고 div 태그 중 className이 "three"를 클릭하면 콘솔에는 아래와 같이 찍힌다.

```
three
two
one
```

### 이벤트 캡쳐링

**이벤트 캡쳐링**이란, 이벤트 버블링과는 반대 방향으로 진행되는 이벤트 전파 방식이다. 특정 이벤트가 발생했을 때, 최상위 요소 태그(body)에서 해당 태그를 찾아 내려가는 것을 말한다. 

```html
<body>
	<div class="one">
		<div class="two">
			<div class="three">
			</div>
		</div>
	</div>
    <script>
        var divs = document.querySelectorAll('div');
        divs.forEach(function(div) {
            div.addEventListener('click', logEvent, {
		        capture: true // default 값은 false
	        });
        });

        function logEvent(event) {
            console.log(event.currentTarget.className);
        }
    </script>  
</body>
```

이벤트 캡쳐링은 위와 같이 구현한다. addEventListener에서 옵션 객체에 capture: true를 설정한다. 그러면 해당 이벤트를 감지하려고 이벤트 버블링과 반대 방향으로 탐색한다.

그러면 콘솔에서는 아래와 같이 나타난다.

```
one
two
three
```

## Event 객체

이벤트 객체는 이벤트를 발생시킨 요소, 발생한 이벤트에 대한 정보를 제공하는 데 쓰인다. 이벤트 객체는 이벤트가 발생했을 때 동적으로 생성되고, 이벤트를 처리할 수 있는 이벤트 핸들러에 인자로 전달된다.(콜백함수)

이벤트 객체는 자동적으로 해당 이벤트핸들러에 바인딩된 함수(이벤트핸들러 함수)에 인자로 전달되기 때문에, 해당 이벤트 핸들러(콜백함수)를 선언할 때 이벤트 객체를 받아올 첫 번째 매개변수를 선언해주어야 한다.

```html
<!DOCTYPE html>
<html>
<body>
  <em class="message"></em>
  <script>
  function showCoords(e, msg) {
    msg.innerHTML =
      'clientX value: ' + e.clientX + '<br>' +
      'clientY value: ' + e.clientY;
  }

  const msg = document.querySelector('.message');

  addEventListener('click', function (e) {
    showCoords(e, msg);
  });
  </script>
</body>
</html>
```

### 이벤트 프로퍼티

* event.target

**이벤트를 발생시킨 요소**를 가리킨다.(이벤트 버블링의 가장 최하위 요소 반환) 즉, `this === e.target` e.target은 해당 이벤트핸들러에 바인딩된 요소를 가리킨다. 만약 특정 폼에 하나의 이벤트핸들러만 사용하여 e.target을 사용할 경우엔, **이벤트 위임**이라는 방법을 사용할 수 있다.

e.target을 가져올 경우, 해당 폼이나 DOM 요소(부모) 자체에 이벤트 핸들러를 바인딩하게되면, 부모 요소 아래에 해당하는 자식 요소들이 해당 이벤트를 발생시킨 경우, 자식 요소에 바인딩된 e.target을 각각 가져올 수 있다. 이것을 이벤트 위임이라고 하고, 이를 통해 코드의 반복성을 줄일 수 있겠다.

* event.currentTarget

**이벤트에 바인딩된 DOM 요소**를 가리킨다.(이벤트에 해당하는 요소 반환) 이는 즉, 어떤 DOM 요소에 묶여있는 이벤트 핸들러가 있으면, 해당 이벤트 핸들러를 일으킬 수 있는 DOM 요소, 그리고 그 하위요소들이 모두 currentTarget이 된다. 

* event.type

발생한 이벤트의 종류를 나타내는 문자열을 반환한다.

* event.cancelable

요소의 기본동작을 취소시킬 수 있는지 Boolean 값으로 나타난다.
그 후 기본동작 중단 가능시 e.preventDefault() 사용하여 기본 동작 중단시킨다.

* event.eventPhase

이벤트 흐름상 어느 단계에 있는지 반환한다.

1. 0 : 이벤트 없음
2. 1 : 캡쳐링 단계
3. 2 : 타깃
4. 3 : 버블링 단계

## 이벤트 위임 (Event Delegation)

이벤트 위임의 경우, 부모 요소로 하여금 자식 요소를 콜백함수를 통해 어떠한 일을 발생시키고 싶을 경우 사용한다. 부모 요소에 해당 이벤트 핸들러에 콜백함수를 달면, 자식 요소에게 해당 이벤트 핸들러가 전달되어 사용이 가능하다. 이를 부모 요소에 이벤트가 위임되었다고 한다.

이는 이벤트가 이벤트 흐름(버블링, 캡쳐링)으로 인해 이벤트를 발생시킨 요소의 부모 요소에도 영향을 미치기 때문에 가능하다. 이를 통해 이벤트를 발생시킨 특정 자식 요소를 알아내는 일은 `event.target`을 사용할 수 있다.

## 이벤트 메소드 중 기본 동작을 변경하는 메소드

* event.preventDefault()

대표적인 예로 form을 submit()하는 경우에 해당 버튼을 클릭하면 폼이 서버로 전송되는 기능이 되는데, 이런 요소들의 기본 동작을 중단시키기 위한 메소드이다.

* event.stopPropagation()

특정 요소의 이벤트핸들러 함수(콜백함수) 실행 후, 해당 이벤트가 부모 요소로 전파되는 것을 중단시키는 메소드이다. 해당 요소의 부모 요소에도 동일한 이벤트에 또 다른 콜백함수가 지정되어 있을 경우 사용된다.

* preventDefault와 stopPropagation

preventDefault와 stopPropagation을 동시에 실시하는 방법은,(기본 동작 중단 및 이벤트 버블링, 캡쳐링 중단 적용) 해당 이벤트핸들러의 콜백함수를 `return false` 하면된다.

 참고한 사이트
 - [https://meetup.toast.com/posts/89]
 - [https://poiemaweb.com/js-event]