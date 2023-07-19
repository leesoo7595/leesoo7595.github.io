---
layout: post
title: "Playwright - Global setup and teardown"
date: 2023-07-19
tags: [E2E, playwright]
comments: true
---

이 글은 playwright [공식문서](https://playwright.dev/docs/test-global-setup-teardown)를 읽고 정리한 글 입니다.

# Global setup and teardown

글로벌로 setup/teardown을 구성하는 방법은 두 가지가 있다. global setup파일을 사용하고 그 안에 `globalSetup`이나 [project dependencies](https://playwright.dev/docs/test-global-setup-teardown#project-dependencies)를 사용하는 것이다. project dependencies를 사용하는 경우, 다른 projects들이 실행되기 전에 project를 선언해주어야한다. 이 설정은 HTML 보고서에 전역 설정이 표시되고, 트레이스 뷰어에 설정 추적이 기록된다. 그리고 fixtures를 사용할 수 있어서 전역 설정을 구성하는 권장 방법이다.

## Project Dependencies

프로젝트 종속성은 다른 프로젝트가 실행되기 전에 실행되어야할 프로젝트 리스트들을 말한다. 얘네들은 글로벌 셋업 구성을 해줌으로써 하나의 프로젝트가 처음 실행될때 이를 의존할 수 있는 부분에서 유용하다. dependencies를 사용하는 것은 글로벌 설정에서 트레이스나 다른 아티팩트를 생성할 수 있다.

### Setup

첫번째로 'setup'으로 새로운 프로젝트를 생성한다. 그리고 global.setup.ts 파일을 `testProject.testMatch` 프로퍼티에 지정한다.

```typescript
// playwright.config.ts
import { defineConfig } from "@playwright/test";

export default defineConfig({
  projects: [
    {
      name: "setup",
      testMatch: /global.setup\.ts/,
    },
    // {
    //   other project
    // }
  ],
});
```

프로젝트에 testProject.dependencies 프로퍼티에 우리가 이전에 선언했던 setup 프로젝트를 배열에 담아 넣어준다.

```typescript
// playwright.config.ts
import { defineConfig } from "@playwright/test";

export default defineConfig({
  projects: [
    {
      name: "setup",
      testMatch: /global.setup\.ts/,
    },
    {
      name: "chromium",
      use: { ...devices["Desktop Chrome"] },
      dependencies: ["setup"],
    },
  ],
});
```

### Setup Example

이 예제는 project dependencies를 사용해서 애플리케이션에 로그인하고 상태를 storage에 저장하는 전역 설정을 만드는 방법을 보여준다. 여러 테스트를 돌리고 싶을 때 각 테스트마다 로그인을 하지 않아도 로그인 상태를 가질 수 있게 해주는 유용한 방법이다.

setup project는 storage state를 'playwright/.auth/user.json' 파일에 작성할 것이다. `STORAGE_STATE`는 상수로 export해서 위치 공유를 할 수 있또록 해준다. 이렇게하면 쿠키 및 로컬스토리지 스냅샷과 함께 브라우저 컨텍스트에 storageState가 적용된다.

아래 예제처럼 하면 logged in chromiun 프로젝트에는 setup 프로젝트를 종속하고, logged out chromium 프로젝트에서는 setup 프로젝트를 종속하지 않는다. 그리고 storageState 옵션도 사용하지 않았다.

```typescript
import { defineConfig } from "@playwright/test";

export const STORAGE_STATE = path.join(__dirname, "playwright/.auth/user.json");

export default defineConfig({
  use: {
    baseURL: "http://localhost:3000/",
  },
  projects: [
    {
      name: "setup",
      testMatch: /global.setup\.ts/,
    },
    {
      name: "logged in chromium",
      testMatch: "**/*.loggedin.spec.ts",
      dependencies: ["setup"],
      use: {
        ...devices["Desktop Chrome"],
        storageState: STORAGE_STATE,
      },
    },
    {
      name: "logged out chromium",
      use: { ...devices["Desktop Chrome"] },
      testIgnore: ["**/*loggedin.spec.ts"],
    },
  ],
});
```

프로젝트의 최상단 레벨에 setup 테스트를 생성해보자, 그리고 어플리케이션에 로그인하고 로그인 작업 수행후 컨텍스트를 스토리지 상태에 채운다. 그렇게 하면 한번만 로그인하면 자격증명이 `STORAGE_STATE` 파일에 저장되어 매번 테스트할 때마다 다시 로그인할 필요가 없다. `STORAGE_STATE`를 playwright 설정 파일에 임포트해서 페이시의 컨텍스트에 스토리지 상태를 저장한다.

```typescript
// global.setup.ts
import { test as setup, expect } from "@playwright/test";
import { STORAGE_STATE } from "../playwright.config";

setup("do login", async ({ page }) => {
  await page.goto("/");
  await page.getByLabel("User Name").fill("user");
  await page.getByLabel("Password").fill("password");
  await page.getByText("Sign in").click();

  // Wait until the page actually signs in.
  await expect(page.getByText("Hello, user!")).toBeVisible();

  await page.context().storageState({ path: STORAGE_STATE });
});
```

```typescript
import { test, expect } from "@playwright/test";

test.beforeEach(async ({ page }) => {
  await page.goto("/");
});

test("menu", async ({ page }) => {
  // You are signed in!
});
```

### Teardown

```typescript
// playwright.config.ts
import { defineConfig } from "@playwright/test";

export default defineConfig({
  projects: [
    {
      name: "setup db",
      testMatch: /global\.setup\.ts/,
      teardown: "cleanup db",
    },
    {
      name: "cleanup db",
      testMatch: /global\.teardown\.ts/,
    },
    // {
    //   other project
    // }
  ],
});
```
