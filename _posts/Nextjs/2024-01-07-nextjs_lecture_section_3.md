---
layout: post
title: "[Nextjs] Nextjs app router 강의 섹션 3"
date: 2024-01-07
tags: [nextjs]
comments: true
---

제로초의 next app router를 기반으로 docs 공부를 보충하였습니다.

# Data Fetching

## Server Actions and Mutations

server actions은 서버에서 실행되는 비동기함수이다. 서버와 클라이언트 컴포넌트에서 폼 양식 제출 및 mutations을 사용될 수 있다.

### Convention

Server Action은 `"use server"`라는 리액트 디렉티브를 정의할 수 있다. 디렉티브를 비동기함수 상단에 두어 해당 함수를 서버 액션으로 표시하거나 별도의 파일 상단에 배치하여 해당 파일을 서버 액션으로 표시할 수 있다.

#### Server Components

서버 컴포넌트는 `"use server"`인라인 함수 레벨이나 모듈 레벨에서 사용할 수 있다. 서버 액션을 인라인하기 위해서는 함수 본문 상단에 디렉티브를 추가한다.

```tsx
// Server Component
export default function Page() {
  // Server Action
  async function create() {
    'use server'

    // ...
  }

  return (
    // ...
  )
}
```

#### Client Components

클라이언트 컴포넌트에서는 module-level로 `"use server"` 디렉티브를 넣은 actions만 임포트할 수 있다.

서버액션을 클라이언트 컴포넌트에서 호출하려면, 새로운 파일을 생성하고 `"use server"` 디렉티브를 상단에 추가한다. 그러면 파일의 모든 함수들은 클라이언트과 서버 컴포넌트 모두에서 재사용되는 서버액션이 된다.

```typescript
// app/actions.ts

"use server";

export async function create() {
  // ...
}
```

```typescript
// app/ui/button.tsx
import { create } from '@/app/actions'

export function Button() {
  return (
    // ...
  )
}
```

클라이언트 컴포넌트의 prop으로도 서버액션을 전달할 수 있다:

```tsx
<ClientComponent updateItem={updateItem} />;

// ClientComponent
("use client");

export default function ClientComponent({ updateItem }) {
  return <form action={updateItem}>{/* ... */}</form>;
}
```

### Behavior

- 서버액션은 <form> 요소의 action 어트리뷰트를 통해서 일어날 수 있다.
  - 서버컴포넌트는 기본적으로 Javascript가 아직 로드되지않아도 않아도 서버액션이 실행되어 form이 submit된다.
  - 클라이언트컴포넌트에서는 서버액션을 호출하는 form은 자바스크립트가 아직 로드되지 않았을 때는 대기열에 추가하여 클라이언트 하이드레이션의 우선순위를 정한다.
  - 하이드레이션 이후에, 브라우저는 form 제출이 이루어질때 브라우저가 리로드되지 않는다.
- 서버액션은 `<form>`뿐만이 아니라 이벤트핸들러, `useEffect`, 써드파티 라이브러리들과 다른 `<button>` 같은 기타 form elements에서도 호출이 가능하다.
- 서버액션은 Nextjs 캐싱과 재검증 아키텍처에서도 통합된다. 액션이 invoke되었을 때, Nextjs는 한번의 서버 통신으로 업데이트된 UI와 새 데이터를 모두 받아올 수 있다.
- 백그라운드에서 액션은 POST 메소드를 사용하고, 이 메소드만이 액션을 invoke할 수 있다.
- 서버액션의 인자와 리턴값을 React에서 직렬화할 수 있어야한다. 그래서 직렬화 가능한 인자와 값들로 이루어져있어야한다.
- 서버액션은 함수들이다. 그래서 어플리케이션 내에 재사용이 가능하다.
- 서버액션은 페이지나 레이아웃에서 런타임을 상속받는다(?)

### Example

#### **Forms**

React에서는 HTML `<form>` 요소를 확장하여 `action`이라는 prop으로 서버액션을 호출할 수 있도록 한다.

form이 호출될 때, 액션은 자동적으로 `FormData` 객체를 받는다. 해당 필드들을 관리하기위해 useState를 쓰는 대신, `FormData` 메소드를 사용해서 data를 추출한다.

```tsx
export default function Page() {
  async function createInvoice(formData: FormData) {
    "use server";

    const rawFormData = {
      customerId: formData.get("customerId"),
      amount: formData.get("amount"),
      status: formData.get("status"),
    };

    // mutate data
    // revalidate cache
  }

  return <form action={createInvoice}>...</form>;
}
```

[자세한건 React `<form>` documentation 참고](https://react.dev/reference/react-dom/components/form#handle-form-submission-with-a-server-action)

- **추가적인 인자 전달**

저바스크립트의 `bind` 메소드를 사용해서 서버액션에 추가 인자를 전달할 수 있다.

```tsx
"use client";

import { updateUser } from "./actions";

export function UserProfile({ userId }: { userId: string }) {
  const updateUserWithId = updateUser.bind(null, userId);

  return (
    <form action={updateUserWithId}>
      <input type="text" name="name" />
      <button type="submit">Update User Name</button>
    </form>
  );
}
```

서버액션은 `userId` 인자를 추가적으로 form data에서 받을 수 있을 것이다.

```js
"use server";

export async function updateUser(userId, formData) {
  // ...
}
```

- **pendig states**

form이 제출되고 pending state일 때 보여주기 위해서`useFormStatus` 훅을 사용할 수 있다.

- `useFormStatus`는 특정 `<form>` 상태를 리턴한다. 그래서 꼭 `<form>` 요소의 자식으로써 정의되어야한다.
- `useFormStatus`는 리액트훅이므로 클라이언트 컴포넌트에서 사용되어야한다.

```tsx
"use client";

import { useFormStatus } from "react-dom";

export function SubmitButton() {
  const { pending } = useFormStatus();

  return (
    <button type="submit" aria-disabled={pending}>
      Add
    </button>
  );
}
```

`<SubmitButton/>`는 모든 form에 네스팅이 가능하다:

```tsx
import { SubmitButton } from "@/app/submit-button";
import { createItem } from "@/app/actions";

// Server Component
export default async function Home() {
  return (
    <form action={createItem}>
      <input type="text" name="field-name" />
      <SubmitButton />
    </form>
  );
}
```

- **서버사이드 재검증과 에러핸들링**

`required`와 `type="email"` 같은 기본적인 클라이언트 사이드 form 유효성인 HTML 유효성을 사용하는 것을 권장한다.

좀 더 고급기능의 서버사이드 유효성을 위해서는 zod 같은 라이브러리를 사용하여 데이터를 mutating하기 전에 form 필드의 유효성을 검증할 수 있다.

```ts
// actions.ts
"use server";

import { z } from "zod";

const schema = z.object({
  email: z.string({
    invalid_type_error: "Invalid Email",
  }),
});

export default async function createUser(formData: FormData) {
  const validatedFields = schema.safeParse({
    email: formData.get("email"),
  });

  // Return early if the form data is invalid
  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
    };
  }

  // Mutate data
}
```

서버에서 필드들이 유효성이 검증되면, 액션에서 직렬화된 객체를 리턴할 수 있고 `useFormState` 훅을 사용해서 사용자에게 메세지를 보여줄 수 있다.

- `useFormState`에 액션을 바이패싱하면 액션의 함수 시그니처가 변경되어 첫 번째 인수로 새로운 prevState 또는 initialState 파라미터를 받는다.
- `useFormState`는 리액트훅이니까~~~ 클라컴포넌트에만!

```ts
"use server";

export async function createUser(prevState: any, formData: FormData) {
  // ...
  return {
    message: "Please enter a valid email",
  };
}
```

그리고 `useFormState` 훅에 액션을 전달하여 리턴받은 `state`를 사용하여 에러메세지를 보여줄 수 있다.

```tsx
"use client";

import { useFormState } from "react-dom";
import { createUser } from "@/app/actions";

const initialState = {
  message: "",
};

export function Signup() {
  const [state, formAction] = useFormState(createUser, initialState);

  return (
    <form action={formAction}>
      <label htmlFor="email">Email</label>
      <input type="text" id="email" name="email" required />
      {/* ... */}
      <p aria-live="polite" className="sr-only">
        {state?.message}
      </p>
      <button>Sign up</button>
    </form>
  );
}
```

데이터를 mutating하기전에 항상 사용자에게 해당 작업을 수행할 권한이 있는지 확인해야한다. 이는 [인증 및 권한 부여](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations#authentication-and-authorization) 참조

- **Optimistic updates**

`useOptimistic` 훅을 사용하여 응답을 기다리지 않고 서버액션이 완료되기 전에 UI를 Optimistic하게 업데이트할 수 있다:

```tsx
"use client";

import { useOptimistic } from "react";
import { send } from "./actions";

type Message = {
  message: string;
};

export function Thread({ messages }: { messages: Message[] }) {
  const [optimisticMessages, addOptimisticMessage] = useOptimistic<Message[]>(
    messages,
    (state: Message[], newMessage: string) => [
      ...state,
      { message: newMessage },
    ]
  );

  return (
    <div>
      {optimisticMessages.map((m, k) => (
        <div key={k}>{m.message}</div>
      ))}
      <form
        action={async (formData: FormData) => {
          const message = formData.get("message");
          addOptimisticMessage(message);
          await send(message);
        }}
      >
        <input type="text" name="message" />
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

- **중첩된 요소들**

중첩된 `<form>`, `<button>` 등등의 요소에 서버액션을 호출할 수 있다. 이 요소들은 `formAction`을 prop 또는 이벤트핸들러에서 받는다.

이는 여러 서버액션들을 form에서 호출하고 싶을때와 같은 특정 케이스들에 유용하다. 예를 들면 `<button>` 요소를 만들어 draft post를 저장하고 추가적으로 퍼블리싱까지 할 때와 같은 경우이다. [React <form> docs 참고](https://react.dev/reference/react-dom/components/form#handle-form-submission-with-a-server-action)

- **프로그래밍 방식의 form submission**

`requestSubmit()` 메소드를 트리거하여 폼양식 제출이 가능하다. 예를 들면, 사용자가 커맨드+엔터키를 눌렀을때, `onKeyDown` 이벤트를 통해 폼양식을 제출하려면:

```tsx
"use client";

export function Entry() {
  const handleKeyDown = (e: React.KeyboardEvent<HTMLTextAreaElement>) => {
    if (
      (e.ctrlKey || e.metaKey) &&
      (e.key === "Enter" || e.key === "NumpadEnter")
    ) {
      e.preventDefault();
      e.currentTarget.form?.requestSubmit();
    }
  };

  return (
    <div>
      <textarea name="entry" rows={20} required onKeyDown={handleKeyDown} />
    </div>
  );
}
```

이런 방식으로 하는 경우, 가장 가까이에 있는 부모의 `<form>`이 트리거되어 서버액션이 호출된다.

#### Non-form Elements

`<form>` 요소를 사용해서 서버액션을 사용하는 것이 일반적이지만, 이벤트핸들러와 `useEffect`와 같은 코드를 사용하여 호출할 수도 있다.

- 이벤트핸들러

onClick 같은 이벤트핸들러를 통해 서버액션을 invoke할 수도 있다. 예를 들면:

```tsx
"use client";

import { incrementLike } from "./actions";
import { useState } from "react";

export default function LikeButton({ initialLikes }: { initialLikes: number }) {
  const [likes, setLikes] = useState(initialLikes);

  return (
    <>
      <p>Total Likes: {likes}</p>
      <button
        onClick={async () => {
          const updatedLikes = await incrementLike();
          setLikes(updatedLikes);
        }}
      >
        Like
      </button>
    </>
  );
}
```

사용자경험을 더 좋게 하기 위해, `useOptimistic`과 `useTransition`과 같은 서버액션에서 응답을 받아오기 전 UI나 pending state 상태일 때를 고려하여 React API를 사용하는 것을 권장한다.

이벤트핸들러를 form elements에도 등록할 수 있다. 예를 들면 `onChange`를 사용하여 form 필드를 저장할 때:

```tsx
"use client";

import { publishPost, saveDraft } from "./actions";

export default function EditPost() {
  return (
    <form action={publishPost}>
      <textarea
        name="content"
        onChange={async (e) => {
          await saveDraft(e.target.value);
        }}
      />
      <button type="submit">Publish</button>
    </form>
  );
}
```

여러 이벤트가 연속적으로 실행될 수 있는 경우에는 불필요한 서버액션 호출을 방지하기 위해서 디바운싱을 권장한다.

- useEffect

React useEffect 훅을 사용하여 컴포넌트가 마운트되었을 때나 의존성이 바뀌었을때 서버액션을 호출할 수도 있다. 이것은 글로벌 이벤트에 의존하거나, 자동으로 트리거되어야하는 mutation에 유용하다. 그 예로 앱의 숏컷으로 onKeyDown이나 무한스크롤링을 위한 옵저버 훅, 또는 컴포넌트가 마운트되어 조회 수를 업데이트할 때 등을 들 수 있다.

```tsx
"use client";

import { incrementViews } from "./actions";
import { useState, useEffect } from "react";

export default function ViewCount({ initialViews }: { initialViews: number }) {
  const [views, setViews] = useState(initialViews);

  useEffect(() => {
    const updateViews = async () => {
      const updatedViews = await incrementViews();
      setViews(updatedViews);
    };

    updateViews();
  }, []);

  return <p>Total Views: {views}</p>;
}
```
