# 202430113 안지혜

## 2026-10-07 (Week 6)
## async를 미리 사용하는 이유

Next.js App Router의 Server Component에서는 데이터 요청을 비동기로 처리하는 경우가 많기 때문에
'await'가 당장 없더라도 'async'를 붙여 사용 가능

- 페이지 함수 작성 방식 통일 가능
- 이후 'fetch()'나 DB 요청 추가 용이
- Server Component는 Promise를 반환하는 형태 지원

#### 2.3 느린 네트워크
네트워크가 느리면 사용자가 링크를 크릭하기 전에 Prefetch가 끝나지 않을 수 있음
이 경우 'loading.tsx'도 미리 준비되지 않아 화면 변화가 늦게 느껴질 수 있음
'useLinkStatus'를 이용하면 이동 중임을 사용자에게 표시 가능

```tsx
'use client'

import { useLinkStatus } from 'next/link'

export default function LoadingIndicator() {
  const { pending } = useLinkStatus()

  return (
    <span className={pending ? 'is-pending' : ''} />
  )
}
```
스피너나 텍스트 효과 등을 이용해 페이지 이동이 진행 중이라는 피드백을 줄 수 있다.

#### 2.4 Prefetch 비활성화
`<Link>`의 `prefetch` 값을 `false`로 설정하면 자동 Prefetch를 끌 수 있다.
```ts
<Link prefetch={false} href="/blog">
  Blog
</Link>
```
링크가 매우 많은 페이지에서는 불필요한 네트워크 요청을 줄이는 데 사용 가능

단점:
- 정적 경로도 클릭 이후에 로드됨
- 동적 경로는 이동 전에 서버 렌더링을 기다려야 할 수 있다.
Prefetch를 완전히 끄는 대신 사용자가 링크에 마우스를 올렸을 때만 실행하는 방법도 사용 가능

#### 2.5 Hydration 지연
`<Link>`를 통한 클라이언트 기능은 Hydration 이후 사용 가능하다.
초기 JS Bundle이 크면 Hydration이 늦어지고 Prefetch 시작도 지연될 수 있다

개선 방법:
- @next/bundle-analyzer로 번들 크기 확인
- 불필요하게 큰 dependency 제거
- 가능하면 Client Component의 로직을 Server Component로 이동

### Server Component & Client Component
Next.js에서는 필요한 기능에 따라 Server Component와 Client Component를 나누어 사용 가능하다

**Client Component**
브라우저에서 실행되어야 하는 기능이 필요할 때 사용한다
- State 사용
- `OnClick`, `onChange` 등의 Event Handler
- `useEffect` 등의 Lifecycle Logic
- Custom Hook
- `window`, `localStorage`, `navigator.geolocation` 등의 Browser API
Client Component로 만드려면 파일 상단에 작성한다
```ts
'use client'
```

### Server Component
Next.js App Router에서 기본적으로 사용되는 Component이다.

다음과 같은 경우에 적합하다.
- DB 또는 API에서 데이터 가져오기
- API Key, Token 등 보안 정보 보호
- Client에 전달하는 JavaScript 양 감소
- 초기 콘텐츠 표시 성능 개선
- Streaming을 이용한 점진적인 콘텐츠 전달


정리하면 **사용자 상호작용이나 브라우저 기능이 필요하면 Client Component**, 데이터 처리와 서버 작업 중심이라면 **Server Component**를 사용하는 것이 적절하다.


---

## 2026-09-30 (Week 5)
### 1. How Navigation Works

Next.js의 네비게이션은 크게 다음 네 가지 방식으로 동작한다.

- Server Rendering
- Prefetching
- Streaming
- Client-side Transition

#### 1.1 Server Rendering

Next.js의 `layout`과 `page`는 기본적으로 React Server Component로 동작한다.

서버 렌더링은 크게 두 가지 방식으로 나뉜다.

- **Static Rendering**: 빌드 또는 재검증 시점에 미리 렌더링하고 결과를 캐시
- **Dynamic Rendering**: 사용자의 요청이 들어온 시점에 렌더링

서버 렌더링은 새 경로를 보여주기 전에 서버 응답을 기다려야 할 수 있다는 단점이 있다.  
Next.js는 이를 줄이기 위해 `Prefetching`과 `Client-side Transition`을 사용한다.

##### Initial Visit

일반적인 CSR 방식의 React 앱은 초기 HTML이 거의 비어 있고, JavaScript가 실행된 후 화면이 만들어진다.

Next.js는 최초 접근 시 서버에서 HTML을 생성해 전달하기 때문에 사용자는 콘텐츠를 더 빠르게 확인할 수 있다.

이후 React의 `hydration`을 거쳐 버튼 클릭 등의 상호작용이 가능해진다.

- 초기 화면 표시 속도 개선
- UX 개선
- SEO에 유리

#### 1.2 Prefetching

Prefetching은 사용자가 실제로 이동하기 전에 다음 페이지에 필요한 데이터를 미리 받아두는 방식이다.

Next.js의 `<Link>` 컴포넌트를 사용하면 연결된 경로를 미리 가져올 수 있다.

```tsx
import Link from "next/link";

export default function Layout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html>
      <body>
        <nav>
          <Link href="/blog">Blog</Link>
          <a href="/contact">Contact</a>
        </nav>
        {children}
      </body>
    </html>
  );
}
```

- `<Link>` : Prefetching 및 Client-side Navigation 지원
- `<a> `: 일반적인 페이지 이동
경로 종류에 따라 Prefetching 범위도 달라진다.
- 정적 경로: 전체 경로 Prefetch 가능
- 동적 경로: Prefetch를 생략하거나 `loading.tsx`가 있는 경우 일부만 Prefetch
동적 경로를 모두 미리 가져오지 않는 이유는 사용자가 방문하지 않을 페이지까지 서버에서 처리하는 낭비를 줄이기 위해서이다.

#### 1.3 Streaming
Streaming은 페이지 전체 렌더링이 끝날 때까지 기다리지 않고 준비된 부분부터 클라이언트에 전달하는 방식
즉 일부 데이터가 늦게 준비되더라도 먼저 사용할 수 있는 UI부터 표시할 수 있다.
동적 경로에서는 다음과 같은 요소를 먼저 보여줄 수 있다.
- 공통 Layout
- Loading UI
- Loading Skeleton

##### loading.tsx
라우트 폴더에 loading.tsx를 추가하면 페이지가 로드되는 동안 보여줄 UI를 정의할 수 있다.
```ts
export default function Loading() {
  return <LoadingSkeleton />;
}
```
Next.js는 내부적으로 페이지 콘텐츠를 `<Suspense>` 경계로 처리하며, 실제 페이지가 준비되기 전까지 `loading.tsx`의 UI를 보여 준다

##### loading.tsx 장점
- 사용자가 이동 즉시 화면 변화를 확인 가능
- 로딩 중에도 공통 Layout 유지
- 페이지 전체가 준비될 때까지 빈 화면을 보는 문제 감소
- Streaming과 함께 사용해 동적 경로의 체감 속도 개선

#### 1.4 Client-side Transition
일반적인 페이지 이동은 전체 문서를 다시 불러오기 때문에 다음과 같은 문제가 생길 수 있다.
- 기존 state 초기화
- 스크롤 위치 초기화
- 페이지 로딩 중 상호작용 중단
Next.js의 <Link>는 전체 페이지를 새로고침하지 않고 필요한 부분만 변경한다.
따라서
- 공통 Layout 유지
- 기존 UI 유지
- Prefetch된 페이지를 빠르게 표시
- Streaming과 함께 빠른 동적 라우팅 가능
즉 서버 렌더링을 사용하면서도 사용자 입장에서는 SPA처럼 자연스러운 페이지 전환을 제공한다.

### 2. Navigation이 느려지는 경우
Next.js는 기본적으로 빠른 네비게이션을 제공하지만, 특정 상황에서는 페이지 이동이 느리게 느껴질 수 있다.

#### 2.1 동적 경로에 loading.tsx가 없는 경우
동적 경로는 서버 응답을 기다린 후 페이지가 표시될 수 있다.
이 경우 사용자는 클릭 후 아무 변화가 없어 앱이 멈춘 것처럼 느낄 수 있다.
따라서 동적 라우트에는 loading.tsx를 추가해 로딩 UI를 보여주는 것이 좋다.
```ts
export default function Loading() {
  return <LoadingSkeleton />;
}
```

#### 2.2 generateStaticParams가 없는 동적 세그먼트
`[slug]` 같은 동적 경로도 미리 경로를 알 수 있다면 빌드 시 정적으로 생성 가능
하지만 `generateStaticParams`가 없으면 필요한 페이지가 요청 시점에 동적으로 렌더링 될 수 있음
```ts
export async function generateStaticParams() {
  const posts = await fetch("https://.../posts").then((res) => res.join());

  return posts.map((post) => ({
    sluh: post.slug,
  }));
}
```
페이지에서는 전달된 `slug`값을 사용 가능
```ts
export default async function Page({
  params,
}: {
  params: Promise<{ slug: string }>;
}) {
  const { slug } = await params;

  // ...
}
```
`generateStaticParams`를 사용하면 미리 알 수 있는 동적 경로를 빌드 시 생성해 요청 시 렌더링 부담 줄일 수 있음

--- 

## 2026-09-23 (Week 4)
### Link Component

- 페이지 이동 시 사용하는 Next.js 컴포넌트
- `href` 속성 필수
- Root Layout은 반드시 존재하며 `html`, `body` 포함
- 하위 페이지의 Layout은 선택 사항

### 중첩 라우트

- 폴더를 중첩하면 URL 경로도 함께 중첩됨
- 예: `/blog/[slug]`
  - `/` : Root Segment
  - `blog` : Segment
  - `[slug]` : Leaf Segment
- `app/blog/page.tsx` → `/blog`
- `app/blog/[slug]/page.tsx` → `/blog/특정값`
- `[slug]`처럼 대괄호를 사용하면 동적 경로 생성 가능
- 게시글, 상품 상세 페이지 등에 활용

### 디렉터리 구조

```text
app/
└─ blog/
   ├─ page.tsx
   └─ [slug]/
      └─ page.tsx
```     

#### slug
- 특정 페이지를 식별하기 위한 URL 일부
- 예: `/blog/nextjs`에서 `nextjs`가 slug
- 데이터를 이용해 동적으로 페이지를 찾을 때 상ㅅㅇ

```ts
export const posts = [
  { slug: "nextjs", title: "Next.js 소개", content: "Next.js는 React 기반의 풀스택 프레임워크입니다." },
  { slug: "routing", title: "라우팅", content: "App Router 알아보기" },
];
```

```ts
const post = posts.find((p) => p.slug === params.slug);
```
- params : 동적 URL 값을 전달받음
- 구조 분해를 통해 params만 사용할 수 있음
- 비동기 데이터 처리 시 async, await 사용 가능

#### searchParams
URL의 검색 조건(Query String)을 읽을 때 사용
```text
/products?category=shoes&page=2
```
- category=shoes
- page=2
- 페이지네이션, 필터링 등에 활용
- App Router에서는 페이지의 props를 통해 전달받아 사용

---

## 2026-09-16 (Week 3)
### Folder and File Conventions
#### Route Group / Private Folder
- Route Group을 이용하면 URL에 영향을 주지 않고 폴더 구조 정리 가능
- `_folder` 형태는 비공개 폴더로 활용 가능
- 폴더를 생성해도 `page.js`, `page.tsx`, `route.js` 등이 없으면 해당 경로 공개 X

#### Parallel / Intercepting Routes
- 복잡한 UI, 모달 형태의 라우팅 등에 사용
- `@folder` : Named Slot
- Intercepting Route를 사용하면 현재 화면을 유지하면서 다른 경로의 내용을 표시 가능
- 목록 위에 상세 페이지를 모달로 띄우는 형태 등에 활용

| 항목 | 설명 |
|---|---|
| `@folder` | Named Slot |
| `(.)folder` | 같은 레벨 경로 가로채기 |
| `(..)folder` | 한 단계 위 경로 가로채기 |
| `(...)folder` | Root 기준 경로 가로채기 |

#### Open Graph Protocol
- 링크 공유 시 미리보기 정보를 제공하기 위한 프로토콜 
- Facebook. Instargram, X, KakaoTalk 등의 링크 미리보기에 활용

#### 프로젝트 구성
- Next.js 는 프로젝트 내부 파일 배치에 비교적 자유로움
- `app` 내부의 폴더 구조가 라우팅 구조 정의
- UI 로직과 라우팅 로직을 구분하여 관리 가능
- 관련 파일을 같은 위치에 두는 Colocation 방식 사용 가능

#### 주요 특수 파일
- `layout.js` / `layout.tsx`
- `template.js`
- `error.js`
- `loading.js`
- `not-found.js`
- `page.js` / `page.tsx`

#### Colocation
- 관련 파일과 폴더를 기능별로 가까이 배치하는 방식
- 프로젝트 구조를 이해하고 관리하기 쉬워짐
- app 안에 파일을 함께 둘 수도 있고 외부에 분리할 수도 있음

#### Layout vs Template
- layout : 페이지 이동 후에도 상태가 유지되는 형태
- template : 이동 시 새로 생성되어 상태가 초기화되는 형태

---

## 2026-09-09 (Week 2)
### pnpm
#### Performant NPM
pnpm은 npm, Yarn과 같은 Node.js 패키지 관리자

- 패키지를 전역 저장소에 한 번 저장하고 프로젝트에서는 링크로 참조
- 동일 패키지를 중복 저장하지 않아 디스크 공간 절약
- 이미 설치된 패키지를 재사용해 설치 속도가 빠름
- pnpm 사용 시 `pnpm-lock.yaml` 생성

#### Next.js에서 pnpm 사용

```bash
pnpm create next-app@latest
cd my-app
rm -rf node_modules package-lock.json
pnpm install
pnpm dev
```

#### Hard Link 
파일은 크게 다음요로소 구성
- Directory Entry: 파일 이름과 inode를 연결
- inode: 권환, 소유자, 크기, 데이터 위치 등의 메타데이터 저장
- Data Block: 실제 파일 데이터 저장

하드 링크는 원본을 복사하는 것이 아니라 **같은 inode를 참조하는 새로운 이름을 만드는 방식**
- 원본과 하드 링크는 같은 데이터를 공유
- 한쪽을 삭제해도 다른 링크가 남아 있으면 데이터 유지
- link count가 0이 되면 실제데이터 삭제

#### Hard Link vs Symbolic Link
##### Hard Link
- 원본과 동일한 inode 사용
- 동일한 Data Block 공유
- 원본 이름을 삭제해도 다른 하드 링크가 있으면 사용 가능

##### Symbolic Link
- 원본과 별도의 inode 사용
- 원본 파일의 경로를 저장
- 원본이 삭제되면 링크가 끊어질 수 있음
- Windows의 바로 가기와 비슷한 개념