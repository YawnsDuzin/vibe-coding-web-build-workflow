# Step 8: SEO 심화 가이드

> **이 단계의 목표:** 메타 태그 이상의 SEO 요소(사이트맵, robots.txt, 구조화 데이터, canonical URL)를 구현하여 검색 엔진 가시성을 극대화한다.

---

## 8-1. SEO 체크리스트 전체 맵

```
[기본] Step 3에서 이미 구현
  ✅ generateMetadata (title, description)
  ✅ OG 태그 (og:title, og:description, og:image)
  ✅ 시맨틱 HTML (<header>, <nav>, <main>, <footer>)

[심화] 이 문서에서 구현
  □ sitemap.xml — 검색 엔진에 전체 페이지 구조 알림
  □ robots.txt — 크롤링 허용/차단 규칙
  □ 구조화 데이터 (JSON-LD) — 리치 스니펫 활성화
  □ canonical URL — 중복 콘텐츠 방지
  □ next/image 최적화 — 이미지 SEO
  □ 성능 최적화 — Core Web Vitals (검색 순위 반영)
```

---

## 8-2. sitemap.xml 자동 생성

Next.js App Router는 `sitemap.ts` 파일을 통해 사이트맵을 자동 생성할 수 있다.

### 파일: `src/app/sitemap.ts`

```typescript
import { MetadataRoute } from "next";
import { getAllSolutionSlugs } from "@/lib/mdx";

// 사이트 기본 URL
const BASE_URL = process.env.NEXT_PUBLIC_SITE_URL || "https://www.company.com";

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  // 정적 페이지 목록
  const staticPages: MetadataRoute.Sitemap = [
    {
      url: BASE_URL,
      lastModified: new Date(),
      changeFrequency: "weekly",
      priority: 1.0,
    },
    {
      url: `${BASE_URL}/about`,
      lastModified: new Date(),
      changeFrequency: "monthly",
      priority: 0.8,
    },
    {
      url: `${BASE_URL}/solutions`,
      lastModified: new Date(),
      changeFrequency: "weekly",
      priority: 0.9,
    },
    {
      url: `${BASE_URL}/careers`,
      lastModified: new Date(),
      changeFrequency: "weekly",
      priority: 0.7,
    },
    {
      url: `${BASE_URL}/contact`,
      lastModified: new Date(),
      changeFrequency: "monthly",
      priority: 0.6,
    },
  ];

  // 동적 페이지: 솔루션 상세
  const solutionSlugs = await getAllSolutionSlugs();
  const solutionPages: MetadataRoute.Sitemap = solutionSlugs.map((slug) => ({
    url: `${BASE_URL}/solutions/${slug}`,
    lastModified: new Date(),
    changeFrequency: "monthly" as const,
    priority: 0.8,
  }));

  return [...staticPages, ...solutionPages];
}
```

### 확인 방법

```bash
# 개발 서버에서 확인
pnpm dev
# http://localhost:3000/sitemap.xml 접속

# 빌드 후 확인
pnpm build
# .next/server/app/sitemap.xml 파일 생성 확인
```

### sitemap 필드 설명

| 필드 | 의미 | 권장 값 |
|------|------|---------|
| `url` | 페이지 절대 URL | `https://www.company.com/about` |
| `lastModified` | 마지막 수정일 | `new Date()` 또는 실제 수정 날짜 |
| `changeFrequency` | 변경 빈도 힌트 | `weekly`, `monthly`, `yearly` |
| `priority` | 상대적 중요도 (0.0~1.0) | 메인 1.0, 주요 페이지 0.8, 보조 페이지 0.6 |

---

## 8-3. robots.txt 설정

### 파일: `src/app/robots.ts`

```typescript
import { MetadataRoute } from "next";

const BASE_URL = process.env.NEXT_PUBLIC_SITE_URL || "https://www.company.com";

export default function robots(): MetadataRoute.Robots {
  return {
    rules: [
      {
        // 모든 검색 엔진 허용
        userAgent: "*",
        allow: "/",
        disallow: [
          "/api/",        // API 라우트 크롤링 차단
          "/_next/",      // Next.js 내부 파일 차단
          "/admin/",      // 관리자 페이지 (있을 경우)
        ],
      },
    ],
    // 사이트맵 위치 알림
    sitemap: `${BASE_URL}/sitemap.xml`,
  };
}
```

### 확인

```
http://localhost:3000/robots.txt 접속 시 출력:

User-Agent: *
Allow: /
Disallow: /api/
Disallow: /_next/
Disallow: /admin/

Sitemap: https://www.company.com/sitemap.xml
```

---

## 8-4. 구조화 데이터 (JSON-LD)

구조화 데이터를 추가하면 Google 검색 결과에 **리치 스니펫**(별점, FAQ, 회사 정보 등)이 표시된다.

### 조직(Organization) 구조화 데이터

#### 파일: `src/app/layout.tsx`에 추가

```tsx
// layout.tsx의 <body> 안, {children} 앞에 추가

<script
  type="application/ld+json"
  dangerouslySetInnerHTML={{
    __html: JSON.stringify({
      "@context": "https://schema.org",
      "@type": "Organization",
      name: "CompanyName",
      url: "https://www.company.com",
      logo: "https://www.company.com/images/logo.png",
      description: "디지털 혁신의 파트너",
      address: {
        "@type": "PostalAddress",
        streetAddress: "서울특별시 강남구 테헤란로 123",
        addressLocality: "서울",
        addressCountry: "KR",
      },
      contactPoint: {
        "@type": "ContactPoint",
        email: "info@company.com",
        contactType: "customer service",
        availableLanguage: "Korean",
      },
      sameAs: [
        "https://www.linkedin.com/company/companyname",
        "https://github.com/companyname",
      ],
    }),
  }}
/>
```

### 제품(Product) 구조화 데이터 — 솔루션 상세 페이지

#### 파일: `src/app/solutions/[slug]/page.tsx`에 추가

```tsx
// 페이지 컴포넌트 안, return JSX 상단에 추가

<script
  type="application/ld+json"
  dangerouslySetInnerHTML={{
    __html: JSON.stringify({
      "@context": "https://schema.org",
      "@type": "Product",
      name: solution.title,
      description: solution.description,
      brand: {
        "@type": "Organization",
        name: "CompanyName",
      },
      offers: {
        "@type": "Offer",
        availability: "https://schema.org/InStock",
        url: `https://www.company.com/solutions/${slug}`,
      },
    }),
  }}
/>
```

### 채용 공고(JobPosting) 구조화 데이터

```tsx
<script
  type="application/ld+json"
  dangerouslySetInnerHTML={{
    __html: JSON.stringify({
      "@context": "https://schema.org",
      "@type": "JobPosting",
      title: career.title,
      description: "채용 공고 상세 설명",
      datePosted: "2024-01-15",
      employmentType: "FULL_TIME",
      hiringOrganization: {
        "@type": "Organization",
        name: "CompanyName",
        sameAs: "https://www.company.com",
      },
      jobLocation: {
        "@type": "Place",
        address: {
          "@type": "PostalAddress",
          addressLocality: career.location,
          addressCountry: "KR",
        },
      },
    }),
  }}
/>
```

### 구조화 데이터 검증 방법

```
1. Google Rich Results Test
   → https://search.google.com/test/rich-results
   → 프로덕션 URL 입력하여 유효성 확인

2. Schema.org Validator
   → https://validator.schema.org
   → JSON-LD 코드 직접 붙여넣기로 검증
```

---

## 8-5. Canonical URL 설정

동일한 콘텐츠가 여러 URL로 접근 가능한 경우, canonical URL로 원본을 지정하여 중복 콘텐츠 패널티를 방지한다.

### generateMetadata에서 설정

```typescript
// src/app/solutions/[slug]/page.tsx

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const solution = await getSolutionBySlug(params.slug);
  const url = `${process.env.NEXT_PUBLIC_SITE_URL}/solutions/${params.slug}`;

  return {
    title: `${solution.title} | CompanyName`,
    description: solution.description,
    // canonical URL 지정
    alternates: {
      canonical: url,
    },
    openGraph: {
      title: solution.title,
      description: solution.description,
      url: url,
      siteName: "CompanyName",
      images: [{ url: "/og-image.png", width: 1200, height: 630 }],
      type: "website",
    },
  };
}
```

### 전역 기본 canonical 설정

```typescript
// src/app/layout.tsx의 metadata

export const metadata: Metadata = {
  metadataBase: new URL(
    process.env.NEXT_PUBLIC_SITE_URL || "https://www.company.com"
  ),
  // metadataBase를 설정하면 상대 경로가 자동으로 절대 URL로 변환됨
};
```

---

## 8-6. 이미지 SEO

### next/image 최적화 규칙

```tsx
import Image from "next/image";

// ✅ 좋은 예
<Image
  src="/images/solutions/cloud-manager.png"
  alt="클라우드 매니저 대시보드 - 멀티 클라우드 통합 관리 화면"
  width={800}
  height={600}
  priority  // 히어로 이미지만 priority 사용
/>

// ❌ 나쁜 예
<img
  src="/images/solutions/cloud-manager.png"
  alt=""  // alt 텍스트 비어있음
/>
```

### 이미지 SEO 체크리스트

| # | 항목 | 규칙 |
|:-:|------|------|
| 1 | alt 텍스트 | 이미지 내용을 구체적으로 설명 (키워드 포함) |
| 2 | 파일명 | 의미 있는 이름 사용 (`cloud-manager.png` > `img001.png`) |
| 3 | 크기 지정 | width/height 반드시 명시 (CLS 방지) |
| 4 | 포맷 | WebP 또는 AVIF 사용 (Next.js가 자동 변환) |
| 5 | priority | 뷰포트 상단 이미지에만 사용 (LCP 개선) |
| 6 | lazy loading | 스크롤 아래 이미지는 자동 lazy (기본값) |

---

## 8-7. 페이지별 SEO 프롬프트

바이브코딩으로 SEO를 적용하는 프롬프트:

### 전체 페이지 SEO 일괄 적용

```
모든 페이지에 SEO를 강화해줘:

1. generateMetadata 확인/보완
   - title: "페이지명 | CompanyName" 형식
   - description: 각 페이지에 맞는 60~120자 설명
   - alternates.canonical: 각 페이지의 정규 URL
   - openGraph: title, description, url, siteName, images

2. src/app/sitemap.ts 생성
   - 정적 페이지 5개 + 동적 솔루션 상세 페이지
   - changeFrequency와 priority 적절히 설정

3. src/app/robots.ts 생성
   - 모든 검색 엔진 허용, /api/와 /_next/ 차단
   - sitemap 위치 명시

4. layout.tsx에 Organization JSON-LD 추가

5. 솔루션 상세에 Product JSON-LD, 채용에 JobPosting JSON-LD 추가
```

---

## 8-8. Google Search Console 등록

사이트맵과 SEO 설정이 완료되면 Google Search Console에 등록한다.

```
① https://search.google.com/search-console 접속

② "URL 접두어" 방식으로 속성 추가
   → https://www.company.com 입력

③ 소유권 인증 (권장: HTML 태그 방식)
   → 제공되는 meta 태그를 layout.tsx <head>에 추가:

   export const metadata: Metadata = {
     verification: {
       google: "인증코드_여기에",
     },
   };

④ 사이트맵 제출
   → "사이트맵" 메뉴 → sitemap.xml 입력 → 제출

⑤ 색인 생성 요청
   → "URL 검사" → 주요 페이지 URL 입력 → "색인 생성 요청"
```

### Search Console에서 모니터링할 항목

| 항목 | 확인 주기 | 의미 |
|------|:---------:|------|
| 색인 생성 상태 | 주 1회 | 페이지가 Google에 등록되었는지 |
| 검색 실적 | 주 1회 | 어떤 키워드로 검색되고, 클릭되는지 |
| 크롤링 오류 | 주 1회 | 404, 서버 에러 등 크롤링 문제 |
| Core Web Vitals | 월 1회 | 사용자 경험 지표 (LCP, FID, CLS) |
| 모바일 사용 편의성 | 월 1회 | 모바일 최적화 상태 |
