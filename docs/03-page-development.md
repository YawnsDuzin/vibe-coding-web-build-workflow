# Step 3: 핵심 페이지 개발 워크플로우

> **이 단계의 목표:** Claude Code에 줄 프롬프트 예시와 활용 팁을 정리하여 바이브코딩으로 각 페이지를 효율적으로 개발한다.

---

## 3-1. 개발 순서 전략

페이지를 아래 순서로 개발하면 의존성과 재사용성을 최대화할 수 있다.

```
① 전역 레이아웃 (Header + Footer)     — 모든 페이지의 공통 뼈대
  ↓
② 메인(홈) 페이지                      — 히어로 + 핵심 가치 + 솔루션 미리보기
  ↓
③ 솔루션 목록/상세 페이지               — MDX 콘텐츠 연동의 핵심
  ↓
④ 회사소개 페이지                      — 정적 콘텐츠 중심
  ↓
⑤ 채용 페이지                         — MDX 기반 채용 공고 목록
  ↓
⑥ 문의 페이지                         — 폼 + Server Action (유일한 인터랙티브 기능)
```

### 왜 이 순서인가?

| 순서 | 이유 |
|:----:|------|
| ①→② | 레이아웃을 먼저 만들어야 이후 페이지에서 일관된 헤더/푸터를 재사용할 수 있다 |
| ②→③ | 메인 페이지의 "솔루션 미리보기"와 솔루션 목록 페이지가 같은 컴포넌트를 공유한다 |
| ③→⑤ | 솔루션과 채용 모두 MDX 기반이므로, 솔루션에서 만든 `lib/mdx.ts` 유틸을 재사용한다 |
| ⑥ 마지막 | 폼 처리는 Server Action이 필요한 유일한 동적 기능이므로 마지막에 집중한다 |

---

## 3-2. 페이지별 프롬프트 예시

### 프롬프트 ①: 전역 레이아웃 (Header + Footer)

```
src/app/layout.tsx와 공통 컴포넌트를 만들어줘.

1. src/components/layout/Header.tsx
   - 로고(텍스트 "CompanyName") 왼쪽
   - 네비게이션 메뉴 가운데: 회사소개, 솔루션, 채용, 문의
   - CTA 버튼 "상담 신청" 오른쪽 (→ /contact 링크)
   - 스크롤 시 배경 white + shadow 효과 (sticky top-0)
   - 모바일: 햄버거 버튼 → 슬라이드 메뉴 (useState로 토글)
   - 현재 경로 표시: usePathname()으로 활성 메뉴 하이라이트

2. src/components/layout/Footer.tsx
   - 3열 구성: 회사 정보 | 페이지 링크 | 연락처
   - 하단에 © 2024 CompanyName. All rights reserved.
   - 모바일에서 1열 스택

3. src/app/layout.tsx
   - Pretendard 웹폰트 적용 (Google Fonts의 Noto Sans KR 대체 가능)
   - <html lang="ko">
   - 메타데이터: title "CompanyName", description "디지털 혁신의 파트너"
   - Header → {children} → Footer 순서로 배치

스타일: Tailwind CSS, 모바일 반응형
색상: primary-600(#2563EB), gray 계열 배경
```

> **왜 효과적인가:** 3개 파일을 한 프롬프트에서 요청하되, 각 파일의 구성 요소를 번호로 구분하여 AI가 누락 없이 생성한다. 모바일 동작(햄버거 메뉴)과 인터랙션(스크롤 효과)까지 구체적으로 명시했다.

---

### 프롬프트 ②: 메인(홈) 페이지

```
메인 페이지(src/app/page.tsx)를 만들어줘.

구성 (위에서 아래로):
1. 히어로 섹션
   - 헤드라인: "디지털 혁신의 파트너"
   - 서브텍스트: "AI와 클라우드 기술로 비즈니스 성장을 가속합니다" (1~2줄)
   - CTA 버튼 2개: "솔루션 보기"(→/solutions, primary), "문의하기"(→/contact, outline)
   - 오른쪽 또는 배경에 그라데이션 장식 (blue-500 → indigo-600)
   - 상하 패딩 충분히 (py-20 lg:py-32)

2. 핵심 가치 섹션
   - 섹션 제목: "왜 우리를 선택해야 할까요?"
   - 카드 3개 (아이콘 + 제목 + 설명 2줄):
     - 🔧 기술력: "최신 기술 스택으로 안정적인 솔루션 제공"
     - 🤝 신뢰성: "200+ 기업이 선택한 검증된 파트너"
     - 📈 확장성: "비즈니스 성장에 맞춰 유연하게 확장"
   - 그리드: 모바일 1열, md 3열

3. 솔루션 미리보기 섹션
   - 섹션 제목: "솔루션" + "전체 보기 →" 링크
   - content/solutions/ 폴더의 MDX에서 상위 4개를 읽어서 카드로 표시
   - 카드 구성: 아이콘 + 제목 + 설명 + "자세히 →" 링크
   - 그리드: 모바일 1열, sm 2열, lg 4열
   - lib/mdx.ts의 getAllSolutions() 함수 필요 (없으면 함께 생성)

4. CTA 배너 섹션
   - 배경: primary-600 (파란색)
   - 흰색 텍스트: "지금 상담을 시작하세요"
   - 흰색 CTA 버튼: "문의하기 →" (→ /contact)

스타일: Tailwind CSS, 모바일 반응형, 깔끔한 기업용 디자인
각 섹션 사이에 적절한 간격 (py-16 lg:py-24)
```

> **왜 효과적인가:** 4개 섹션을 번호와 구체적인 텍스트, 색상, 레이아웃까지 지정하여 AI의 추측 범위를 최소화한다. MDX 데이터 연동까지 범위에 포함시켜 한 번에 동작하는 페이지를 받는다.

---

### 프롬프트 ③: 솔루션 목록 + 상세 페이지

```
솔루션 관련 페이지 2개와 유틸 함수를 만들어줘.

1. src/lib/mdx.ts — MDX 유틸 함수
   - getAllSolutions(): content/solutions/*.mdx에서 모든 솔루션 목록 반환
     (frontmatter: title, description, icon, features[], order)
   - getSolutionBySlug(slug): 특정 솔루션 1개 반환 (frontmatter + MDX 본문)
   - getAllSolutionSlugs(): 전체 slug 목록 (generateStaticParams용)
   - gray-matter로 frontmatter 파싱, next-mdx-remote/serialize로 MDX 직렬화

2. src/types/content.ts — 타입 정의
   - SolutionFrontmatter 인터페이스
   - SolutionData 인터페이스 (frontmatter + slug + content)

3. src/app/solutions/page.tsx — 솔루션 목록 페이지
   - 페이지 제목: "솔루션"
   - 부제: "비즈니스 성장을 위한 맞춤 솔루션을 만나보세요"
   - 카드 그리드: 모바일 1열, sm 2열, lg 3열
   - 카드 클릭 → /solutions/[slug] 이동
   - order 필드로 정렬

4. src/app/solutions/[slug]/page.tsx — 솔루션 상세 페이지
   - generateStaticParams로 빌드 시 정적 생성
   - generateMetadata로 SEO 메타 태그 동적 생성
   - 레이아웃:
     - 상단: 아이콘 + 제목 + 설명 히어로
     - lg 이상: 왼쪽 2/3 MDX 본문 (prose 클래스) + 오른쪽 1/3 기능 목록 사이드바
     - lg 미만: 본문 → 기능 목록 순서로 스택
   - 하단: "다른 솔루션 보기" 카드 (현재 솔루션 제외, 최대 3개)
   - 뒤로가기: "← 솔루션 목록" 링크

content/solutions/ 에 샘플 MDX 파일 4개도 만들어줘:
- product-a.mdx: "클라우드 매니저" (클라우드 인프라 통합 관리)
- product-b.mdx: "데이터 인사이트" (실시간 데이터 분석 대시보드)
- product-c.mdx: "AI 어시스턴트" (업무 자동화 AI 솔루션)
- product-d.mdx: "시큐리티 가드" (통합 보안 모니터링)
```

> **왜 효과적인가:** 데이터 흐름(MDX 파일 → lib/mdx.ts → 페이지 컴포넌트)을 전체적으로 명시하고, 타입 정의·유틸·페이지·샘플 데이터까지 한 번에 요청하여 누락 없는 완성된 기능 단위를 받는다.

---

### 프롬프트 ④: 회사소개 페이지

```
src/app/about/page.tsx 회사소개 페이지를 만들어줘.

구성:
1. 히어로 영역
   - 제목: "회사소개"
   - 서브텍스트: "기술로 세상을 더 좋게 만듭니다"
   - 배경: 연한 그레이 (bg-gray-50)

2. 비전/미션 섹션
   - 2열 카드: 비전 "디지털 혁신의 글로벌 리더" | 미션 "기술로 기업의 잠재력을 극대화"
   - 각 카드에 아이콘과 2~3줄 설명

3. 회사 연혁 섹션
   - 타임라인 UI (세로 라인 + 연도별 이벤트)
   - 샘플 데이터 4개: 2020 창립, 2021 시리즈A, 2022 해외 진출, 2023 100인 돌파
   - 모바일에서도 보기 좋은 반응형 타임라인

4. 팀 소개 섹션
   - 섹션 제목: "팀을 만나보세요"
   - 4열 그리드 (모바일 2열): 이름 + 직책 + 한 줄 소개
   - 이미지는 placeholder(bg-gray-200 원형)
   - 샘플 4명: CEO, CTO, VP of Engineering, Head of Design

5. 숫자로 보는 회사 섹션
   - 가로 나열 카드 4개: 직원 수 "100+", 고객사 "200+", 프로젝트 "500+", 만족도 "98%"
   - 숫자는 크게 (text-4xl font-bold primary 색상), 라벨은 작게

generateMetadata 포함
```

> **왜 효과적인가:** 정적 콘텐츠 페이지도 섹션별 구조, 샘플 데이터, 반응형 동작을 상세히 명시하여 결과물의 완성도를 높인다.

---

### 프롬프트 ⑤: 채용 페이지

```
채용 페이지를 만들어줘.

1. src/lib/mdx.ts에 채용 관련 함수 추가
   - getAllCareers(): content/careers/*.mdx에서 전체 채용 공고 반환
   - frontmatter: title, team, location, type, isOpen

2. src/types/content.ts에 CareerFrontmatter 인터페이스 추가

3. src/app/careers/page.tsx
   - 히어로: 제목 "채용", 부제 "함께 성장할 동료를 찾습니다"
   - 회사 문화 섹션: 4개 카드 (유연근무, 교육 지원, 건강 관리, 팀 문화)
   - 채용 공고 목록:
     - isOpen: true인 공고만 표시
     - 각 공고: 제목 + 팀 + 위치 + 근무 형태 배지
     - 공고 없으면 "현재 열린 포지션이 없습니다" 메시지
   - 하단 CTA: "찾는 포지션이 없나요? careers@company.com으로 이력서를 보내주세요"

content/careers/ 에 샘플 MDX 2개:
- frontend-engineer.mdx: 프론트엔드 엔지니어 (개발팀, 서울, 정규직, isOpen: true)
- backend-engineer.mdx: 백엔드 엔지니어 (개발팀, 서울, 정규직, isOpen: true)
각 MDX 본문에 자격 요건, 우대 사항, 주요 업무를 작성해줘.
```

---

### 프롬프트 ⑥: 문의 페이지

```
src/app/contact/page.tsx에 문의 폼 페이지를 만들어줘.

1. 페이지 구성 (2열 레이아웃, lg 이상)
   왼쪽: 문의 폼
   오른쪽: 회사 연락처 정보 (이메일, 전화, 주소, 영업시간)
   모바일: 폼 → 연락처 순서로 스택

2. 문의 폼 (src/components/contact/ContactForm.tsx, 'use client')
   필드:
   - 이름 (text, 필수)
   - 이메일 (email, 필수, 형식 검증)
   - 회사명 (text, 선택)
   - 문의 유형 (select 드롭다운: 제품 문의 / 파트너십 / 채용 / 기타)
   - 메시지 (textarea, 필수, 최소 10자)

   기능:
   - 클라이언트 사이드 유효성 검사 (필수 필드 비어있으면 에러 메시지, 이메일 형식 체크)
   - useActionState(React 19)로 Server Action 연결
   - 제출 중 버튼 disabled + "전송 중..." 텍스트
   - 성공 시 초록색 알림: "문의가 접수되었습니다. 빠른 시일 내 답변 드리겠습니다."
   - 실패 시 빨간색 알림: "전송에 실패했습니다. 다시 시도해 주세요."
   - 성공 후 폼 초기화

3. Server Action (src/app/contact/actions.ts)
   - 'use server' 지시어
   - 서버 사이드 유효성 재검증
   - 현재는 console.log로 데이터 출력 (나중에 이메일 API 연결 가능하도록 구조화)
   - 반환: { success: boolean, message: string }

generateMetadata 포함
```

> **왜 효과적인가:** 폼 필드, 유효성 검사 규칙, 성공/실패 UX, Server Action 반환 타입까지 정확히 명시하여 프론트엔드와 백엔드가 통합된 결과물을 한 번에 받는다.

---

## 3-3. 프롬프트 작성 체크리스트

모든 페이지 프롬프트에 아래 항목을 포함하면 결과 품질이 높아진다.

```
□ 파일 경로 (src/app/about/page.tsx)
□ 섹션 구성 (번호 매기기)
□ 각 섹션의 구체적 텍스트/데이터
□ 반응형 동작 (모바일 → 데스크톱 열 수)
□ 색상 지정 (primary-600, gray-50 등)
□ 간격 지정 (py-16, gap-8 등)
□ 데이터 소스 (MDX 폴더 경로, frontmatter 필드)
□ 인터랙션 (클릭, 호버, 스크롤 동작)
□ 메타데이터 (generateMetadata 포함 여부)
□ 필요한 유틸 함수 (없으면 함께 생성 요청)
```

---

## 3-4. 레퍼런스 사이트 활용 방법

바이브코딩에서 디자인 레퍼런스를 활용하는 3가지 방법:

### 방법 1: URL 직접 참조

```
메인 페이지의 히어로 섹션을 만들어줘.
레이아웃 참고: https://stripe.com 의 히어로 섹션처럼
왼쪽에 텍스트+CTA, 오른쪽에 일러스트 영역으로 구성해줘.
```

Claude Code가 해당 사이트의 일반적인 레이아웃 패턴을 알고 있으므로, URL만으로도 방향을 잡을 수 있다. 단, AI가 실제로 URL에 접속하는 것은 아니므로 유명 사이트일수록 효과적이다.

### 방법 2: 스크린샷 첨부

```
첨부한 스크린샷과 비슷한 레이아웃의 솔루션 카드 그리드를 만들어줘.
- 카드 간격, 그림자, 라운딩을 유사하게
- 색상은 우리 primary 파란색 계열로
- 반응형: 모바일 1열, 태블릿 2열, 데스크톱 3열
```

Claude Code에 이미지 파일 경로를 넘기면 시각적 레이아웃을 분석하여 유사한 구조를 생성한다. 전체 페이지보다는 **특정 섹션의 스크린샷**을 주는 것이 더 정확한 결과를 낸다.

### 방법 3: 구체적 키워드 조합

```
Vercel 홈페이지 스타일의 미니멀한 네비게이션 바를 만들어줘.
- 로고 왼쪽, 메뉴 가운데, CTA 버튼 오른쪽
- 스크롤 시 배경 blur 효과 (backdrop-blur)
- 모바일에서 햄버거 메뉴
```

유명 사이트 이름 + "스타일" 키워드로 디자인 방향을 전달하면, AI가 해당 디자인 패턴을 적용한다.

### 방법별 비교

| 방법 | 정확도 | 편의성 | 적합한 상황 |
|------|:------:|:------:|------------|
| URL 참조 | ⭐⭐ | ⭐⭐⭐ | 유명 사이트의 전체적인 분위기를 참고할 때 |
| 스크린샷 | ⭐⭐⭐ | ⭐⭐ | 특정 섹션의 레이아웃을 정확히 재현하고 싶을 때 |
| 키워드 조합 | ⭐⭐ | ⭐⭐⭐ | 빠르게 디자인 방향을 전달하고 싶을 때 |

---

## 3-5. 개발 중 자주 쓰는 추가 프롬프트

### SEO 메타데이터 일괄 추가

```
모든 페이지에 generateMetadata를 추가해줘.
- title: "페이지명 | CompanyName" 형식
- description: 각 페이지에 맞는 설명 (60~120자)
- openGraph: title, description, url, siteName, images
- OG 이미지: /og-image.png (1200x630)
```

### 로딩/에러 상태 추가

```
아래 파일들을 만들어줘:
- src/app/loading.tsx: 스켈레톤 UI (pulse 애니메이션)
- src/app/error.tsx: 에러 페이지 ('use client', reset 버튼 포함)
- src/app/not-found.tsx: 404 페이지 (홈으로 돌아가기 링크)
```

### 접근성 개선

```
Header.tsx의 접근성을 개선해줘:
- 시맨틱 태그: <nav>, <header> 사용
- aria-label 추가: 네비게이션 "메인 메뉴"
- 모바일 메뉴: aria-expanded 상태 관리
- 키보드 내비게이션: Tab 이동, Enter/Space 토글
- 포커스 스타일: focus-visible:ring-2
```

> **다음 단계:** [Step 4: CI/CD 파이프라인](./04-cicd-pipeline.md)에서 자동 검증·배포 파이프라인을 구축한다.
