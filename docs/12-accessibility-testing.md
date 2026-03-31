# Step 12: 접근성 테스트

> **이 단계의 목표:** 웹 접근성(a11y) 기준을 충족하도록 자동 검사 도구와 수동 체크리스트를 적용하고, CI에 접근성 검사를 추가한다.

---

## 12-1. 왜 접근성이 중요한가

| 관점 | 이유 |
|------|------|
| **법적 의무** | 한국: 장애인차별금지법, 웹 접근성 인증 (공공기관 필수, 민간 권장) |
| **사용자 확대** | 시각/청각/운동 장애, 고령자, 일시적 장애(한 손 사용) 등 다양한 사용자 포용 |
| **SEO 연관** | 시맨틱 HTML, alt 텍스트 등 접근성 요소가 검색 엔진 최적화에도 기여 |
| **기업 이미지** | 접근성을 고려한 사이트는 기업의 사회적 책임을 보여줌 |

### 목표 기준

```
WCAG 2.1 Level AA 준수
 → Lighthouse Accessibility 점수 90+ 달성
```

---

## 12-2. 자동 검사 도구

### 도구 비교

| 도구 | 용도 | 비용 | 통합 방식 |
|------|------|:----:|----------|
| **axe-core** | 자동 접근성 검사 엔진 | 무료 | Playwright 테스트에 통합 |
| **Lighthouse** | 종합 품질 검사 (접근성 포함) | 무료 | Chrome DevTools / CI |
| **axe DevTools** | 브라우저 확장 프로그램 | 무료 | Chrome/Firefox 확장 |
| **WAVE** | 웹 기반 접근성 검사 | 무료 | 웹 사이트에서 URL 입력 |

---

## 12-3. axe-core + Playwright 접근성 테스트

### 설치

```bash
# axe-core Playwright 통합 패키지
pnpm add -D @axe-core/playwright
```

### 테스트 파일: `e2e/accessibility.spec.ts`

```typescript
import { test, expect } from "@playwright/test";
import AxeBuilder from "@axe-core/playwright";

// 모든 주요 페이지에 대해 접근성 검사 실행
const pages = [
  { name: "메인", path: "/" },
  { name: "회사소개", path: "/about" },
  { name: "솔루션 목록", path: "/solutions" },
  { name: "채용", path: "/careers" },
  { name: "문의", path: "/contact" },
];

for (const { name, path } of pages) {
  test(`${name} 페이지 접근성 검사 (WCAG 2.1 AA)`, async ({ page }) => {
    await page.goto(path);

    const results = await new AxeBuilder({ page })
      // WCAG 2.1 Level AA 기준으로 검사
      .withTags(["wcag2a", "wcag2aa", "wcag21aa"])
      // 알려진 예외 규칙 (필요 시 추가)
      // .disableRules(["color-contrast"])
      .analyze();

    // 위반 사항이 있으면 상세 정보 출력
    const violations = results.violations.map((v) => ({
      rule: v.id,
      impact: v.impact,
      description: v.description,
      nodes: v.nodes.length,
      help: v.helpUrl,
    }));

    if (violations.length > 0) {
      console.log(`\n[${name}] 접근성 위반 사항:`);
      console.table(violations);
    }

    // 심각(critical) 및 중요(serious) 위반이 0이어야 통과
    const criticalViolations = results.violations.filter(
      (v) => v.impact === "critical" || v.impact === "serious"
    );
    expect(
      criticalViolations,
      `${name} 페이지에 심각한 접근성 위반 ${criticalViolations.length}건`
    ).toHaveLength(0);
  });
}
```

### 실행

```bash
# 접근성 테스트만 실행
pnpm exec playwright test e2e/accessibility.spec.ts

# 전체 테스트와 함께 실행
pnpm test
```

---

## 12-4. CI에 접근성 검사 추가

### `.github/workflows/ci.yml`에 추가

axe-core 테스트가 E2E 테스트에 포함되어 있으므로, [Step 10](./10-testing-strategy.md)에서 추가한 E2E 테스트 step이 접근성도 함께 검사한다.

추가로 Lighthouse CI를 별도로 실행할 수도 있다:

```yaml
      # ── Lighthouse CI (접근성 + 성능 종합 검사) ──
      - name: Lighthouse CI
        uses: treosh/lighthouse-ci-action@v12
        with:
          urls: |
            http://localhost:3000/
            http://localhost:3000/about
            http://localhost:3000/solutions
            http://localhost:3000/contact
          configPath: ./lighthouserc.json
          uploadArtifacts: true
```

### `lighthouserc.json`

```json
{
  "ci": {
    "assert": {
      "assertions": {
        "categories:accessibility": ["error", { "minScore": 0.9 }],
        "categories:seo": ["error", { "minScore": 0.9 }],
        "categories:performance": ["warn", { "minScore": 0.8 }]
      }
    },
    "collect": {
      "startServerCommand": "pnpm start",
      "startServerReadyPattern": "ready on",
      "numberOfRuns": 1
    }
  }
}
```

> 접근성(0.9)과 SEO(0.9)는 error로, 성능(0.8)은 warn으로 설정하여 접근성 미달 시 PR 머지를 차단한다.

---

## 12-5. 시맨틱 HTML 체크리스트

바이브코딩으로 페이지를 생성할 때, 아래 시맨틱 구조를 준수한다.

### 페이지 전체 구조

```html
<body>
  <header>             <!-- 헤더: 로고 + 네비게이션 -->
    <nav>              <!-- 메인 네비게이션 -->
      ...
    </nav>
  </header>

  <main>               <!-- 본문 콘텐츠 (페이지당 1개) -->
    <section>          <!-- 논리적 섹션 단위 -->
      <h1>페이지 제목</h1>  <!-- h1은 페이지당 1개 -->
      ...
    </section>
    <section>
      <h2>섹션 제목</h2>
      ...
    </section>
  </main>

  <footer>             <!-- 푸터: 링크 + 저작권 -->
    ...
  </footer>
</body>
```

### 필수 접근성 속성

| 요소 | 필수 속성 | 예시 |
|------|----------|------|
| `<img>` / `<Image>` | `alt` | `alt="클라우드 매니저 대시보드 화면"` |
| `<a>` (외부 링크) | `target`, `rel` | `target="_blank" rel="noopener noreferrer"` |
| `<nav>` | `aria-label` | `aria-label="메인 메뉴"` |
| `<button>` (아이콘만) | `aria-label` | `aria-label="메뉴 열기"` |
| `<input>` | `id` + `<label>` | `<label htmlFor="email">이메일</label>` |
| `<select>` | `id` + `<label>` | `<label htmlFor="type">문의 유형</label>` |
| 모바일 메뉴 | `aria-expanded` | `aria-expanded={isOpen}` |
| 모달/다이얼로그 | `role`, `aria-modal` | `role="dialog" aria-modal="true"` |

---

## 12-6. 색상 대비 검사

WCAG AA 기준: 일반 텍스트 4.5:1, 큰 텍스트(18px+ bold 또는 24px+) 3:1

### Tailwind CSS 기본 색상의 대비율

| 조합 | 대비율 | AA 통과 |
|------|:------:|:-------:|
| `text-gray-900` on `bg-white` | 15.3:1 | ✅ |
| `text-gray-700` on `bg-white` | 9.1:1 | ✅ |
| `text-gray-500` on `bg-white` | 4.6:1 | ✅ (경계선) |
| `text-gray-400` on `bg-white` | 3.0:1 | ❌ |
| `text-white` on `bg-primary-600` (#2563EB) | 4.6:1 | ✅ |
| `text-white` on `bg-primary-500` (#3B82F6) | 3.4:1 | ❌ |

### 안전한 색상 조합 규칙

```
본문 텍스트: text-gray-700 이상 (gray-700, gray-800, gray-900)
보조 텍스트: text-gray-500 이상
버튼 텍스트: text-white on bg-primary-600 이상 (primary-600, 700, 800)
링크 텍스트: text-primary-700 이상
```

### 대비율 검사 도구

```
온라인: https://webaim.org/resources/contrastchecker/
Chrome 확장: axe DevTools → Color Contrast 자동 검사
```

---

## 12-7. 키보드 내비게이션 테스트

### 수동 테스트 절차

```
① Tab 키로 페이지 전체를 순회
   → 모든 인터랙티브 요소(링크, 버튼, 입력 필드)에 도달 가능한지 확인
   → 포커스 순서가 시각적 순서와 일치하는지 확인

② 포커스 표시(focus indicator) 확인
   → 현재 포커스된 요소가 시각적으로 구별되는지 확인
   → Tailwind: focus-visible:ring-2 focus-visible:ring-primary-500

③ Enter/Space 키로 동작 확인
   → 버튼: Enter 또는 Space로 클릭
   → 링크: Enter로 이동
   → 드롭다운: Space로 열기, 화살표로 선택

④ Escape 키로 닫기
   → 모바일 메뉴: Escape로 닫기
   → 모달/다이얼로그: Escape로 닫기

⑤ 포커스 트랩 확인 (모달)
   → 모달이 열리면 Tab이 모달 내부에서만 순환
   → 모달 닫히면 원래 요소로 포커스 복귀
```

### 포커스 스타일 프롬프트

```
모든 인터랙티브 요소에 키보드 포커스 스타일을 추가해줘:
- 링크, 버튼: focus-visible:outline-none focus-visible:ring-2
  focus-visible:ring-primary-500 focus-visible:ring-offset-2
- 입력 필드: focus:ring-2 focus:ring-primary-500 focus:border-primary-500
- focus-visible을 사용하여 마우스 클릭 시에는 링이 보이지 않게
```

---

## 12-8. 스크린 리더 테스트

### 무료 스크린 리더

| 스크린 리더 | OS | 사용법 |
|------------|:--:|--------|
| **NVDA** | Windows | 무료 다운로드. 가장 많이 사용되는 무료 스크린 리더 |
| **VoiceOver** | macOS/iOS | 내장. `Cmd+F5`로 활성화 |
| **TalkBack** | Android | 내장. 설정 → 접근성에서 활성화 |

### 스크린 리더 테스트 체크리스트

| # | 확인 항목 | 방법 |
|:-:|----------|------|
| 1 | 페이지 제목 읽기 | 페이지 진입 시 `<title>` 내용을 읽는가 |
| 2 | 헤딩 구조 탐색 | H1→H2→H3 순서로 논리적으로 탐색되는가 |
| 3 | 이미지 대체 텍스트 | 이미지에 도달하면 alt 텍스트를 읽는가 |
| 4 | 폼 라벨 | 입력 필드에 포커스하면 라벨을 읽는가 |
| 5 | 버튼 용도 | 아이콘 버튼의 aria-label을 읽는가 |
| 6 | 링크 목적 | 링크 텍스트만으로 목적지를 알 수 있는가 ("여기 클릭" ❌) |
| 7 | 에러 메시지 | 폼 유효성 에러 발생 시 에러 메시지를 읽는가 |

---

## 12-9. 바이브코딩으로 접근성 개선하기

### 일괄 개선 프롬프트

```
전체 프로젝트의 접근성을 WCAG 2.1 AA 기준으로 개선해줘:

1. 시맨틱 HTML
   - <div> 남용을 <section>, <article>, <aside>로 교체
   - <header>, <nav>, <main>, <footer> 구조 확인
   - 각 페이지에 <h1> 1개만 존재하는지 확인

2. ARIA 속성
   - Header.tsx: <nav aria-label="메인 메뉴">
   - 모바일 햄버거: aria-expanded, aria-controls
   - 아이콘 버튼: aria-label 추가

3. 폼 접근성
   - ContactForm.tsx: 모든 input에 <label> 연결 (htmlFor + id)
   - 필수 필드: aria-required="true"
   - 에러 메시지: aria-live="polite"로 실시간 알림

4. 포커스 관리
   - 모든 인터랙티브 요소에 focus-visible 스타일
   - 모바일 메뉴 열릴 때 첫 메뉴 항목으로 포커스 이동
   - Skip navigation 링크 추가 (맨 위에 "본문으로 건너뛰기")

5. 이미지
   - 모든 <Image>에 의미 있는 alt 텍스트
   - 장식용 이미지: alt="" (빈 문자열)
```

### Skip Navigation 링크

```tsx
// src/app/layout.tsx의 <body> 맨 위에 추가
<a
  href="#main-content"
  className="sr-only focus:not-sr-only focus:absolute focus:top-4 focus:left-4
             focus:z-50 focus:px-4 focus:py-2 focus:bg-primary-600 focus:text-white
             focus:rounded-md"
>
  본문으로 건너뛰기
</a>

// <main> 태그에 id 추가
<main id="main-content">
  {children}
</main>
```

> `sr-only`로 평소에는 숨기고, Tab 키로 포커스되면 화면에 나타난다.

---

## 12-10. 접근성 테스트 종합 체크리스트

| # | 카테고리 | 항목 | 검사 방법 |
|:-:|---------|------|----------|
| 1 | 자동 검사 | axe-core 위반 0건 | `pnpm test` (E2E) |
| 2 | 자동 검사 | Lighthouse Accessibility 90+ | Chrome DevTools |
| 3 | 색상 | 텍스트 대비율 4.5:1 이상 | WebAIM Contrast Checker |
| 4 | 키보드 | Tab으로 모든 요소 접근 가능 | 수동 테스트 |
| 5 | 키보드 | 포커스 인디케이터 표시 | 수동 테스트 |
| 6 | 키보드 | Escape로 메뉴/모달 닫기 | 수동 테스트 |
| 7 | 스크린 리더 | 헤딩 구조 논리적 | VoiceOver/NVDA |
| 8 | 스크린 리더 | 이미지 alt 텍스트 적절 | VoiceOver/NVDA |
| 9 | 스크린 리더 | 폼 라벨 연결 | VoiceOver/NVDA |
| 10 | 반응형 | 200% 줌에서 콘텐츠 잘림 없음 | 브라우저 줌 |
