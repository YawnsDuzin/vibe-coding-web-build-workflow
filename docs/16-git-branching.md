# Step 16: Git 브랜칭 전략

> **이 단계의 목표:** 1인 개발 + 바이브코딩 환경에 맞는 Git 브랜칭 규칙을 정의하여 안전한 코드 관리와 배포를 보장한다.

---

## 16-1. 브랜칭 모델 선택

### 후보 비교

| 모델 | 복잡도 | 적합한 팀 규모 | 바이브코딩 적합성 |
|------|:-----:|:------------:|:---------------:|
| **GitHub Flow** | ⭐ | 1~5명 | ⭐⭐⭐ |
| Git Flow | ⭐⭐⭐ | 5명+ | ⭐ |
| Trunk-Based | ⭐ | 1~3명 | ⭐⭐ |

### ✅ 선정: GitHub Flow (간소화 버전)

```
main (프로덕션)
  │
  ├── feature/add-blog-page        ← 기능 브랜치
  ├── fix/contact-form-validation   ← 버그 수정 브랜치
  ├── content/update-solution-desc  ← 콘텐츠 수정 브랜치
  └── hotfix/fix-500-error          ← 긴급 수정 브랜치
```

> **원칙:** `main`은 항상 배포 가능한 상태를 유지한다. 모든 변경은 브랜치 → PR → 머지로 진행한다.

---

## 16-2. 브랜치 네이밍 규칙

### 형식

```
{타입}/{간단한-설명}
```

### 타입별 규칙

| 타입 | 용도 | 예시 |
|------|------|------|
| `feature/` | 새 기능 추가 | `feature/add-blog-page` |
| `fix/` | 버그 수정 | `fix/contact-form-validation` |
| `content/` | 콘텐츠만 수정 | `content/update-product-a-desc` |
| `style/` | 디자인/스타일 변경 | `style/update-hero-gradient` |
| `refactor/` | 기능 변경 없는 코드 개선 | `refactor/extract-mdx-utils` |
| `hotfix/` | 긴급 프로덕션 수정 | `hotfix/fix-500-on-solutions` |
| `chore/` | 설정/의존성 변경 | `chore/upgrade-next-15` |

### 네이밍 규칙

```
✅ 좋은 예:
   feature/add-career-detail-page
   fix/mobile-menu-not-closing
   content/update-q4-careers

❌ 나쁜 예:
   my-branch              ← 타입 없음
   feature/update         ← 설명이 너무 모호
   Feature/AddBlogPage    ← 대문자, camelCase 사용
   fix/bug                ← 어떤 버그인지 알 수 없음
```

---

## 16-3. 커밋 메시지 컨벤션

### Conventional Commits 형식

```
{타입}: {변경 내용 요약}

(선택) 상세 설명
```

### 커밋 타입

| 타입 | 용도 | 예시 |
|------|------|------|
| `feat` | 새 기능 | `feat: 블로그 목록 페이지 추가` |
| `fix` | 버그 수정 | `fix: 문의 폼 이메일 유효성 검사 오류` |
| `content` | 콘텐츠 변경 | `content: 클라우드 매니저 설명 업데이트` |
| `style` | 스타일 변경 | `style: 히어로 섹션 그라데이션 변경` |
| `refactor` | 리팩토링 | `refactor: MDX 파싱 유틸 함수 분리` |
| `chore` | 설정/도구 | `chore: ESLint 규칙 추가` |
| `docs` | 문서 | `docs: README 업데이트` |
| `test` | 테스트 | `test: 솔루션 페이지 E2E 테스트 추가` |
| `ci` | CI/CD | `ci: Lighthouse CI 추가` |

### 좋은 커밋 메시지 예시

```bash
# ✅ 좋은 예 — 무엇을 왜 변경했는지 알 수 있음
git commit -m "feat: 솔루션 상세 페이지에 관련 솔루션 추천 섹션 추가"
git commit -m "fix: 모바일에서 햄버거 메뉴 클릭 시 닫히지 않는 문제 수정"
git commit -m "content: 2024년 Q4 채용 공고 3건 추가"

# ❌ 나쁜 예 — 정보가 부족함
git commit -m "update"
git commit -m "fix bug"
git commit -m "작업 완료"
```

---

## 16-4. 워크플로우: 일반 기능 개발

```bash
# ① main 최신 상태로 업데이트
git checkout main
git pull origin main

# ② 기능 브랜치 생성
git checkout -b feature/add-blog-page

# ③ 바이브코딩으로 개발
# Claude Code에게 프롬프트 → 코드 생성 → 확인 → 반복

# ④ 변경사항 커밋 (작업 단위별로 나누어 커밋)
git add src/app/blog/page.tsx src/components/blog/
git commit -m "feat: 블로그 목록 페이지 및 카드 컴포넌트 추가"

git add content/blog/
git commit -m "content: 샘플 블로그 포스트 3개 추가"

# ⑤ 원격에 push
git push -u origin feature/add-blog-page

# ⑥ GitHub에서 PR 생성
#    제목: "feat: 블로그 기능 추가"
#    본문: 변경 내용 요약, 스크린샷

# ⑦ CI 통과 + 프리뷰 URL 확인

# ⑧ PR 머지 (Squash and merge 권장)

# ⑨ 로컬 정리
git checkout main
git pull origin main
git branch -d feature/add-blog-page
```

---

## 16-5. 워크플로우: 긴급 핫픽스

```bash
# ① main에서 직접 핫픽스 브랜치 생성
git checkout main
git pull origin main
git checkout -b hotfix/fix-500-on-contact

# ② 최소한의 수정만 진행
# "src/app/contact/actions.ts에서 500 에러 발생해. 수정해줘: [에러 메시지]"

# ③ 커밋
git add src/app/contact/actions.ts
git commit -m "hotfix: 문의 폼 Server Action 500 에러 수정"

# ④ push + PR 생성 + 빠르게 머지
git push -u origin hotfix/fix-500-on-contact
# PR 생성 → CI 통과 확인 → 즉시 머지

# ⑤ (선택) CI 대기가 급하면 Vercel Instant Rollback 먼저 실행
```

---

## 16-6. 워크플로우: 비개발자 콘텐츠 수정

```
비개발자는 GitHub 웹 UI에서:
  ① content/ 파일 편집
  ② "Create a new branch" 선택
  ③ 자동으로 content/update-xxx 형태의 브랜치 생성
  ④ PR 생성 → CI + 프리뷰 확인
  ⑤ 개발자가 승인 → 머지
```

> 비개발자에게는 브랜치 이름을 직접 정하라고 요구하지 않는다. GitHub 웹 UI가 자동으로 생성하는 이름을 사용해도 무방하다.

---

## 16-7. PR 템플릿

### 파일: `.github/pull_request_template.md`

```markdown
## 변경 내용
<!-- 어떤 변경이 포함되어 있나요? -->
- 

## 변경 이유
<!-- 왜 이 변경이 필요한가요? -->
- 

## 확인 사항
<!-- 체크리스트를 완료해주세요 -->
- [ ] 로컬에서 `pnpm build` 성공
- [ ] 모바일 반응형 확인
- [ ] 프리뷰 URL에서 변경 내용 확인

## 스크린샷 (선택)
<!-- UI 변경이 있으면 스크린샷을 첨부해주세요 -->
```

---

## 16-8. PR 머지 전략

### Squash and Merge (권장)

```
기능 브랜치의 여러 커밋을 하나의 깔끔한 커밋으로 합쳐서 main에 머지한다.

장점:
  ✅ main 브랜치의 커밋 히스토리가 깔끔
  ✅ 기능별로 하나의 커밋이므로 롤백이 쉬움
  ✅ 바이브코딩 시 여러 시행착오 커밋이 합쳐짐

설정:
  GitHub → Settings → General → Pull Requests
  → "Allow squash merging" ✅
  → "Allow merge commits" ✅ (필요 시)
  → "Allow rebase merging" ❌ (비활성화 권장)
  → Default: "Squash and merge"
```

### Squash 커밋 메시지 형식

```
feat: 블로그 기능 추가 (#42)

- 블로그 목록 페이지 (/blog)
- 블로그 상세 페이지 (/blog/[slug])
- MDX 기반 블로그 포스트 3개
- E2E 테스트 추가
```

---

## 16-9. 브랜치 보호 규칙 요약

[Step 4](./04-cicd-pipeline.md)에서 설정한 브랜치 보호 규칙과 함께:

```
GitHub → Settings → Branches → main 보호 규칙:

✅ Require a pull request before merging
   → 직접 push 차단, PR 필수

✅ Require status checks to pass
   → CI(lint + build + test) 통과 필수

✅ Require branches to be up to date
   → main과 동기화 후 머지

✅ Automatically delete head branches
   → 머지 후 기능 브랜치 자동 삭제 (정리)
```

---

## 16-10. CLAUDE.md에 추가할 Git 규칙

```markdown
## Git 규칙
- 브랜치: {타입}/{설명} 형식 (feature/, fix/, content/, hotfix/)
- 커밋: Conventional Commits ({타입}: {설명})
- main 브랜치에 직접 push 금지 → PR 필수
- PR 머지: Squash and merge
- 바이브코딩 시 작업 단위별로 커밋 분리 (한 커밋에 너무 많은 변경 넣지 않기)
```
