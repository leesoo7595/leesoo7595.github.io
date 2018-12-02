---
layout: post
title: "[HTML] Text, Anchor, Image"
date: 2018-12-02
tags: [HTML]
comments: true
---

# HTML

## Block

Block 요소를 사용했을 때 이 요소가 차지하는 부분은 가로는 브라우저의 길이만큼, 세로는 글꼴의 크기만큼이다. 그러므로 Block 요소를 사용하면 그 다음 요소는 해당 부분 아랫줄에 붙게 된다.

h1, p, div 요소는 Block 형태이다.

## Inline

Inline 요소를 사용했을 땐 반대로 줄바꿈 현상이 일어나지 않고, 자신의 내용만큼의 가로너비를 가진다. 그래서 Inline 요소를 사용할 경우 그 다음 요소는 해당 부분 옆에 바로 붙게 된다.

Block 요소는 Inline 요소를 포함할 수 있지만, Inline 요소는 Block 요소를 포함할 수 없다.

strong, a, span 요소는 Inline 형태이다.

## Layout

Layout 요소는 div, span이 있다.

단순히 레이아웃을 구현하기 위해 사용된다. div는 Block 방식의 레이아웃, span은 Inline 방식의 레이아웃을 구현하는 데에 사용한다.

```html
<div>
  <p>Block 요소 내부에 <span>Inline 요소를 사용한다.</span></p>
</div>
```

## Text tags

### Heading

h1 ~ h6 태그를 의미한다. 웹 페이지의 개요를 나타낼 때 쓰인다.

### Line breaks

줄 바꾸기 태그를 의미하고, p와 br 태그가 있다.

p태그는 아래에 공백에 생기고, br테그는 줄바꿈 간에 여백이 존재하지 않는다.

### 기타

* hr태그 (Horizontal rule)

```html
<h3>Horizontal rule</h3>
<hr>
```

* blockquote태그 (인용문)

* pre태그(Performatted text, 형식화된 텍스트)

### 줄바꿈 없는 Text tags

텍스트 태그 중에서 Inline인 요소를 말한다.

* strong, b : 해당 태그는 강조, 굵게 표시하는 데에 쓰인다.

* em, i : 문맥상 특정부분 강소, 이탤릭 글씨체 표시

* mark : 형광펜 효과

## Image, Hyperlink tags

* Anchor (링크)

- a : `<a href="http://www.naver.com" target="_blank" title="네이버 열기">sdf</a>`

* Image

- img : `<img src="이미지 경로" width="100" height="200" alt="이미지 설명">`
