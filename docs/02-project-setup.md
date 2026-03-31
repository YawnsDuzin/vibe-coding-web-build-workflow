# Step 2: 프로젝트 초기 세팅

> **이 단계의 목표:** 터미널 명령어만으로 프로젝트를 생성하고, 바이브코딩에 최적화된 디렉토리 구조와 설정 파일을 완성한다.

---

## 2-1. 사전 준비 확인

프로젝트를 시작하기 전에 아래 도구들이 설치되어 있는지 확인한다.

```bash
# Node.js 20 이상 확인
node -v
# 예: v20.11.0

# pnpm 설치 (없으면 설치)
corepack enable
corepack prepare pnpm@latest --activate
pnpm -v
# 예: 9.1.0

# Git 확인
git -v
# 예: git version 2.43.0

# Claude Code 확인 (바이브코딩 도구)
claude --version
```

### corepack이 안 되는 경우

```bash
# npm으로 직접 pnpm 설치
npm install -g pnpm@latest
```

---

## 2-2. 프로젝트 생성

### 명령어 블록 (순서대로 실행)

```bash
# ① Next.js 프로젝트 생성
# --typescript: TypeScript 활성화
# --tailwind: Tailwind CSS 포함
# --eslint: ESLint 기본 설정 포함
# --app: App Router 사용 (Pages Router 대신)
# --src-dir: src/ 디렉토리 구조 사용
# --import-alias: @ 별칭으로 절대 경로 import
pnpm create next-app@latest company-homepage \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*"

# ② 프로젝트 디렉토리 진입
cd company-homepage

# ③ MDX 관련 의존성 설치
# next-mdx-remote: MDX 파일을 Next.js에서 렌더링
# gray-matter: MDX frontmatter(메타데이터) 파싱
pnpm add next-mdx-remote gray-matter

# ④ 개발용 의존성 설치
# @tailwindcss/typography: 마크다운 콘텐츠에 타이포그래피 스타일 적용
pnpm add -D @tailwindcss/typography

# ⑤ 콘텐츠 디렉토리 생성
mkdir -p content/solutions
mkdir -p content/careers
mkdir -p public/images

# ⑥ 개발 서버 실행 확인
pnpm dev
# http://localhost:3000 에서 Next.js 기본 페이지 확인 후 Ctrl+C로 종료
```

### 각 명령어 설명

| 순서 | 명령어 | 무엇을 하는가 |
|:----:|--------|---------------|
| ① | `pnpm create next-app` | Next.js 보일러플레이트 생성. 옵션으로 TS, Tailwind, ESLint, App Router를 한 번에 세팅 |
| ③ | `pnpm add next-mdx-remote gray-matter` | MDX 콘텐츠를 읽고 렌더링하기 위한 핵심 라이브러리 2개 설치 |
| ④ | `pnpm add -D @tailwindcss/typography` | `prose` 클래스로 마크다운 본문에 읽기 좋은 타이포그래피를 적용 |
| ⑤ | `mkdir -p content/...` | 비개발자가 편집할 콘텐츠 파일을 저장할 디렉토리 생성 |

---

## 2-3. 디렉토리 구조

```
company-homepage/
│
├── src/                           # 소스 코드 루트
│   ├── app/                       # Next.js App Router — 파일 기반 라우팅
│   │   ├── layout.tsx             # 전역 레이아웃 (헤더, 푸터, 메타 태그)
│   │   ├── page.tsx               # 메인(홈) 페이지 → /
│   │   ├── globals.css            # Tailwind 지시어 + 전역 CSS
│   │   │
│   │   ├── about/
│   │   │   └── page.tsx           # 회사소개 → /about
│   │   │
│   │   ├── solutions/
│   │   │   ├── page.tsx           # 솔루션 목록 → /solutions
│   │   │   └── [slug]/
│   │   │       └── page.tsx       # 솔루션 상세 → /solutions/product-a
│   │   │
│   │   ├── careers/
│   │   │   └── page.tsx           # 채용 → /careers
│   │   │
│   │   └── contact/
│   │       ├── page.tsx           # 문의 폼 → /contact
│   │       └── actions.ts         # Server Action (폼 처리 로직)
│   │
│   ├── components/                # 재사용 가능한 UI 컴포넌트
│   │   ├── layout/
│   │   │   ├── Header.tsx         # 네비게이션 바 (로고 + 메뉴 + CTA)
│   │   │   └── Footer.tsx         # 푸터 (링크 + 저작권)
│   │   ├── home/
│   │   │   ├── Hero.tsx           # 메인 히어로 섹션
│   │   │   ├── ValueCards.tsx     # 핵심 가치 카드 3개
│   │   │   └── SolutionPreview.tsx # 솔루션 미리보기 그리드
│   │   ├── solutions/
│   │   │   ├── SolutionCard.tsx   # 솔루션 카드 (목록용)
│   │   │   └── FeatureSidebar.tsx # 기능 목록 사이드바 (상세용)
│   │   └── contact/
│   │       └── ContactForm.tsx    # 문의 폼 (클라이언트 컴포넌트)
│   │
│   ├── lib/                       # 유틸리티 함수
│   │   ├── mdx.ts                 # MDX 파일 읽기/파싱 (getMdxBySlug, getAllSlugs)
│   │   └── constants.ts           # 사이트 전역 상수 (회사명, 메뉴 항목 등)
│   │
│   └── types/                     # TypeScript 타입 정의
│       └── content.ts             # MDX frontmatter 타입 (Solution, Career 등)
│
├── content/                       # 콘텐츠 파일 (비개발자 편집 영역)
│   ├── solutions/                 # 솔루션별 MDX 파일
│   │   ├── product-a.mdx          # 제품 A 상세 설명
│   │   ├── product-b.mdx          # 제품 B 상세 설명
│   │   ├── product-c.mdx          # 제품 C 상세 설명
│   │   └── product-d.mdx          # 제품 D 상세 설명
│   └── careers/                   # 채용 공고 MDX 파일
│       ├── frontend-engineer.mdx  # 프론트엔드 엔지니어
│       └── backend-engineer.mdx   # 백엔드 엔지니어
│
├── public/                        # 정적 파일 (빌드 시 그대로 복사)
│   ├── images/
│   │   ├── hero/                  # 히어로 섹션 이미지
│   │   ├── solutions/             # 솔루션별 이미지
│   │   └── team/                  # 팀 소개 사진
│   ├── favicon.ico                # 파비콘
│   └── og-image.png               # OG(소셜 미리보기) 이미지
│
├── .github/
│   └── workflows/
│       ├── ci.yml                 # PR 검증 (lint + build)
│       └── deploy.yml             # 프로덕션 자동 배포
│
├── CLAUDE.md                      # 바이브코딩 컨텍스트 파일
├── .env.example                   # 환경변수 템플릿
├── .env.local                     # 실제 환경변수 (git 무시됨)
├── .gitignore                     # Git 무시 파일 목록
├── next.config.ts                 # Next.js 설정
├── tailwind.config.ts             # Tailwind CSS 커스텀 설정
├── tsconfig.json                  # TypeScript 설정
├── eslint.config.mjs              # ESLint 설정
├── package.json                   # 프로젝트 메타 + 의존성
└── pnpm-lock.yaml                 # pnpm 잠금 파일
```

### 디렉토리 설계 원칙

| 원칙 | 설명 |
|------|------|
| **관심사 분리** | `app/`(라우팅) → `components/`(UI) → `lib/`(로직) → `content/`(데이터)로 역할 분리 |
| **기능별 그룹핑** | `components/` 하위를 `layout/`, `home/`, `solutions/`, `contact/` 등 기능별 폴더로 구성 |
| **AI 예측 가능성** | Next.js 공식 컨벤션을 충실히 따라 Claude Code가 파일 위치를 정확히 예측할 수 있게 함 |
| **콘텐츠 분리** | `content/` 폴더를 소스 코드와 분리하여 비개발자가 수정할 영역을 명확히 구분 |

---

## 2-4. CLAUDE.md (바이브코딩 컨텍스트 파일)

이 파일은 Claude Code가 프로젝트의 규칙과 패턴을 이해하기 위한 **가장 중요한 파일**이다.
프로젝트 루트에 `CLAUDE.md`를 생성하면 Claude Code가 매 세션 시작 시 자동으로 읽는다.

```markdown
# CLAUDE.md — 프로젝트 컨벤션

## 프로젝트 개요
회사 공식 홈페이지. Next.js 14 App Router + Tailwind CSS + MDX 기반.
바이브코딩으로 개발 중이며, 이 파일의 규칙을 반드시 따를 것.

## 기술스택
- Framework: Next.js 14+ (App Router, TypeScript strict mode)
- Styling: Tailwind CSS v4 (유틸리티 클래스 우선, 커스텀 CSS 최소화)
- Content: MDX 파일 (content/ 디렉토리), next-mdx-remote + gray-matter로 파싱
- Hosting: Vercel (Hobby)
- Package Manager: pnpm (npm, yarn 사용 금지)

## 코드 컨벤션

### 파일 네이밍
- 컴포넌트 파일: PascalCase (예: HeroSection.tsx, SolutionCard.tsx)
- 유틸/헬퍼 파일: camelCase (예: getMdxContent.ts, formatDate.ts)
- 페이지 파일: Next.js App Router 규칙 (page.tsx, layout.tsx, loading.tsx, error.tsx)
- 콘텐츠 파일: kebab-case (예: product-a.mdx, frontend-engineer.mdx)

### import 순서
1. React/Next.js 내장 모듈 (react, next/link, next/image 등)
2. 외부 라이브러리 (next-mdx-remote, gray-matter 등)
3. 내부 컴포넌트 (@/components/...)
4. 내부 유틸 (@/lib/...)
5. 타입 (@/types/...)

### 컴포넌트 작성 규칙
- 함수 선언식 export default 사용: `export default function ComponentName()`
- Props 타입은 컴포넌트 파일 상단에 interface로 정의
- 'use client' 지시어는 꼭 필요한 컴포넌트에만 사용 (기본은 Server Component)
- 이미지는 반드시 next/image의 <Image> 컴포넌트 사용

## 스타일 규칙
- Tailwind 유틸리티 클래스를 JSX className에 직접 작성
- 반응형: mobile-first 접근 (sm: → md: → lg: → xl:)
- 색상: primary(파란색 계열 #2563EB), gray(Tailwind 기본 gray)
- 간격: Tailwind 기본 스페이싱 사용 (p-4, m-6, gap-8 등)
- 다크모드: 지원하지 않음 (라이트 모드 전용)
- 최대 너비: 컨테이너 max-w-7xl mx-auto 패턴 사용

## 콘텐츠 구조

### 솔루션 MDX (content/solutions/*.mdx)
---
title: "제품명"
description: "한 줄 설명"
icon: "아이콘 이름"
features:
  - "기능 1"
  - "기능 2"
order: 1
---

### 채용 MDX (content/careers/*.mdx)
---
title: "포지션명"
team: "팀명"
location: "서울"
type: "정규직"
isOpen: true
---

## 명령어
- pnpm dev: 개발 서버 (localhost:3000)
- pnpm build: 프로덕션 빌드
- pnpm lint: ESLint 검사
- pnpm tsc --noEmit: TypeScript 타입 검사

## 주의사항
- node_modules/, .next/, .env.local은 절대 커밋하지 않을 것
- API 키나 시크릿을 코드에 하드코딩하지 않을 것
- content/ 폴더의 MDX 파일은 비개발자도 편집하므로, frontmatter 스키마를 변경할 때는 기존 파일도 함께 업데이트할 것
```

### CLAUDE.md 활용 팁

| 상황 | CLAUDE.md에 추가할 내용 |
|------|----------------------|
| 새 컴포넌트 패턴 도입 | "카드 컴포넌트는 항상 rounded-xl shadow-sm border 패턴 사용" |
| 자주 발생하는 실수 교정 | "Link 컴포넌트는 next/link에서 import (react-router 아님)" |
| 특정 라이브러리 사용법 | "MDX 렌더링은 항상 next-mdx-remote의 MDXRemote 사용" |
| 디자인 토큰 변경 | "primary 색상을 #1D4ED8에서 #2563EB로 변경함" |

---

## 2-5. 환경변수 설정

### .env.example (Git에 커밋 — 템플릿)

```bash
# ============================================
# 환경변수 템플릿
# 이 파일을 .env.local로 복사하여 실제 값을 입력하세요
# cp .env.example .env.local
# ============================================

# --- 필수 ---

# 사이트 기본 URL (SEO 메타 태그, OG 이미지 등에 사용)
NEXT_PUBLIC_SITE_URL=https://www.company.com

# --- 선택: 문의 폼 ---

# 문의 폼 수신 이메일
CONTACT_EMAIL_TO=info@company.com

# 이메일 발송 서비스 (Resend, SendGrid 등)
# RESEND_API_KEY=re_xxxxxxxxxxxx

# --- 선택: 분석 ---

# Google Analytics 측정 ID
# NEXT_PUBLIC_GA_ID=G-XXXXXXXXXX

# --- 선택: 이미지 CDN ---

# Cloudinary (대용량 이미지 관리 시)
# NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME=your-cloud-name
```

### .env.local 생성 (Git에서 무시됨)

```bash
# 프로젝트 루트에서 실행
cp .env.example .env.local

# .env.local을 열어 실제 값으로 수정
# 에디터로 열거나 Claude Code에게 요청:
# "env.local 파일의 NEXT_PUBLIC_SITE_URL을 https://mycompany.com으로 변경해줘"
```

### .gitignore 확인

Next.js가 자동 생성하는 `.gitignore`에 이미 포함되어 있지만, 확인할 항목:

```gitignore
# 환경변수 (반드시 무시)
.env.local
.env*.local

# 빌드 출력
.next/
out/

# 의존성
node_modules/

# Vercel
.vercel/

# OS 파일
.DS_Store
Thumbs.db
```

---

## 2-6. Tailwind CSS 커스텀 설정

`tailwind.config.ts`에 프로젝트에 맞는 커스텀 설정을 추가한다.

```typescript
import type { Config } from "tailwindcss";

const config: Config = {
  content: [
    "./src/**/*.{js,ts,jsx,tsx,mdx}",
    "./content/**/*.{md,mdx}",
  ],
  theme: {
    extend: {
      colors: {
        // 회사 브랜드 색상
        primary: {
          50: "#EFF6FF",
          100: "#DBEAFE",
          200: "#BFDBFE",
          300: "#93C5FD",
          400: "#60A5FA",
          500: "#3B82F6",
          600: "#2563EB",  // 메인 primary
          700: "#1D4ED8",
          800: "#1E40AF",
          900: "#1E3A8A",
          950: "#172554",
        },
      },
      fontFamily: {
        // 한글 웹폰트 (Google Fonts에서 로드)
        sans: [
          "Pretendard",
          "-apple-system",
          "BlinkMacSystemFont",
          "system-ui",
          "sans-serif",
        ],
      },
      // 최대 너비 커스텀
      maxWidth: {
        "8xl": "88rem",  // 1408px
      },
    },
  },
  plugins: [
    require("@tailwindcss/typography"),
  ],
};

export default config;
```

---

## 2-7. 개발 서버 실행 및 확인

```bash
# 개발 서버 시작
pnpm dev

# 정상 실행 시 출력:
#   ▲ Next.js 14.x.x
#   - Local:        http://localhost:3000
#   - Environments: .env.local
#   ✓ Ready in XXXms
```

### 확인 체크리스트

| # | 확인 항목 | 방법 |
|:-:|----------|------|
| 1 | 개발 서버 정상 시작 | `pnpm dev` → 에러 없이 Ready 메시지 출력 |
| 2 | 브라우저 접속 | http://localhost:3000 → Next.js 기본 페이지 표시 |
| 3 | Tailwind 동작 | 기본 페이지에 Tailwind 스타일이 적용되어 있음 |
| 4 | TypeScript 동작 | `pnpm tsc --noEmit` → 에러 없음 |
| 5 | ESLint 동작 | `pnpm lint` → 에러 없음 |
| 6 | 빌드 성공 | `pnpm build` → 에러 없이 완료 |

### 문제 해결

| 증상 | 원인 | 해결 |
|------|------|------|
| `pnpm: command not found` | pnpm 미설치 | `npm install -g pnpm` |
| 포트 3000 이미 사용 중 | 다른 프로세스가 점유 | `pnpm dev -p 3001` 또는 기존 프로세스 종료 |
| Tailwind 스타일 미적용 | `content` 경로 누락 | `tailwind.config.ts`의 content 배열 확인 |
| MDX import 에러 | 패키지 미설치 | `pnpm add next-mdx-remote gray-matter` |

---

## 2-8. Git 초기 커밋

```bash
# 현재 상태 확인
git status

# 초기 파일 스테이징 (불필요한 파일 제외 확인)
git add .

# 초기 커밋
git commit -m "chore: Next.js 프로젝트 초기 세팅

- Next.js 14 App Router + TypeScript + Tailwind CSS
- MDX 콘텐츠 관리 구조 (content/ 디렉토리)
- CLAUDE.md 바이브코딩 컨텍스트 파일
- ESLint + 환경변수 템플릿"

# GitHub 원격 저장소 연결 (새 저장소인 경우)
git remote add origin https://github.com/your-org/company-homepage.git
git push -u origin main
```

> **다음 단계:** [Step 3: 핵심 페이지 개발 워크플로우](./03-page-development.md)에서 실제 페이지를 바이브코딩으로 개발한다.
