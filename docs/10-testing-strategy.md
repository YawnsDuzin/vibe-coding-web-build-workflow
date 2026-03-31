# Step 10: 테스트 전략

> **이 단계의 목표:** Playwright E2E 테스트를 도입하여 페이지 라우팅, 문의 폼, MDX 콘텐츠 렌더링이 정상 동작하는지 자동으로 검증한다.

---

## 10-1. 테스트 전략 개요

### 회사 홈페이지에 필요한 테스트 범위

```
                       ┌──────────────────────┐
                       │   E2E 테스트 (Playwright)  │  ← 이 문서에서 구현
                       │   · 페이지 라우팅          │
                       │   · 문의 폼 제출           │
                       │   · MDX 콘텐츠 렌더링      │
                       │   · 반응형 레이아웃         │
                       └──────────────────────┘
               ┌───────────────────────────────────┐
               │        통합 테스트 (선택)              │
               │   · Server Action 동작              │
               │   · MDX 파싱 유틸 함수               │
               └───────────────────────────────────┘
       ┌───────────────────────────────────────────────┐
       │              단위 테스트 (선택)                    │
       │   · 유효성 검사 함수                              │
       │   · 유틸리티 함수                                 │
       └───────────────────────────────────────────────┘
```

### 왜 E2E 테스트를 우선하는가?

| 관점 | 이유 |
|------|------|
| 바이브코딩 특성 | AI가 생성한 코드는 내부 구현이 자주 바뀌므로, 최종 사용자 관점의 E2E가 가장 안정적 |
| 정적 사이트 특성 | 비즈니스 로직이 적고 UI 렌더링이 핵심이므로 E2E가 가성비 최고 |
| 비개발자 수정 | MDX 콘텐츠 변경 시 빌드+렌더링 정상 여부를 E2E로 검증 |

---

## 10-2. Playwright 설치 및 설정

### 설치

```bash
# Playwright 설치 (테스트 러너 + 브라우저)
pnpm add -D @playwright/test

# 브라우저 바이너리 설치 (Chromium만 — 빌드 시간 절약)
pnpm exec playwright install chromium
```

### 설정 파일: `playwright.config.ts`

```typescript
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  // 테스트 파일 위치
  testDir: "./e2e",

  // 각 테스트 최대 실행 시간
  timeout: 30_000,

  // 테스트 실패 시 재시도 (CI에서만)
  retries: process.env.CI ? 2 : 0,

  // 병렬 실행 워커 수
  workers: process.env.CI ? 1 : undefined,

  // 리포터 설정
  reporter: process.env.CI ? "github" : "html",

  // 전역 설정
  use: {
    // 테스트할 기본 URL
    baseURL: "http://localhost:3000",

    // 실패 시 스크린샷 저장
    screenshot: "only-on-failure",

    // 실패 시 트레이스 저장 (디버깅용)
    trace: "on-first-retry",
  },

  // 테스트할 브라우저/디바이스
  projects: [
    {
      name: "desktop-chrome",
      use: { ...devices["Desktop Chrome"] },
    },
    {
      name: "mobile-chrome",
      use: { ...devices["Pixel 7"] },
    },
  ],

  // 테스트 전에 개발 서버 자동 시작
  webServer: {
    command: "pnpm dev",
    url: "http://localhost:3000",
    reuseExistingServer: !process.env.CI,
    timeout: 120_000,
  },
});
```

### package.json 스크립트 추가

```json
{
  "scripts": {
    "test": "playwright test",
    "test:ui": "playwright test --ui",
    "test:headed": "playwright test --headed"
  }
}
```

---

## 10-3. 테스트 파일 작성

### 디렉토리 구조

```
e2e/
├── navigation.spec.ts     # 페이지 라우팅 + 네비게이션
├── home.spec.ts           # 메인 페이지 섹션 검증
├── solutions.spec.ts      # 솔루션 목록 + 상세 페이지
├── contact.spec.ts        # 문의 폼 유효성 검사 + 제출
└── responsive.spec.ts     # 모바일 반응형 레이아웃
```

### 테스트 ①: 페이지 라우팅 + 네비게이션

```typescript
// e2e/navigation.spec.ts
import { test, expect } from "@playwright/test";

test.describe("페이지 라우팅", () => {
  test("모든 페이지가 200으로 응답한다", async ({ page }) => {
    const routes = ["/", "/about", "/solutions", "/careers", "/contact"];

    for (const route of routes) {
      const response = await page.goto(route);
      expect(response?.status()).toBe(200);
    }
  });

  test("존재하지 않는 페이지는 404를 반환한다", async ({ page }) => {
    const response = await page.goto("/nonexistent-page");
    expect(response?.status()).toBe(404);
  });
});

test.describe("헤더 네비게이션", () => {
  test("로고 클릭 시 홈으로 이동한다", async ({ page }) => {
    await page.goto("/about");
    await page.getByRole("link", { name: /CompanyName/i }).click();
    await expect(page).toHaveURL("/");
  });

  test("네비게이션 메뉴가 올바른 페이지로 이동한다", async ({ page }) => {
    await page.goto("/");

    const menuItems = [
      { name: "회사소개", url: "/about" },
      { name: "솔루션", url: "/solutions" },
      { name: "채용", url: "/careers" },
      { name: "문의", url: "/contact" },
    ];

    for (const item of menuItems) {
      await page.getByRole("link", { name: item.name }).first().click();
      await expect(page).toHaveURL(item.url);
    }
  });
});
```

### 테스트 ②: 메인 페이지

```typescript
// e2e/home.spec.ts
import { test, expect } from "@playwright/test";

test.describe("메인 페이지", () => {
  test.beforeEach(async ({ page }) => {
    await page.goto("/");
  });

  test("히어로 섹션이 표시된다", async ({ page }) => {
    await expect(page.getByRole("heading", { level: 1 })).toBeVisible();
    // CTA 버튼 2개 확인
    await expect(page.getByRole("link", { name: /솔루션 보기/i })).toBeVisible();
    await expect(page.getByRole("link", { name: /문의하기/i })).toBeVisible();
  });

  test("솔루션 미리보기 카드가 표시된다", async ({ page }) => {
    // 솔루션 카드가 최소 1개 이상 표시되는지 확인
    const solutionCards = page.locator("[data-testid='solution-card']");
    await expect(solutionCards.first()).toBeVisible();
  });

  test("메타 태그가 올바르게 설정되어 있다", async ({ page }) => {
    const title = await page.title();
    expect(title).toContain("CompanyName");

    const description = page.locator('meta[name="description"]');
    await expect(description).toHaveAttribute("content", /.+/);
  });
});
```

### 테스트 ③: 솔루션 페이지

```typescript
// e2e/solutions.spec.ts
import { test, expect } from "@playwright/test";

test.describe("솔루션 목록 페이지", () => {
  test("솔루션 카드 목록이 표시된다", async ({ page }) => {
    await page.goto("/solutions");
    const heading = page.getByRole("heading", { name: /솔루션/i });
    await expect(heading).toBeVisible();

    // 카드가 1개 이상 표시
    const cards = page.locator("[data-testid='solution-card']");
    expect(await cards.count()).toBeGreaterThan(0);
  });

  test("카드 클릭 시 상세 페이지로 이동한다", async ({ page }) => {
    await page.goto("/solutions");
    const firstCard = page.locator("[data-testid='solution-card']").first();
    await firstCard.click();
    await expect(page).toHaveURL(/\/solutions\/.+/);
  });
});

test.describe("솔루션 상세 페이지", () => {
  test("MDX 콘텐츠가 렌더링된다", async ({ page }) => {
    await page.goto("/solutions/product-a");

    // 제목 표시 확인
    await expect(page.getByRole("heading", { level: 1 })).toBeVisible();

    // 본문 콘텐츠 영역 확인
    const content = page.locator("article, .prose");
    await expect(content.first()).toBeVisible();
  });

  test("다른 솔루션 링크가 표시된다", async ({ page }) => {
    await page.goto("/solutions/product-a");
    const otherSolutions = page.getByText(/다른 솔루션/i);
    await expect(otherSolutions).toBeVisible();
  });
});
```

### 테스트 ④: 문의 폼

```typescript
// e2e/contact.spec.ts
import { test, expect } from "@playwright/test";

test.describe("문의 폼", () => {
  test.beforeEach(async ({ page }) => {
    await page.goto("/contact");
  });

  test("모든 폼 필드가 표시된다", async ({ page }) => {
    await expect(page.getByLabel(/이름/i)).toBeVisible();
    await expect(page.getByLabel(/이메일/i)).toBeVisible();
    await expect(page.getByLabel(/회사명/i)).toBeVisible();
    await expect(page.getByLabel(/문의 유형/i)).toBeVisible();
    await expect(page.getByLabel(/메시지/i)).toBeVisible();
    await expect(page.getByRole("button", { name: /전송|보내기|제출/i })).toBeVisible();
  });

  test("필수 필드가 비어있으면 에러 메시지를 표시한다", async ({ page }) => {
    // 빈 폼 제출 시도
    await page.getByRole("button", { name: /전송|보내기|제출/i }).click();

    // 에러 메시지 또는 HTML5 validation 확인
    const nameInput = page.getByLabel(/이름/i);
    const isInvalid = await nameInput.evaluate(
      (el: HTMLInputElement) => !el.validity.valid
    );
    expect(isInvalid).toBe(true);
  });

  test("유효한 데이터로 폼을 제출할 수 있다", async ({ page }) => {
    // 폼 필드 채우기
    await page.getByLabel(/이름/i).fill("홍길동");
    await page.getByLabel(/이메일/i).fill("test@example.com");
    await page.getByLabel(/회사명/i).fill("테스트 회사");
    await page.getByLabel(/문의 유형/i).selectOption({ index: 1 });
    await page.getByLabel(/메시지/i).fill("테스트 문의 메시지입니다. 10자 이상 작성합니다.");

    // 제출
    await page.getByRole("button", { name: /전송|보내기|제출/i }).click();

    // 성공 메시지 확인 (5초 대기)
    await expect(page.getByText(/접수|완료|감사/i)).toBeVisible({ timeout: 5000 });
  });

  test("잘못된 이메일 형식은 거부한다", async ({ page }) => {
    await page.getByLabel(/이름/i).fill("홍길동");
    await page.getByLabel(/이메일/i).fill("invalid-email");
    await page.getByLabel(/메시지/i).fill("테스트 메시지입니다. 충분히 길게 작성합니다.");

    await page.getByRole("button", { name: /전송|보내기|제출/i }).click();

    // 이메일 필드가 invalid 상태인지 확인
    const emailInput = page.getByLabel(/이메일/i);
    const isInvalid = await emailInput.evaluate(
      (el: HTMLInputElement) => !el.validity.valid
    );
    expect(isInvalid).toBe(true);
  });
});
```

### 테스트 ⑤: 모바일 반응형

```typescript
// e2e/responsive.spec.ts
import { test, expect, devices } from "@playwright/test";

test.describe("모바일 반응형", () => {
  test.use(devices["Pixel 7"]);

  test("모바일에서 햄버거 메뉴가 동작한다", async ({ page }) => {
    await page.goto("/");

    // 데스크톱 메뉴는 숨겨지고 햄버거 버튼이 보여야 함
    const hamburger = page.getByRole("button", { name: /메뉴|menu/i });
    await expect(hamburger).toBeVisible();

    // 햄버거 클릭 → 모바일 메뉴 열림
    await hamburger.click();
    await expect(page.getByRole("link", { name: /회사소개/i })).toBeVisible();
  });

  test("모바일에서 가로 스크롤이 없다", async ({ page }) => {
    const routes = ["/", "/about", "/solutions", "/contact"];

    for (const route of routes) {
      await page.goto(route);
      const hasHorizontalScroll = await page.evaluate(() => {
        return document.documentElement.scrollWidth > document.documentElement.clientWidth;
      });
      expect(hasHorizontalScroll, `${route}에서 가로 스크롤 발생`).toBe(false);
    }
  });
});
```

---

## 10-4. CI에 테스트 추가

### `.github/workflows/ci.yml` 수정

기존 CI 워크플로우에 E2E 테스트 step을 추가한다.

```yaml
      # ── (기존 Step 5~7 이후에 추가) ──

      # 8. Playwright 브라우저 설치
      - name: Playwright 브라우저 설치
        run: pnpm exec playwright install chromium --with-deps

      # 9. E2E 테스트 실행
      - name: E2E 테스트
        run: pnpm test

      # 10. 테스트 실패 시 리포트 업로드
      - name: 테스트 리포트 업로드
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 7
```

---

## 10-5. 테스트 실행 방법

```bash
# 전체 테스트 실행
pnpm test

# UI 모드로 실행 (브라우저에서 테스트 실시간 확인)
pnpm test:ui

# 특정 파일만 실행
pnpm exec playwright test e2e/contact.spec.ts

# 특정 테스트만 실행
pnpm exec playwright test -g "문의 폼"

# 브라우저를 띄워서 실행 (디버깅용)
pnpm test:headed

# 실패한 테스트 리포트 보기
pnpm exec playwright show-report
```

---

## 10-6. 바이브코딩으로 테스트 추가하기

새 페이지/기능을 만들 때 함께 요청하는 프롬프트:

```
채용 페이지를 만들어줘.
[...페이지 구성 상세...]

추가로 e2e/careers.spec.ts에 E2E 테스트도 작성해줘:
- 채용 공고 목록이 표시되는지
- isOpen: true인 공고만 보이는지
- 각 공고에 팀, 위치, 근무 형태가 표시되는지
```

---

## 10-7. 트러블슈팅

| 증상 | 원인 | 해결 |
|------|------|------|
| `browserType.launch: Executable doesn't exist` | 브라우저 미설치 | `pnpm exec playwright install chromium` |
| 테스트 타임아웃 | 개발 서버 시작 지연 | `playwright.config.ts`의 webServer.timeout 증가 |
| CI에서만 실패 | 환경 차이 | `--with-deps` 플래그로 시스템 의존성 함께 설치 |
| 셀렉터를 못 찾음 | 컴포넌트에 접근성 속성 누락 | `data-testid`, `aria-label`, `role` 추가 |
