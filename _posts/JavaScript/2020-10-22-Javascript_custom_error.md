---
layout: post
title: "Javascript - Custom Error"
date: 2020-10-22
tags: [JavaScript]
categories:
- Javascript
comments: true
---

## 커스텀 에러와 에러 확장

개발을 하다보면 에러 클래스를 직접 만들어서 활용하는 경우가 많이 생기게 된다. 에러 클래스를 만들 경우, 전에 정리한 글에서 계속 얘기했듯이 에러 객체 표준에 맞게 정의되어야 한다.

그것을 활용하는 방법이 `Error` 클래스를 상속받아 커스텀 에러 클래스를 만드는 것이다. 이로써 커스텀 에러 클래스 또한 에러 객체임을 식별할 수 있게 되기 때문이다.

### 에러 확장

```javascript
// 자바스크립트 자체 내장 에러 클래스
class Error {
  constructor(message) {
    this.message = message;
    this.name = "Error"; // (name은 내장 에러 클래스마다 다릅니다.)
    this.stack = [];
  }
}

class ValidationError extends Error {
  constructor(message) {
    super(message); // (1)
    this.name = "ValidationError"; // (2)
  }
}

// 사용법
function readUser(json) {
  let user = JSON.parse(json);

  if (!user.age) {
    throw new ValidationError("No field: age");
  }
  if (!user.name) {
    throw new ValidationError("No field: name");
  }

  return user;
}

// try..catch와 readUser를 함께 사용한 예시

try {
  let user = readUser('{ "age": 25 }');
} catch (err) {
  if (err instanceof ValidationError) {
    alert("Invalid data: " + err.message); // Invalid data: No field: name
  } else if (err instanceof SyntaxError) { // (*)
    alert("JSON Syntax Error: " + err.message);
  } else {
    throw err; // 알려지지 않은 에러는 재던지기 합니다. (**)
  }
}
```

`ValidationError`는 `Error` 클래스를 상속받고 있다. 여기서 `ValidationError`는 자식 생성자로 super를 호출하여야 자식 생성자가 따로 설정한 정보 이외는 상속자의 정보를 따라가게 된다. name 프로퍼티를 신경써서 에러명을 지정해놓을 필요가 있다.

### 상속 중첩

`ValidationError` 객체를 지정하는 것만으로는 에러가 포괄적일 수 있다. 그래서 한번 더 `ValidationError`를 상속 받아 더욱 구체적인 에러 클래스를 구현할 수 있다.

```javascript

class PropertyRequiredError extends ValidationError {
  constructor(property) {
    super("No property: " + property);
    this.name = "PropertyRequiredError";
    this.property = property;
  }
}

function readUser(json) {
  let user = JSON.parse(json);

  if (!user.age) {
    throw new PropertyRequiredError("age");
  }
  if (!user.name) {
    throw new PropertyRequiredError("name");
  }

  return user;
}

// try..catch와 readUser를 함께 사용하면 다음과 같습니다.
try {
  let user = readUser('{ "age": 25 }');
} catch (err) {
  if (err instanceof ValidationError) {
    alert("Invalid data: " + err.message); // Invalid data: No property: name
    alert(err.name); // PropertyRequiredError
    alert(err.property); // name
  } else if (err instanceof SyntaxError) {
    alert("JSON Syntax Error: " + err.message);
  } else {
    throw err; // 알려지지 않은 에러는 재던지기 합니다.
  }
}

```

### 에러 클래스 네이밍을 자동으로 추가하기

에러 클래스의 생성자 안에서 기본 에러 클래스를 지정하고 `this.constructor.name`을 에러 클래스의 네이밍으로 지정할 수 있도록 추가하는 방법이 있다.

```javascript
class MyError extends Error {
  constructor(message) {
    super(message);
    this.name = this.constructor.name;
  }
}

class ValidationError extends MyError { }

class PropertyRequiredError extends ValidationError {
  constructor(property) {
    super("No property: " + property);
    this.property = property;
  }
}

// 제대로 된 이름이 출력됩니다.
alert( new PropertyRequiredError("field").name ); // PropertyRequiredError

```

### 여러 에러 클래스를 감싸서 필요한 정보를 보여주는 에러를 감싸는 에러

```javascript
class ReadError extends Error {
  constructor(message, cause) {
    super(message);
    this.cause = cause;
    this.name = 'ReadError';
  }
}


function readUser(json) {
  let user;

  try {
    user = JSON.parse(json);
  } catch (err) {
    if (err instanceof SyntaxError) {
      throw new ReadError("Syntax Error", err);
    } else {
      throw err;
    }
  }

  try {
    validateUser(user);
  } catch (err) {
    if (err instanceof ValidationError) {
      throw new ReadError("Validation Error", err);
    } else {
      throw err;
    }
  }

}

try {
  readUser('{잘못된 형식의 json}');
} catch (e) {
  if (e instanceof ReadError) {
    alert(e);
    // Original error: SyntaxError: Unexpected token b in JSON at position 1
    alert("Original error: " + e.cause);
  } else {
    throw e;
  }
}
```

위의 ReadError 객체는 `cause`라는 아이를 받음으로써 ReadError 안에서 어떤 종류의 에러로 인해 발생했는지를 다 수집하여 하나로 모아서 처리할 수 있게 된다.

## Reference

* [커스텀 에러와 에러 확장](https://ko.javascript.info/custom-errors)