---
layout: post
title: "[JavaScript] Template Literals"
date: 2019-01-01
tags: [JavaScript]
comments: true
---

ES6는 Template Literal이라고 불리는 새로운 문자열 표기법을 도입하였다. 템플릿 리터럴은 백틱 문자를 사용한다.

```JavaScript
const first = 'Ung-mo';
const last = 'Lee';

// ES5: 문자열 연결
console.log('My name is ' + first + ' ' + last + '.');
// "My name is Ung-mo Lee."

// ES6: String Interpolation
console.log(`My name is ${first} ${last}.`);
// "My name is Ung-mo Lee."
```

템플릿 리터럴은 + 연산자를 사용하지 않아도 간단한 방법으로 새로운 문자열을 삽입할 수 있다. 이를 문자열 인터폴레이션이라 한다.

```JavaScript
console.log(`1 + 1 = ${1 + 1}`); // "1 + 1 = 2"
```

문자열 인터폴레이션은 `${...}`로 표현식을 감싼다. 이는 문자열로 강제 타입 변환된다.
