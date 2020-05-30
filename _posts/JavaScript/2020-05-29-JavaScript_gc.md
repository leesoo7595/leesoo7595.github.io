---
layout: post
title: "Javascript - Garbage Collection"
date: 2020-05-29
tags: [JavaScript]
categories:
- Javascript
comments: true
---

## 가비지 컬렉션

자바스크립트는 눈에 보이지 않는 곳에서 메모리 관리를 수행한다. 원시값, 객체, 함수 등 모든 것은 메모리를 차지한다. 여기서 쓸모 없는 것들의 경우, GC가 처리하게 되는데, 자바스크립트 엔진이 이 처리 과정을 어떻게 진행하는지 알아보자.

### 기준

자바스크립트는 도달 가능성(`reachability`)라는 개념을 사용하여 메모리 관리를 수행한다.

즉, **도달 가능한**이라는 뜻은 접근하거나 사용할 수 있는 값을 의미하는데, 도달 가능한 값은 메모리에서 삭제되지 않는다.

* 현재 함수의 지역 변수와 매개변수
* 중첩 함수의 체인에 있는 함수에서 사용되는 변수, 매개변수
* 전역 변수
* 기타

이런 값들의 경우 root 값이라고 한다.

root에서 참조하는 값이나 체이닝으로 root에서 참조할 수 있는 것들은 도달 가능한 값이 된다.

```javascript
// user엔 객체 참조 값이 저장됩니다.
// 도달 가능한 상태 : <global> => Object {name: John}
let user = {
  name: "John"
};

// 도달 불가능한 상태
user = null;
```

#### 참조 두 개

```javascript
// user엔 객체 참조 값이 저장됩니다.
let user = {
  name: "John"
};

// 여전히 도달 가능
let admin = user;
user = null;
```

### 알고리즘

`mark-and-sweep`이라는 가비지 컬렉션 기본 알고리즘이다.

* GC는 `root` 정보를 수집하고 이를 `mark`한다.
* `root`가 참조하는 모든 객체를 방문하고 이를 `mark`한다.
* `mark`된 모든 객체에 방문하고 그 객체들이 참조하는 객체도 `mark`한다. 한번 방문한 객체는 모두 `mark`해서 다시 방문할 일은 없다.
* 반복반복
* `mark` 되지 않은 모든 객체를 메모리에서 삭제

**자바스크립트는 코드 실행에 영향을 미치지 않으면서, GC를 더 빠르게 하는 다양한 최적화 기법을 적용한다.**

1. generational collection 

객체를 **새로운 객체**와 **오래된 객체**로 나눈다. 객체의 상당수는 생성 이후 자신의 역할을 수행하고 금방 쓸모가 없어진다. 이런 류를 **새로운 객체**로 구분하고, 메모리에서 제거한다. 일정 시간 이상 계속 쓰이는 객체는 **오래된 객체**로 분류하고 GC에서 감시를 조금한다.

2. incremental collection

**GC를 여러 조각으로 분리해서, 각 조각을 별도로 수행하게 한다.** 작업을 분리하고 변경 사항을 추적하는 데에 추가 작업이 들어가지만, 긴 지연을 짧은 지연으로 여러 개 분산 시켜 리소스를 줄일 수 있다.

3. idle-time collection 

실행에 주는 영향을 최소화하기 위해 CPU가 유휴 상태일 때만 GC를 실행한다. 

### Reference

- [모던자바스크립트 - Garbage Collection](https://ko.javascript.info/garbage-collection)