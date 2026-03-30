# 바이브코딩으로 회사 홈페이지 구축하기 — 실전 워크플로우 가이드

> 바이브코딩(AI 보조 개발) 방식으로 회사 홈페이지를 신규 구축하고 CI/CD까지 완성하는 전체 과정을 단계별로 정리한다.
> 이 문서만 보고 처음부터 재현할 수 있어야 한다.

---

## 프로젝트 조건 요약

| 항목 | 내용 |
|------|------|
| 개발자 | 1명 (Claude Code + 바이브코딩) |
| 기존 홈페이지 | 없음 (신규 구축) |
| 콘텐츠 관리 | 비개발 직원도 텍스트/이미지 수정 가능 |
| 예산 | 호스팅 월 $0 (무료 티어) |
| 필요 페이지 | 메인, 회사소개, 솔루션(4~5개), 채용, 문의 |

---

## Step 1: 기술스택 선정

> **이 단계의 목표:** 바이브코딩에 최적화된 기술스택을 비교·선정한다.

### 1-1. 프레임워크

| 후보 | 바이브코딩 친화성 | 무료 호스팅 | 비개발자 수정 | 초기 개발 속도 | 커뮤니티/문서 |
|------|:-:|:-:|:-:|:-:|:-:|
| **Next.js (App Router)** | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| Astro | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| Gatsby | ⭐⭐ | ⭐⭐ | ⭐⭐ | ⭐⭐ | ⭐⭐ |

**✅ 선정: Next.js 14+ (App Router)**
Claude Code가 가장 많이 학습한 React 기반 프레임워크이며, Vercel 무료 티어와 완벽 호환된다. App Router의 파일 기반 라우팅은 AI가 페이지를 생성/수정할 때 구조를 예측하기 쉽다.

### 1-2. 스타일링

| 후보 | 바이브코딩 친화성 | 번들 크기 | 커스터마이징 | 커뮤니티/문서 |
|------|:-:|:-:|:-:|:-:|
| **Tailwind CSS** | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| CSS Modules | ⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐ |
| styled-components | ⭐⭐ | ⭐⭐ | ⭐⭐⭐ | ⭐⭐ |

**✅ 선정: Tailwind CSS v4**
유틸리티 클래스 기반이라 AI가 별도 CSS 파일 없이 JSX 안에서 스타일을 직접 생성할 수 있다. 프롬프트 한 번으로 레이아웃+스타일이 동시에 완성되므로 바이브코딩에 가장 효율적이다.

### 1-3. 콘텐츠 관리 (CMS)

| 후보 | 비개발자 편집 | 무료 티어 | 바이브코딩 통합 | 커뮤니티/문서 |
|------|:-:|:-:|:-:|:-:|
| **MDX 파일 기반** | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| Contentlayer + MD | ⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐ |
| Sanity CMS | ⭐⭐⭐ | ⭐⭐ | ⭐⭐ | ⭐⭐⭐ |

**✅ 선정: MDX 파일 기반 (content/ 디렉토리)**
외부 서비스 의존 없이 Git 저장소 안의 `.mdx` 파일만 수정하면 콘텐츠가 바뀐다. 비개발자도 GitHub 웹 UI에서 마크다운 텍스트를 직접 편집할 수 있고, Claude Code가 콘텐츠 파일을 직접 생성/수정하기에 가장 심플하다.

### 1-4. 호스팅

| 후보 | 무료 티어 범위 | Next.js 지원 | 배포 편의성 | 커뮤니티/문서 |
|------|:-:|:-:|:-:|:-:|
| **Vercel** | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| Cloudflare Pages | ⭐⭐⭐ | ⭐⭐ | ⭐⭐ | ⭐⭐ |
| Netlify | ⭐⭐ | ⭐⭐ | ⭐⭐⭐ | ⭐⭐ |

**✅ 선정: Vercel (Hobby 플랜)**
Next.js 제작사의 호스팅이라 zero-config 배포가 가능하고, Git push만으로 프리뷰/프로덕션 배포가 자동 완료된다. 무료 티어로 월 100GB 대역폭, 서버리스 함수 포함.

### 1-5. 패키지 매니저

| 후보 | 속도 | 바이브코딩 친화성 | 안정성 |
|------|:-:|:-:|:-:|
| **pnpm** | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| npm | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| yarn | ⭐⭐⭐ | ⭐⭐ | ⭐⭐ |

**✅ 선정: pnpm**
디스크 효율이 높고 설치 속도가 빠르며, Claude Code가 pnpm 명령어를 정확하게 생성한다.

### 최종 기술스택 요약

```
Next.js 14+ (App Router) + Tailwind CSS v4 + MDX + Vercel + pnpm
```

---

## Step 2: 프로젝트 초기 세팅

> **이 단계의 목표:** 터미널 명령어만으로 프로젝트를 생성하고, 바이브코딩에 최적화된 디렉토리 구조를 잡는다.

### 2-1. 프로젝트 생성 명령어

```bash
# 1. Next.js 프로젝트 생성
pnpm create next-app@latest company-homepage \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*"

# 2. 프로젝트 디렉토리 진입
cd company-homepage

# 3. 추가 의존성 설치
pnpm add next-mdx-remote gray-matter    # MDX 콘텐츠 처리
pnpm add -D @tailwindcss/typography     # 마크다운 타이포그래피

# 4. 개발 서버 실행 확인
pnpm dev
```

### 2-2. 디렉토리 구조

```
company-homepage/
├── src/
│   ├── app/                    # Next.js App Router 페이지
│   │   ├── layout.tsx          # 전역 레이아웃 (헤더/푸터)
│   │   ├── page.tsx            # 메인(홈) 페이지
│   │   ├── about/
│   │   │   └── page.tsx        # 회사소개 페이지
│   │   ├── solutions/
│   │   │   ├── page.tsx        # 솔루션 목록 페이지
│   │   │   └── [slug]/
│   │   │       └── page.tsx    # 솔루션 상세 페이지 (동적 라우팅)
│   │   ├── careers/
│   │   │   └── page.tsx        # 채용 페이지
│   │   └── contact/
│   │       └── page.tsx        # 문의 페이지
│   ├── components/             # 재사용 UI 컴포넌트
│   │   ├── Header.tsx          # 네비게이션 헤더
│   │   ├── Footer.tsx          # 푸터
│   │   ├── Hero.tsx            # 히어로 섹션
│   │   └── ContactForm.tsx     # 문의 폼
│   └── lib/
│       └── mdx.ts              # MDX 파일 읽기/파싱 유틸
├── content/                    # 콘텐츠 파일 (비개발자 편집 영역)
│   ├── solutions/              # 솔루션별 MDX 파일
│   │   ├── product-a.mdx
│   │   ├── product-b.mdx
│   │   └── ...
│   └── careers/                # 채용 공고 MDX 파일
│       └── frontend-engineer.mdx
├── public/                     # 정적 파일 (이미지, 파비콘 등)
│   └── images/
├── CLAUDE.md                   # 바이브코딩 컨텍스트 파일
├── .env.example                # 환경변수 템플릿
├── .env.local                  # 실제 환경변수 (git 무시)
└── package.json
```

### 2-3. CLAUDE.md 초안

```markdown
# CLAUDE.md — 프로젝트 컨벤션

## 프로젝트 개요
회사 공식 홈페이지. Next.js 14 App Router + Tailwind CSS + MDX 기반.

## 기술스택
- Framework: Next.js 14+ (App Router, TypeScript)
- Styling: Tailwind CSS v4 (유틸리티 클래스 우선, 커스텀 CSS 최소화)
- Content: MDX 파일 (content/ 디렉토리)
- Hosting: Vercel
- Package Manager: pnpm

## 코드 컨벤션
- 컴포넌트: PascalCase (예: HeroSection.tsx)
- 유틸/헬퍼: camelCase (예: getMdxContent.ts)
- 페이지 파일: Next.js 규칙 (page.tsx, layout.tsx)
- import 순서: React → Next.js → 외부 라이브러리 → 내부 모듈 → 타입

## 스타일 규칙
- Tailwind 유틸리티 클래스를 JSX에 직접 작성
- 반응형: mobile-first (sm → md → lg → xl)
- 색상: tailwind.config.ts의 커스텀 색상 팔레트 사용
- 다크모드: 지원하지 않음 (라이트 모드 전용)

## 콘텐츠 구조
- content/solutions/*.mdx: 솔루션 상세 (frontmatter: title, description, icon)
- content/careers/*.mdx: 채용 공고 (frontmatter: title, team, location, type)
- 이미지: public/images/ 하위에 페이지별 폴더로 분류

## 명령어
- pnpm dev: 개발 서버 (localhost:3000)
- pnpm build: 프로덕션 빌드
- pnpm lint: ESLint 검사
```

### 2-4. .env.example

```bash
# 문의 폼 이메일 발송용 (선택사항)
CONTACT_EMAIL_TO=info@company.com

# Google Analytics (선택사항)
NEXT_PUBLIC_GA_ID=G-XXXXXXXXXX

# 사이트 기본 URL
NEXT_PUBLIC_SITE_URL=https://www.company.com
```

---

## Step 3: 핵심 페이지 개발 워크플로우

> **이 단계의 목표:** Claude Code에 줄 프롬프트 예시와 활용 팁을 정리하여 바이브코딩 효율을 극대화한다.

### 3-1. 페이지별 예시 프롬프트

#### 프롬프트 1: 메인 페이지

```
메인 페이지(src/app/page.tsx)를 만들어줘.

구성:
1. 히어로 섹션: "디지털 혁신의 파트너" 헤드라인, 서브텍스트, CTA 버튼 2개(솔루션 보기, 문의하기)
2. 핵심 가치 섹션: 아이콘 + 제목 + 설명 카드 3개 (기술력, 신뢰성, 확장성)
3. 솔루션 미리보기: content/solutions/ MDX에서 읽어와 카드 4개 그리드 표시
4. CTA 배너: "지금 상담을 시작하세요" + 문의 페이지 링크

스타일: Tailwind CSS, 모바일 반응형, 깔끔한 기업용 디자인
색상: 파란색 계열 primary (#2563EB), 그레이 계열 배경
```

> **왜 효과적인가:** 섹션별 구성·텍스트·색상을 구체적으로 지정하여 AI의 추측을 최소화하고 한 번에 원하는 결과물을 얻는다.

#### 프롬프트 2: 솔루션 상세 페이지

```
src/app/solutions/[slug]/page.tsx를 동적 라우팅으로 만들어줘.

요구사항:
- content/solutions/ 폴더의 MDX 파일을 slug로 매칭
- frontmatter(title, description, icon, features[])를 읽어서 표시
- 레이아웃: 왼쪽에 본문 내용, 오른쪽에 기능 목록 사이드바 (lg 이상)
- 하단에 "다른 솔루션 보기" 링크 (현재 솔루션 제외)
- generateStaticParams로 빌드 시 정적 생성
- lib/mdx.ts에 getMdxBySlug, getAllSlugs 함수도 함께 작성
```

> **왜 효과적인가:** 데이터 흐름(MDX → frontmatter → UI)을 명시하고, 필요한 유틸 함수까지 범위를 지정하여 누락 없이 완성된 기능을 받는다.

#### 프롬프트 3: 문의 폼 페이지

```
src/app/contact/page.tsx에 문의 폼을 만들어줘.

폼 필드: 이름, 이메일, 회사명, 문의 유형(드롭다운: 제품 문의/파트너십/채용/기타), 메시지(textarea)
기능:
- 클라이언트 사이드 유효성 검사 (필수 필드, 이메일 형식)
- Server Action으로 폼 제출 처리
- 제출 성공/실패 시 토스트 메시지 표시
- 제출 중 버튼 로딩 상태

Server Action(src/app/contact/actions.ts)도 함께 만들어줘.
일단은 콘솔 로그만 찍고, 나중에 이메일 API 연결할 수 있도록 구조만 잡아줘.
```

> **왜 효과적인가:** 폼 필드를 정확히 나열하고, 프론트와 백엔드(Server Action)를 한 프롬프트에서 함께 요청하여 통합된 결과물을 얻는다.

### 3-2. 레퍼런스 사이트 활용 방법

바이브코딩에서 디자인 레퍼런스를 활용하는 3가지 방법:

#### 방법 1: URL 직접 참조

```
메인 페이지의 히어로 섹션을 만들어줘.
레이아웃 참고: https://stripe.com 의 히어로 섹션처럼
왼쪽에 텍스트+CTA, 오른쪽에 일러스트 영역으로 구성해줘.
```

> Claude Code가 해당 사이트의 일반적인 레이아웃 패턴을 알고 있으므로, URL만으로도 방향을 잡을 수 있다.

#### 방법 2: 스크린샷 첨부

```
첨부한 스크린샷과 비슷한 레이아웃의 솔루션 카드 그리드를 만들어줘.
- 카드 간격, 그림자, 라운딩을 유사하게
- 색상은 우리 primary 파란색 계열로
- 반응형: 모바일 1열, 태블릿 2열, 데스크톱 3열
```

> Claude Code에 이미지 파일 경로를 넘기면 시각적 레이아웃을 분석하여 유사한 구조를 생성한다.

#### 방법 3: 구체적 키워드 조합

```
Vercel 홈페이지 스타일의 미니멀한 네비게이션 바를 만들어줘.
- 로고 왼쪽, 메뉴 가운데, CTA 버튼 오른쪽
- 스크롤 시 배경 blur 효과 (backdrop-blur)
- 모바일에서 햄버거 메뉴
```

> 유명 사이트 이름 + "스타일" 키워드로 디자인 방향을 전달하면, AI가 해당 디자인 패턴을 적용한다.

---

## Step 4: CI/CD 파이프라인

> **이 단계의 목표:** GitHub Actions로 PR 검증 + 프로덕션 자동 배포 파이프라인을 구축한다.

### 4-1. GitHub Actions 워크플로우

#### 파일: `.github/workflows/ci.yml` — PR 검증

```yaml
name: CI — PR 검증

# PR이 열리거나 업데이트될 때 실행
on:
  pull_request:
    branches: [main]

# 동시 실행 방지: 같은 PR에 새 커밋이 오면 이전 실행 취소
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  validate:
    name: Lint & Build 검증
    runs-on: ubuntu-latest

    steps:
      # 1. 코드 체크아웃
      - name: 코드 체크아웃
        uses: actions/checkout@v4

      # 2. pnpm 설치
      - name: pnpm 설치
        uses: pnpm/action-setup@v4
        with:
          version: 9

      # 3. Node.js 설정 (pnpm 캐시 활성화)
      - name: Node.js 20 설정
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      # 4. 의존성 설치
      - name: 의존성 설치
        run: pnpm install --frozen-lockfile

      # 5. ESLint 검사
      - name: 린트 검사
        run: pnpm lint

      # 6. TypeScript 타입 검사
      - name: 타입 검사
        run: pnpm tsc --noEmit

      # 7. 프로덕션 빌드 테스트
      - name: 빌드 검증
        run: pnpm build
```

#### 파일: `.github/workflows/deploy.yml` — 프로덕션 배포

```yaml
name: Deploy — 프로덕션 배포

# main 브랜치에 머지될 때만 실행
on:
  push:
    branches: [main]

# 동시 배포 방지
concurrency:
  group: deploy-production
  cancel-in-progress: false

jobs:
  deploy:
    name: Vercel 프로덕션 배포
    runs-on: ubuntu-latest

    steps:
      # 1. 코드 체크아웃
      - name: 코드 체크아웃
        uses: actions/checkout@v4

      # 2. pnpm 설치
      - name: pnpm 설치
        uses: pnpm/action-setup@v4
        with:
          version: 9

      # 3. Node.js 설정
      - name: Node.js 20 설정
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      # 4. 의존성 설치
      - name: 의존성 설치
        run: pnpm install --frozen-lockfile

      # 5. 프로덕션 빌드
      - name: 프로덕션 빌드
        run: pnpm build

      # 6. Vercel CLI로 배포
      - name: Vercel CLI 설치
        run: pnpm add -g vercel

      # 7. 프로덕션 배포 실행
      - name: 프로덕션 배포
        run: vercel deploy --prod --token=${{ secrets.VERCEL_TOKEN }}
        env:
          VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
          VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}
```

### 4-2. GitHub Secrets 체크리스트

GitHub 저장소 → Settings → Secrets and variables → Actions에 아래 값을 등록한다.

| Secret 이름 | 설명 | 얻는 방법 |
|-------------|------|-----------|
| `VERCEL_TOKEN` | Vercel API 토큰 | [Vercel 대시보드](https://vercel.com/account/tokens) → Create Token |
| `VERCEL_ORG_ID` | Vercel 조직(팀) ID | 프로젝트 루트에서 `vercel link` 실행 → `.vercel/project.json`의 `orgId` |
| `VERCEL_PROJECT_ID` | Vercel 프로젝트 ID | 위와 동일 파일의 `projectId` |

**초기 설정 순서:**

```bash
# 1. Vercel CLI 설치 및 로그인
pnpm add -g vercel
vercel login

# 2. 프로젝트 연결 (최초 1회)
vercel link

# 3. .vercel/project.json에서 orgId, projectId 확인
cat .vercel/project.json

# 4. GitHub Secrets에 3개 값 등록
#    - VERCEL_TOKEN: Vercel 대시보드에서 생성
#    - VERCEL_ORG_ID: project.json의 orgId
#    - VERCEL_PROJECT_ID: project.json의 projectId
```

### 4-3. 배포 후 정상 작동 확인 방법

```bash
# 1. Vercel 배포 상태 확인
vercel ls

# 2. 프로덕션 URL에 HTTP 요청 테스트
curl -I https://www.company.com

# 3. 확인할 항목 체크리스트
```

| 확인 항목 | 방법 | 기대 결과 |
|-----------|------|-----------|
| 페이지 로드 | 브라우저에서 모든 페이지 접속 | 200 OK, 콘텐츠 정상 표시 |
| 모바일 반응형 | Chrome DevTools → 모바일 뷰 | 레이아웃 깨짐 없음 |
| 메타 태그/OG | [ogp.me 검증기](https://www.opengraph.xyz/) | 제목/설명/이미지 정상 |
| 문의 폼 | 테스트 데이터로 제출 | 성공 메시지 표시, 서버 로그 확인 |
| 빌드 시간 | GitHub Actions 로그 | 3분 이내 완료 |
| Lighthouse | Chrome DevTools → Lighthouse | Performance 90+, SEO 90+ |

---

## 부록: 바이브코딩 팁

### 효과적인 프롬프트 작성 원칙

1. **구체적으로 지정하라** — 섹션 구성, 필드 이름, 색상 코드까지 명시하면 수정 횟수가 줄어든다
2. **파일 경로를 포함하라** — `src/app/page.tsx`처럼 정확한 경로를 주면 AI가 올바른 위치에 코드를 생성한다
3. **데이터 흐름을 설명하라** — "MDX에서 읽어서 카드로 표시"처럼 입력→출력을 명시한다
4. **한 번에 하나의 페이지** — 여러 페이지를 동시에 요청하면 품질이 떨어진다. 한 페이지씩 완성 후 다음으로 넘어간다
5. **CLAUDE.md를 유지하라** — 프로젝트 컨벤션을 CLAUDE.md에 기록해두면 AI가 일관된 코드를 생성한다
