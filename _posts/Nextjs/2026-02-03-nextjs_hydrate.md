# Next.js Hydration 개념 정리

## Hydration이란?

**Hydration**은 서버에서 렌더링된 정적 HTML에 JavaScript를 연결하여 인터랙티브하게 만드는 과정입니다.

쉽게 말해, "마른 HTML에 물(JavaScript)을 부어 살아있는 React 앱으로 만드는 것"입니다.

## 렌더링 방식 비교

### CSR (Client-Side Rendering)

```
브라우저 요청 → 빈 HTML 수신 → JS 다운로드 → React 실행 → 화면 렌더링
```

- 초기 로딩 시 빈 화면 (흰 화면)
- JS가 모두 로드될 때까지 콘텐츠 없음
- SEO 불리

### SSR (Server-Side Rendering) + Hydration

```
브라우저 요청 → 완성된 HTML 수신 → 화면 즉시 표시 → JS 다운로드 → Hydration → 인터랙티브
```

- 초기 로딩 시 콘텐츠 바로 보임
- SEO 유리
- 단, Hydration 전까지는 버튼 클릭 등 인터랙션 불가

## Hydration 과정 상세

```
[서버]
1. React 컴포넌트를 HTML 문자열로 변환
2. 완성된 HTML을 클라이언트에 전송

[클라이언트]
3. HTML을 파싱하여 화면에 표시 (이 시점에 사용자는 콘텐츠를 볼 수 있음)
4. JavaScript 번들 다운로드
5. React가 기존 HTML에 이벤트 리스너 연결 (Hydration)
6. 이제 완전한 React 앱으로 동작
```

### 타임라인 예시

```
0ms     100ms    300ms    500ms    1000ms
|--------|--------|--------|--------|
   HTML     화면      JS       Hydration
   수신     표시    다운로드    완료

         👁️ 볼 수 있음    👆 클릭 가능
```

## Hydration Mismatch 문제

서버에서 렌더링한 HTML과 클라이언트에서 렌더링한 결과가 다르면 **Hydration Mismatch** 에러가 발생합니다.

### 발생하는 경우

```tsx
// ❌ 문제: 서버와 클라이언트의 결과가 다름
function Component() {
  return <div>{new Date().toLocaleTimeString()}</div>
  // 서버: "오후 2:30:00"
  // 클라이언트: "오후 2:30:05" (5초 후)
}
```

```tsx
// ❌ 문제: 클라이언트에서만 존재하는 값 사용
function Component() {
  return <div>{window.innerWidth}</div>
  // 서버: window is not defined 에러
}
```

```tsx
// ❌ 문제: 로컬스토리지/쿠키 등 클라이언트 전용 데이터
function Component() {
  const user = localStorage.getItem('user')
  return <div>{user ? '로그인됨' : '로그아웃'}</div>
  // 서버: 항상 '로그아웃' (localStorage 없음)
  // 클라이언트: '로그인됨' 또는 '로그아웃'
}
```

### 콘솔 에러 메시지

```
Warning: Text content did not match. Server: "로그아웃" Client: "로그인됨"

Hydration failed because the initial UI does not match what was rendered on the server.
```

## 해결 방법

### 1. useEffect + useState 패턴 (isMounted)

```tsx
function Component() {
  const [isMounted, setIsMounted] = useState(false)
  const user = useUserStore() // Zustand 등 클라이언트 상태

  useEffect(() => {
    setIsMounted(true)
  }, [])

  // 마운트 전에는 서버와 동일한 결과 반환
  if (!isMounted) {
    return <div>로딩중...</div>
  }

  // 마운트 후에는 실제 데이터 사용
  return <div>{user ? '로그인됨' : '로그아웃'}</div>
}
```

### 2. dynamic import + ssr: false

```tsx
import dynamic from 'next/dynamic'

// 이 컴포넌트는 서버에서 렌더링되지 않음
const ClientOnlyComponent = dynamic(() => import('./ClientOnlyComponent'), {
  ssr: false,
})

function Page() {
  return <ClientOnlyComponent />
}
```

### 3. suppressHydrationWarning

```tsx
// 의도적으로 다를 수 있는 경우에만 사용 (시간 표시 등)
function Clock() {
  return <time suppressHydrationWarning>{new Date().toLocaleTimeString()}</time>
}
```

### 4. useId (React 18+)

```tsx
import { useId } from 'react'

function Form() {
  // 서버와 클라이언트에서 동일한 ID 생성
  const id = useId()

  return (
    <>
      <label htmlFor={id}>이름</label>
      <input id={id} />
    </>
  )
}
```

## Next.js App Router에서의 차이

### Server Components (기본값)

```tsx
// app/page.tsx
// 서버에서만 실행됨, Hydration 없음
async function Page() {
  const data = await fetch('...')
  return <div>{data}</div>
}
```

### Client Components

```tsx
// app/components/Counter.tsx
'use client' // 이 지시어가 있으면 클라이언트 컴포넌트

import { useState } from 'react'

function Counter() {
  const [count, setCount] = useState(0)
  // 이 컴포넌트는 Hydration 됨
  return <button onClick={() => setCount(count + 1)}>{count}</button>
}
```

## 정리

| 개념                   | 설명                                                     |
| ---------------------- | -------------------------------------------------------- |
| **SSR**                | 서버에서 HTML 생성                                       |
| **Hydration**          | 서버 HTML에 JS 연결하여 인터랙티브하게 만듦              |
| **Hydration Mismatch** | 서버/클라이언트 렌더링 결과 불일치                       |
| **해결법**             | isMounted 패턴, dynamic import, suppressHydrationWarning |

## 참고 자료

- [React 공식 문서 - Hydration](https://react.dev/reference/react-dom/client/hydrateRoot)
- [Next.js 공식 문서](https://nextjs.org/docs)
