---
layout: post
title: "[CSS] Transition"
date: 2018-12-31
tags: [CSS]
categories:
- CSS
comments: true
---

# Positioning Schemes

CSS 2.1 버전에서는, 3 가지의 positioning shemes에 따라 해당 박스의 레이아웃이 결정된다.

1. Normal flow : `normal flow`는 블록 레벨의 영역에는 `block formatting`을 포함하고, 인라인 레벨의 영역에는 `inline formatting`을 포함한다. 그리고 `relative positioning`에는 블록 레벨과 인라인 레벨의 영역을 포함한다.
2. Floats : float 형태는, 기존 영역이 `normal flow`를 따라가지만, 그 상태에서 왼쪽이나 오른쪽으로 이동할 수 있다. 
3. Absolute positioning : `absolute positioning` 모델은, 해당 박스의 기존에 따라가는 `normal flow` 속성을 모두 없애버리고, 해당 포함하는 블록에 새롭게 위치를 할당해준다.

## Block Formatting Context?

`Block Formatting Context`란, 페이지 상에서 블록 요소를 렌더링할 때 사용되는 CSS의 `Visual Formatting Model` 중 하나이다. 이 안에서 블록박스의 레이아웃이 결정된다. 

`Block Formatting Context`는, 가장 위에 있는 `Containing Block`을 가지고 있는 요소에서부터 시작하여 박스가 수직으로 순서 배치된다. 두 개의 형제 요소들이 세로로 위치될 때, 이는 `margin`으로 결정된다. `Block Formatting Context` 안에 해당하는 두 개의 인접 블록 요소 사이에 있는 세로 `margin`이 줄어든다...(먼ㄴ말이지..)

내용적으로 죽 읽어보니 `Block Formatting Context`는 어떤 요소가 영역을 가질 때, 해당 요소의 `display` 속성이 `block`인 경우 `Block Formatting Context`라는 영역을 생성하는 정도로 이해하면 될 듯 하다.

### Containing Block

`Block Formatting Context`가 뭔지 알아보려다보니, `Containing Block`이라는 개념을 숙지하는 것이 우선일 것 같다.(꼬리물기식 구글링....정말 힘들다)

`Containing Block`은 요소의 크기나 위치에 영향을 주는 영역이다. CSS를 사용할 때, 요소는 항상 박스를 생성한다.(영역) 해당 박스는 네 가지의 영역으로 나뉘어진다.

1. Context
2. Padding
3. Border
4. Margin

<img width="553" alt="Screen Shot 2020-02-02 at 1 35 39 AM" src="https://user-images.githubusercontent.com/39291812/73595466-67436800-455c-11ea-88e0-1e41f85d9236.png">

`width`, `height`, `padding`, `margin`의 속성의 값이나 `position`의 종류로 설정된 속성 값은 `Containing Block`으로부터 계산된다.

여기서, `Containing Block`은 `position`의 속성에 따라 바뀔 수 있다. 

* `static`, `relative`, `sticky`의 경우, 가장 가까운 parent의 블록 컨테이너(`inline-block, block, list-item` 등의 요소)이거나, 가장 가까운 `establishes a formatting context`(table, flex, grid, 블록 컨테이너 자기 자신)의 영역에 따라 형성된다.
* `absolute`인 경우, `position`의 속성 값이 `static`이 아닌 나머지 중 가장 가까운 parent의 내부 여백 영역에 따라 형성된다.
* `fixed`인 경우, `Containing Block`은 뷰포트나 페이지 영역으로 형성된다.
* `absolute`, `fixed`인 경우엔, 다음 조건 중 하나를 만족하는 가장 가까운 parent의 내부 여백 영역으로 형성된다.
    * `transform`이나 `perspective` 속성이 `none`이 아닌 경우
    * `will-change` 속성이 `transform`이나 `perspective` 속성인 경우
    * `filter` 속성이 `none` 인 경우
    * `contain` 속성이 `paint`인 경우

위의 경우로 Containing Block이 다르게 형성된다.

## Visual Formatting Model

CSS에서 `visual formatting model`은 DOM에서 각 요소를 변환하여, CSS의 각각 여러 formatting models에 맞게 영역을 생성한다. 해당 visual formatting model을 결정하는 요소는 다음과 같다.

* 영역의 종류나 상관관계
* Positioning Scheme(normal flow, float, absolute positioning)
* DOM구조 안의 요소와 상관관계
* 외부 요소(viewport size 등)

### 영역(Box) 생성

CSS에서 맨 처음 영역을 만들 때, 이는 `Visual formatting model`의 종류 중 하나로 생성될 것이다. 이는 `display`의 속성 값에 따라 달라진다.

#### Block Box

특정 요소가 `Block-level Box`라고 한다면, 이 요소는 `display` 속성 값이 `block`이거나, `list-item`, `table`일 때를 의미한다. 블록 요소는 `width`, `height의` 속성을 지정할 수 있고, 블록 요소 뒤에 등장하는 다른 요소는 항상 다음줄에 렌더링된다.

#### Inline Box

특정 요소가 `Inline-level Box`일 때, `display` 속성 값이 `inline`이거나, `inline-block`, `inline-table`일 때이다. 인라인 요소는 블록 요소와는 달리 줄 바꿈이 되지 않고, `width`, `height` 속성 지정이 불가능하다. 글 자체에 효과를 주기위한 단위이다.(이탤릭체, 볼드체 등) `inline-block` 요소는 `block`과 `inline`의 중간 형태인데, 줄 바꿈이 되지 않지만 `height`, `width`를 지정할 수 있다.

## Position property

드디어 position...

특정 요소가 어떤 `position` 속성이나 `float`을 가지는지에 따라 요소의 영역이 달라질 수 있다. 

<img width="497" alt="Screen Shot 2020-02-02 at 1 50 22 PM" src="https://user-images.githubusercontent.com/39291812/73603205-fd11de00-45c2-11ea-938b-a15949b9fd59.png">

### static

이 영역은 표준 영역에 해당하며, `normal flow`를 따른다고 말할 수 있다. `static` 속성은 top, right, bottom, left 속성이 적용되지 않는다.

### relative

해당 속성은 `normal flow`의 상태인 영역 내에서 `static`과 같이 배치를 하지만, 해당 요소를 기준으로 상대적인 위치(offset)를 지정해줄 수 있는 속성이다. 즉, 특정 요소의 `position` 속성이 `relative`일 때, 해당 요소의 자식 요소들은 상대적인 위치를 지정할 수 있게 해준다.

### absolute

`absolute` 속성인 요소는, `normal flow`를 따르고 있는 특징을 제거한다. 그리고 해당 레이아웃에 어떤 공간도 배정해주지 않는다. 가장 가까운 위치 지정(`relative`) parent 요소를 기준으로 배치된다. 배치되는 위치는 top, right, bottom, left의 지정된 위치로 배치되며, 지정될 Parent 요소가 없다면 최상위에 있는 `containing block`을 기준으로 배치된다. 또한 해당 요소는 z-index가 auto가 아닌 경우, `stacking context`를 생성한다. 이는 z축이 있다고 정한 html 요소의 3차원 공간이다.

### fixed

`fixed` 속성 또한 absolute와 마찬가지로 `normal flow` 특징을 제거한다. 해당 페이지에서 요소를 `static`으로 배치되는 공간은 생성되지 않는다. absolute와 차이점은, `viewport` 기준으로 지정된 위치에 배치된다는 것이다. 그래서 스크롤이 되어도 해당 위치에는 고정적으로 자리를 가지게 된다. 이 속성도 또한 `stacking context`를 생성한다. 해당 요소의 parent의 `transform`이 none이 아닌 다른 것으로 설정되면, 해당 parent의 viewport 기준이 아니라 컨테이너 기준으로 사용된다. (해당 설명은 추후 보충 설명이 더 필요할듯...)

### sticky

`sticky` 속성은 `normal flow`을 따라서 요소가 배치된다. 그리고 `relative`와 마찬가지로 top, right, bottom, left 값을 기준으로 해당 요소를 포함하는 parent의 `containing block`을 기준으로 상대적인 위치에 자리를 잡게 된다. 이 속성 또한 `stacking context`를 생성한다.

## 참고

- [w3 - css2 문서](https://www.w3.org/TR/CSS2/visuren.html#propdef-position)
- [css 레이아웃 Normal Flow & Floats](https://medium.com/@shlee1353/css-%EB%A0%88%EC%9D%B4%EC%95%84%EC%9B%83-normal-flow-floats-bee7b8da72b4)
- [containing block - mdn](https://developer.mozilla.org/ko/docs/Web/CSS/All_About_The_Containing_Block)
- [visual formatting model - mdn](https://developer.mozilla.org/ko/docs/Web/Guide/CSS/Visual_formatting_model)
- [display 속성 - ofcourse](https://ofcourse.kr/css-course/display-%EC%86%8D%EC%84%B1)
- [position - mdn](https://developer.mozilla.org/ko/docs/Web/CSS/position)
- [position sticky - w3c](https://www.w3schools.com/howto/howto_css_sticky_element.asp)
