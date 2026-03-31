# Step 17: 성능 예산 (Performance Budget)

> **이 단계의 목표:** Lighthouse CI로 PR마다 성능 점수를 자동 체크하고, 기준 미달 시 머지를 차단하여 성능 퇴행을 방지한다.

---

## 17-1. 성능 예산이란?

성능 예산(Performance Budget)은 웹사이트의 성능 지표에 대한 **최소 기준선**이다.
새로운 코드가 이 기준을 밑돌면 PR 머지를 차단하여, 배포할수록 느려지는 것을 막는다.

### 왜 바이브코딩에서 특히 중요한가

```
바이브코딩의 잠재적 위험:
  AI가 생성한 코드에 불필요한 의존성이 포함될 수 있음
  대형 이미지가 최적화 없이 추가될 수 있음
  클라이언트 컴포넌트('use client')가 과도하게 사용될 수 있음
  → 배포할수록 번들 크기 증가, 로딩 속도 저하

성능 예산의 역할:
  PR마다 자동으로 성능을 측정하여 퇴행을 즉시 감지
  AI 생성 코드의 성능 영향을 객관적으로 검증
```

---

## 17-2. 성능 기준 설정

### Core Web Vitals 기준

| 지표 | 약자 | 좋음 (Good) | 개선 필요 | 나쁨 | 우리 기준 |
|------|:----:|:----------:|:--------:|:----:|:---------:|
| Largest Contentful Paint | LCP | ≤ 2.5s | ≤ 4.0s | > 4.0s | **≤ 2.5s** |
| First Input Delay | FID | ≤ 100ms | ≤ 300ms | > 300ms | **≤ 100ms** |
| Cumulative Layout Shift | CLS | ≤ 0.1 | ≤ 0.25 | > 0.25 | **≤ 0.1** |
| Interaction to Next Paint | INP | ≤ 200ms | ≤ 500ms | > 500ms | **≤ 200ms** |

### Lighthouse 점수 기준

| 카테고리 | 최소 점수 | 위반 시 |
|----------|:--------:|:------:|
| Performance | **85** | PR 머지 차단 (error) |
| Accessibility | **90** | PR 머지 차단 (error) |
| SEO | **90** | PR 머지 차단 (error) |
| Best Practices | **80** | 경고만 (warn) |

### 번들 크기 기준

| 항목 | 최대 크기 | 측정 방법 |
|------|:---------:|----------|
| First Load JS | **150KB** (gzip) | `pnpm build` 출력의 "First Load JS" |
| 페이지별 JS | **50KB** (gzip) | 각 라우트의 "Size" 열 |
| 총 이미지 | **500KB**/페이지 | Lighthouse "Total Image Weight" |

---

## 17-3. Lighthouse CI 설정

### 설치

```bash
# Lighthouse CI CLI 설치
pnpm add -D @lhci/cli
```

### 설정 파일: `lighthouserc.json`

```json
{
  "ci": {
    "collect": {
      "startServerCommand": "pnpm start",
      "startServerReadyPattern": "Ready",
      "startServerReadyTimeout": 30000,
      "url": [
        "http://localhost:3000/",
        "http://localhost:3000/about",
        "http://localhost:3000/solutions",
        "http://localhost:3000/contact"
      ],
      "numberOfRuns": 3,
      "settings": {
        "preset": "desktop",
        "throttling": {
          "cpuSlowdownMultiplier": 1
        }
      }
    },
    "assert": {
      "assertions": {
        "categories:performance": ["error", { "minScore": 0.85 }],
        "categories:accessibility": ["error", { "minScore": 0.9 }],
        "categories:seo": ["error", { "minScore": 0.9 }],
        "categories:best-practices": ["warn", { "minScore": 0.8 }],
        "first-contentful-paint": ["warn", { "maxNumericValue": 2000 }],
        "largest-contentful-paint": ["error", { "maxNumericValue": 2500 }],
        "cumulative-layout-shift": ["error", { "maxNumericValue": 0.1 }],
        "total-byte-weight": ["warn", { "maxNumericValue": 500000 }]
      }
    },
    "upload": {
      "target": "temporary-public-storage"
    }
  }
}
```

### 설정 항목 설명

| 항목 | 값 | 설명 |
|------|:--:|------|
| `numberOfRuns` | 3 | 3회 측정 후 중앙값 사용 (편차 보정) |
| `preset` | desktop | 데스크톱 환경 기준 (모바일은 더 엄격) |
| `minScore` | 0.85 | 100점 만점 중 85점 이상 |
| `maxNumericValue` | 2500 | LCP 2.5초 이내 |
| `target` | temporary-public-storage | Lighthouse 리포트를 임시 URL로 업로드 |

### package.json 스크립트 추가

```json
{
  "scripts": {
    "lhci": "lhci autorun",
    "lhci:collect": "lhci collect",
    "lhci:assert": "lhci assert"
  }
}
```

---

## 17-4. CI에 Lighthouse CI 추가

### `.github/workflows/ci.yml`에 추가

기존 CI 워크플로우의 빌드 step 이후에 추가한다.

```yaml
      # ── (기존 빌드 검증 step 이후) ──

      # Lighthouse CI — 성능 예산 검사
      - name: 프로덕션 빌드 (Lighthouse용)
        run: pnpm build

      - name: Lighthouse CI 실행
        run: |
          pnpm exec lhci autorun
        env:
          LHCI_GITHUB_APP_TOKEN: ${{ secrets.LHCI_GITHUB_APP_TOKEN }}

      - name: Lighthouse 리포트 업로드
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: lighthouse-report
          path: .lighthouseci/
          retention-days: 7
```

### (선택) Lighthouse CI GitHub App

PR에 Lighthouse 점수를 코멘트로 남기려면:

```
① https://github.com/apps/lighthouse-ci 에서 GitHub App 설치
② 앱 설치 시 발급되는 토큰을 GitHub Secrets에 등록:
   LHCI_GITHUB_APP_TOKEN = lhci_xxxxxxxxxx
③ PR마다 자동으로 성능 리포트 코멘트가 작성됨
```

---

## 17-5. 번들 크기 모니터링

### @next/bundle-analyzer 설정

```bash
pnpm add -D @next/bundle-analyzer
```

#### `next.config.ts` 수정

```typescript
import bundleAnalyzer from "@next/bundle-analyzer";

const withBundleAnalyzer = bundleAnalyzer({
  enabled: process.env.ANALYZE === "true",
});

const nextConfig = {
  // 기존 설정...
};

export default withBundleAnalyzer(nextConfig);
```

#### 분석 실행

```bash
# 번들 분석 리포트 생성 (브라우저에서 자동 열림)
ANALYZE=true pnpm build

# 출력: .next/analyze/ 폴더에 HTML 리포트 생성
# → 어떤 패키지가 번들 크기를 차지하는지 시각화
```

### 빌드 출력에서 크기 확인

```bash
pnpm build

# 출력 예시:
# Route (app)                    Size     First Load JS
# ┌ ○ /                          5.2 kB   89.3 kB
# ├ ○ /about                     3.1 kB   87.2 kB
# ├ ● /solutions                 4.5 kB   88.6 kB
# ├ ● /solutions/[slug]          6.8 kB   90.9 kB
# ├ ○ /careers                   3.8 kB   87.9 kB
# └ ○ /contact                   8.2 kB   92.3 kB
# + First Load JS shared by all  84.1 kB
#
# ○ (Static)  ● (SSG)
```

### CI에서 번들 크기 체크

```yaml
      # 번들 크기 검사
      - name: 번들 크기 확인
        run: |
          pnpm build 2>&1 | tee build-output.txt
          # First Load JS shared가 150KB 초과하면 실패
          SHARED_SIZE=$(grep "First Load JS shared" build-output.txt | grep -oP '[\d.]+\s*kB' | head -1 | grep -oP '[\d.]+')
          echo "Shared JS size: ${SHARED_SIZE}kB"
          if (( $(echo "$SHARED_SIZE > 150" | bc -l) )); then
            echo "::error::First Load JS shared (${SHARED_SIZE}kB) exceeds budget (150kB)"
            exit 1
          fi
```

---

## 17-6. 성능 개선 프롬프트

성능 예산을 초과했을 때 바이브코딩으로 해결하는 프롬프트:

### LCP 개선

```
Lighthouse에서 LCP가 3.2초로 기준(2.5초)을 초과해.
개선해줘:
- 히어로 이미지에 priority 속성 추가
- 히어로 섹션의 불필요한 애니메이션 제거
- next/font로 폰트 최적화 (preload)
- 크리티컬 CSS가 인라인되는지 확인
```

### 번들 크기 감소

```
First Load JS가 160kB로 예산(150kB)을 초과해.
줄여줘:
1. ANALYZE=true pnpm build로 번들 분석
2. 큰 패키지를 dynamic import로 분할
3. 'use client' 컴포넌트를 Server Component로 전환 가능한지 확인
4. 미사용 의존성 제거 (pnpm prune)
```

### CLS 개선

```
CLS가 0.15로 기준(0.1)을 초과해. 수정해줘:
- 모든 Image 컴포넌트에 width/height 명시
- 웹폰트 로딩 시 font-display: swap + size-adjust
- 동적 콘텐츠 영역에 min-height 설정
```

---

## 17-7. 성능 예산 대시보드

### 로컬에서 빠르게 확인

```bash
# Lighthouse 로컬 실행
pnpm exec lhci collect --url="http://localhost:3000"
pnpm exec lhci assert

# 또는 한 번에
pnpm lhci
```

### PR에서 확인

```
PR → Checks 탭 → Lighthouse CI
  → 각 페이지별 Performance, Accessibility, SEO 점수
  → 기준 미달 항목 빨간색 표시
  → 리포트 URL 클릭하면 상세 개선 제안 확인
```

### 추이 모니터링

```
Lighthouse CI Server (자체 호스팅) 또는
Vercel Speed Insights에서 시간에 따른 성능 추이 확인

주간 체크:
  □ 평균 LCP가 2.5초 이내인가?
  □ CLS가 0.1 이내인가?
  □ 번들 크기가 꾸준히 증가하고 있지는 않은가?
```

---

## 17-8. 성능 예산 종합 체크리스트

| # | 항목 | 기준 | 검사 시점 |
|:-:|------|:----:|:---------:|
| 1 | Lighthouse Performance | ≥ 85 | PR마다 (CI) |
| 2 | Lighthouse Accessibility | ≥ 90 | PR마다 (CI) |
| 3 | Lighthouse SEO | ≥ 90 | PR마다 (CI) |
| 4 | LCP | ≤ 2.5s | PR마다 (CI) |
| 5 | CLS | ≤ 0.1 | PR마다 (CI) |
| 6 | First Load JS (shared) | ≤ 150KB | PR마다 (CI) |
| 7 | 페이지별 JS | ≤ 50KB | 빌드 출력 수동 확인 |
| 8 | 총 이미지 무게 | ≤ 500KB/페이지 | Lighthouse 리포트 |
| 9 | Core Web Vitals (실사용자) | 모두 Good | Vercel Speed Insights (주간) |
| 10 | 번들 크기 추이 | 증가 추세 없음 | 월간 번들 분석 |
