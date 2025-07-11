---
layout: post
title: "React Native - What is Codegen"
date: 2025-07-11
tags: [React Native]
comments: true
---

# Codegen

## What is Codegen?

코드젠이란 많은 반복적인 코드들을 쓰는 작업을 피하기 위한 툴이다. Codegen을 사용하는 것은 의무는 아니다: 너는 그 모든 코드를 수동적으로 작성할 수 있다. 그러나 Codegen은 스캐폴딩 코드를 생성해줌으로써 많은 시간을 절약할 수 있게 해준다.

React Native는 항상 iOS와 android가 빌드될때 코드젠을 실행한다. 주기적으로 어떤 타입과 파일들이 실제로 생성되는지 수동적으로 코드젠 스크립트를 돌려야한다: 이것은 Turbo Native Modules와 Fabric Native Components를 개발할 때 일반적인 시나리오이다.

## How does Codegen works

코드젠 작업은 RN앱과 강하게 커풀링되어있다. 코드젠 스크립트는 react-native npm 패키지와 앱이 그 스크립트를 빌드할 때 항상 돌아간다.

코드젠은 프로젝트 안에 폴더로 숨겨져있고, 너가 명시하는 package.json의 디렉토리에서부터 시작하고, 거기서 어떤 custom modules와 components를 명시한 specs를 포함한 js 파일들을 찾는다. Spec 파일들은 typed dialect로 작성된 js 파일들이다. RN은 현재 Flow, ts를 지원한다.

Codegen은 항상 spec 파일을 찾는다. 그것은 spec 파일과 연결되어있는 보일러플레이트 코드를 생성한다. 코드젠은 C++ glue-code를 생성하고 안드로이드의 java와 iOS의 object-c++ 코드로 platform-specific 코드를 생성한다.

## Reference

- (https://reactnative.dev/docs/the-new-architecture/what-is-codegen)
