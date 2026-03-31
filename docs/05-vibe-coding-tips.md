# Step 5: 바이브코딩 팁 & 부록

> **이 단계의 목표:** 바이브코딩(AI 보조 개발)의 효율을 극대화하는 실전 팁, 트러블슈팅, 참고 자료를 정리한다.

---

## 5-1. 효과적인 프롬프트 작성 원칙

### 원칙 1: 구체적으로 지정하라

```
❌ 나쁜 예: "솔루션 페이지 만들어줘"
✅ 좋은 예: "src/app/solutions/page.tsx를 만들어줘.
             카드 그리드 3열(모바일 1열), primary-600 색상,
             content/solutions/*.mdx에서 데이터를 읽어와 표시"
```

구체적 지정의 체크리스트:
- [ ] 파일 경로 명시
- [ ] 색상 코드 또는 Tailwind 클래스 지정
- [ ] 반응형 열 수 (모바일/태블릿/데스크톱)
- [ ] 데이터 소스 위치 및 필드명
- [ ] UI 텍스트 (제목, 버튼 라벨 등)

### 원칙 2: 파일 경로를 포함하라

```
❌ "헤더 컴포넌트를 수정해줘"
✅ "src/components/layout/Header.tsx의 네비게이션 메뉴에
    '블로그' 링크를 '채용'과 '문의' 사이에 추가해줘"
```

AI가 올바른 파일을 찾고, 올바른 위치에 코드를 수정하기 위해 경로는 필수다.

### 원칙 3: 데이터 흐름을 설명하라

```
❌ "솔루션 카드를 만들어줘"
✅ "content/solutions/*.mdx의 frontmatter(title, description, icon)를
    lib/mdx.ts의 getAllSolutions()로 읽어서
    SolutionCard 컴포넌트에 props로 전달하고
    솔루션 목록 페이지에서 3열 그리드로 렌더링해줘"
```

입력(데이터 소스) → 처리(유틸 함수) → 출력(UI 컴포넌트) 흐름을 명시한다.

### 원칙 4: 한 번에 하나의 페이지

```
❌ "메인, 회사소개, 솔루션, 채용, 문의 페이지를 모두 만들어줘"
✅ "메인 페이지(src/app/page.tsx)를 만들어줘. [상세 구성...]"
   → 완성 후 →
   "회사소개 페이지(src/app/about/page.tsx)를 만들어줘. [상세 구성...]"
```

여러 페이지를 동시에 요청하면 각 페이지의 품질이 떨어진다. 한 페이지를 완성하고 확인한 후 다음으로 넘어간다.

### 원칙 5: CLAUDE.md를 꾸준히 업데이트하라

```markdown
# 프로젝트 진행 중 CLAUDE.md에 추가할 내용 예시

## 최근 변경사항
- 2024-01-15: primary 색상을 #2563EB → #1D4ED8로 변경
- 2024-01-16: 솔루션 카드에 hover 시 shadow-lg 효과 추가

## 패턴
- 카드 컴포넌트: 항상 rounded-xl shadow-sm border border-gray-100 패턴 사용
- 섹션 간격: py-16 lg:py-24 통일
- 섹션 제목: text-3xl font-bold text-center mb-12
```

CLAUDE.md는 AI의 "장기 기억"이다. 프로젝트가 진행될수록 새로운 패턴과 결정사항을 기록해야 AI가 일관된 코드를 생성한다.

---

## 5-2. 바이브코딩 워크플로우 패턴

### 패턴 A: 생성 → 확인 → 수정 반복

```
[1차 프롬프트] 페이지 전체 생성 요청
  ↓
[확인] 브라우저에서 결과 확인
  ↓
[2차 프롬프트] 세부 수정 요청
  "히어로 섹션의 서브텍스트를 2줄로 줄이고,
   CTA 버튼 간격을 gap-4로 넓혀줘"
  ↓
[확인] 다시 확인
  ↓
[완성] 다음 페이지로 이동
```

> 핵심: 1차에 80%를 만들고, 2~3차 수정으로 100%를 완성한다.

### 패턴 B: 기존 코드 참조 지시

```
"Header.tsx와 동일한 스타일 패턴으로
 채용 페이지의 상단 히어로를 만들어줘.
 배경색은 gray-50, 제목은 '채용'으로"
```

이미 만들어진 컴포넌트를 참조하라고 명시하면, AI가 기존 코드의 스타일 패턴을 읽고 일관된 디자인을 생성한다.

### 패턴 C: 에러 메시지 복붙

```
"pnpm build 실행 시 아래 에러가 발생해. 수정해줘:

Error: Cannot find module '@/lib/mdx'
  at /src/app/solutions/page.tsx:3:1"
```

에러 메시지를 그대로 복사하여 전달하면, AI가 정확한 원인을 파악하고 수정한다. 에러를 설명하려 하지 말고 원본 메시지를 붙여넣는 것이 가장 효과적이다.

### 패턴 D: 점진적 복잡도 증가

```
[1단계] "간단한 문의 폼을 만들어줘. 이름, 이메일, 메시지 필드만"
  ↓
[2단계] "문의 유형 드롭다운(제품 문의/파트너십/채용/기타)을 추가해줘"
  ↓
[3단계] "클라이언트 사이드 유효성 검사를 추가해줘. 필수 필드, 이메일 형식 체크"
  ↓
[4단계] "Server Action으로 폼 제출 처리를 연결해줘"
```

복잡한 기능은 한 번에 완성하려 하지 말고, 단계별로 기능을 쌓아 올린다.

---

## 5-3. 자주 발생하는 문제와 해결법

### 빌드 에러

| 에러 메시지 | 원인 | 해결 프롬프트 |
|------------|------|-------------|
| `Module not found: '@/lib/mdx'` | 파일 경로 오류 | "src/lib/mdx.ts 파일이 있는지 확인하고, 없으면 만들어줘" |
| `Type 'string' is not assignable to type...` | TypeScript 타입 불일치 | "이 에러를 수정해줘: [에러 전문 붙여넣기]" |
| `'use client' must be at the top` | 지시어 위치 오류 | "ContactForm.tsx 파일 맨 위에 'use client'를 추가해줘" |
| `Image is missing required "alt"` | next/image alt 속성 누락 | "모든 Image 컴포넌트에 alt 텍스트를 추가해줘" |

### 스타일 문제

| 증상 | 원인 | 해결 프롬프트 |
|------|------|-------------|
| Tailwind 클래스 미적용 | tailwind.config content 경로 누락 | "tailwind.config.ts의 content에 모든 소스 경로가 포함되어 있는지 확인해줘" |
| 모바일에서 가로 스크롤 | 컨테이너 overflow 문제 | "모바일에서 가로 스크롤이 발생해. overflow-hidden 추가해줘" |
| 폰트 미적용 | 웹폰트 로딩 실패 | "layout.tsx에서 Pretendard 폰트 로딩 설정을 확인해줘" |

### MDX 관련

| 증상 | 원인 | 해결 프롬프트 |
|------|------|-------------|
| frontmatter 데이터 undefined | gray-matter 파싱 오류 | "content/solutions/product-a.mdx의 frontmatter 형식을 확인해줘" |
| MDX 렌더링 안 됨 | next-mdx-remote 설정 오류 | "솔루션 상세 페이지의 MDX 렌더링 코드를 점검해줘" |
| 한글 slug 문제 | URL 인코딩 이슈 | "slug를 영문 kebab-case로 통일해줘 (예: cloud-manager)" |

---

## 5-4. 성능 최적화 프롬프트

프로젝트 완성 후, 아래 프롬프트로 성능을 개선할 수 있다.

### 이미지 최적화

```
모든 페이지에서 <img> 태그를 next/image의 <Image>로 교체해줘.
- width, height를 적절히 지정
- priority는 히어로 이미지에만 true
- loading="lazy"는 스크롤 아래 이미지에 적용
- placeholder="blur"는 로컬 이미지에만 적용
```

### 번들 크기 최적화

```
next.config.ts에 bundle-analyzer를 추가하고,
번들 크기를 분석해서 큰 의존성이 있으면 알려줘.

pnpm add -D @next/bundle-analyzer
```

### Core Web Vitals 개선

```
Lighthouse 점수를 개선해줘:
1. LCP: 히어로 이미지에 priority 속성 추가
2. CLS: 이미지/폰트에 width/height 명시로 레이아웃 시프트 방지
3. FID: 불필요한 클라이언트 JS 줄이기 ('use client' 최소화)
```

---

## 5-5. 유지보수 가이드

### 새 솔루션 추가하기

```
1. content/solutions/new-product.mdx 파일 생성
2. frontmatter 작성:
   ---
   title: "새 제품명"
   description: "한 줄 설명"
   icon: "아이콘 이름"
   features:
     - "기능 1"
     - "기능 2"
   order: 5
   ---
3. 본문에 제품 상세 설명 마크다운 작성
4. PR 생성 → CI 통과 → 머지 → 자동 배포
```

### 새 채용 공고 추가하기

```
1. content/careers/new-position.mdx 파일 생성
2. frontmatter 작성:
   ---
   title: "포지션명"
   team: "팀명"
   location: "서울"
   type: "정규직"
   isOpen: true
   ---
3. 본문에 자격 요건, 우대 사항, 주요 업무 작성
4. PR 생성 → 머지 → 자동 배포
```

### 채용 공고 마감하기

```
해당 MDX 파일의 frontmatter에서 isOpen: true → isOpen: false로 변경
```

---

## 5-6. 확장 시나리오

프로젝트가 성장하면서 추가할 수 있는 기능:

| 기능 | 난이도 | 프롬프트 예시 |
|------|:------:|-------------|
| 블로그 | ⭐⭐ | "content/blog/ 폴더 기반 블로그 기능을 추가해줘. 목록 + 상세 + 태그 필터" |
| 다국어 (i18n) | ⭐⭐⭐ | "next-intl로 한국어/영어 다국어를 지원해줘. URL은 /ko, /en 프리픽스" |
| 다크모드 | ⭐⭐ | "next-themes로 다크모드를 추가해줘. 토글 버튼을 Header에 배치" |
| 뉴스레터 구독 | ⭐ | "푸터에 이메일 구독 폼을 추가해줘. Server Action + Resend API 연동" |
| 사이트맵 | ⭐ | "next-sitemap으로 sitemap.xml을 자동 생성해줘" |
| RSS 피드 | ⭐ | "블로그 포스트의 RSS 피드를 /feed.xml에 생성해줘" |
| 검색 기능 | ⭐⭐ | "Fuse.js로 솔루션+블로그 통합 검색 기능을 만들어줘" |

---

## 5-7. 참고 자료

### 공식 문서

| 기술 | 문서 URL |
|------|---------|
| Next.js App Router | https://nextjs.org/docs/app |
| Tailwind CSS | https://tailwindcss.com/docs |
| next-mdx-remote | https://github.com/hashicorp/next-mdx-remote |
| gray-matter | https://github.com/jonschlinkert/gray-matter |
| Vercel 배포 | https://vercel.com/docs |
| GitHub Actions | https://docs.github.com/en/actions |
| pnpm | https://pnpm.io/ko/ |

### Claude Code 관련

| 자료 | 설명 |
|------|------|
| CLAUDE.md 가이드 | Claude Code가 프로젝트 컨텍스트를 읽는 파일 작성법 |
| Claude Code CLI | 터미널에서 `claude --help`로 사용법 확인 |

---

## 5-8. 전체 워크플로우 요약

```
[Step 0] 프로젝트 조건 정의
  "무엇을, 왜, 어떤 제약 하에 만드는가?"
           ↓
[Step 1] 기술스택 선정
  "Next.js + Tailwind + MDX + Vercel + pnpm"
           ↓
[Step 2] 프로젝트 초기 세팅
  "pnpm create next-app → 디렉토리 구조 → CLAUDE.md"
           ↓
[Step 3] 핵심 페이지 개발
  "레이아웃 → 메인 → 솔루션 → 회사소개 → 채용 → 문의"
  "각 페이지를 구체적 프롬프트로 하나씩 생성"
           ↓
[Step 4] CI/CD 파이프라인
  "GitHub Actions(ci.yml + deploy.yml) → Vercel 자동 배포"
           ↓
[Step 5] 바이브코딩 팁 적용
  "CLAUDE.md 업데이트 → 성능 최적화 → 유지보수"
           ↓
[완성] 회사 홈페이지 운영 중 🎉
```
