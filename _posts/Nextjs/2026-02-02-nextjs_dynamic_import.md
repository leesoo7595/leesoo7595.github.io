# TIL: Next.js `dynamic` import 제대로 이해하기

## 1. `dynamic`이란?

Next.js에서 제공하는 **코드 스플리팅 + 지연 로딩** 기능. React의 `lazy()` + `Suspense`를 래핑한 것.

```tsx
import dynamic from 'next/dynamic'

const HeavyChart = dynamic(() => import('./HeavyChart'))
```

---

## 2. 동작 원리

```
일반 import                          dynamic import
──────────────────────────────────────────────────────────────────
빌드 시 → 하나의 번들에 포함           빌드 시 → 별도 청크(.js)로 분리

페이지 로드 → 전체 번들 다운로드        페이지 로드 → 메인 번들만 다운로드
                                          → 컴포넌트 필요 시 청크 요청
```

---

## 3. 핵심 옵션

```tsx
const Component = dynamic(() => import('./Component'), {
  // 1. SSR 비활성화 (window, document 사용 컴포넌트)
  ssr: false,

  // 2. 로딩 중 표시할 UI
  loading: () => <Skeleton />,
})
```

| 옵션      | 기본값 | 설명                              |
| --------- | ------ | --------------------------------- |
| `ssr`     | `true` | `false`면 클라이언트에서만 렌더링 |
| `loading` | `null` | 로딩 중 보여줄 컴포넌트           |

---

## 4. 언제 사용해야 하는가

### ✅ 사용하면 좋은 경우

| 케이스                | 예시                                |
| --------------------- | ----------------------------------- |
| **무거운 라이브러리** | Chart.js, Monaco Editor, PDF Viewer |
| **조건부 렌더링**     | Modal, BottomSheet, Dialog          |
| **클라이언트 전용**   | window/document 의존 컴포넌트       |
| **Below the fold**    | 스크롤해야 보이는 영역              |

```tsx
// ✅ 좋은 예시
const Modal = dynamic(() => import('./Modal'))
const Chart = dynamic(() => import('./Chart'), { ssr: false })
const PDFViewer = dynamic(() => import('./PDFViewer'), {
  ssr: false,
  loading: () => <p>PDF 로딩중...</p>,
})
```

### ❌ 사용하면 안 좋은 경우

| 케이스                        | 이유                                 |
| ----------------------------- | ------------------------------------ |
| **항상 렌더링되는 컴포넌트**  | 어차피 로드해야 함 → 오버헤드만 추가 |
| **작은 컴포넌트**             | 청크 분리 비용 > 이득                |
| **초기 화면(Above the fold)** | LCP 지연, 레이아웃 시프트 발생       |

```tsx
// ❌ 나쁜 예시 - 항상 보이는 컴포넌트
const Header = dynamic(() => import('./Header')) // 항상 표시됨
const Button = dynamic(() => import('./Button')) // 너무 작음
const AccountInfo = dynamic(() => import('./AccountInfo')) // 핵심 UI
```

---

## 5. 흔한 오해와 진실

### Q1. 페이지 이동 후 돌아오면 다시 HTTP 요청하나?

> **아니오.** 브라우저가 JS 파일을 캐시하므로 재요청하지 않음.

### Q2. 그럼 성능 문제 없나?

> **있을 수 있음.** 컴포넌트가 다시 마운트되면서:
>
> - dynamic resolve 과정 발생
> - 로딩 UI 깜빡임 가능
> - 상태 초기화

### Q3. 모든 컴포넌트에 쓰면 좋은가?

> **아니오.** 청크 수가 많아지면:
>
> - HTTP 요청 수 증가 (HTTP/1.1에서 병목)
> - 번들 분석/디버깅 어려움
> - 미세한 로딩 지연 누적

---

## 6. 성능 최적화 전략

```tsx
// 1. Named export 시
const Component = dynamic(() =>
  import('./Components').then((mod) => mod.MyComponent),
)

// 2. 여러 컴포넌트를 하나의 청크로 묶기
const ModalBundle = dynamic(() => import('./modals'))
// modals/index.ts에서 Modal, Dialog, BottomSheet 모두 export

// 3. Intersection Observer로 뷰포트 진입 시 로드
const LazySection = dynamic(() => import('./LazySection'))

function Page() {
  const [isVisible, setIsVisible] = useState(false)
  // ... intersection observer 로직
  return isVisible ? <LazySection /> : null
}
```

---

## 7. App Router에서의 주의점

```tsx
// App Router에서는 서버 컴포넌트가 기본
// dynamic은 클라이언트 컴포넌트에서만 의미 있음

'use client' // ← 필수

import dynamic from 'next/dynamic'
const Chart = dynamic(() => import('./Chart'), { ssr: false })
```

---

## 8. 실무 체크리스트

사용 전 자문해볼 질문:

- [ ] 이 컴포넌트가 항상 렌더링되는가? → `Yes`면 일반 import
- [ ] 무거운 라이브러리를 포함하는가? → `Yes`면 dynamic
- [ ] 조건부로 렌더링되는가? → `Yes`면 dynamic 고려
- [ ] window/document를 사용하는가? → `Yes`면 `ssr: false`
- [ ] 초기 화면에 보이는가? → `Yes`면 loading UI 필수

---

## 9. 결론

> **"dynamic import는 도구일 뿐, 만능이 아니다"**

- 무분별한 사용 → 오히려 성능 저하
- 적재적소에 사용 → 초기 로딩 속도 개선
- 핵심은 **"이 컴포넌트가 정말 지연 로딩이 필요한가?"** 판단하기
