---
layout: post
title: "[CSS] min-height, max-width"
date: 2018-12-27
tags: [CSS]
comments: true
---


## min-height

```css
div {
  min-height: 100%;
  width: 100%;  
}
```

div의 가로 길이는 무조건 원본이미지나 바깥에서 주고있는 가로의 크기의 100%이다.

하지만 min-height의 의미는 최소 원본이미지의 세로, 또는 div 바깥에서 주고있는 세로 크기의 100% 이상이 되어야한다. 즉, 적어도 100%이며 가로에 따라 더 커질 수도 있는 것

## max-width

```css
div {
  max-width: 100%;
  height: 100%;
}
```

div의 세로 길이는 무조건 원본이미지, 또는 바깥에서 주고있는 세로 크기의 100%이다.

max-width의 의미는 최대 원본이미지의 가로, 또는 바깥에서 주고있는 가로 크기여야한다. 즉 세로 길이에 따라서 더 작아질 수도 있는 것이다. 더 커질 수는 없음
