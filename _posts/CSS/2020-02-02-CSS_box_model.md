---
layout: post
title: "[CSS] Box Model"
date: 2020-02-02
tags: [CSS]
categories:
- CSS
comments: true
---

# Box Model

DOM에 레이아웃이 그려질 때, CSS의 표준 영역 모델(CSS basic box model)에 따라서 만들어진다. 이 영역은 size, position, 그리고 그 밖의 여러 속성들에 의해서 만들어진다.

모든 영역은 4개로 나뉘어진다. 이것은 모두 알고 있듯이

* content
* padding
* border
* margin

<img width="572" alt="Screen Shot 2020-02-02 at 3 15 08 PM" src="https://user-images.githubusercontent.com/39291812/73604131-1751b900-45cf-11ea-98cb-9e6e655ab3bd.png">

위 사진은 사실 position 주제 설명할 때랑 거의 같다.

## Content Area

`content`에는, `content edge`로 둘러쌓여 있으며, 요소의 "real" content를 담고 있는 부분이다. 즉, text, img, video 같은 컨텐츠를 품고 있는 영역이다. 이 영역은 content `width`, `height`로 이루어져있다.

### Padding Edge

`padding`은, 그림과 같이 `content` 영역 다음으로 둘러싸고 있는 영역이다. `padding`의 `width`나 `height`가 0이면, `padding edge`는 `content edge`와 같다. 

### Border Edge

`border edge`는 마찬가지로 `width`, `height`가 0일 경우엔 `padding edge`와 같다. 

### Margin Edge / outer Edge

`margin edge` 또한 위와 같이 `width`, `height`가 0이면, `border edge`와 같다.

**`background style` 같은 경우, `content`, `padding`, `border` area까지 적용이 된다. `margin`의 `background style`은 항상 투명하다.**

**또한, `padding`, `border` 모두 `width`에 포함되지 않는다는 점이다. 그래서 `content`와 `padding`, `border` 길이까지 포함한 `width`를 계산하려면 `content`의 `width`에서 더 더해주어야 한다.**

이러한 계산이 귀찮은 경우, `box-sizing: border-box`로 처리하면, 해당 요소의 `width`, `height`는 `content`의 너비가 아닌 `border`까지 포함한 길이가 된다. default로는 `box-sizing: content-box`이다.

## margin

`margin`은 다른 속성들과는 달리 해당 요소와 그 다음 요소들과의 거리를 나타낸다. 그래서 `box-sizing`과는 상관없이 `width`에 포함되지 않는다. `border`까지는 컨텐츠 영역의 테두리라고 표현되지만, margin은 그 밖에 있는 속성이다. 

### collapse margin

margin으로 요소의 바깥 길이를 줄 때, 때로는 해당 margin이 결합될 수 있다.(collapse) 해당 경우는 다음을 충족할 때 발생한다.

* adjacent siblings

특정 요소들이 같은 `containing block` 안에 존재할 때, 이를 `siblings` 관계라고 한다. 인접해있는 `siblings` 관계의 요소일 경우, 각각의 `margin`이 결합된다.

*  부모와 자식 요소 간에 분리해주는 요소가 없는 경우

부모의 `margin`과 자식의 `margin`에서 그 사이를 분리해주는 `border`, `padding`, `inline-contents`가 존재하지 않는 경우, 해당 `margin`은 결합되어 부모 바깥의 `margin`이 된다.

<img width="300" alt="Screen Shot 2020-02-02 at 4 32 01 PM" src="https://user-images.githubusercontent.com/39291812/73604780-9a780c80-45d9-11ea-8145-5db5b287d834.png">

* Empty blocks

만약, 분리되어있는 블록에서 `border`, `padding`, `inline content`나 `height` 등이 없는 상태로 `margin`을 갖고 있다면, 해당 분리된 요소들의 `margin`은 결합된다.

## 참고
- [box model - w3.org](https://www.w3.org/TR/CSS2/box.html#box-dimensions)
- [Introduction Box model - mdn](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Box_Model/Introduction_to_the_CSS_box_model)
- [CSS box model - zerocho](https://www.zerocho.com/category/CSS/post/582ddf81d4416a001860e75d)
- [mastering margin collapsing - mdn](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing)