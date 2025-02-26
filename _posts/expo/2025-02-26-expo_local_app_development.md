---
layout: post
title: "expo - local app development"
date: 2025-02-26
tags: [expo]
comments: true
---

- [local-app-development](https://docs.expo.dev/guides/local-app-development/#local-builds-with-expo-dev-client)

expo-dev-client을 설치하면, 프로젝트의 디버그 빌드에 expo-dev-client UI와 툴링이 포함되는데, 이를 development build라고 한다.

development build를 생성하면 https://docs.expo.dev/guides/local-app-development/#local-app-compilation 커맨드를 사용할 수 있다. 이 커맨드로 디버그 빌드를 생성하고 개발서버를 시작할 수 있다.

Local app compilation

프로젝트를 로컬로 빌드하기 위해서는 expo cli의 커맨드를 통해 android와 ios 디렉토리를 생성한다.

- `npx expo run:android`
- `npx expo run:ios`

위 커맨드는 로컬에 설치되어있는 xcode, android sdk를 사용하여 프로젝트를 앱의 디버그 빌드로 컴파일한다.

- 위 커맨드는 빌드하기 전에 네이티브 디렉터리가 없으면 prebuild를 먼저 진행하여 네이티브 디렉터리를 만든다. 이미 존재하면 건너뛴다!
- `--device` 플래그를 추가하여 앱을 실행할 디바이스를 선택할 수 있다. 물리디바이스를 선택할 수도 있다.
- `--variant release` 또는 `--configuration Release`를 플래그로 전달하여 앱의 프로덕션 빌드를 할 수 잇다. 하지만 이러한 빌드는 signed되지않고 앱스토어에 제출할 수 없다. 프로덕션 빌드를 sign하려면 local app production 참조하기 https://docs.expo.dev/guides/local-app-production/

빌드 후 프로젝트 구성이나 네이티브 코드를 수정하려면 빌드를 다시해야한다. npx expo prebuild를 실행하면 기존 파일 위에 변경 사항이 업데이트된다. 이것은 빌드 후 다른 결과가 발생할 수도 있다.

이를 방지하려면 `.gitignore`에 네이티브 디렉터리를 추가하고 npx expo prebuild --clean을 사용하면 프로젝트를 관리할 수 있다. `--clean` 플래그는 기존 디렉터리를 다시 생성하기 전에 삭제를 먼저 한다.
