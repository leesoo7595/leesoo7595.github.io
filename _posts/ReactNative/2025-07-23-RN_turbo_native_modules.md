---
layout: post
title: "React Native - Turbo Native Modules"
date: 2025-07-23
tags: [React Native]
comments: true
---

# Turbo Native Modules

RN으로 작업해본적이 있다면 네이티브 모듈이라는 개념에 익숙할 것이다. 네이티브 모듈은 JS와 플랫폼 네이티브 코드가 'Bridge'를 통해 JSON을 통해서 플랫폼 간의 직렬화를 처리하는 방식으로 통신할 수 있도록 해준다.

터보 네이티브 모듈은 네이티브 모듈의 다음 버전으로 몇 가지의 장점을 제공한다:

- 플랫폼 간의 일관성 있는 강력한 타입된 인터페이스
- C++로 코드를 단독으로, 또는 다른 네이티브 언어와 통합하여 작성할 수 있으므로 여러 플랫폼에서 구현을 중복할 필요가 줄어든다.
- 모듈의 lazy loading 지원으로 앱의 시작속도를 향상시킨다.
- 네이티브 코드를 위한 자바스크립트 인터페이스인 JSI를 사용하여, 기존 아키텍처의 브릿지보다 js와 네이티브 코드 간의 효율적인 통신이 가능하다.

터보 모듈을 작성하는 단계는 다음과 같다:

1. Flow나 TS를 사용해서 JS 스펙을 정의한다.
2. 작성한 스펙에서 코드를 생성하도록 디펜던시 관리 시스템을 구성한다.
3. 네이티브 코드를 구현한다.
4. 앱의 코드와 통합한다.

이 가이드는 RN 최신 버전의 터보 네이티브 모듈을 어떻게 만드는지 보여줄 것이다.

| Note
|
| You can also setup local library containing Turbo Native Module with one command. Read the guide to Local libraries setup for more details.

## How to Create a Turbo Native Module

터보 네이티브 모듈을 만들려면 먼저:

1. JS 스펙을 정의한다.
2. 모듈을 구성하여 코드젠이 스캐폴딩을 생성할 수 있도록 한다.
3. 네이티브 코드를 작성하여 모듈 구현을 끝낸다.

### 1. Folder Setup

모듈을 앱에서 분리하려면 모듈을 앱과 별도로 정의한 다음 나중에 앱에 종속성으로 추가하는 것이 좋다. 이는 나중에 오픈소스 라이브러리로 배포할 수 있는 터보 네이티브 모듈을 작성할 때도 마찬가지이다.

앱 엽에 RTNCalculator라는 폴더를 만들자. RTN은 React Native를 의미하고, React Native 모듈에 권장되는 접두사이다.

그리고 subfolders를 js, ios, android 3개 만든다.

```
TurboModulesGuide
├── MyApp
└── RTNCalculator
    ├── android
    ├── ios
    └── js
```

### 2. JavaScript Specification

새로운 아키텍처는 Flow나 TS를 사용한 JS의 타입화된 인터페이스를 요구한다. Codegen은 이 스펙을 사용하여 C++, Objective-C++, Java 등 강력하게 타입화된 언어로 코드를 생성한다.

이 스펙이 포함된 파일은 두 가지의 요구사항을 충족하여야한다:

1. 파일 이름은 `Native<MODULE_NAME>`이어야 하고, 확장자는 Flow를 사용하는 경우 `.js`, `.jsx`여야하고, TS를 사용하는 경우 `.ts`, `.tsx`여야 한다. Codegen은 이 패턴과 일치하는 파일만 찾는다.
2. 파일은 `TurboModuleRegistrySpec` 객체를 export해야한다.

```js
// @flow
import type { TurboModule } from "react-native/Libraries/TurboModule/RCTExport";
import { TurboModuleRegistry } from "react-native";

export interface Spec extends TurboModule {
  add(a: number, b: number): Promise<number>;
}
export default (TurboModuleRegistry.get<Spec>("RTNCalculator"): ?Spec);
```

```ts
import { TurboModule, TurboModuleRegistry } from "react-native";

export interface Spec extends TurboModule {
  add(a: number, b: number): Promise<number>;
}

export default TurboModuleRegistry.get<Spec>("RTNCalculator") as Spec | null;
```

생성한 파일을 `js/NativeRTNCalculator.ts`와 같은 `js` 디렉토리에 넣는다.

스펙 파일의 시작 부분에는 다음을 임포트해야한다:

- `TurboModule` : 이 타입은 모든 터보 네이티브 모듈의 기본 인터페이스를 정의한다.
- `TurboModuleRegistry` : 터보 네이티브 모듈을 로드하는 함수가 포함된 js 모듈

파일의 두 번째 섹션에는 터보 네이티브 모듈의 인터페이스 `Spec`이 포함되어 있다. 위의 예시의 경우 Promise를 반환하는 add 함수를 정의한다. 이 인터페이스 타입은 터보 네이티브 모듈의 `Spec`으로 네이밍해야한다.

마지막에는 `TurboModuleRegistry.get`에 모듈의 이름을 전달하여 invoke해야한다. 그러면 터보 네이티브 모듈이 로드된다.

### 3. Module Configuration

그 다음은 Codegen과 auto-linking을 위한 구성을 추가해야한다.

어떤 환경설정 파일은 iOS와 android를 공유하는 반면, 다른 아이들은 플랫폼별로 다르다.

**Shared**

공유되는 환경설정파일은 너의 모듈을 인스톨하는 `package.json`이다. RTNCalculator 디렉토리의 루트에 package.json을 생성한다.
