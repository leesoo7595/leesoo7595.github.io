---
layout: post
title: "Javascript - Basic"
date: 2020-05-21
tags: [JavaScript]
categories:
- Javascript
comments: true
---

## Hello, World

### Script 태그

`<script>` 태그엔 자바스크립트가 들어가게 된다. 브라우저는 해당 태그를 보면 자동으로 안의 코드를 처리하게 된다. `<script>` 태그를 사용할 때, `HTML4`에서는 기본적으로 `type`을 명시해주어야했다. 현재에는 `type` 명시가 필수가 아닌 다른 속성으로써 쓰인다. 자세한건 다음 기회에...

```html
<script type="text/javascript">
  console.log('hello')
</script>

// 옛날 자바스크립트 코드가 쓰일 때 스크립트 내부에 주석 처리는 다음과 같이 처리하였다.
<script type="text/javascript"><!--
    ...
//--></script>
```

**외부 스크립트를 삽입하는 경우**, 절대경로(root에서부터 시작)와 상대경로 모두 사용 가능하다. 현재는 거의 스크립트 파일을 html에서 받아서 사용한다고 생각하고 사용하는 것이 좋다. 즉, 매우 간단한 자바스크립트 코드일 경우에만 html 내부에 스크립트 명시를 해놓고 자바스크립트 코드를 직접 넣도록 하는 것이 좋다.

그 이유는, 스크립트를 별도의 파일로 분리하여 불러오게되면, 브라우저는 외부 스크립트를 다운받아 캐시에 저장해놓기 때문에, 성능상의 이점이 존재하기 때문이다. 즉 여러 페이지에서 동일한 외부 스크립트를 사용할 경우, 이를 한 번만 다운로드해서 반복적으로 사용하게 되어 이점이 된다.

**script 태그 안에 src가 사용되었을 때**, `src` 속성 안에 다른 외부 경로가 존재함에도 자바스크립트 코드가 내부에 있다면 내부의 코드는 무시된다. `script` 태그만 사용하여 내부에 코드를 작성하거나, `script` 태그에 `src` 속성만 적어두고 비워놓던지 두 가지 중 하나만 행해야 한다. 외부 스크립트와 내부의 코드 모두 동작하게 하려면, `script` 태그를 두 번 사용하면 된다.

### 세미콜론

코드에서는, 서로 다른 문으로는 세미콜론으로 구분하도록 되어있다. 자바스크립트에서는 줄바꿈이 있다면, 세미콜론을 생략해도 문제가 없다. 즉, 세미콜론을 자동 삽입한다. 하지만 예외적으로 자바스크립트가 세미콜론을 자동으로 삽입하지 못하는 경우가 있다.

```javascript
alert("에러가 발생합니다.");
[1, 2].forEach(alert) // 1, 2

// 세미콜론 생략한 경우
alert("에러가 발생합니다.")[1, 2].forEach(alert)
```

위의 경우, 자바스크립트가 예외적으로 세미콜론을 자동 삽입해주지 못하므로 세미콜론을 추가해줘야 에러가 나지 않을 것이다. 즉, 위 코드에서 세미콜론을 생략할 경우 하나의 단일 문으로 인식하여 코드가 돌아간다. 이런 경우가 가끔 존재하므로, 줄바꿈을 했더라도 세미콜론을 넣어주는 것이 코드의 안전에 좋다. (하지만 난 안넣지롱....)

### strict mode

ES5 전까지인 2009년까지 자바스크립트라는 언어는 기존의 기능을 변경하지 않으면서 새로운 기능이 추가되었다. 그래서 기존의 코드가 절대 망가지지 않는다는 장점이 있지만, 자바스크립트가 만들어진 당시 불안정한 것들이 계속 존재한다는 단점도 있었다. ES5부터는 새로운 기능이 추가되고 기존 기능이 변경되면서 하위 호환성에 문제가 생길 수 있게 되었는데, 이로 인해 ES5의 변경사항이 기본 모드에서는 활성화되지 않도록 해놨다고 한다. 그래서 변경사항을 적용하기 위해 `use strict`이라는 지시자를 사용한다.

* `use strict`은 스크립트의 최상단에 위치해야 한다. 
* `use strict`가 적용된 스크립트를 다시 되돌릴 수는 없다.(자바스크립트 엔진을 이전 방식으로 되돌리는)

```javascript
// use strict 가 없는 예제

num = 5; // 변수 "num"이 정의되어있지 않더라도, 단순 할당만으로 변수가 생성

alert(num); // 5

// use strict 예제인 경우
"use strict";

num = 5; // error: num is not defined
```

### 변수와 상수

* 함수형 언어

함수형 언어에서는 함수값 변경을 금지하고 있다. 스칼라와 얼랭 같은 대표적인 함수형 언어는 변수(상수) 내부에 값이 저장되면, 그 값을 영원히 유지한다. 다른 값을 저장하고 싶다면 새로운 변수를 선언해서 할당해야 한다.

### 자료형

* BigInt

```javascript
// 끝에 'n'이 붙으면 BigInt형 자료
const bigInt = 1234567890123456789012345678901234567890n;
```

* typeof

```javascript
typeof undefined // "undefined"
typeof 0 // "number"
typeof 10n // "bigint"
typeof true // "boolean"
typeof "foo" // "string"
typeof Symbol("id") // "symbol"
typeof Math // "object"  (1)
typeof null // "object"  (2)
typeof alert // "function"  (3)
```

1. Math는 자바스크립트의 내장 객체이므로 object가 출력
2. null의 결과가 object인건 언어상의 오류이다. null이 Object라는 의미가 아님. 하위 호환성을 위해 남겨둔 상황
3. typeof의 피연산자가 함수면, function을 반환한다. 하지만 function 타입은 존재하지않는다. 자바스크립트에서 function은 object이기 때문

### 브라우저와 상호작용할 수 있는 세 가지 함수

1. alert

메세지를 보여준다.

2. prompt

사용자에게 텍스트를 입력하라는 메시지를 띄워주고, 입력 필드까지 제공해준다. 확인을 누르면 `prompt` 함수는 사용자가 입력한 문자열을 반환하고, 취소 또는 `Esc`를 누르면 `null`을 반환한다.

3. confirm

사용자가 확인 또는 취소 버튼을 누를 때까지 메세지가 창에 보여진다. 확인 버튼은 true, 취소 버튼은 false를 반환한다.

위 함수들은 모두 모달을 띄워주는데, 모달이 떠 있는 동안엔 브라우저의 동작이 일시정지된다. 사용자가 창을 닫기 전까지는 다른 동작을 할 수가 없다. 그리고 해당 모달창을 수정할 수 없다.

### 형 변환

자바스크립트에서 함수, 연산자에 전달되는 값은 자동으로 타입이 맞춰서 코드가 돌아간다. 이런 과정을 **형 변환** `type conversion` 이라고 한다. 자바스크립트가 암시적으로 형 변환을 해주는 경우도 있고, 개발자가 명시적으로 원하는 타입으로 변환해주는 경우도 있다.

```javascript
// 문자형 변환
let value = true;
alert(typeof value); // boolean

value = String(value); // 변수 value엔 문자열 "true"가 저장된다.
alert(typeof value); // string

// 숫자형 변환
alert( "6" / "2" );  
// 3 숫자형으로 자동 변환 후 연산 수행
let str = "123";
alert(typeof str); // string

let num = Number(str); // 문자열 "123"이 숫자 123으로 변환됩니다.

alert(typeof num); // number

// 불린형 변환
alert( Boolean(1) ); // 숫자 1(true)
alert( Boolean(0) ); // 숫자 0(false)

alert( Boolean("hello") ); // 문자열(true)
alert( Boolean("") ); // 빈 문자열(false)
```

<img width="753" alt="Screen Shot 2020-05-22 at 1 04 18 AM" src="https://user-images.githubusercontent.com/39291812/82579075-42850600-9bc8-11ea-989a-4592ee602031.png">

