---
layout: post
title: "[JavaScript] Hoisting"
date: 2018-12-21
tags: [JavaScript]
comments: true
---

## 함수의 호이스팅

### let

```javascript
x;          // ReferenceError: x는 정의되지 않음
let x = 3;  // 에러가 나서 실행이 멈춰 도달할 수 없다.
```

### var

```javascript
x;          // undefined
var x = 3;  
x;          // 3
```

var를 쓰면 변수를 선언하기 전에도 접근이 가능하게된다. var로 선언한 변수는 끌어올린다는 뜻의 호이스팅을 따른다.

자바스크립트는 함수나 전역 스코프 전체를 살펴보고 var로 선언한 변수를 맨 위로 끌어올린다.

여기서 중요한 것은 선언만 끌어올려진다는 것이고, 할당은 끌어올려지지 않는다.

```javascript
var x;      // 선언(할당은 아닌)이 끌어올려진다.
x;          // undefined
x = 3;
x;          // 3
```
