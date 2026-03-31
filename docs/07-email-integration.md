# Step 7: 문의 폼 이메일 연동

> **이 단계의 목표:** 문의 폼에서 제출된 데이터를 실제 이메일로 발송하도록 Resend API를 연동한다.

---

## 7-1. 이메일 서비스 선택

### 후보 비교

| 서비스 | 무료 티어 | Next.js 통합 | 설정 난이도 | 바이브코딩 친화성 |
|--------|----------|:------------:|:---------:|:---------------:|
| **Resend** | 월 3,000건 | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| SendGrid | 월 100건/일 | ⭐⭐ | ⭐⭐ | ⭐⭐ |
| AWS SES | 월 3,000건 (EC2) | ⭐⭐ | ⭐ | ⭐⭐ |
| Nodemailer + Gmail | 일 500건 | ⭐⭐ | ⭐ | ⭐⭐ |

### ✅ 선정: Resend

- **무료 티어:** 월 3,000건 (문의 폼 용도로 충분)
- **설정이 매우 간단:** API 키 하나로 즉시 사용 가능
- **Next.js 공식 지원:** Server Action에서 바로 호출할 수 있는 SDK 제공
- **React Email 호환:** React 컴포넌트로 이메일 템플릿을 작성할 수 있음

---

## 7-2. Resend 가입 및 API 키 발급

```
① https://resend.com 접속 → "Sign Up" 클릭

② GitHub 또는 이메일로 계정 생성

③ 대시보드 → "API Keys" 메뉴 클릭

④ "Create API Key" 클릭
   - Name: "company-homepage"
   - Permission: "Sending access"
   - Domain: "All domains" (초기 설정)
   → 생성된 키 복사 (re_xxxxxxxxx 형식)

⑤ (선택) 커스텀 도메인 인증
   - "Domains" 메뉴 → "Add Domain"
   - company.com 입력
   - 표시되는 DNS 레코드(SPF, DKIM)를 도메인 DNS에 추가
   - 인증 완료 후 @company.com 발신 주소 사용 가능
```

---

## 7-3. 의존성 설치

```bash
# Resend SDK 설치
pnpm add resend

# (선택) React Email — 이메일 템플릿을 React로 작성할 때
pnpm add @react-email/components
```

---

## 7-4. 환경변수 설정

### .env.example에 추가

```bash
# 이메일 발송 (Resend)
RESEND_API_KEY=re_xxxxxxxxxxxxxxxxxxxxxxxxxxxx

# 문의 수신 이메일
CONTACT_EMAIL_TO=info@company.com

# 문의 발신 이메일 (Resend에서 인증된 도메인)
# 도메인 인증 전: onboarding@resend.dev 사용
CONTACT_EMAIL_FROM=noreply@company.com
```

### .env.local에 실제 값 입력

```bash
RESEND_API_KEY=re_실제_API_키
CONTACT_EMAIL_TO=info@company.com
CONTACT_EMAIL_FROM=onboarding@resend.dev
```

### Vercel 환경변수에도 등록

```
Vercel 대시보드 → 프로젝트 → Settings → Environment Variables
→ 위 3개 변수를 Production/Preview 환경에 추가
```

---

## 7-5. Server Action 구현

### 파일: `src/app/contact/actions.ts`

```typescript
"use server";

import { Resend } from "resend";

// Resend 인스턴스 생성
const resend = new Resend(process.env.RESEND_API_KEY);

// 폼 데이터 타입
interface ContactFormData {
  name: string;
  email: string;
  company?: string;
  inquiryType: string;
  message: string;
}

// 응답 타입
interface ActionResult {
  success: boolean;
  message: string;
}

// 서버 사이드 유효성 검사
function validateFormData(data: ContactFormData): string | null {
  if (!data.name || data.name.trim().length === 0) {
    return "이름을 입력해주세요.";
  }
  if (!data.email || !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(data.email)) {
    return "올바른 이메일 주소를 입력해주세요.";
  }
  if (!data.message || data.message.trim().length < 10) {
    return "메시지를 10자 이상 입력해주세요.";
  }
  return null;
}

export async function submitContactForm(
  _prevState: ActionResult | null,
  formData: FormData
): Promise<ActionResult> {
  // 1. 폼 데이터 추출
  const data: ContactFormData = {
    name: formData.get("name") as string,
    email: formData.get("email") as string,
    company: (formData.get("company") as string) || "",
    inquiryType: formData.get("inquiryType") as string,
    message: formData.get("message") as string,
  };

  // 2. 서버 사이드 유효성 검사
  const validationError = validateFormData(data);
  if (validationError) {
    return { success: false, message: validationError };
  }

  // 3. 이메일 발송
  try {
    await resend.emails.send({
      from: process.env.CONTACT_EMAIL_FROM || "onboarding@resend.dev",
      to: process.env.CONTACT_EMAIL_TO || "info@company.com",
      replyTo: data.email,
      subject: `[홈페이지 문의] ${data.inquiryType} - ${data.name}`,
      html: `
        <h2>새로운 문의가 접수되었습니다</h2>
        <table style="border-collapse: collapse; width: 100%; max-width: 600px;">
          <tr>
            <td style="padding: 8px; border: 1px solid #ddd; font-weight: bold;">이름</td>
            <td style="padding: 8px; border: 1px solid #ddd;">${data.name}</td>
          </tr>
          <tr>
            <td style="padding: 8px; border: 1px solid #ddd; font-weight: bold;">이메일</td>
            <td style="padding: 8px; border: 1px solid #ddd;">${data.email}</td>
          </tr>
          <tr>
            <td style="padding: 8px; border: 1px solid #ddd; font-weight: bold;">회사명</td>
            <td style="padding: 8px; border: 1px solid #ddd;">${data.company || "-"}</td>
          </tr>
          <tr>
            <td style="padding: 8px; border: 1px solid #ddd; font-weight: bold;">문의 유형</td>
            <td style="padding: 8px; border: 1px solid #ddd;">${data.inquiryType}</td>
          </tr>
          <tr>
            <td style="padding: 8px; border: 1px solid #ddd; font-weight: bold;">메시지</td>
            <td style="padding: 8px; border: 1px solid #ddd; white-space: pre-wrap;">${data.message}</td>
          </tr>
        </table>
        <p style="color: #666; font-size: 12px; margin-top: 16px;">
          이 메일은 홈페이지 문의 폼에서 자동 발송되었습니다.
          회신은 ${data.email}로 보내주세요.
        </p>
      `,
    });

    return {
      success: true,
      message: "문의가 접수되었습니다. 빠른 시일 내 답변 드리겠습니다.",
    };
  } catch (error) {
    console.error("이메일 발송 실패:", error);
    return {
      success: false,
      message: "전송에 실패했습니다. 잠시 후 다시 시도해 주세요.",
    };
  }
}
```

### 코드 흐름 설명

```
클라이언트 (ContactForm.tsx)
  ↓ useActionState로 Server Action 호출
Server Action (actions.ts)
  ↓ ① FormData에서 필드 추출
  ↓ ② 서버 사이드 유효성 검사
  ↓ ③ Resend API로 이메일 발송
  ↓ ④ 결과 반환 { success, message }
클라이언트
  ↓ 성공/실패 메시지 표시
```

---

## 7-6. (선택) React Email 템플릿

HTML 문자열 대신 React 컴포넌트로 이메일 템플릿을 작성할 수 있다.

### 파일: `src/lib/emails/contact-notification.tsx`

```tsx
import {
  Html,
  Head,
  Body,
  Container,
  Section,
  Text,
  Heading,
  Hr,
} from "@react-email/components";

interface ContactNotificationProps {
  name: string;
  email: string;
  company?: string;
  inquiryType: string;
  message: string;
}

export default function ContactNotification({
  name,
  email,
  company,
  inquiryType,
  message,
}: ContactNotificationProps) {
  return (
    <Html lang="ko">
      <Head />
      <Body style={{ fontFamily: "sans-serif", backgroundColor: "#f9fafb" }}>
        <Container style={{ maxWidth: "600px", margin: "0 auto", padding: "20px" }}>
          <Heading as="h2">새로운 문의가 접수되었습니다</Heading>
          <Section style={{ backgroundColor: "#fff", padding: "20px", borderRadius: "8px" }}>
            <Text><strong>이름:</strong> {name}</Text>
            <Text><strong>이메일:</strong> {email}</Text>
            <Text><strong>회사명:</strong> {company || "-"}</Text>
            <Text><strong>문의 유형:</strong> {inquiryType}</Text>
            <Hr />
            <Text><strong>메시지:</strong></Text>
            <Text style={{ whiteSpace: "pre-wrap" }}>{message}</Text>
          </Section>
          <Text style={{ color: "#666", fontSize: "12px" }}>
            이 메일은 홈페이지 문의 폼에서 자동 발송되었습니다.
          </Text>
        </Container>
      </Body>
    </Html>
  );
}
```

### Server Action에서 React Email 사용

```typescript
import ContactNotification from "@/lib/emails/contact-notification";

// resend.emails.send 호출 시 html 대신 react 속성 사용:
await resend.emails.send({
  from: process.env.CONTACT_EMAIL_FROM || "onboarding@resend.dev",
  to: process.env.CONTACT_EMAIL_TO || "info@company.com",
  replyTo: data.email,
  subject: `[홈페이지 문의] ${data.inquiryType} - ${data.name}`,
  react: ContactNotification({
    name: data.name,
    email: data.email,
    company: data.company,
    inquiryType: data.inquiryType,
    message: data.message,
  }),
});
```

---

## 7-7. 테스트 방법

### 로컬 테스트

```bash
# 1. .env.local에 Resend API 키 설정 확인
cat .env.local | grep RESEND

# 2. 개발 서버 실행
pnpm dev

# 3. http://localhost:3000/contact 접속

# 4. 테스트 데이터로 폼 제출
#    이름: 테스트
#    이메일: test@example.com
#    메시지: 테스트 문의입니다.

# 5. CONTACT_EMAIL_TO 이메일 수신함 확인
```

### Resend 대시보드에서 확인

```
https://resend.com/emails 접속
→ 발송 이력, 성공/실패 상태, 열람 여부 확인 가능
```

### 주의사항

| 항목 | 설명 |
|------|------|
| 도메인 미인증 시 | `onboarding@resend.dev`에서만 발송 가능. 테스트용으로는 충분 |
| 무료 티어 한도 | 월 3,000건. 초과 시 429 에러 발생 |
| replyTo 설정 | 수신자가 이메일에서 "답장" 클릭 시 문의자 이메일로 바로 회신 |
| 스팸 방지 | 프로덕션에서는 반드시 커스텀 도메인 인증 후 사용 권장 |

---

## 7-8. 트러블슈팅

| 에러 | 원인 | 해결 |
|------|------|------|
| `Missing API key` | RESEND_API_KEY 미설정 | .env.local 및 Vercel 환경변수 확인 |
| `Validation error: from` | 미인증 도메인에서 발송 시도 | `onboarding@resend.dev` 사용 또는 도메인 인증 |
| `Rate limit exceeded` | 무료 티어 한도 초과 | Resend 대시보드에서 사용량 확인, 다음 달 초기화 대기 |
| 이메일이 스팸함으로 감 | 도메인 인증 미완료 | SPF, DKIM DNS 레코드 추가 후 도메인 인증 |
| 로컬에서는 되는데 배포 후 안 됨 | Vercel 환경변수 누락 | Vercel Settings → Environment Variables 확인 |
