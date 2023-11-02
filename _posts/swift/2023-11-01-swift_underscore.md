---
layout: post
title: "Swift : underscore"
date: 2023-11-01
tags: [Swift]
comments: true
---

## Swift - Underscore 의미

스위프트 문법에는 종종 알 수 없는 언더스코어(`_`)가 보인다.

### 함수의 인자에서 `_`

함수를 사용할 때 종종 언더스코어가 보인다.

```swift
func greet(name: String) {
  return "Hello \(name)"
}

greet(name: "Sue")
```

함수에서 인자를 넘겨줄 때 기본적으로 위와 같이 라벨을 같이 지정해주면서 호출할 수 있다.

그런데 이 라벨 작성을 생략하고 싶을 경우 언더스코어를 사용할 수 있다.

```swift
func greet(_ name: String) {
  return "Hello \(name)"
}

greet("Sue")
```

## Reference

- [Swift underscore(\_)](https://medium.com/@codenamehong/swift-underscore-90dcbec5072f)
