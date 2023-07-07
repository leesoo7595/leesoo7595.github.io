---
layout: post
title: "Playwright 시작해보기"
date: 2023-07-05
tags: [E2E, playwright]
comments: true
---

이 글은 [Playwright 공식문서](https://playwright.dev/docs/intro#whats-installed)를 읽고 정리한 글 입니다.

# Installation

Playwright 테스트는 E2E 테스트를 수행하기 위해 만들어졌다. Playwright는 Chromium, WebKit, Firefox를 포함해서 모든 최신 [렌더링 엔진](https://developer.mozilla.org/en-US/docs/Glossary/Rendering_engine)을 지원한다. Windows, Linux, macOS, 로컬이나 CI, 헤드리스나 안드로이드 네이티브 모바일 에뮬레이션과 모바일 사파리에서 테스트를 할 수 있다.

## Installed

```zsh
npm init playwright@latest
```

- 타입스크립트 사용여부 체크(default: typescript)
- 테스트 파일들을 담을 폴더 이름 정하기(프로젝트 안에 테스트 폴더를 가지고 있을때는 tests 또는 e2e로 되어있음)
- CI에서 쉽게 테스트 돌릴 수 있도록 깃헙액션 워크플로우 추가할건지
- playwright 브라우저 설치여부(default: true)

`playwright.config.ts`에서 playwright를 사용하기 위한 구성을 추가할 수 있다. 이미 존재하는 프로젝트 내에서 테스트를 실해하면 종속성이 package.json에 직접 추가된다.

`tests` 폴더는 테스트를 시작하는데 도움을 주기 위한 기본적인 예제 테스트를 포함하고 있다. 더 자세한 예제는 tests-example 폴더에 포함되어있는 todo app 테스트를 통해 확인할 수 있다.

## Running the Example Test

기본적으로 테스트는 3개의 워커를 사용해서 chromium, firefox, webkit 3개의 브라우저에서 돌아간다. 이것 또한 `playwright.config` 파일에서 설정을 할 수 있다. 테스트는 헤드리스 모드로 돌아간다. 헤드리스 모드란 테스트를 실행시킬 때 브라우저를 열지 않는다는 의미이다. 테스트 결과와 테스트 로그는 터미널에 보여진다.

```
npx playwright test
```

헤드모드 테스트, 멀티테스트, 또는 특정 테스트를 실행하려면 [해당 문서](https://playwright.dev/docs/running-tests)를 참조하기

간단히 요약하면 다양한 조건의 테스트를 할 수가 있는데, **UI Mode**로 할려면 `--ui`를 붙여서 실행해주면되고, UI Mode는 디버깅을하거나 워치모드 등등을 할때 더 나은 경험흘 할 수 있다.

**커맨드라인**으로는 모든 테스트를 돌리거나 하나의 파일, 폴더단위, 또는 파일이름이나 테스트 제목으로 테스트를 선택해서 할 수 있다. `headed mode`의 경우는 `--headed` 옵션을 붙여서 돌리면 되고, 특정 프로젝트를 돌리는 옵션은 `--project=chromium`식으로 할 수 있다.

**디버깅 옵션**도 가능, `--debug` 옵션을 붙이면 된다. 특정 코드라인만을 디버깅할 수도 있다. [디버깅 가이드](https://playwright.dev/docs/debug)를 체크해서 Playwright Inspector이나 브라우저 개발자 도구로 디버깅하는 방법 등을 알아볼 수 있다.

## HTML Test Reports

**HTML Reporter**는 테스트에 대한 full report를 보여주고, 이는 브라우저, 패스한 테스트, 실패한 테스트, 건너뛴 테스트, 불완전한 테스트를 기준으로 보고서를 필터링할 수 있다. 기본적으로 일부 테스트가 실패한 경우 자동으로 HTML Reporter가 열리도록 되어있다.

```
npx playwright show-report
```

## System requirements

- Node.js 16+
- Windows 10+, Windows Server 2016+ or Windows Subsystem for Linux (WSL).
- MacOS 12 Monterey or MacOS 13 Ventura.
- Debian 11, Ubuntu 20.04 or Ubuntu 22.04.

# Writing tests

Playwright는 심플하다.

- actions를 수행하고
- 기대와 다른 상태를 assert한다.

주저리주저리 써있는데 playwright가 테스트 시간이 빠른 이유~~ 이런게 적혀있고..

```typescript
import { test, expect } from "@playwright/test";

test("has title", async ({ page }) => {
  await page.goto("https://playwright.dev/");

  // Expect a title "to contain" a substring.
  await expect(page).toHaveTitle(/Playwright/);
});

test("get started link", async ({ page }) => {
  await page.goto("https://playwright.dev/");

  // Click the get started link.
  await page.getByRole("link", { name: "Get started" }).click();

  // Expects the URL to contain intro.
  await expect(page).toHaveURL(/.*intro/);
});
```

## Actions

### navigation

대부분의 테스트는 페이지의 URL을 알려주는 것부터 시작할 것이다. 그러고 나서 페이지의 요소들과 상호작용이 가능하기 때문이다.

```typescript
await page.goto("https://playwright.dev/");
```

Playwright은 페이지가 로드 상태에 도달할 때까지 기다렸다가 앞으로 이동한다.

### interactions

actions을 수행하려면 요소의 위치를 찾는 것부터 시작한다. playwright는 Locators API를 사용한다. Locators는 페이지에서 언제든지 요소를 찾을 수 있는 방법을 가지고 있으며, 사용 가능한 다양한 유형의 Locators를 알아볼 수 있다. playwright는 요소를 실행할 수 있을 때까지 기다렸다가 동작을 수행하므로, 사용할 수 있게 될 때까지 기다려야하는 신경을 쓰지 않아도 된다.

```typescript
// Create a locator.
const getStarted = page.getByRole("link", { name: "Get started" });
// Click it.
await getStarted.click();
```

대부분의 케이스에서는 한줄으로 바로 작성할 수 있다.

```typescript
await page.getByRole("link", { name: "Get started" }).click();
```

### Basic actions

playwright에서는 수많은 actions 함수들이 있다. Locator API를 통해서 더 많은 actions들을 알아볼 수 있다.

<img width="861" alt="스크린샷 2023-06-26 오전 11 54 02" src="https://github.com/leesoo7595/leesoo7595.github.io/assets/39291812/34439981-4053-4bcd-ae2e-559cfa1e6ac0">

## Assertions

playwright는 `expect` 함수 형태의 테스트 어설션을 포함하고 있다. assertion을 만들려면, `expect(value)`를 호출하고 예상하는 결과를 확인할 수 있는 **매쳐**를 고른다.

`toEqual`, `toContain`, `toBeTruthy`와 같은 assert 조건에 따라 사용할 수 있는 많은 제네릭 매쳐들이 있다.

```typescript
expect(success).toBeTruthy();
```

playwright은 예상하는 조건을 만나려면 기다려야하는 경우에 사용하는 async 매쳐도 포함한다. 이러한 매쳐들을 사용하면 테스트가 흔들리지않고 탄력적으로 진행될 수 있다.

```typescript
await expect(page).toHaveTitle(/Playwright/);
```

위 코드는 Playwright라는 제목을 가진 페이지를 만날때까지 기다리는 코드이다.

<img width="557" alt="스크린샷 2023-07-05 오후 7 27 47" src="https://github.com/leesoo7595/leesoo7595.github.io/assets/39291812/d85e3f83-393d-42e3-8633-032bf30881e6">

### Test Isolation

playwright 테스트는 test fixtures 개념을 기반으로 한다. Test fixtures는 각 테스트를 위한 환경을 설정하는데 사용되고, 테스트에 필요한 모든 것을 제공한다. Test fixtures는 테스트 간에 환경이 독립적으로 되어있다. fixtures를 사용하면, 일반적인 설정 대신 테스트의 의미에 따라 그룹화가 가능하다.

자세한건 다음 포스팅에서..

### Using Test Hooks

다양한 test hooks를 사용할 수 있다. 예를 들면 `test.describe`를 사용해서 테스트 그룹을 선언하고, 각 테스트 전후에 실행되는 `test.beforeEach`나 `test.afterEach`를 선언할 수 있다. 반대로 모든 테스트 전후에 실행되는 beforeAll, afterAll도 있음

## Test Generator

playwright는 테스트를 즉시 생성할 수 있는 기능을 제공하고, 이를 통해 빠르게 테스트를 시작할 수 있게 해준다. 테스트하려는 웹사이드와 인터렉션할 수 있는 브라우저 창과 테스트를 기록, 복사, 삭제 등등이 가능한 playwright inspector 창이 열린다.

-

### Running Codegen

```
npx playwright codegen demo.playwright.dev/todomvc
```

codegen 명령어를 사용하여 test generator를 실행할 수 있으며, 옵셔널하게 URL을 입력할 수 있다.

## Trace viewer

playwright 트레이스 뷰어는 GUI 도구이다. 테스트의 각 동작을 앞뒤로 이동하여 동작 중 어떤 일이 발생했는지 시각적으로 확인할 수 있다.

### Recording a Trace

기본적으로 `playwright.config` 파일엔 `trace.zip` 파일을 생성하는 데에 필요한 구성이 포함되어있다. Traces는 `on-first-retry` 옵션으로 설정되어있는데, 이는 실패한 테스트의 첫 번째 재시도시 실행된다. 이 뜻은 traces는 실패한 테스트의 첫번째 재시도엔 기록되지만, 첫번째 실행이거나 두번째 재시도땐 그렇지 않다.

```typescript
import { defineConfig } from '@playwright/test';
export default defineConfig({
  retries: process.env.CI ? 2 : 0, // set to 2 when running on CI
  ...
  use: {
    trace: 'on-first-retry', // record traces on first retry of each test
  },
});
```

trace를 기록하는데에 적용가능한 옵션들을 보려면 [여기](https://playwright.dev/docs/trace-viewer)를 보자

로컬에서 디버깅 메서드를 사용해서 테스트가 디버깅이 가능하기에 traces는 일반적으로 CI 환경에서 실행된다. 그래도 만약 로컬에서 trace를 돌리고 싶다면 `--trace on` 옵션을 추가하면 된다.

```
npx playwright test --trace on
```

### Viewing the Trace

각 작업을 클릭하거나 타임라인을 사용해서 마우스를 가져가면 테스트의 trace를 확인하고 작업 전후의 페이지 상태를 확인할 수 있다. 테스트의 각 단계에서 로그, 소스, 네트워크를 inspect할 수 있다. trace viewer는 DOM 스냅샷을 생성하므로 개발도구를 열거나 완전히 인터렉팅이 가능하다.
