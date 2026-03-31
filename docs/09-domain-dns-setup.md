# Step 9: 도메인 & DNS 설정

> **이 단계의 목표:** 커스텀 도메인을 구매하고, Vercel에 연결하여 `https://www.company.com`으로 사이트를 운영한다.

---

## 9-1. 전체 흐름 개요

```
① 도메인 구매 (레지스트라에서)
   ↓
② Vercel에 도메인 추가
   ↓
③ DNS 레코드 설정 (레지스트라 또는 Vercel DNS)
   ↓
④ SSL 인증서 자동 발급 (Vercel이 처리)
   ↓
⑤ 정상 작동 확인
```

---

## 9-2. 도메인 구매

### 레지스트라(도메인 판매처) 비교

| 레지스트라 | .com 가격 (연) | 특징 | 추천 |
|-----------|:-------------:|------|:----:|
| **Namecheap** | ~$9 | 가격 대비 좋은 UI, WhoisGuard 무료 | ✅ |
| Cloudflare Registrar | ~$10 | 원가 판매, DNS 통합 관리 | ✅ |
| Google Domains → Squarespace | ~$12 | Google에서 이관됨 | |
| GoDaddy | ~$12 (갱신 $20+) | 초기 가격은 저렴하나 갱신 비용이 높음 | |
| 가비아 (.kr 도메인) | ₩16,500~ | 한국 .kr, .co.kr 도메인 구매 시 | ✅ (.kr) |

### 도메인 선택 팁

```
✅ 권장:
   - company.com (가장 보편적, 신뢰성 높음)
   - company.co.kr (한국 기업임을 강조)

⚠️ 피할 것:
   - 너무 긴 도메인 (digital-transformation-solutions.com)
   - 하이픈이 많은 도메인 (my-company-name.com)
   - 비주류 TLD (.xyz, .io는 기업 홈페이지에 부적합할 수 있음)
```

---

## 9-3. Vercel에 도메인 추가

### 방법 A: Vercel 대시보드에서 설정 (권장)

```
① Vercel 대시보드 접속 → 프로젝트 선택

② Settings → Domains 클릭

③ 도메인 입력 → "Add" 클릭
   - company.com 입력
   - www.company.com도 추가 (리다이렉트 설정)

④ Vercel이 필요한 DNS 레코드를 안내해줌
```

### 방법 B: Vercel CLI로 설정

```bash
# 도메인 추가
vercel domains add company.com

# www 서브도메인 추가
vercel domains add www.company.com

# 도메인 목록 확인
vercel domains ls
```

---

## 9-4. DNS 레코드 설정

Vercel에 도메인을 추가하면, 아래 DNS 레코드를 레지스트라(또는 DNS 관리 서비스)에 설정해야 한다.

### 옵션 1: Vercel DNS 사용 (가장 간편)

Vercel을 네임서버로 사용하면 DNS 레코드가 자동 관리된다.

```
레지스트라의 네임서버 설정에서 다음으로 변경:

ns1.vercel-dns.com
ns2.vercel-dns.com
```

> 네임서버 변경 후 전파에 최대 48시간 소요될 수 있다 (보통 1~2시간).

### 옵션 2: 외부 DNS 유지 (레지스트라/Cloudflare)

기존 DNS를 유지하면서 Vercel로 연결하는 경우:

#### Apex 도메인 (company.com)

| 레코드 타입 | 호스트 | 값 | TTL |
|:----------:|:------:|:--:|:---:|
| A | @ | `76.76.21.21` | 300 |

#### www 서브도메인 (www.company.com)

| 레코드 타입 | 호스트 | 값 | TTL |
|:----------:|:------:|:--:|:---:|
| CNAME | www | `cname.vercel-dns.com` | 300 |

### DNS 레코드 설정 방법 (레지스트라별)

#### Namecheap

```
1. Dashboard → Domain List → Manage 클릭
2. Advanced DNS 탭
3. "ADD NEW RECORD" 클릭
4. 위 표의 레코드 추가
5. 기존 A 레코드나 CNAME이 있으면 삭제 후 추가
```

#### Cloudflare

```
1. 대시보드 → 도메인 선택 → DNS
2. "Add record" 클릭
3. 위 표의 레코드 추가
4. Proxy status(주황색 구름): OFF로 설정
   → Vercel의 SSL과 충돌 방지
```

#### 가비아 (.kr 도메인)

```
1. My가비아 → 도메인 관리 → DNS 관리
2. 레코드 추가
3. 위 표의 레코드 추가
```

---

## 9-5. www vs non-www 리다이렉트 설정

하나의 정규 URL을 정하고, 나머지는 리다이렉트한다.

### Vercel 대시보드에서 설정

```
Settings → Domains에서:

방법 1: www → non-www (company.com이 정규 URL)
  company.com        → Primary
  www.company.com    → Redirects to company.com (308)

방법 2: non-www → www (www.company.com이 정규 URL) [권장]
  www.company.com    → Primary
  company.com        → Redirects to www.company.com (308)
```

### 어느 것을 선택할까?

| 옵션 | 장점 | 추천 |
|------|------|:----:|
| `www.company.com` | CDN/DNS 레벨 유연성, 쿠키 분리 용이 | ✅ 기업 사이트 |
| `company.com` | URL이 더 짧고 모던함 | 스타트업/개인 |

---

## 9-6. SSL 인증서

### Vercel 자동 SSL

Vercel은 도메인이 연결되면 **자동으로 Let's Encrypt SSL 인증서를 발급**한다.

```
별도 설정 불필요:
  ✅ HTTPS 자동 활성화
  ✅ HTTP → HTTPS 자동 리다이렉트
  ✅ 인증서 자동 갱신 (90일마다)
  ✅ 와일드카드 인증서 지원
```

### SSL 발급 실패 시

| 원인 | 해결 |
|------|------|
| DNS 레코드 미전파 | 24~48시간 대기 후 재확인 |
| DNS 설정 오류 | A 레코드/CNAME 값 재확인 |
| Cloudflare Proxy 활성화 | Proxy status를 OFF(DNS only)로 변경 |
| CAA 레코드 충돌 | CAA에 `letsencrypt.org` 허용 추가 |

---

## 9-7. 확인 방법

### DNS 전파 확인

```bash
# A 레코드 확인
dig company.com A +short
# 예상 출력: 76.76.21.21

# CNAME 확인
dig www.company.com CNAME +short
# 예상 출력: cname.vercel-dns.com.

# 또는 온라인 도구 사용:
# https://dnschecker.org — 글로벌 DNS 전파 상태 확인
```

### HTTPS 및 리다이렉트 확인

```bash
# HTTPS 정상 동작 확인
curl -I https://www.company.com
# HTTP/2 200 ← 정상

# HTTP → HTTPS 리다이렉트 확인
curl -I http://www.company.com
# HTTP/1.1 308 Permanent Redirect
# Location: https://www.company.com/ ← 정상

# www 리다이렉트 확인 (non-www → www인 경우)
curl -I https://company.com
# HTTP/2 308
# Location: https://www.company.com/ ← 정상
```

### 전체 체크리스트

| # | 확인 항목 | 방법 | 기대 결과 |
|:-:|----------|------|-----------|
| 1 | DNS 전파 | `dig company.com A` | Vercel IP (76.76.21.21) |
| 2 | HTTPS 접속 | 브라우저에서 접속 | 🔒 자물쇠 + 사이트 정상 표시 |
| 3 | HTTP 리다이렉트 | `curl -I http://...` | 308 → https:// |
| 4 | www 리다이렉트 | `curl -I https://company.com` | 308 → www 또는 반대 |
| 5 | SSL 인증서 | 브라우저 자물쇠 클릭 | Let's Encrypt 발급, 유효기간 확인 |
| 6 | 전체 페이지 접속 | 모든 경로 직접 테스트 | 404 없음 |
| 7 | 프리뷰 배포 | PR 생성 후 프리뷰 URL 확인 | 커스텀 도메인과 별도로 정상 동작 |

---

## 9-8. 도메인 설정 후 업데이트할 항목

도메인이 확정되면 프로젝트에서 아래 항목을 업데이트한다.

### 환경변수

```bash
# .env.local 및 Vercel 환경변수 업데이트
NEXT_PUBLIC_SITE_URL=https://www.company.com
```

### 메타데이터

```typescript
// src/app/layout.tsx
export const metadata: Metadata = {
  metadataBase: new URL("https://www.company.com"),
  // ...
};
```

### sitemap.ts

```typescript
// BASE_URL이 환경변수에서 읽히므로 자동 반영
const BASE_URL = process.env.NEXT_PUBLIC_SITE_URL;
```

### Google Search Console

```
Search Console에서 속성 추가:
→ https://www.company.com
→ 사이트맵 재제출
```

### 바이브코딩 프롬프트

```
도메인이 www.company.com으로 확정되었어.
아래 파일들의 URL을 모두 업데이트해줘:
- .env.example의 NEXT_PUBLIC_SITE_URL
- layout.tsx의 metadataBase
- JSON-LD의 url/sameAs 필드
- OG 이미지 절대 경로
```

---

## 9-9. 트러블슈팅

| 증상 | 원인 | 해결 |
|------|------|------|
| "DNS_PROBE_FINISHED_NXDOMAIN" | DNS 미설정 또는 미전파 | DNS 레코드 확인, 48시간 대기 |
| "ERR_SSL_VERSION_OR_CIPHER_MISMATCH" | SSL 미발급 | Vercel 대시보드에서 SSL 상태 확인, DNS 전파 후 재시도 |
| 무한 리다이렉트 루프 | Cloudflare Flexible SSL + Vercel HTTPS 충돌 | Cloudflare SSL을 "Full (strict)"로 변경 또는 Proxy OFF |
| 프리뷰 URL은 되는데 커스텀 도메인 안 됨 | DNS 설정 오류 | A 레코드/CNAME 값 재확인 |
| "도메인이 다른 Vercel 프로젝트에 연결됨" | 이전 프로젝트에 도메인 등록 | 이전 프로젝트에서 도메인 제거 후 재추가 |
