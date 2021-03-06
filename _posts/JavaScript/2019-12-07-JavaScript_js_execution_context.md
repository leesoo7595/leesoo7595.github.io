---
layout: post
title: "[JavaScript] Execution Context"
date: 2019-12-07
tags: [JavaScript]
comments: true
---

# 실행 컨텍스트와 자바스크립트의 동작 원리

## 실행 컨텍스트

실행 컨텍스트란, 자바스크립트의 핵심 원리이다. 실행 가능한 코드가 실행되기 위해 필요한 환경이라고 한다.

* 전역 코드 : 전역 영역에 존재하는 코드
* Eval 코드 : eval 함수로 실행되는 코드
* 함수 코드 : 함수 내에 존재하는 코드

자바스크립트 엔진은 코드를 실행하기 위해서 필요한 여러 정보(`변수, 함수 선언, 변수의 스코프, this`)를 가지고 있어야 한다.

## 실행 컨텍스트의 객체

실행 컨텍스트는 실행 가능한 코드를 형상화하고 구분하는 역할을 하지만, 물리적으로는 **객체의 형태**를 가진다. 또한 3가지의 프로퍼티를 소유하고 있다.

* Variable Object : vars, function declarations, arguments 등
* Scope chain : Variable object + all parent scopes
* thisValue : Context object

### Variable Object (VO / 변수객체)

실행 컨텍스트가 생성되면, 자바스크립트 엔진은 실행에 필요한 여러 정보들을 담을 객체를 생성한다. 이를 VO라고 한다. VO는 코드가 실행될 때 엔진에 의해 참조되어 코드에서는 접근할 수 없다.

VO에 담는 객체는 변수, 매개변수와 인수 정보, 그리고 함수 선언이 있다.(함수 표현식은 제외)

VO는 실행 컨텍스트의 프로퍼티이다. 그래서 값을 가진다. 어떤 값을 가르키느냐!!면 **전역 컨텍스트**의 경우와 **함수 컨텍스트**의 경우에 따라 다르다. 

* 전역 컨텍스트

VO가 유일하고, 최상위에 위치하며 모든 전역 변수, 전역 함수 등을 포함하는 전역 객체(Global Object)를 가리킨다. 전역 객체는 전역에 선언된 변수 및 전역 함수를 프로퍼티로 소유한다.

<img width="647" alt="Screen Shot 2019-12-07 at 11 32 40 PM" src="https://user-images.githubusercontent.com/39291812/70376214-e1cf7d00-1949-11ea-8c31-3b8423e7a399.png">

* 함수 컨텍스트

VO는 Activation Object(AO), 또는 활성 객체를 가리킨다. 매개변수와 인수들의 정보를 배열의 형태로 담고 있는 arguments object가 추가된다.

<img width="694" alt="Screen Shot 2019-12-07 at 11 33 19 PM" src="https://user-images.githubusercontent.com/39291812/70376222-f90e6a80-1949-11ea-9cea-5a422c7953e3.png">

### Scope Chain

스코프 체인은 해당 전역이나 함수가 참조할 수 있는 변수, 함수 선언 등의 정보를 담고 있는 **GO 또는 AO의 리스트**를 가리킨다. 현재 실행 컨텍스트의 AO를 시작으로 마지막으로는 GO를 가리킨다.

**스코프 체인은 변수를 검색하는 매커니즘이다.** 변수가 아닌 객체의 프로퍼티(메소드)를 검색하는 매커니즘(체인)은 프로토타입 체인이다.

자바스크립트 엔진은 스코프 체인을 통해 렉시컬 스코프를 파악한다. 함수 안에 또 함수가 있는 중첩 상태일 때, 부모 함수의 스코프가 자식 함수의 스코프 체인에 포함되도록 한다.

**즉, 함수 실행 중에 변수를 만날 경우, 해당 변수를 현재 스코프(AO)에서 검색하고, 실패할 경우 스코프 체인에 담겨진 순서대로 검색을 계속한다.**


### this value

실행 컨텍스트 내에서 this 프로퍼티에는 말 그대로 this값이 할당되는데, 이는 함수 호출 패턴에 의해 결정된다.

## 실행 컨텍스트 생성 과정

1. 전역 객체(GO)가 생성된다. 빌트인 객체와 BOM, DOM이 설정되어있다.
2. 전역 코드로 컨트롤이 진입하면서 실행 컨텍스트가 생성되고(전역 실행 컨텍스트), 스택이 쌓인다.
3. 스택에 쌓이는 실행 컨텍스트를 기준으로 VO, SC, this 프로퍼티에 따라서 코드를 처리한다.

### 스코프 체인의 생성과 초기화

해당 실행 컨텍스트가 생성되면, 스코프 체인의 생성 및 초기화가 제일 먼저 실행된다. **전역 컨텍스트의 경우, 스코프 체인은 전역 객체의 레퍼런스를 포함하는 리스트가 된다.** 스코프 체인과 전역 객체가 연결되어있다고 생각하면 될듯??

### Variable Instantiation (변수 객체화) 실행

스코프 체인 생성이 끝나면, 변수 객체화가 실행되게 된다.

변수 객체화는 VO에 프로퍼티 및 값을 추가하는 과정을 의미한다. 즉, 변수, 매개변수 및 인수(arguments), 함수 선언을 VO에 추가하여 객체화한다.

전역 코드의 경우엔, VO는 GO를 가리키게 된다. 리스트로 들어있는 스코프 체인과는 차이가 있는 듯 하다. VO는 직접 쓸 수 있는 변수, 매개변수, 함수 선언을 한다면, 스코프 체인은 GO에 대한 정보가 있는 차이인 듯 하다..

추가 정보 알아보기!

VO에 프로퍼티와 값을 추가하는 순서는 아래와 같다. 무조건 저 순서로 동작함!!

* 함수 코드의 경우, 매개변수는 VO의 프로퍼티, 인수(arguments)는 값으로 설정된다.
* 함수 표현식을 제외한 코드 내의 함수 선언은 함수명이 VO의 프로퍼티로 지정되고, 함수 객체는 값으로 설정된다. (함수 호이스팅)
* 변수 선언은 변수명이 VO의 프로퍼티로, 값은 undefined로 설정된다. (변수 호이스팅)

1. 함수의 선언 처리

* `[[Scopes]]`

함수 선언의 경우, 함수명이 VO의 프로퍼티, 그리고 생성된 함수 객체는 값으로 설정된다.

- 생성된 함수 객체는 `[[Scopes]]` 프로퍼티를 가지게 된다. 
- `[[Scopes]]` 프로퍼티는 함수 객체만이 소유하는 내부 프로퍼티이다.
- 함수 객체가 실행되는 환경을 가리킨다.
- 현재 실행 컨텍스트의 스코프 체인이 참조하고 있는 객체를 값으로 설정한다.
- 실행 컨텍스트 스택으로 쌓여있는 순을 의미하는 듯 : 현재 실행 환경, 현재 실행 환경을 포함하는 외부 함수의 실행 환경, 전역 객체를 가리킨다.

자신을 포함하는 외부 함수의 실행 컨텍스트는 소멸하였지만, `[[Scopes]]` 프로퍼티가 가리키는 외부 함수의 실행 환경(AO)은 소멸하지 않고 참조할 수 있는 것을 클로저라고 한다. 자세한 건 클로저 단원에서...

<img width="575" alt="Screen Shot 2019-12-09 at 12 58 41 AM" src="https://user-images.githubusercontent.com/39291812/70392064-106b5780-1a1f-11ea-9601-45f3fc1f5c1d.png">

<img width="630" alt="Screen Shot 2019-12-09 at 12 58 59 AM" src="https://user-images.githubusercontent.com/39291812/70392066-182afc00-1a1f-11ea-8b69-86bce19b3aa3.png">

2. 변수의 선언 처리

코드 내에서 선언된 변수명은 VO의 프로퍼티로 설정되며, 값은 undefined로 설정된다.

- 선언 단계 -> VO에 변수 등록, 스코프가 변수의 객체를 참조할 수 있다.
- 초기화 단계 -> VO에 등록된 변수를 메모리에 할당하고 undefined로 초기화된다.
- 할당 단계 -> 실제값 할당

var 키워드로 선언된 변수는 선언 + 초기화 단계가 동시에 이루어진다. 즉, 변수를 선언하기 전에 변수에 접근하여도 에러가 나지 않는다. VO에 변수가 존재하기 때문이다. 이를 변수 호이스팅이라고 한다.

<img width="623" alt="Screen Shot 2019-12-09 at 1 04 33 AM" src="https://user-images.githubusercontent.com/39291812/70392147-fc742580-1a1f-11ea-94b5-5ad4a6acae5c.png">

3. this value

함수 선언, 변수 선언 처리가 끝나면 this value가 결정된다. **this가 결정되기 전에, this는 전역 객체를 가리키고 있다. 그리고 함수 호출 패턴에 의해 this가 할당되는 값이 정해진다.** 전역 코드(컨텍스트)의 경우 this는 전역 객체를 가리킨다.

### 전역 코드의 실행

```javascript
var x = 'xxx';

function foo () {
  var y = 'yyy';

  function bar () {
    var z = 'zzz';
    console.log(x + y + z);
  }
  bar();
}

foo();
```
