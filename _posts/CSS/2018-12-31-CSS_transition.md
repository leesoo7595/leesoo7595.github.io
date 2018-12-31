---
layout: post
title: "[CSS] Transition"
date: 2018-12-31
tags: [CSS]
comments: true
---

## transition

transition은 전환이라고 하고, 이는 효과가 변경되었을 때 부드럽게 처리해주는 CSS의 기능이다.

```css
a {
  font-size: 3rem;
  display: inline-block;
}

a:active {
  transform: translate(20px, 20px);
}
```

인라인블록이나 블록 형태에서만 tranform 기능을 사용할 수 있다. a가 실행되었을 때 20px, 20px씩 (x, y) 위치를 이동시키는 기능이다.

위 코드로 실행했을 때 갑자기 순간이동하는 것처럼 부자연스럽게 이동한다. 그래서 이를 자연스럽게 하기 위해 transition을 사용한다.

```css
a {
  font-size: 3rem;
  display: inline-block;
  transition-property: all;
  transition-duration: 1s;
}

a:active {
  transform: translate(20px, 20px);
}
```

* transition-property: all;

모든 효과에 대해서(all) 변화가 일어났을 때 적용시킨다.

* transition-duration: 1s;

1초 동안 전환이 이루어진다.
