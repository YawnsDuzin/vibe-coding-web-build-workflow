# Step 4: CI/CD 파이프라인

> **이 단계의 목표:** GitHub Actions로 PR 검증(lint + build) + 프로덕션 자동 배포 파이프라인을 구축하고, Vercel 연동을 완료한다.

---

## 4-1. CI/CD 아키텍처 개요

```
개발자가 코드 수정
     ↓
Feature 브랜치에 push
     ↓
Pull Request 생성
     ↓
┌──────────────────────────────┐
│  CI 워크플로우 (ci.yml)        │
│  ① 코드 체크아웃               │
│  ② pnpm 설치 + 캐싱           │
│  ③ 의존성 설치                 │
│  ④ ESLint 검사                │
│  ⑤ TypeScript 타입 검사       │
│  ⑥ 프로덕션 빌드 테스트         │
│  → 하나라도 실패하면 PR 머지 차단 │
└──────────────────────────────┘
     ↓ (모두 통과 + 코드 리뷰 승인)
main 브랜치에 머지
     ↓
┌──────────────────────────────┐
│  Deploy 워크플로우 (deploy.yml) │
│  ① 코드 체크아웃               │
│  ② 빌드                      │
│  ③ Vercel CLI로 프로덕션 배포   │
│  → 프로덕션 URL 자동 업데이트   │
└──────────────────────────────┘
```

---

## 4-2. CI 워크플로우 — PR 검증

### 파일: `.github/workflows/ci.yml`

```yaml
name: CI — PR 검증

# ──────────────────────────────────────────────
# 트리거: PR이 열리거나 업데이트될 때 실행
# ──────────────────────────────────────────────
on:
  pull_request:
    branches: [main]
    # 문서 파일만 변경된 PR은 CI 스킵 (빌드 시간 절약)
    paths-ignore:
      - '**.md'
      - 'docs/**'
      - 'LICENSE'

# ──────────────────────────────────────────────
# 동시 실행 방지
# 같은 PR에 새 커밋이 push되면 이전 실행을 취소하여
# 불필요한 빌드 시간 낭비를 막는다
# ──────────────────────────────────────────────
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  validate:
    name: Lint & Build 검증
    runs-on: ubuntu-latest
    # 최대 실행 시간 10분 (무한 루프 방지)
    timeout-minutes: 10

    steps:
      # ── Step 1: 코드 체크아웃 ──
      # PR의 소스 브랜치 코드를 체크아웃한다
      - name: 코드 체크아웃
        uses: actions/checkout@v4

      # ── Step 2: pnpm 설치 ──
      # pnpm을 GitHub Actions 러너에 설치한다
      # version은 package.json의 packageManager 필드와 일치시킨다
      - name: pnpm 설치
        uses: pnpm/action-setup@v4
        with:
          version: 9

      # ── Step 3: Node.js 설정 ──
      # Node.js 20을 설치하고 pnpm 캐시를 활성화한다
      # 캐시를 사용하면 두 번째 실행부터 의존성 설치가 훨씬 빨라진다
      - name: Node.js 20 설정
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      # ── Step 4: 의존성 설치 ──
      # --frozen-lockfile: pnpm-lock.yaml과 package.json이
      # 일치하지 않으면 에러를 발생시킨다 (CI 환경에서 필수)
      - name: 의존성 설치
        run: pnpm install --frozen-lockfile

      # ── Step 5: ESLint 검사 ──
      # 코드 스타일과 잠재적 버그를 정적으로 검사한다
      - name: 린트 검사
        run: pnpm lint

      # ── Step 6: TypeScript 타입 검사 ──
      # --noEmit: JS 파일을 생성하지 않고 타입만 검사한다
      # 빌드와 별도로 실행하여 타입 에러를 빠르게 발견한다
      - name: 타입 검사
        run: pnpm tsc --noEmit

      # ── Step 7: 프로덕션 빌드 테스트 ──
      # 실제 프로덕션 빌드가 성공하는지 확인한다
      # MDX 파싱 에러, 이미지 경로 오류 등이 여기서 발견된다
      - name: 빌드 검증
        run: pnpm build
```

### CI 워크플로우 동작 설명

| Step | 실패 시 의미 | 해결 방법 |
|------|------------|----------|
| 린트 검사 | 코드 스타일 위반, 미사용 변수 등 | `pnpm lint --fix`로 자동 수정 후 재커밋 |
| 타입 검사 | TypeScript 타입 불일치 | 에러 메시지의 파일:줄번호를 확인하고 타입 수정 |
| 빌드 검증 | import 오류, MDX 파싱 실패 등 | 로컬에서 `pnpm build` 실행하여 에러 재현 후 수정 |

---

## 4-3. Deploy 워크플로우 — 프로덕션 배포

### 파일: `.github/workflows/deploy.yml`

```yaml
name: Deploy — 프로덕션 배포

# ──────────────────────────────────────────────
# 트리거: main 브랜치에 push(머지)될 때만 실행
# ──────────────────────────────────────────────
on:
  push:
    branches: [main]
    # CI와 마찬가지로 문서 변경은 배포 스킵
    paths-ignore:
      - '**.md'
      - 'docs/**'
      - 'LICENSE'

# ──────────────────────────────────────────────
# 동시 배포 방지
# cancel-in-progress: false로 설정하여
# 진행 중인 배포가 완료될 때까지 새 배포를 대기시킨다
# (배포 중 취소하면 중간 상태가 될 수 있으므로)
# ──────────────────────────────────────────────
concurrency:
  group: deploy-production
  cancel-in-progress: false

jobs:
  deploy:
    name: Vercel 프로덕션 배포
    runs-on: ubuntu-latest
    timeout-minutes: 15

    steps:
      # ── Step 1: 코드 체크아웃 ──
      - name: 코드 체크아웃
        uses: actions/checkout@v4

      # ── Step 2: pnpm 설치 ──
      - name: pnpm 설치
        uses: pnpm/action-setup@v4
        with:
          version: 9

      # ── Step 3: Node.js 설정 ──
      - name: Node.js 20 설정
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      # ── Step 4: 의존성 설치 ──
      - name: 의존성 설치
        run: pnpm install --frozen-lockfile

      # ── Step 5: 프로덕션 빌드 ──
      # Vercel CLI로 배포하기 전에 로컬에서 빌드를 확인한다
      - name: 프로덕션 빌드
        run: pnpm build

      # ── Step 6: Vercel CLI 설치 ──
      # Vercel CLI를 글로벌로 설치한다
      - name: Vercel CLI 설치
        run: pnpm add -g vercel

      # ── Step 7: 프로덕션 배포 실행 ──
      # --prod 플래그로 프로덕션 도메인에 배포한다
      # 환경변수는 GitHub Secrets에서 주입된다
      - name: 프로덕션 배포
        run: vercel deploy --prod --token=${{ secrets.VERCEL_TOKEN }}
        env:
          VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
          VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}

      # ── Step 8: 배포 완료 알림 (선택) ──
      # Slack 웹훅으로 배포 완료를 알린다
      # 사용하려면 SLACK_WEBHOOK_URL 시크릿을 추가한다
      # - name: Slack 배포 알림
      #   if: success()
      #   run: |
      #     curl -X POST -H 'Content-type: application/json' \
      #       --data '{"text":"✅ 프로덕션 배포 완료: ${{ github.event.head_commit.message }}"}' \
      #       ${{ secrets.SLACK_WEBHOOK_URL }}
```

---

## 4-4. GitHub Secrets 설정 가이드

GitHub 저장소 → **Settings** → **Secrets and variables** → **Actions**에서 아래 3개의 시크릿을 등록한다.

### 필수 시크릿

| Secret 이름 | 설명 | 값 형식 예시 |
|-------------|------|-------------|
| `VERCEL_TOKEN` | Vercel API 인증 토큰 | `vrc_xxxxxxxxxxxxxxxxxxxx` |
| `VERCEL_ORG_ID` | Vercel 조직(팀) ID | `team_xxxxxxxxxxxxxxxxxxxx` |
| `VERCEL_PROJECT_ID` | Vercel 프로젝트 ID | `prj_xxxxxxxxxxxxxxxxxxxx` |

### 선택 시크릿

| Secret 이름 | 설명 | 용도 |
|-------------|------|------|
| `SLACK_WEBHOOK_URL` | Slack 웹훅 URL | 배포 완료 알림 |

### 시크릿 값 얻는 방법 (단계별)

```bash
# ────────────────────────────────────────────
# 1단계: Vercel CLI 설치 및 로그인
# ────────────────────────────────────────────
pnpm add -g vercel
vercel login
# → 브라우저가 열리고 Vercel 계정으로 로그인
# → 터미널에 "Congratulations!" 메시지 확인

# ────────────────────────────────────────────
# 2단계: 프로젝트 연결
# ────────────────────────────────────────────
cd company-homepage
vercel link
# → "Set up and deploy?" → y
# → 조직 선택 (개인 계정 또는 팀)
# → 기존 프로젝트 연결 또는 새 프로젝트 생성
# → .vercel/ 폴더가 생성됨

# ────────────────────────────────────────────
# 3단계: orgId, projectId 확인
# ────────────────────────────────────────────
cat .vercel/project.json
# 출력 예시:
# {
#   "orgId": "team_xxxxxxxxxxxxxxxxxxxx",    ← VERCEL_ORG_ID
#   "projectId": "prj_xxxxxxxxxxxxxxxxxxxx"  ← VERCEL_PROJECT_ID
# }

# ────────────────────────────────────────────
# 4단계: Vercel API 토큰 생성
# ────────────────────────────────────────────
# Vercel 대시보드 접속:
#   → Settings → Tokens → Create Token
#   → 이름: "github-actions-deploy"
#   → Scope: Full Account (또는 특정 프로젝트)
#   → 생성된 토큰 복사 → VERCEL_TOKEN으로 사용
```

### GitHub에 시크릿 등록하는 방법

```
1. GitHub 저장소 페이지 접속
2. Settings 탭 클릭
3. 왼쪽 메뉴 → Secrets and variables → Actions
4. "New repository secret" 버튼 클릭
5. Name에 시크릿 이름 (예: VERCEL_TOKEN), Secret에 값 입력
6. "Add secret" 클릭
7. 나머지 2개도 동일하게 반복
```

---

## 4-5. 브랜치 보호 규칙 설정

CI가 의미를 가지려면, main 브랜치에 직접 push를 막고 PR + CI 통과를 필수로 만들어야 한다.

### 설정 경로

```
GitHub 저장소 → Settings → Branches → Add branch protection rule
```

### 권장 설정

| 설정 항목 | 값 | 설명 |
|----------|:-:|------|
| Branch name pattern | `main` | main 브랜치에 적용 |
| Require a pull request before merging | ✅ | 직접 push 차단, PR 필수 |
| Require approvals | 1 | 최소 1명 리뷰 승인 (1인 개발이면 0으로) |
| Require status checks to pass | ✅ | CI 통과 필수 |
| Status checks: "Lint & Build 검증" | ✅ | ci.yml의 job 이름 선택 |
| Require branches to be up to date | ✅ | main과 동기화 후 머지 |
| Include administrators | ✅ | 관리자도 규칙 적용 |

---

## 4-6. Vercel 프리뷰 배포 (자동)

Vercel에 프로젝트를 연결하면, **PR마다 프리뷰 URL이 자동 생성**된다. 별도 설정이 필요 없다.

### 동작 방식

```
PR 생성/업데이트
  ↓
Vercel이 자동으로 프리뷰 빌드
  ↓
PR 코멘트에 프리뷰 URL 표시
  예: https://company-homepage-abc123-your-team.vercel.app
  ↓
비개발자도 프리뷰 URL로 변경사항 미리 확인 가능
```

### Vercel 프리뷰와 GitHub Actions CI의 관계

| 항목 | Vercel 프리뷰 | GitHub Actions CI |
|------|:------------:|:-----------------:|
| 트리거 | PR 생성 시 자동 | PR 생성 시 자동 |
| 역할 | 실제 사이트 미리보기 제공 | 코드 품질 검증 (lint + type + build) |
| 실패 시 | 프리뷰 URL 접속 불가 | PR 머지 차단 |
| 비용 | Vercel 무료 티어에 포함 | GitHub Free에 포함 (2,000분/월) |

> 두 가지를 함께 사용하면: CI가 코드 품질을 검증하고, Vercel 프리뷰가 시각적 확인을 제공한다.

---

## 4-7. 배포 후 정상 작동 확인

### CLI로 확인

```bash
# Vercel 배포 목록 확인
vercel ls

# 프로덕션 URL 응답 확인
curl -I https://www.company.com
# HTTP/2 200 ← 이 응답이 오면 정상
```

### 체크리스트

| # | 확인 항목 | 방법 | 기대 결과 |
|:-:|----------|------|-----------|
| 1 | 전체 페이지 로드 | 브라우저에서 /, /about, /solutions, /careers, /contact 접속 | 모든 페이지 200 OK |
| 2 | 모바일 반응형 | Chrome DevTools → Toggle device toolbar → iPhone 14 Pro | 레이아웃 깨짐 없음, 햄버거 메뉴 동작 |
| 3 | 솔루션 상세 | /solutions/product-a 접속 | MDX 콘텐츠 정상 렌더링 |
| 4 | 문의 폼 | 테스트 데이터로 폼 제출 | 유효성 검사 동작, 성공 메시지 표시 |
| 5 | 메타 태그 | 페이지 소스 보기 (Ctrl+U) | title, description, og:image 존재 |
| 6 | OG 미리보기 | 소셜 공유 시뮬레이터에서 URL 입력 | 제목/설명/이미지 정상 표시 |
| 7 | Lighthouse | Chrome DevTools → Lighthouse → Generate report | Performance 90+, SEO 90+ |
| 8 | 빌드 시간 | GitHub Actions 로그 확인 | CI 전체 3분 이내, 배포 5분 이내 |
| 9 | 404 페이지 | 존재하지 않는 URL 접속 (예: /asdf) | 커스텀 404 페이지 표시 |
| 10 | HTTPS | 브라우저 주소 표시줄 확인 | 🔒 자물쇠 아이콘 표시 |

### 문제 발생 시 디버깅

| 증상 | 원인 | 해결 |
|------|------|------|
| 배포 실패: "VERCEL_TOKEN is not set" | GitHub Secrets 미등록 | Settings → Secrets에서 3개 시크릿 확인 |
| 빌드 실패: "Module not found" | 의존성 누락 | `pnpm-lock.yaml`이 커밋되었는지 확인 |
| 프리뷰 URL 404 | Vercel 프로젝트 미연결 | `vercel link` 재실행 |
| CI 통과했는데 배포 실패 | 환경변수 차이 | Vercel 대시보드 → Settings → Environment Variables 확인 |
| 이미지 깨짐 | public/ 경로 오류 | 이미지 경로가 `/images/...`로 시작하는지 확인 |

---

## 4-8. 비개발자 콘텐츠 수정 워크플로우

CI/CD가 구축되면, 비개발자도 아래 흐름으로 안전하게 콘텐츠를 수정할 수 있다.

```
① GitHub 웹에서 content/ 폴더의 MDX 파일 열기
  ↓
② 연필 아이콘(✏️) 클릭 → 텍스트 수정
  ↓
③ "Create a new branch and start a pull request" 선택
  ↓
④ PR 생성 → CI 자동 실행 + Vercel 프리뷰 URL 생성
  ↓
⑤ 프리뷰 URL에서 변경사항 확인
  ↓
⑥ 개발자가 PR 승인 → main에 머지
  ↓
⑦ 프로덕션 자동 배포 완료
```

> **핵심:** 비개발자는 GitHub 웹 UI에서 마크다운만 편집하면 되고, 나머지(검증·프리뷰·배포)는 모두 자동화된다.

> **다음 단계:** [Step 5: 바이브코딩 팁 & 부록](./05-vibe-coding-tips.md)에서 바이브코딩 효율을 높이는 실전 팁과 참고 자료를 정리한다.
