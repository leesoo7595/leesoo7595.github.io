---
layout: post
title: 'Javascript - 이벤트 버블링과 캡쳐링'
date: 2022-03-18
tags: [JavaScript]
categories:
  - Javascript
comments: true
---

## 이벤트 버블링과 캡쳐링

일전에 이벤트 버블링과 캡쳐링에 대해서 질문을 받았는데 확실하게 대답을 하지 못했다. 그래서 다시 제대로 정리해보는 이벤트 버블링 & 캡쳐링 개념!!!

### 버블링

한 요소에 이벤트가 발생할 경우, 해당 요소의 핸들러가 동작하고, 해당 요소의 부모 요소의 핸들러 또한 동작한다. 그렇게 최상단 요소의 핸들러까지 이 과정이 반복되고, 이를 이벤트 버블링이라 한다.

여기서 최상단 요소의 핸들러까지라 함은, **document 객체를 만날때까지의** 각 요소 핸들러를 동작시킨다는 의미이다.

#### 거의 모든 요소는 버블링이 된다.

focus와 같은 이벤트의 경우 버블링이 되지 않는다.

#### event.target

부모 요소의 핸들러의 경우 이벤트가 어디서 발생했는지에 대한 정보를 알 수 있다.

이벤트가 발생한 가장 안쪽 깊은 요소의 경우 `event.target`으로 접근이 가능하다.

**event.target과 event.currentTarget(this)의 차이점?**

- event.target은 **실제 이벤트가 시작된 타깃 요소**를 말한다. 즉, 버블링이 진행되어도 변하지 않는다.
- 반대로 event.currentTarget은 현재 요소를 말한다. **현재 실행중인 핸들러**의 요소를 의미한다.

#### 버블링을 중단하는 stopPropagation

버블링은 타깃 이벤트부터 시작해서, html태그를 거쳐서 document객체를 만날때까지 각 노드에서 모두 발생한다. 어떤 이벤트는 window까지 올라간다.

이를 어떤 이벤트까지 실행시킨 이후, 이후 부모 요소에게 버블링을 중단시킬 수도 있는데, 그렇게하려면 stopPropagation을 사용하면 된다!

- **event.stopImmediatePropagation()**의 경우, 한 요소에 작동하는 이벤트핸들러가 여러개일 경우, 해당 함수만 실행하고 나머지 핸들러 & 윗 요소 핸들러까지 다 막으려면 사용!!
- stopPropagation은 꼭 이벤트를 막아야하는 경우에만 사용하는 것이 좋다!
  - 이유1) ga 같은 사용자 분석도구는 이벤트를 막으면 기록되지 않는다.

### 캡쳐링

1. 캡처링 단계 – 이벤트가 하위 요소로 전파되는 단계
2. 타깃 단계 – 이벤트가 실제 타깃 요소에 전달되는 단계
3. 버블링 단계 – 이벤트가 상위 요소로 전파되는 단계

캡처링 단계에서 이벤트를 잡아내는 방법은, 핸들러함수에서 {capture: true}를 추가해주는 방법이 있다.

```javascript
elem.addEventListener(..., {capture: true})
// 아니면, 아래 같이 {capture: true} 대신, true를 써줘도 됩니다.
elem.addEventListener(..., true)
```

```javascript
<style>
  body * {
    margin: 10px;
    border: 1px solid blue;
  }
</style>

<form>FORM
  <div>DIV
    <p>P</p>
  </div>
</form>

<script>
  for(let elem of document.querySelectorAll('*')) {
    elem.addEventListener("click", e => alert(`캡쳐링: ${elem.tagName}`), true);
    elem.addEventListener("click", e => alert(`버블링: ${elem.tagName}`));
  }
</script>
```

캡처링을 하면, html 태그부터 현재 타깃요소까지 이벤트를 캐치하면서 내려온다. 버블링은 반대로 현재 타깃요소에서 html태그까지 올라간다.

## Reference

- [이벤트 버블링과 캡쳐링](https://ko.javascript.info/bubbling-and-capturing)
