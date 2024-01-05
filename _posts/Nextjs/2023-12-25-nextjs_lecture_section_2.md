---
layout: post
title: "[Nextjs] Nextjs app router 강의 섹션 2"
date: 2023-12-25
tags: [nextjs]
comments: true
---

# Nextjs

제로초의 next app router를 기반으로 docs 공부를 보충하였습니다.

## Linking and Navigating

nextjs에서 라우트 간의 네이게이팅을 두 가지 방법이 있다.

- `<Link> Component`
- `useRouter Hook`

이 페이지에서는 `<Link>`, `useRouter()`를 어떻게 사용하는지, 그리고 네비게이션은 어떻게 동작하는지 알아보자.

### `<Link>` Component

`<Link>`는 HTML의 `<a>` 태그를 확장해서 라우트 간의 prefetching과 client-side 네이게이션을 제공하는 기본 빌트인 컴포넌트이다. Nextjs에서 라우트 간의 네비게이션을 하는 기본 방식이다.

`next/link`를 임포트하여 사용할 수 있고, href prop을 전달한다.

```tsx
import Link from "next/link";

export default function Page() {
  return <Link href="/dashboard">Dashboard</Link>;
}

// dynamic segments
export default function PostList({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>
          <Link href={`/blog/${post.slug}`}>{post.title}</Link>
        </li>
      ))}
    </ul>
  );
}
```

#### 활성화된 Link 확인

`usePathname()`을 사용하여 link가 활성화되어있는지 확인할 수 있다.

```tsx
"use client";

import { usePathname } from "next/navigation";
import Link from "next/link";

export function Links() {
  const pathname = usePathname();

  return (
    <nav>
      <ul>
        <li>
          <Link className={`link ${pathname === "/" ? "active" : ""}`} href="/">
            Home
          </Link>
        </li>
        <li>
          <Link
            className={`link ${pathname === "/about" ? "active" : ""}`}
            href="/about"
          >
            About
          </Link>
        </li>
      </ul>
    </nav>
  );
}
```

#### `id`로 스크롤하기

nextjs app router의 기본 동작은 새 라우트의 상단으로 스크롤하거나, 뒤로 또는 앞으로 네비게이팅할 때 스크롤 위치를 유지한다.

특정 id로 스크롤하고 싶다면, URL을 #와 함께 추가하거나 href prop을 #와 함께 전달한다. 이것은 `<Link>`가 `<a>`로 렌더링하기 때문에 가능하다.

```tsx
<Link href="/dashboard#settings">Settings</Link>

// Output
<a href="/dashboard#settings">Settings</a>
```

#### scroll 복원 비활성화

nextjs app router의 기본 동작은 새 라우트의 상단으로 스크롤하거나, 뒤로 또는 앞으로 네이게이팅할 때 스크롤 위치를 유지한다. 만약 이 행동을 비활성화하고 싶다면, `scroll={false}`를 `<Link>` 컴포넌트에 전달하거나 `router.push()`, `router.replace()`에 `scroll: false`를 전달한디.

```tsx
// next/link
<Link href="/dashboard" scroll={false}>
  Dashboard
</Link>;

// useRouter
import { useRouter } from "next/navigation";

const router = useRouter();

router.push("/dashboard", { scroll: false });
```

### `useRouter()` Hook

`useRouter` hook은 클라이언트 컴포넌트에서 라우팅을 프로그램적으로 변경할 때 사용한다. 서버 컴포넌트에서는 `redirect()`를 대신 사용하여야한다.

```tsx
"use client";

import { useRouter } from "next/navigation";

export default function Page() {
  const router = useRouter();

  return (
    <button type="button" onClick={() => router.push("/dashboard")}>
      Dashboard
    </button>
  );
}
```

### 라우팅과 네비게이션 동작 방식

App Router는 라우팅과 네이게이션을 할 떄 하이브리드식 접근방법을 사용한다. 서버에서는 애플리케이션 코드가 route segments로 자동으로 코드 스플리트된다. 클라이언트에서는 nextjs route segments를 prefetches와 caches한다. 이것은 user가 새로운 라우트에 이동해도 브라우저는 페이지를 리로드하지 않는다. 그리고 변경되는 route segments만 다시 리렌더링하여 네비게이션 환경과 성능을 개선한다.

### useSelectedLayoutSegment

`useSelectedLayoutSegment`는 호출된 레이아웃보다 한 단계 아래에서 active route 세그먼트를 읽을 수 있는 클라이언트 컴포넌트 훅이다.

네비게이션 UI에 유용하다. 예를 들면 부모 레이아웃 안에 탭이 있고, active child segment에 따라 스타일이 바뀌는 경우이다.

```tsx
"use client";

import { useSelectedLayoutSegment } from "next/navigation";

export default function ExampleClientComponent() {
  const segment = useSelectedLayoutSegment();

  return <p>Active segment: {segment}</p>;
}
```

- 이 훅은 클라이언트 컴포넌트 훅이고, 레이아웃의 경우 기본적으로 서버 컴포넌트이기 때문에, layoutSegment는 일반적으로 레이아웃에 임포트된 클라이언트 컴포넌트를 통해 호출된다.
- `useSelectedLayoutSegment`는 한 레벨 낮은 세그먼트를 리턴한다. 모든 활성화된 세그먼트를 리턴하고 싶으면 `useSelectedLayoutSegments`를 사용하자.
