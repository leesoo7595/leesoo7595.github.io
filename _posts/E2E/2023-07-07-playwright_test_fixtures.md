---
layout: post
title: "Playwright - Test Fixtures"
date: 2023-07-07
tags: [E2E, playwright]
comments: true
---

이 글은 [Playwright 공식문서](https://playwright.dev/docs/test-fixtures)를 읽고 정리한 글이다.

# Fixtures

playwright 테스트는 test fixtures의 개념을 기반으로 한다. Test fixtures는 각 테스트에 필요한 환경을 설정하는 데에 사용되고, 테스트에 필요한 것만을 제공한다. Test fixtures는 테스트 간에 독립적인 환경을 구성한다. fixtures를 사용하면, 테스트의 의미에 따라 테스트를 그룹화할 수 있다.

**built-in fixtures**는 주로 테스트에서 반복적으로 사용되는 아이들을 미리 선언한 fixtures를 의미하고, 이것들을 테스트 함수에서 사용할 수 있다.

- page
- context
- browser
- browserName
- request

fixtures를 사용하면 before/after hooks를 사용하는데 많은 이점을 가져갈 수 있다.

- encapsulate
- reusable
- on-demand
- composable
- flexible
- grouping

### Creating a fixture

**Page object models**란, 대규모 테스트에 걸맞는 관리, 유지를 최적화하도록 구조화 할 수 있는데, 이 모델은 테스트에 적절하게 구성할 수 있도록 접근하는 방식 중 하나이다.

page 객체는 웹 어플리케이션의 한 부분을 나타낸다. e-commerce 웹 어플리케이션은 하나의 home 페이지를 가지고, 리스트 페이지와 체크아웃 페이지를 가질 것이다. 각각 페이지들이 page object models에 나타낼 수 있다.

page 객체들은 애플리케이션에 적합한 상위 수준의 API를 생성하고, 한 곳의 요소 selectors를 캡쳐하고 코드를 재사용하여 반복을 피함으로써 유지를 단순화시켜 관리를 간소화한다.

fixture를 생성하려면, `text.extend()`를 사용해서 fixture를 포함할 새로운 테스트 객체를 생성한다.

```typescript
import { test as base } from "@playwright/test";
import { TodoPage } from "./todo-page";
import { SettingsPage } from "./settings-page";

// Declare the types of your fixtures.
type MyFixtures = {
  todoPage: TodoPage;
  settingsPage: SettingsPage;
};

// Extend base test by providing "todoPage" and "settingsPage".
// This new "test" can be used in multiple test files, and each of them will get the fixtures.
export const test = base.extend<MyFixtures>({
  todoPage: async ({ page }, use) => {
    // Set up the fixture.
    const todoPage = new TodoPage(page);
    await todoPage.goto();
    await todoPage.addToDo("item1");
    await todoPage.addToDo("item2");

    // Use the fixture value in the test.
    await use(todoPage);

    // Clean up the fixture.
    await todoPage.removeAll();
  },

  settingsPage: async ({ page }, use) => {
    await use(new SettingsPage(page));
  },
});
```

위 코드는 page object modal 기반으로 만든 두개의 fixtures TodoPage, SettingsPage를 생성해서 base test 객체에 extends하여 todoPage, settingsPage fixtures를 새로 정의한 test 객체이다.

### Using a fixture

위처럼 선언하게 되면 그 이후부터는 이 fixtures를 사용할 수 있다.

```typescript
import { test, expect } from "./my-test";

test.beforeEach(async ({ settingsPage }) => {
  await settingsPage.switchToDarkMode();
});

test("basic test", async ({ todoPage, page }) => {
  await todoPage.addToDo("something nice");
  await expect(page.getByTestId("todo-title")).toContainText([
    "something nice",
  ]);
});
```

### Overriding fixtures

나만의 fixtures를 생성할수도 있지만, 기존에 존재하는 fixtures를 오버라이딩할 수도 있다.

```typescript
import { test as base } from "@playwright/test";

export const test = base.extend({
  page: async ({ baseURL, page }, use) => {
    await page.goto(baseURL);
    await use(page);
  },
});
```

위 코드는 page 라는 fixture를 오버라이딩해서 자동으로 baseURL로 네비게이팅할 수 있도록 한 것이다. 위 코드의 경우 구성 파일이나 `test.use()`를 사용하여 테스트 파일에서 로컬로 baseURL을 구성할 수도 있다.

```typescript
test.use({ baseURL: "https://playwright.dev" });
```

특정 기본 fixture를 완전히 재정의하는 것도 가능하다.

```typescript
import { test as base } from "@playwright/test";

export const test = base.extend({
  storageState: async ({}, use) => {
    const cookie = await getAuthCookie();
    await use({ cookies: [cookie] });
  },
});
```

### Worker-scoped fixtures

playwright 테스트는 테스트 파일들을 실행할 때 [worker processes](https://playwright.dev/docs/test-parallel)를 사용한다. 각각의 테스트가 실행할 때 test fixtures를 세팅하는 방법과 비슷하게, worker fixtures도 각 worker process를 세팅할 수 있다. 여기에서 서비스를 설정하고, 서버를 실행하는 등의 작업을 수행할 수 있다. playwright 테스트는 worker fixtures가 일치하고 환경이 동일하다면, 가능한 많은 테스트 파일에 대해 worker process를 재사용한다.

```typescript
import { test as base } from "@playwright/test";

type Account = {
  username: string;
  password: string;
};

// Note that we pass worker fixture types as a second template parameter.
export const test = base.extend<{}, { account: Account }>({
  account: [
    async ({ browser }, use, workerInfo) => {
      // Unique username.
      const username = "user" + workerInfo.workerIndex;
      const password = "verysecure";

      // Create the account with Playwright.
      const page = await browser.newPage();
      await page.goto("/signup");
      await page.getByLabel("User Name").fill(username);
      await page.getByLabel("Password").fill(password);
      await page.getByText("Sign up").click();
      // Make sure everything is ok.
      await expect(page.getByTestId("result")).toHaveText("Success");
      // Do not forget to cleanup.
      await page.close();

      // Use the account value.
      await use({ username, password });
    },
    { scope: "worker" },
  ],

  page: async ({ page, account }, use) => {
    // Sign in with our account.
    const { username, password } = account;
    await page.goto("/signin");
    await page.getByLabel("User Name").fill(username);
    await page.getByLabel("Password").fill(password);
    await page.getByText("Sign in").click();
    await expect(page.getByTestId("userinfo")).toHaveText(username);

    // Use signed-in page in the test.
    await use(page);
  },
});
export { expect } from "@playwright/test";
```

위 코드는 같은 worker에 해당하는 모든 테스트가 공유하는 account fixture를 생성한다. 그리고 각 테스트에서 이 account로 로그인을 하도록 page fixture를 오버라이딩한다. 특정 유니크한 계정을 생성하기 위해,모든 테스트 또는 fixtures에서 사용할 수 있는 `workerInfo.workerIndex`를 사용할 것이다. **테스트 러너가 worker당 한번씩 이 fixtures를 설정할 수 있도록 `{ scope: 'worker' }`를 전달해야한다.**

### Automatic fixtures

각 test/worker에서 직접적으로 코드에 매번 적지않더라도 자동으로 설정하도록 하는 것이다. 이를 만들려면 튜플 구문을 사용하여 `{ auto: true }`를 전달한다.

아래는 테스트가 실패했을 때 디버그 로그를 자동으로 붙일 수 있도록 설정하는 fixtures이다. 그러면 나중에 reporter에서 로그를 확인할 수 있다. TestInfo 객체를 사용하여 테스트가 실행될 때 metadata를 조회할 수 있다.

```typescript
import * as debug from "debug";
import * as fs from "fs";
import { test as base } from "@playwright/test";

export const test = base.extend<{ saveLogs: void }>({
  saveLogs: [
    async ({}, use, testInfo) => {
      // Collecting logs during the test.
      const logs = [];
      debug.log = (...args) => logs.push(args.map(String).join(""));
      debug.enable("myserver");

      await use();

      // After the test we can check whether the test passed or failed.
      if (testInfo.status !== testInfo.expectedStatus) {
        // outputPath() API guarantees a unique file name.
        const logFile = testInfo.outputPath("logs.txt");
        await fs.promises.writeFile(logFile, logs.join("\n"), "utf8");
        testInfo.attachments.push({
          name: "logs",
          contentType: "text/plain",
          path: logFile,
        });
      }
    },
    { auto: true },
  ],
});
export { expect } from "@playwright/test";
```

### Fixture timeout

기본적으로 fixtures는 테스트에 대한 timeout을 공유한다. slow fixtures에서는 특히 `worker-scoped`에 해당하는 경우, timeout을 분리하는 것이 편리하다. 이렇게 하는 경우 timeout을 작게 유지한 채로 slow fixtures에 더 많은 timeout 시간을 제공하는 것이 가능하다.

```typescript
import { test as base, expect } from "@playwright/test";

const test = base.extend<{ slowFixture: string }>({
  slowFixture: [
    async ({}, use) => {
      // ... perform a slow operation ...
      await use("hello");
    },
    { timeout: 60000 },
  ],
});

test("example test", async ({ slowFixture }) => {
  // ...
});
```

### Fixtures-options

playwright는 여러 테스트 projects를 실행시킬 때 설정을 분리할 수 있도록 제공한다. `option` fixtures를 사용할 수 있는데, 이를 사용하여 환경 옵션을 선언적이고 타입 세이프한 옵션으로 구현할 수 있다.

아래 예시는 defaultItem 이라는 옵션을 생성하여 todoPage fixture에서 사용하도록 하였다. 이 옵션은 configuration 파일에서 세팅하는 것이 좋고, `{option: true}`라는 튜플구문을 사용하여야한다.

```typescript
import { test as base } from "@playwright/test";
import { TodoPage } from "./todo-page";

// Declare your options to type-check your configuration.
export type MyOptions = {
  defaultItem: string;
};
type MyFixtures = {
  todoPage: TodoPage;
};

// Specify both option and fixture types.
export const test = base.extend<MyOptions & MyFixtures>({
  // Define an option and provide a default value.
  // We can later override it in the config.
  defaultItem: ["Something nice", { option: true }],

  // Our "todoPage" fixture depends on the option.
  todoPage: async ({ page, defaultItem }, use) => {
    const todoPage = new TodoPage(page);
    await todoPage.goto();
    await todoPage.addToDo(defaultItem);
    await use(todoPage);
    await todoPage.removeAll();
  },
});
export { expect } from "@playwright/test";
```
