# Step 11: 에러 모니터링

> **이 단계의 목표:** Sentry와 Vercel Analytics를 연동하여 프로덕션 에러를 실시간 추적하고 성능을 모니터링한다.

---

## 11-1. 모니터링 아키텍처

```
사용자 브라우저 / 서버 런타임
  ├─→ Sentry (에러 추적)
  │     · 런타임 에러 캡처
  │     · 에러 발생 빈도/영향 범위 분석
  │     · Slack/이메일 알림
  │
  └─→ Vercel Analytics (성능 모니터링)
        · Core Web Vitals (LCP, FID, CLS)
        · 페이지별 로딩 성능
        · 방문자 통계
```

---

## 11-2. Sentry 설정

### 가입 및 프로젝트 생성

```
① https://sentry.io 접속 → 무료 계정 생성 (Developer 플랜)
② "Create Project" 클릭
③ Platform: "Next.js" 선택
④ 프로젝트 이름: "company-homepage"
⑤ DSN(Data Source Name) 복사 → 환경변수에 사용
```

### Sentry 무료 티어 (Developer)

| 항목 | 한도 |
|------|------|
| 에러 이벤트 | 월 5,000건 |
| 성능 트랜잭션 | 월 10,000건 |
| 세션 리플레이 | 월 50건 |
| 데이터 보존 | 30일 |
| 팀원 | 1명 |

> 회사 홈페이지 규모에서는 무료 티어로 충분하다.

### 설치

```bash
# Sentry Next.js SDK 설치 (설정 위저드 포함)
pnpm exec @sentry/wizard@latest -i nextjs
```

위저드가 자동으로 아래 파일을 생성/수정한다:

```
생성되는 파일:
├── sentry.client.config.ts    # 클라이언트 사이드 Sentry 초기화
├── sentry.server.config.ts    # 서버 사이드 Sentry 초기화
├── sentry.edge.config.ts      # Edge 런타임 Sentry 초기화
└── src/app/global-error.tsx   # 전역 에러 바운더리 (Sentry 연동)

수정되는 파일:
├── next.config.ts             # withSentryConfig 래퍼 추가
└── .env.local                 # SENTRY_DSN 등 환경변수 추가
```

### 환경변수

```bash
# .env.local에 추가 (위저드가 자동 생성)
SENTRY_DSN=https://xxxx@o1234.ingest.sentry.io/5678
SENTRY_ORG=your-org
SENTRY_PROJECT=company-homepage

# 선택: 소스맵 업로드용 인증 토큰
SENTRY_AUTH_TOKEN=sntrys_xxxxxxxxxxxx
```

### 수동 설정 (위저드 없이)

위저드를 사용하지 않는 경우 수동으로 설정하는 방법:

```bash
pnpm add @sentry/nextjs
```

#### `sentry.client.config.ts`

```typescript
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,

  // 성능 모니터링: 트랜잭션의 10%만 샘플링 (무료 티어 절약)
  tracesSampleRate: 0.1,

  // 세션 리플레이: 에러 발생 시에만 캡처
  replaysOnErrorSampleRate: 1.0,
  replaysSessionSampleRate: 0,

  // 개발 환경에서는 비활성화
  enabled: process.env.NODE_ENV === "production",
});
```

#### `sentry.server.config.ts`

```typescript
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  tracesSampleRate: 0.1,
  enabled: process.env.NODE_ENV === "production",
});
```

#### `next.config.ts` 수정

```typescript
import { withSentryConfig } from "@sentry/nextjs";

const nextConfig = {
  // 기존 Next.js 설정
};

export default withSentryConfig(nextConfig, {
  // 빌드 시 소스맵을 Sentry에 업로드
  // 프로덕션에서 에러 발생 위치를 원본 코드로 매핑
  org: process.env.SENTRY_ORG,
  project: process.env.SENTRY_PROJECT,

  // 소스맵은 Sentry에만 업로드하고 클라이언트에는 노출하지 않음
  hideSourceMaps: true,

  // 빌드 로그 비활성화
  silent: true,
});
```

### Vercel 환경변수 등록

```
Vercel 대시보드 → Settings → Environment Variables:

SENTRY_DSN            → Production, Preview
NEXT_PUBLIC_SENTRY_DSN → Production, Preview
SENTRY_ORG            → Production, Preview
SENTRY_PROJECT        → Production, Preview
SENTRY_AUTH_TOKEN     → Production, Preview
```

---

## 11-3. 커스텀 에러 캡처

### Server Action 에러 추적

```typescript
// src/app/contact/actions.ts
import * as Sentry from "@sentry/nextjs";

export async function submitContactForm(
  _prevState: ActionResult | null,
  formData: FormData
): Promise<ActionResult> {
  try {
    // ... 이메일 발송 로직
  } catch (error) {
    // Sentry에 에러 보고 + 추가 컨텍스트
    Sentry.captureException(error, {
      extra: {
        formData: {
          name: formData.get("name"),
          email: formData.get("email"),
          inquiryType: formData.get("inquiryType"),
          // message는 개인정보일 수 있으므로 전송하지 않음
        },
      },
    });

    return {
      success: false,
      message: "전송에 실패했습니다. 잠시 후 다시 시도해 주세요.",
    };
  }
}
```

### MDX 파싱 에러 추적

```typescript
// src/lib/mdx.ts
import * as Sentry from "@sentry/nextjs";

export async function getSolutionBySlug(slug: string) {
  try {
    // ... MDX 파싱 로직
  } catch (error) {
    Sentry.captureException(error, {
      tags: { type: "mdx-parsing" },
      extra: { slug },
    });
    throw error;
  }
}
```

---

## 11-4. Sentry 알림 설정

### Slack 연동

```
① Sentry 대시보드 → Settings → Integrations → Slack
② Slack 워크스페이스 연결
③ Alert Rules 설정:
   - "새로운 에러 발생 시" → #website-alerts 채널에 알림
   - "에러 빈도 급증 시" → @개발자 멘션
```

### 알림 규칙 권장 설정

| 규칙 | 조건 | 액션 |
|------|------|------|
| 새 에러 | 처음 보는 에러 발생 | Slack 알림 |
| 에러 급증 | 1시간 내 같은 에러 10회 이상 | Slack 알림 + 이메일 |
| 고빈도 에러 | 24시간 내 같은 에러 100회 이상 | Slack 긴급 알림 |

---

## 11-5. Vercel Analytics 설정

Vercel Analytics는 Vercel 대시보드에서 클릭 한 번으로 활성화할 수 있다.

### 활성화

```
Vercel 대시보드 → 프로젝트 → Analytics 탭 → "Enable" 클릭
```

### 코드 연동

```bash
# Vercel Analytics 패키지 설치
pnpm add @vercel/analytics
```

```tsx
// src/app/layout.tsx
import { Analytics } from "@vercel/analytics/react";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ko">
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  );
}
```

### Vercel Speed Insights (Core Web Vitals)

```bash
pnpm add @vercel/speed-insights
```

```tsx
// src/app/layout.tsx
import { Analytics } from "@vercel/analytics/react";
import { SpeedInsights } from "@vercel/speed-insights/next";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ko">
      <body>
        {children}
        <Analytics />
        <SpeedInsights />
      </body>
    </html>
  );
}
```

### Vercel Analytics 무료 티어

| 항목 | Hobby (무료) |
|------|:------------:|
| 데이터 포인트 | 월 2,500건 |
| Speed Insights | 월 5,000 데이터 포인트 |
| 데이터 보존 | 1일 (Hobby) |

---

## 11-6. 모니터링 대시보드 확인 체크리스트

| # | 확인 항목 | 위치 | 주기 |
|:-:|----------|------|:----:|
| 1 | 미해결 에러 | Sentry → Issues | 매일 |
| 2 | 에러 추이 그래프 | Sentry → Discover | 주 1회 |
| 3 | Core Web Vitals | Vercel → Speed Insights | 주 1회 |
| 4 | 페이지별 방문 수 | Vercel → Analytics | 주 1회 |
| 5 | 서버 에러 로그 | Vercel → Logs | 에러 발생 시 |

---

## 11-7. 트러블슈팅

| 증상 | 원인 | 해결 |
|------|------|------|
| Sentry에 이벤트가 안 올라옴 | DSN 미설정 또는 `enabled: false` | 환경변수, NODE_ENV 확인 |
| 소스맵이 매핑 안 됨 | SENTRY_AUTH_TOKEN 미설정 | Vercel 환경변수에 토큰 추가 |
| 무료 한도 초과 알림 | 이벤트 5,000건 초과 | `tracesSampleRate`을 0.05로 낮추기 |
| Analytics 데이터 미표시 | `<Analytics />` 컴포넌트 누락 | layout.tsx에 추가 확인 |
