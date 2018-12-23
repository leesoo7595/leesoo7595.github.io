---
layout: post
title: "[JavaScript] 제어문"
date: 2018-12-23
tags: [JavaScript]
comments: true
---

참고도서 : 러닝 자바스크립트

## for...in

객체의 프로퍼티에 루프를 실행하도록 설계된 루프이다.

```javascript
for (variable in object)
  statement
```

```javascript
const player = {
  name: "Thomas",
  rank: "Midshipman",
  age: 25
};

for (let prop in player) {
  if (!player.hasOwnProperty(prop)) continue;
  console.log(prop + ': ' + player[prop]);
}
```

## for...of 

컬렉션의 요소에 루프를 실행하는 다른 방법이다.

```javascript
for (variable of object)
  statement
```

for...of 루프는 배열, 이터러블 객체에 모두 사용할 수 있는 범용적인 루프이다.

```javascript
const hand = [randFace(), randFace(), randFace()];
for (let face of hand)
  console.log(`You rolled...${face}!`);
```

위의 식은 배열을 for...of 루프로 돌린 것이다. 이는 배열에 루프를 실행해야 하지만 각 요소의 인덱스를 알 필요는 없을 때 알맞다. 인덱스를 알아야한다면 for 루프를 사용한다.
