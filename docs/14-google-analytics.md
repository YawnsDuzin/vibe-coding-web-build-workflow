# Step 14: Google Analytics 연동

> **이 단계의 목표:** GA4(Google Analytics 4)를 Next.js 프로젝트에 연동하여 방문자 통계, 페이지뷰, 이벤트를 추적한다.

---

## 14-1. GA4 계정 및 속성 생성

### 단계별 절차

```
① https://analytics.google.com 접속 → Google 계정 로그인

② "측정 시작" 클릭
   → 계정 이름: "Company Name"
   → 데이터 공유 설정: 기본값 유지

③ 속성 생성
   → 속성 이름: "Company Homepage"
   → 보고 시간대: 대한민국 (GMT+09:00)
   → 통화: 대한민국 원 (KRW)

④ 비즈니스 정보
   → 업종: 해당 업종 선택
   → 비즈니스 규모: 소규모

⑤ 데이터 스트림 생성
   → "웹" 선택
   → URL: https://www.company.com
   → 스트림 이름: "Company Homepage Web"
   → "스트림 만들기" 클릭

⑥ 측정 ID 복사
   → G-XXXXXXXXXX 형식의 ID를 복사
   → 환경변수에 사용
```

---

## 14-2. 환경변수 설정

```bash
# .env.local에 추가
NEXT_PUBLIC_GA_ID=G-XXXXXXXXXX
```

```bash
# .env.example에도 추가 (템플릿)
# Google Analytics 측정 ID
NEXT_PUBLIC_GA_ID=G-XXXXXXXXXX
```

```
# Vercel 환경변수에도 등록
Vercel 대시보드 → Settings → Environment Variables
→ NEXT_PUBLIC_GA_ID = G-실제측정ID (Production, Preview)
```

---

## 14-3. GA4 스크립트 삽입

### 방법 A: Next.js Script 컴포넌트 (권장)

#### 파일: `src/components/analytics/GoogleAnalytics.tsx`

```tsx
"use client";

import Script from "next/script";

const GA_ID = process.env.NEXT_PUBLIC_GA_ID;

export default function GoogleAnalytics() {
  // 측정 ID가 없으면 렌더링하지 않음 (로컬 개발 시)
  if (!GA_ID) return null;

  return (
    <>
      {/* GA4 gtag.js 로드 */}
      <Script
        src={`https://www.googletagmanager.com/gtag/js?id=${GA_ID}`}
        strategy="afterInteractive"
      />
      {/* GA4 초기화 */}
      <Script
        id="google-analytics"
        strategy="afterInteractive"
        dangerouslySetInnerHTML={{
          __html: `
            window.dataLayer = window.dataLayer || [];
            function gtag(){dataLayer.push(arguments);}
            gtag('js', new Date());
            gtag('config', '${GA_ID}', {
              page_path: window.location.pathname,
            });
          `,
        }}
      />
    </>
  );
}
```

#### `src/app/layout.tsx`에 추가

```tsx
import GoogleAnalytics from "@/components/analytics/GoogleAnalytics";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ko">
      <body>
        <GoogleAnalytics />
        {/* Header, children, Footer ... */}
      </body>
    </html>
  );
}
```

### 방법 B: @next/third-parties 사용 (간편)

```bash
pnpm add @next/third-parties
```

```tsx
// src/app/layout.tsx
import { GoogleAnalytics } from "@next/third-parties/google";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ko">
      <body>
        {children}
        {process.env.NEXT_PUBLIC_GA_ID && (
          <GoogleAnalytics gaId={process.env.NEXT_PUBLIC_GA_ID} />
        )}
      </body>
    </html>
  );
}
```

> 방법 B가 코드가 더 간결하지만, 커스텀 이벤트 추적 시 방법 A가 더 유연하다.

---

## 14-4. 페이지뷰 자동 추적

Next.js App Router는 클라이언트 사이드 네비게이션을 사용하므로, 페이지 전환 시 GA에 별도로 페이지뷰를 보고해야 한다.

#### 파일: `src/components/analytics/PageViewTracker.tsx`

```tsx
"use client";

import { usePathname, useSearchParams } from "next/navigation";
import { useEffect, Suspense } from "react";

const GA_ID = process.env.NEXT_PUBLIC_GA_ID;

function PageViewTrackerInner() {
  const pathname = usePathname();
  const searchParams = useSearchParams();

  useEffect(() => {
    if (!GA_ID) return;

    const url = pathname + (searchParams?.toString() ? `?${searchParams.toString()}` : "");

    // GA4에 페이지뷰 이벤트 전송
    window.gtag?.("config", GA_ID, {
      page_path: url,
    });
  }, [pathname, searchParams]);

  return null;
}

export default function PageViewTracker() {
  return (
    <Suspense fallback={null}>
      <PageViewTrackerInner />
    </Suspense>
  );
}
```

```tsx
// src/app/layout.tsx에 추가
import PageViewTracker from "@/components/analytics/PageViewTracker";

// <body> 안에 추가
<PageViewTracker />
```

> `@next/third-parties`의 `GoogleAnalytics` 컴포넌트를 사용하는 경우 페이지뷰가 자동 추적되므로 이 컴포넌트는 불필요하다.

---

## 14-5. 커스텀 이벤트 추적

### 이벤트 유틸 함수

#### 파일: `src/lib/analytics.ts`

```typescript
// GA4 커스텀 이벤트 전송 유틸
export function trackEvent(
  eventName: string,
  parameters?: Record<string, string | number | boolean>
) {
  if (typeof window === "undefined") return;
  if (!process.env.NEXT_PUBLIC_GA_ID) return;

  window.gtag?.("event", eventName, parameters);
}

// 미리 정의된 이벤트들
export const analytics = {
  // CTA 버튼 클릭
  clickCTA: (label: string) =>
    trackEvent("click_cta", { button_label: label }),

  // 솔루션 카드 클릭
  viewSolution: (solutionName: string) =>
    trackEvent("view_solution", { solution_name: solutionName }),

  // 문의 폼 제출
  submitContact: (inquiryType: string) =>
    trackEvent("submit_contact_form", { inquiry_type: inquiryType }),

  // 문의 폼 제출 성공
  contactSuccess: () =>
    trackEvent("contact_form_success"),

  // 채용 공고 클릭
  viewCareer: (position: string) =>
    trackEvent("view_career", { position_title: position }),

  // 외부 링크 클릭
  clickExternalLink: (url: string) =>
    trackEvent("click_external_link", { link_url: url }),
};
```

### TypeScript 타입 선언

#### 파일: `src/types/gtag.d.ts`

```typescript
interface Window {
  gtag?: (
    command: "config" | "event" | "js",
    targetId: string | Date,
    config?: Record<string, unknown>
  ) => void;
  dataLayer?: unknown[];
}
```

### 사용 예시

```tsx
// 솔루션 카드 컴포넌트에서
import { analytics } from "@/lib/analytics";

<Link
  href={`/solutions/${slug}`}
  onClick={() => analytics.viewSolution(title)}
>
  {title}
</Link>

// 문의 폼에서
const handleSubmit = async () => {
  analytics.submitContact(inquiryType);
  const result = await submitContactForm(formData);
  if (result.success) {
    analytics.contactSuccess();
  }
};
```

---

## 14-6. 추적할 이벤트 목록

| 이벤트 | 트리거 | GA4 이벤트명 | 파라미터 |
|--------|--------|:----------:|---------|
| CTA 클릭 | 히어로/배너 CTA 버튼 클릭 | `click_cta` | `button_label` |
| 솔루션 조회 | 솔루션 상세 페이지 진입 | `view_solution` | `solution_name` |
| 문의 제출 | 문의 폼 전송 버튼 클릭 | `submit_contact_form` | `inquiry_type` |
| 문의 성공 | 문의 폼 전송 성공 | `contact_form_success` | — |
| 채용 조회 | 채용 공고 클릭 | `view_career` | `position_title` |
| 외부 링크 | 외부 URL 클릭 | `click_external_link` | `link_url` |

---

## 14-7. GA4 대시보드 확인

### 실시간 데이터

```
GA4 대시보드 → 보고서 → 실시간
→ 현재 접속자 수, 페이지뷰, 이벤트 확인
→ 연동 직후 테스트 시 유용
```

### 주요 보고서

| 보고서 | 위치 | 확인 내용 |
|--------|------|----------|
| 사용자 획득 | 보고서 → 획득 → 사용자 획득 | 어떤 채널(검색, 직접, 소셜)에서 유입되는지 |
| 페이지 및 화면 | 보고서 → 참여도 → 페이지 및 화면 | 어떤 페이지가 가장 많이 조회되는지 |
| 이벤트 | 보고서 → 참여도 → 이벤트 | 커스텀 이벤트(CTA 클릭, 문의 등) 빈도 |
| 기술 세부정보 | 보고서 → 기술 → 기술 세부정보 | 방문자 디바이스, 브라우저, OS |

### 유용한 맞춤 탐색

```
탐색 → 새 탐색 만들기

"문의 전환 퍼널":
  ① 메인 페이지 조회 (page_view, page_path = /)
  ② 솔루션 페이지 조회 (view_solution)
  ③ 문의 페이지 진입 (page_view, page_path = /contact)
  ④ 문의 폼 제출 (submit_contact_form)
  ⑤ 문의 성공 (contact_form_success)
```

---

## 14-8. 개인정보 보호 고려사항

### 쿠키 동의 배너 (한국법 기준)

한국 개인정보보호법상 GA 쿠키 사용 시 사용자 동의가 권장된다.

```tsx
// 간단한 쿠키 동의 배너 프롬프트
"푸터 위에 쿠키 동의 배너를 만들어줘:
- 텍스트: '이 웹사이트는 사용자 경험 개선을 위해 쿠키를 사용합니다.'
- 버튼 2개: '동의' (쿠키 허용), '거부' (GA 비활성화)
- localStorage에 동의 상태 저장
- 동의하지 않으면 GA 스크립트를 로드하지 않음
- 한 번 선택하면 30일간 배너 숨김"
```

### GA에서 IP 익명화

GA4는 기본적으로 IP 주소를 저장하지 않는다 (UA와 달리 별도 설정 불필요).

---

## 14-9. 트러블슈팅

| 증상 | 원인 | 해결 |
|------|------|------|
| 실시간에 데이터 안 뜸 | 측정 ID 오류 또는 환경변수 미설정 | `NEXT_PUBLIC_GA_ID` 확인, 브라우저 콘솔에서 `gtag` 함수 존재 확인 |
| 페이지 전환 미추적 | SPA 네비게이션 시 페이지뷰 미전송 | `PageViewTracker` 컴포넌트 또는 `@next/third-parties` 사용 |
| 로컬에서 추적됨 | 개발 환경에서 GA 활성화 | `if (!GA_ID) return null` 조건 확인, .env.local에서 GA_ID 제거 |
| 광고 차단기로 차단 | 사용자의 AdBlocker | 서버 사이드 분석(Vercel Analytics)을 병행 |
| 이벤트 미수신 | `trackEvent` 호출 시점 오류 | 브라우저 콘솔에서 `dataLayer` 배열 확인 |
