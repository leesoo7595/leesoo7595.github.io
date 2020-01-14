---
layout: post
title: "Design and Programming"
date: 2020-01-14
tags: [Data Structure]
comments: true
---

# 좋은 소프트웨어의 설계

* Correctness
* Robustness
* Flexibility
* Usability and Reusability
* Efficiency

## Object-Oriented Design

설계라는 것은, 객체지향적인 디자인적으로 하는 것이다. 현실 세계에서 존재하는 추상적인 개념을 어떻게 프로그래밍화하느냐?라는 고민에서 객체지향적인 설계를 통해 한다라는 의미에서 나왔다.

특정 개념을 프로그래밍하기 위해 특성을 생각하여 기준을 세우도록 한다.

**추상화라는 것은 우리가 관심을 가지고 있는 것을 목적에 맞게끔 해당 기능으로만 간략화한 것을 의미한다.**

### Class & Instance

Class는 하나의 디자인이다. 설계도라고도 설명할 수 있다. 그래서 Class에는 컨셉, 즉 추상화를 가지고 있다. 이 디자인을 활용하여 여러개의 Instance를 만들어낼 수 있다. 이 Instance는 모두 같은 컨셉을 가지고 있게 된다.

**추상화를 통해서 균일하게 설계된 여러 개의 인스턴스를 가지게 되는 것을 객체지향 설계라고 한다.**

# UML notation

소프트웨어 디자인은 설계와 같다. 여기서 여러 명과 소프트웨어 설계에 관련해서 의사소통이 필요할 때, UML이라는 표준을 가지고 어떻게 소프트웨어 설계를 더 잘할 수 있을지 고민하게 된다.

## Class 설계에 필요한 요소

* Abstract Class
* Class
* Instance
    - Named
    - UnNamed
* Attribute / Property / Method

## Encapsulation

* Object = Data + Behavior

특정 객체의 내부가 어떻게 동작이 되고 있는지와는 상관없이, 해당 기능만을 이용해서 데이터를 변형할 수 있다는 의미이다.

* Utilizing the visibility

접근 제한자를 사용하여 특정 데이터를 변형할 수 있는 권한을 나눈다.

++ 자바스크립트의 경우, 정말 얼마전에 접근제한자가 생겼다!!!

[자바스크립트 접근제한자 쓰는 방법](https://github.com/tc39/proposal-private-methods)

- private
- protected
- public

하나의 클래스 속에서도, 여러 기능들을 정확히 구분짓는 행위가 꼭 필요하다.

## Inheritance

상속! 부모 클래스를 상속하면, 부모가 가지고 있는 메소드를 그대로 사용할 수 있다. 그리고 자식 클래스는 또 다른 새로운 메소드를 가질 수 있다. 그리고 부모가 가지고 있는 메소드를 자식이 재활용하여 좀 다르게 가질 수 있다.

* superclass : 부모 객체
* subclass : 자식 객체

파이썬의 경우, `Multiple Inheritance`가 가능하다. 

```python
class Fater(Object): 
    ...

class Mother(Object):
    ...

class Child(Father, Mother):
    ...
    
```

자바스크립트는 다중상속을 할 수 없지만, 흉내는 낼 수 있따..
안하는 것을 추천!

아는척 -> -> **자바스크립트의 내적 동질성!**

```javascript
const Parent = class {
	wrap() {
		this.action();
	}
	action() {
		console.log('Parent');
	}
}
const Child = class extends Parent{
	action() {
		console.log('Child');
	}
}

const a = new Child();
a.action();
a.wrap();
```

from 태민님 notion에서....
