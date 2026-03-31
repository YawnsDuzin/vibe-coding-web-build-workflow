# 바이브코딩으로 회사 홈페이지 구축하기 — 실전 워크플로우 가이드

> 바이브코딩(AI 보조 개발) 방식으로 회사 홈페이지를 신규 구축하고 CI/CD까지 완성하는 **전체 과정을 단계별로 정리**한 가이드이다.
> 이 문서만 보고 처음부터 재현할 수 있어야 한다.

---

## 프로젝트 조건

| 항목 | 내용 |
|------|------|
| 개발 방식 | 1명 + Claude Code (바이브코딩) |
| 기술스택 | Next.js 14 + Tailwind CSS v4 + MDX + Vercel + pnpm |
| 콘텐츠 관리 | MDX 파일 기반 (비개발자 GitHub 웹 편집 가능) |
| 호스팅 예산 | 월 $0 (Vercel Hobby 무료 티어) |
| 필요 페이지 | 메인, 회사소개, 솔루션(4~5개), 채용, 문의 |

---

## 가이드 목차

### 핵심 가이드 (Step 0~5)

| Step | 문서 | 내용 요약 |
|:----:|------|----------|
| 0 | [프로젝트 개요 및 조건](docs/00-overview.md) | 프로젝트 배경, 목표, 제약 조건, 의사결정 우선순위 정의 |
| 1 | [기술스택 선정](docs/01-tech-stack.md) | 프레임워크·스타일링·CMS·호스팅·패키지매니저 후보 비교 및 최종 선정 |
| 2 | [프로젝트 초기 세팅](docs/02-project-setup.md) | 프로젝트 생성 명령어, 디렉토리 구조, CLAUDE.md, 환경변수 설정 |
| 3 | [핵심 페이지 개발 워크플로우](docs/03-page-development.md) | 페이지별 프롬프트 예시 6개, 레퍼런스 활용법, 개발 순서 전략 |
| 4 | [CI/CD 파이프라인](docs/04-cicd-pipeline.md) | GitHub Actions YAML, Vercel 배포, Secrets 설정, 배포 확인 체크리스트 |
| 5 | [바이브코딩 팁 & 부록](docs/05-vibe-coding-tips.md) | 프롬프트 작성 원칙, 워크플로우 패턴, 트러블슈팅, 유지보수 가이드 |

### 운영 가이드 (Step 6~9)

| Step | 문서 | 내용 요약 |
|:----:|------|----------|
| 6 | [비개발자 콘텐츠 편집 가이드](docs/06-content-editing-guide.md) | MDX 편집법, GitHub 웹 UI 사용법, 이미지 업로드, FAQ |
| 7 | [문의 폼 이메일 연동](docs/07-email-integration.md) | Resend API 연동, Server Action 구현, React Email 템플릿 |
| 8 | [SEO 심화](docs/08-seo-advanced.md) | sitemap.xml, robots.txt, JSON-LD 구조화 데이터, canonical URL |
| 9 | [도메인 & DNS 설정](docs/09-domain-dns-setup.md) | 도메인 구매, Vercel 연결, DNS 레코드, SSL 인증서, 리다이렉트 |

### 품질 관리 가이드 (Step 10~13)

| Step | 문서 | 내용 요약 |
|:----:|------|----------|
| 10 | [테스트 전략](docs/10-testing-strategy.md) | Playwright E2E 테스트, 페이지 라우팅·폼·반응형 테스트, CI 통합 |
| 11 | [에러 모니터링](docs/11-error-monitoring.md) | Sentry 에러 추적, Vercel Analytics 성능 모니터링, Slack 알림 |
| 12 | [접근성 테스트](docs/12-accessibility-testing.md) | axe-core 자동 검사, WCAG 2.1 AA, 키보드·스크린 리더 테스트 |
| 13 | [Vercel 무료 티어 관리](docs/13-vercel-free-tier.md) | 한도 상세, 사용량 모니터링, 절약 팁, Pro 전환 시점, 대안 호스팅 |

### 고도화 가이드 (Step 14~17)

| Step | 문서 | 내용 요약 |
|:----:|------|----------|
| 14 | [Google Analytics 연동](docs/14-google-analytics.md) | GA4 설정, 페이지뷰 추적, 커스텀 이벤트, 전환 퍼널, 쿠키 동의 |
| 15 | [롤백 절차](docs/15-rollback-strategy.md) | Vercel Instant Rollback, git revert, 판단 플로차트, 사후 조치 |
| 16 | [Git 브랜칭 전략](docs/16-git-branching.md) | GitHub Flow, 브랜치/커밋 네이밍 규칙, PR 템플릿, 머지 전략 |
| 17 | [성능 예산](docs/17-performance-budget.md) | Lighthouse CI, Core Web Vitals 기준, 번들 크기 모니터링 |

---

## 빠른 시작

```bash
# 1. 프로젝트 생성
pnpm create next-app@latest company-homepage \
  --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"

# 2. 의존성 설치
cd company-homepage
pnpm add next-mdx-remote gray-matter
pnpm add -D @tailwindcss/typography

# 3. 개발 서버 실행
pnpm dev
```

자세한 과정은 [Step 2: 프로젝트 초기 세팅](docs/02-project-setup.md)을 참고한다.

---

## 전체 워크플로우

```
[Step 0] 프로젝트 조건 정의
         ↓
[Step 1] 기술스택 선정 → Next.js + Tailwind + MDX + Vercel + pnpm
         ↓
[Step 2] 프로젝트 초기 세팅 → pnpm create next-app → CLAUDE.md 작성
         ↓
[Step 3] 핵심 페이지 개발 → 레이아웃 → 메인 → 솔루션 → 회사소개 → 채용 → 문의
         ↓
[Step 4] CI/CD 파이프라인 → GitHub Actions + Vercel 자동 배포
         ↓
[Step 5] 바이브코딩 팁 적용 → 성능 최적화 → 유지보수
         ↓
[Step 6] 비개발자 콘텐츠 편집 가이드 → 마케팅/HR 팀 온보딩
         ↓
[Step 7] 문의 폼 이메일 연동 → Resend API 연결
         ↓
[Step 8] SEO 심화 → sitemap, 구조화 데이터, Search Console
         ↓
[Step 9] 도메인 & DNS 설정 → 커스텀 도메인 운영
         ↓
[Step 10] 테스트 전략 → Playwright E2E 테스트 도입
         ↓
[Step 11] 에러 모니터링 → Sentry + Vercel Analytics
         ↓
[Step 12] 접근성 테스트 → WCAG 2.1 AA 준수
         ↓
[Step 13] Vercel 무료 티어 관리 → 사용량 모니터링
         ↓
[Step 14] Google Analytics → 방문자 분석 + 이벤트 추적
         ↓
[Step 15] 롤백 절차 → 장애 대응 프로세스 구축
         ↓
[Step 16] Git 브랜칭 전략 → 안전한 코드 관리
         ↓
[Step 17] 성능 예산 → Lighthouse CI 자동 검사
         ↓
       회사 홈페이지 운영
```

---

## 문서 구조

```
vibe-coding-web-build-workflow/
├── README.md                      ← 현재 파일 (목차 & 개요)
└── docs/
    ├── 00-overview.md             ← 프로젝트 개요 및 조건
    ├── 01-tech-stack.md           ← 기술스택 선정
    ├── 02-project-setup.md        ← 프로젝트 초기 세팅
    ├── 03-page-development.md     ← 핵심 페이지 개발 워크플로우
    ├── 04-cicd-pipeline.md        ← CI/CD 파이프라인
    ├── 05-vibe-coding-tips.md     ← 바이브코딩 팁 & 부록
    ├── 06-content-editing-guide.md ← 비개발자 콘텐츠 편집 가이드
    ├── 07-email-integration.md    ← 문의 폼 이메일 연동
    ├── 08-seo-advanced.md         ← SEO 심화
    ├── 09-domain-dns-setup.md     ← 도메인 & DNS 설정
    ├── 10-testing-strategy.md     ← 테스트 전략
    ├── 11-error-monitoring.md     ← 에러 모니터링
    ├── 12-accessibility-testing.md ← 접근성 테스트
    ├── 13-vercel-free-tier.md     ← Vercel 무료 티어 관리
    ├── 14-google-analytics.md     ← Google Analytics 연동
    ├── 15-rollback-strategy.md    ← 롤백 절차
    ├── 16-git-branching.md        ← Git 브랜칭 전략
    └── 17-performance-budget.md   ← 성능 예산
```
