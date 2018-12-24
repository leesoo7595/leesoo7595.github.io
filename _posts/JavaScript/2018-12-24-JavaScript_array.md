---
layout: post
title: "[JavaScript] Array"
date: 2018-12-24
tags: [JavaScript]
comments: true
---

참고도서: 러닝 자바스크립트

## 배열의 요소를 추가하거나 제거 - push, pop, unshift, shift

`push`나 `pop`은 배열의 끝에 요소를 추가하거나 제거한다. 이는 데이터를 수직으로 쌓아 올리는 `stack`에 해당하는 행동이다.
`unshift`나 `shift`는 배열의 처음에 요소를 추가하거나 제거한다. 이는 대기열과 비슷한 `queue`에 해당하는 행동이다.

`push`, `unshift`는 새 요소를 추가해서 늘어난 길이를 반환하고, `pop`, `shift`는 제거된 요소를 반환한다.

```javascript
const arr = ["b", "c", "d"];

arr.push("e");                // 4, arr = ["b", "c", "d". "e"]
arr.pop();                    // "e". arr =  ["b", "c", "d"]

arr.unshift("a");             // 4, arr = ["a", "b", "c", "d"]
arr.shift();                  // "a". arr = ["b", "c", "d"]
```

## 배열의 끝에 여러 요소를 추가 - concat

`concat` 메서드는 배열의 끝에 여러 요소를 추가한 사본을 반환한다. `concat`에 배열을 넘기면 이 메서드는 배열을 분해해서 원래 배열에 추가한 사본을 반환한다.
배열 원본은 변하지 않는다.

```javascript
const arr = [1, 2, 3];

arr.concat(4, 5, 6);        // [1, 2, 3, 4, 5, 6]
arr.concat([4, 5, 6]);
arr.concat([4, 5], 6);
arr.concat([4, [5, 6]]);    // [1, 2, 3, 4, [5, 6]]
```

concat은 배열을 한 번만 분해한다. 배열 안에 있는 배열을 다시 분해하지는 않는다.

## 배열 일부 가져오기 - slice

`slice` 메서드는 매개변수 두 개를 받는다. 첫 번째 매개변수는 어디서부터 가져올지, 두 번째 매개변수는 어디까지 가져올지를 지정한다. 두 번째 매개변수를 생략하면 배열의 마지막까지 반환한다. 음수 인덱스를 쓰면 배열의 끝에서부터 요소를 센다. 배열 원본은 바뀌지 않는다.

```javascript
const arr = [1, 2, 3, 4, 5];

arr.slice(3);       // [4, 5]
arr.slice(2, 4);    // [3, 4]
arr.slice(-2);      // [4, 5]
arr.slice(1, -2);   // [2, 3]
arr.slice(-2, -1);  // [4]
```

## 임의의 위치에 요소를 추가하거나 제거 - splice

`splice`는 배열을 자유롭게 수정할 수 있다. 첫 번째 매개변수는 수정을 시작할 인덱스이고, 두 번째 매개변수는 제거할 요소 숫자이다. 아무 요소도 제거하지 않을 때는 0을 넘긴다. 그 후 매개변수는 배열에 추가될 요소이다.

## 배열 안에서 요소 교체하기

`copyWithin` 메서드는 배열 요소를 복사해서 다른 위치에 붙여놓고 기존의 요소를 덮어쓴다. 첫 번째 매개변수는 복사한 요소를 붙여넣을 위치이고, 두 번째 매개변수는 복사를 시작할 위치이고, 세 번째 매개변수는 복사를 끝낼 위치이다. 이는 생략할 수 있다. concat은 원본은 건드리지 않은 채 새로운 배열로 사본을 만들지만, 이는 원본 자체를 교체한다.

```javascript
var fruits = ["Banana", "Orange", "Apple", "Mango"];
fruits.copyWithin(2,0);         // Banana,Orange,Banana,Orange
```

## 특정 값으로 배열 채우기

`fill` 메서드는 정해진 값으로 배열은 채운다.

```javascript
const arr = new Array(5).fill(1);   // arr = [1, 1, 1, 1, 1]로 초기화
arr.fill("a");                      // arr은 이제 ["a", "a", "a", "a", "a"] 이다.
arr.fill("b", 1);                   // ["a", "b", "b", "b", "b"]
arr.fill("c", 2, 4);                // ["a", "b", "c", "c", "b"]
arr.fill(5.5, -4);                  // ["a", 5.5, 5.5, 5.5, 5.5]
arr.fill(0, -3, -1);                // ["a", 5.5, 0, 0, 5.5]
```

## 배열 정렬 - reverse(), sort()

배열 요소의 순서를 반대로 바꾸는 것은 reverse()이고, 배열 요소의 순서를 올바르게 정렬하는 것은 sort()이다. sort는 정렬 함수를 받을 수 있다. 일반적으로 객체가 있는 배열을 정렬할 수 없지만, 정렬 함수를 사용하면 가능하다.

## 배열 검색 - indexOf, lastIndexOf

* `indexOf()` : 찾고자 하는 것과 정확히 일치(===)하는 첫 번쨰 요소의 `인덱스`를 반환한다.
* `lastIndexOf()` : 이는 `indexOf()`와 반대로 배열의 끝에서부터 검색한다. 일치하는 것을 찾지 못하면 둘다 -1 반환
* `findIndex()` : 보조 함수를 써서 검색 조건을 지정할 수 있다. 하지만 검색을 시작할 인덱스를 지정할 수 없다. 일치하는 것을 찾지 못하면 -1 반환
* `some()` : 조건에 맞는 요소를 찾으면 바로 검색을 멈추고 true를 반환한다.
* `every()` : 배열의 모든 요소가 조건에 맞아야 true를 반환한다.

```javascript
const arr = [1, "name", 5, "age", 17.2];

arr.indexOf(1) // 0이 나옴
arr.indexOf("age") // 3이 나옴

const findArr = [{name:"lemon", age:17}, {name:"candy", age:27}];
findArr.find(c => c.name==="candy"); // {name:"candy", age:27}
```

## map, filter

배열 메서드 중 가장 유용한 메서드이다. map은 배열 요소를 변형한다. 일정한 형식의 배열을 다른 형식으로 바꿀 때 쓰인다. map과 filter 모두 사본을 반환하며 원래 배열을 바뀌지 않는다.

```javascript
const cart = [ { name: 'Widget', price: 9.95 }, { name: 'Gadget', price: 22.95 }];
const names = cart.map(x => x.name);            // ['Widget', 'Gadget']
const prices = cart.map(x => x.price);
const discountPrices = prices.map(x => x*0.8);
```

filter는 배열에서 필요한 것들만 남긴다. 어떤 요소를 남길지 판단할 함수를 매개변수로 넘긴다.

```javascript
// 카드
const cards = [];
for (let suit of ['H', 'C', 'D', 'S'])  // 하트, 클로버, 다이아몬드, 스페이드
  for (let value=1; value<=13; value++)
    cards.push({ suit, value });

// value가 2인 카드
cards.filter(c => c.value === 2);
// [
//   { suit: 'H', value: 2},
//   { suit: 'C', value: 2},
//   { suit: 'D', value: 2},
//   { suit: 'S', value: 2}
// ]
```

## reduce

map이 배열의 각 요소를 변형한다면 reduce는 배열 자체를 변형한다. reduce라는 메서드는 보통 배열을 값 하나로 줄이는 데 쓰인다. 즉, 배열에 있는 숫자를 더하거나 평균을 구하는 것은 배열을 값 하나로 줄이는 동작이다. reduce가 반환하는 값 하나는 객체일 수도 있고, 다른 배열일 수도 있다.

```javascript
const arr = [5, 7, 2, 4];
const sum = arr.reduce((a, x) => a += x, 0);
```

1. 첫 번째 배열 요소 5에서 익명 함수를 호출한다. a의 초기값은 0이고, x의 값은 5이다. 함수는 a와 x(5)의 합을 반환한다.
2. ....
3. 마지막 배열 요소인 4에서 함수를 호출하여 총 합인 18을 반환한다. 이는 reduce의 값이고 sum에 할당되는 값이다.

위의 식에선 누적값을 0으로 설정하여 초깃값을 0으로 지정하게되어 계산하였지만, 만약 누적값을 설정해주지 않는다면 초깃값은 첫번째 배열 요소가 된다. 이렇게 해도 결과는 같다. 즉, 첫 번째 요소가 초기값이 되는 경우에는 누적값을 따로 설정해주지 않아도 된다.

## Array 메서드

map, filter, reduce는 삭제되거나 정의되지 않은 요소들에서 콜백 함수를 호출하지 안흔다.

`const arr = Array(10).map(function(x) { return 5; });`

위의 arr 요소는 전부 undefined이다.

```javascript
const arr = [1, 2, 3, 4, 5];
delete arr[2];
arr.map(x => 0);  // [0, 0, undefined, 0, 0]
```

그리고 배열 중간의 요소를 삭제하고 map을 호출하면 배열 가운데 구멍이 생긴다.

## 문자열 병합

`Array.prototype.join`은 매개변수로 구분자 하나를 받고 요소들을 하나로 합친 문자열을 반환한다.

```javascript
const arr = [1, null, "hello", "world", true, undefined];
delete arr[3];
arr.join();   // "1,,hello,,true,"
```
