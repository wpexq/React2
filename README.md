# 202430113 안지혜
## 2026-09-23 (Week 4)

## Link Component

- 페이지 이동 시 사용하는 Next.js 컴포넌트
- `href` 속성 필수
- Root Layout은 반드시 존재하며 `html`, `body` 포함
- 하위 페이지의 Layout은 선택 사항

## 중첩 라우트

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