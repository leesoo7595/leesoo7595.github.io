---
layout: post
title: "프론트엔드 개발환경의 이해를 듣고 정리하는 웹팩 기초"
date: 2020-10-04
tags: [Webpack]
categories:
- TIL
- Webpack
comments: true
---

이 게시글은 인프런 김정환 강사님의 **프론트엔드 개발 환경의 이해**라는 강의를 듣고 정리한 글 입니다.

## 배경

기본적으로 자바스크립트 모듈이란 필요한 라이브러리나 어떠한 상황으로 인해 다른 경로에 존재하는 코드를 해당 파일에 삽입하여 사용하게 될 경우 `import/export` 라는 문법을 사용하여 외부 파일들을 쓸 수 있도록 지원하게 되는 것을 말한다.

대표적인 자바스크립트 모듈
* AMD
* CommonJS
* UMD

`CommonJS`는 `exports` 키워드로 모듈을 만들고 `require()` 함수로 불러 들이는 방식이다. `Node.js`에서 `CommonJS`를 사용하고 있다.

`AMD`(Asynchronous Module Definition)는 비동기로 로딩되는 환경에서 모듈을 사용하는 것이 목표다. 주로 브라우저 환경에서 `AMD`를 사용하고 있다.

`UMD`(Universal Module Definition)는 `AMD` 기반으로 `CommonJS` 방식까지 지원하는 통합 형태이다.

이렇게 여러가지의 스펙이 나오다가, ES2015에서 표준 모듈 시스템이 나와서 이를 따르도록 하고 있다. 이 표준 모듈 시스템을 사용하기 위해 바벨과 웹팩을 이용하여 표준 모듈 시스템을 맞추게 되었다. 하지만 이 표준 모듈 시스템을 모든 브라우저가 지원하지는 않고 있다. 그래서 이를 해결하기 위해 웹팩이 꼭 필요하게 되는 것이다.

## 엔트리/아웃풋

웹팩은 여러 개의 파일들을 하나의 파일로 합쳐주는 번들러 역할을 한다. 하나의 엔트리에서 연결되어있는 모듈을 모두 찾아내 하나의 결과물로 만들어 준다.

번들 작업을 하는 webpack과 웹팩 터미널 도구인 webpack-cli를 기본으로 설치하여 작업한다.

```terminal
$ npm install -D webpack webpack-cli
```

번들링을 하는 데엔 `--mode`, `--entry`, `--output` 이 세 개의 옵션은 필수적으로 필요하며, 이를 이용하여 하나의 결과물이 나올 수 있다.

```terminal
$ node_modules/.bin/webpack --mode development --entry ./src/app.js --output dist/main.js
```

위 명령어를 예시로 웹팩 번들링을 할 수 있다.

옵션 중에는 `--config`라는 옵션이 존재하는데, 이 옵션을 활용하여 웹팩 컨피그 파일을 만들어서 프로젝트에 붙여서 자동으로 웹팩 번들링을 할 수 있도록 한다.

```javascript
// webpack.config.js
const path = require("path")

module.exports = {
  mode: "development",
  entry: {
    main: "./src/app.js",
  },
  output: {
    filename: "[name].js",
    path: path.resolve("./dist"),
  },
}
```

그리고 컨피그 파일 내부에서 필수적인 옵션인 mode, entry, output을 설정하도록 한다. 그리고 `package.json`에 웹팩 번들링 실행을 위한 명령어를 추가한다.

```
// 프로젝트마다 명령어는 다를 수 있다.

{
  "scripts": {
    "build": "webpack"
  }
}
```

모든 옵션을 웹팩 설정 파일로 옮겨서 webpack 명령어만 실행하면 된다. 이렇게 스크립트로 명령어를 등록해놓으면 `npm run build`로 웹팩 작업을 할 수 있다.

## Reference

- [프론트엔드 개발환경의 이해: 웹팩(기본)](https://jeonghwan-kim.github.io/series/2019/12/10/frontend-dev-env-webpack-basic.html)

