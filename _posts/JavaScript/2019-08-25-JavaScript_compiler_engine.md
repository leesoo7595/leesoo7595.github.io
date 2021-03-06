---
layout: post
title: "[JavaScript] Compiler & Engine"
date: 2019-08-25
tags: [JavaScript]
comments: true
---

참고 : You don't know JS 책의 스코프 단원

## 자바스크립트 언어를 처리하는 과정?

### 기존의 컴파일러 과정

#### 토크나이징 / 렉싱

문자열을 나누어 토큰이라 불리는 조각으로 만드는 과정을 뜻한다. 

예를 들면

`var a = 2;` : `var`, `a`, `=`, `2`, `;`

#### 파싱

토큰 배열을 프로그램의 문법 구조를 반영하여 중첩 원소를 갖는 트리 형태로 바꾸는 과정이다. (무슨 소리인지 모르겟...)

파싱의 결과로 만들어진 트리를 AST(추상 구문 트리)라 부른다.

예를 들면

`var a = 2;`의 트리는 먼저 변수 선언이라 부르는 최상위 노드에서 시작하고, 최상위 노드는 `a`의 값을 가지는 확이낮와 대입 수식이라 부르는 자식 노드를 가진다.

여기서 대입 수식 노드는 `2`라는 값을 가지는 숫자 리터럴을 자식 노드로 가진다.

#### 코드 생성

AST(추상 구문 트리)를 컴퓨터에서 실행 코드로 바꾸는 과정이다. `var a = 2;`를 나타내는 AST를 기계어 집합으로 바꾸어 실제로 `a`라는 변수를 생성하고 값을 저장하는 역할을 처리한다고 본다.


### 자바스크립트 엔진은?

자바스크립트 엔진이 기존 컴파일러와 다른 점은 자바스크립트 컴파일레이션을 미리 수행하지 않아 최적화할 시간이 많지 않다는 것이다.

자바스크립트 엔진은 가능한 한 가장 빠른 성능을 내기 위해 여러 방법을 사용한다. 요약하면, 어떤 자바스크립트 코드라도 실행되려면 먼저 (바로 직전) 컴파일되어야 한다. 그리고 바로 실행할 수 있도록 한다.

### 스코프 이해

* 엔진 : 자바스크립트 프로그램 실행을 책임진다.
* 컴파일러 : 파싱과 코드 생성의 모든 잡일을 한다.
* 스코프 : 선언된 모든 변수 검색 목록을 작성 및 유지한다. 

#### 두 개의 구문으로 보는 엔진

`var a = 2;`이라는 구문을 엔진은 두 개의 서로 다른 구문으로 본다.

1. 컴파일러가 컴파일레이션 과정에서 처리할 구문
2. 실행 과정에서 엔진이 처리할 구문

첫 번째, 컴파일러는 렉싱을 통해 구문을 토큰으로 쪼갠다. 그 후 토큰을 파싱하여 트리 구조로 만든다.
그 후는 코드 생성의 처리 과정이다.

* 컴파일러가 `var a`를 만나면 스코프에게 변수 a가 특정한 스코프 컬렉션 안에 있는지 물어본다. 변수 a가 존재한다면 컴파일러는 선언을 무시하고 지나간다. 그렇지 않으면 컴파일러는 새로운 변수 a를 스코프 컬렉션 내에 선언하라고 요청한다.

* 그 후 컴파일러는 `a = 2` 대입문을 처리하기 위해 나중에 엔진이 실행할 수 있는 코드를 생성한다. 엔진이 실행하는 코드는 먼저 스코프에게 a라 부르는 변수가 현재 스코프 컬렉션 내에서 접근할 수 있는지 확인한다. 가능하면 엔진은 변수 a를 사용하고, 아니라면 엔진은 다른 곳을 살핀다. (스코프 체인)

엔진이 변수를 찾으면 변수에 값 2를 넣고, 못 찾으면 엔진은 에러를 발생시킨다. 

즉, 두 가지 동작을 취하여 변수 대입문을 처리하는데, 첫째 컴파일러가 변수를 선언하고, 둘쨰 엔진이 스코프에서 변수를 찾고 변수가 있다면 값을 대입한다. 

#### 컴파일러체

컴파일러가 생성한 코드를 실행하면 엔진은 변수 a가 선언된 적이 있는지 스코프에서 검색하는데, 이 때 엔진이 어떤 종류의 검색을 하느냐에 따라 검색 결과가 달라진다.

1. LHS 검색

변수가 대입 연산자의 왼쪽에 있을 때 수행한다. 이 때 값을 넣어야하므로 변수 컨테이너 자체를 찾는다.

예를 들면,
`a = 2;`

a에 대한 참조는 LHS 참조이다. 현재 a 값을 신경쓸 필요 없이 `= 2;` 대입 연산을 수행할 대상 변수를 찾기 때문이다.

2. RHS 검색

변수가 대입 연산자의 오른쪽에 있을 때 수행한다. 이는 단순히 특정 변수의 값을 찾는 것과 같다. (왼편이 아닌 쪽)

예를 들면, 

`console.log(a);`

a에 대한 참조는 RHS 참조이다. 구문에서 a에 아무것도 대입하지 않기 때문이다. 대신 a의 값을 가져와 console.log()에 넘겨준다.