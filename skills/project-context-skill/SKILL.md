---
name: project-context-skill
description: 프로젝트 기술 스택을 분석하여 맞춤형 컨텍스트와 코드 패턴 가이드를 생성합니다. 프로젝트별 .claude/project-context.md와 .claude/project-patterns.md를 생성하여 일관된 코드 품질을 유지합니다.
version: "1.0.0"
---

# Project Context Skill

프로젝트의 기술 스택을 자동 분석하여 해당 프로젝트에 맞는 컨텍스트 문서와 코드 패턴 가이드를 생성합니다.

## 명령어

| 명령어 | 동작 |
|-------|------|
| `컨텍스트 생성` / `프로젝트 컨텍스트 생성` | 최초 생성 |
| `컨텍스트 재생성` / `프로젝트 컨텍스트 재생성` | 삭제 후 새로 생성 |

---

## 생성 파일

### 1. `.claude/project-context.md`
프로젝트 메타 정보 (기술 스택, 구조, 컨벤션)

### 2. `.claude/project-patterns.md`
해당 프로젝트에서 사용하는 기술의 코드 패턴 가이드

---

## 분석 대상

### 기술 스택 감지

```
package.json 분석:
├── dependencies
├── devDependencies
└── scripts

추가 파일 분석:
├── next.config.js/mjs    → Next.js
├── vite.config.ts        → Vite
├── tsconfig.json         → TypeScript
├── tailwind.config.js    → Tailwind CSS
├── .eslintrc.*           → ESLint 설정
├── prisma/schema.prisma  → Prisma
└── supabase/             → Supabase
```

### 감지 가능한 기술

| 카테고리 | 기술 | 감지 방법 |
|---------|------|----------|
| **Framework** | Next.js | `next` in deps |
| | Vite | `vite` in deps |
| | CRA | `react-scripts` in deps |
| **Language** | TypeScript | `typescript` in deps or tsconfig.json |
| **Styling** | Tailwind CSS | `tailwindcss` in deps |
| | Shadcn/ui | `components/ui/` 폴더 존재 |
| **State** | Zustand | `zustand` in deps |
| | Jotai | `jotai` in deps |
| | Redux Toolkit | `@reduxjs/toolkit` in deps |
| | Recoil | `recoil` in deps |
| **Data Fetching** | TanStack Query | `@tanstack/react-query` in deps |
| | SWR | `swr` in deps |
| **Forms** | React Hook Form | `react-hook-form` in deps |
| | Zod | `zod` in deps |
| **Database** | Prisma | `prisma` in deps |
| | Supabase | `@supabase/supabase-js` in deps |
| | Drizzle | `drizzle-orm` in deps |
| **Auth** | NextAuth | `next-auth` in deps |
| | Clerk | `@clerk/nextjs` in deps |
| **Testing** | Vitest | `vitest` in deps |
| | Jest | `jest` in deps |
| | Playwright | `@playwright/test` in deps |

---

## 실행 프로세스

### 컨텍스트 생성

```
1. 프로젝트 루트 확인
   - package.json 존재 여부
   - 기존 .claude/ 폴더 확인

2. 기술 스택 분석
   - package.json 파싱
   - 설정 파일 확인
   - 폴더 구조 분석

3. 프로젝트 구조 분석
   - src/ or app/ 구조
   - components/, hooks/, lib/ 등
   - 네이밍 컨벤션 추론

4. 파일 생성
   - .claude/project-context.md
   - .claude/project-patterns.md

5. 결과 요약 출력
```

### 컨텍스트 재생성

```
1. 기존 파일 삭제
   - .claude/project-context.md
   - .claude/project-patterns.md

2. "컨텍스트 생성" 프로세스 실행
```

---

## 생성 템플릿

### project-context.md 템플릿

```markdown
# Project Context

> 자동 생성됨 | 마지막 업데이트: {timestamp}

## 기술 스택

### Core
- **Framework**: {framework} {version}
- **Language**: {language} {version}
- **Runtime**: {runtime}

### UI & Styling
- **CSS Framework**: {css_framework}
- **Component Library**: {component_library}
- **Icons**: {icons}

### State Management
- **Client State**: {client_state}
- **Server State**: {server_state}

### Forms & Validation
- **Form Library**: {form_library}
- **Validation**: {validation}

### Backend & Database
- **Database**: {database}
- **ORM**: {orm}
- **Auth**: {auth}

### Testing
- **Unit/Integration**: {test_framework}
- **E2E**: {e2e_framework}

---

## 프로젝트 구조

```
{detected_structure}
```

---

## 코드 컨벤션

### 네이밍
- **컴포넌트**: {component_naming}
- **훅**: {hook_naming}
- **유틸리티**: {util_naming}
- **타입**: {type_naming}

### 파일 구조
- **컴포넌트 위치**: {component_location}
- **훅 위치**: {hook_location}
- **타입 위치**: {type_location}

---

## 환경 변수

| 변수명 | 용도 | 필수 |
|-------|------|------|
{env_variables}

---

## 주요 스크립트

| 명령어 | 동작 |
|-------|------|
{scripts}
```

### project-patterns.md 템플릿

```markdown
# Project Code Patterns

> 자동 생성됨 | 이 프로젝트에서 사용하는 기술의 Best Practice 패턴

{detected_patterns}

---

## 패턴 출처

이 문서는 다음 기술의 공식 문서 기반 패턴을 포함합니다:
{pattern_sources}
```

---

## 패턴 조합 규칙

감지된 기술에 따라 `references/` 폴더의 패턴을 조합합니다:

```
감지: Next.js + TypeScript + Jotai + TanStack Query

조합되는 패턴:
├── references/nextjs.md
├── references/typescript.md
├── references/jotai.md
└── references/tanstack-query.md

결과: .claude/project-patterns.md에 통합
```

### 패턴 우선순위

동일 카테고리에서 여러 기술 감지 시:

| 카테고리 | 우선순위 |
|---------|---------|
| Framework | Next.js > Vite > CRA |
| State | 감지된 모든 기술 포함 |
| Forms | 감지된 모든 기술 포함 |

---

## 재생성 안내 시점

다음 상황에서 `컨텍스트 재생성`을 **안내**합니다 (자동 실행 아님):

### 1. 라이브러리 마이그레이션 완료 후

```
사용자: "Zustand를 Jotai로 마이그레이션해줘"

작업 완료 후 안내:
"마이그레이션이 완료되었습니다.

💡 의존성 제거 후 '컨텍스트 재생성'을 실행하면
   코드 패턴이 새 기술 스택에 맞게 업데이트됩니다.

   pnpm remove zustand
   → '컨텍스트 재생성해줘'"
```

### 2. 대규모 의존성 변경 후

```
사용자: "이 프로젝트에서 Redux 관련 코드 모두 제거해줘"

작업 완료 후 안내:
"Redux 관련 코드가 제거되었습니다.

💡 '컨텍스트 재생성'으로 패턴을 정리할 수 있습니다."
```

### 3. 프로젝트 구조 대규모 변경 후

```
사용자: "src/ 구조를 app/ 기반 App Router로 마이그레이션해줘"

작업 완료 후 안내:
"App Router 마이그레이션이 완료되었습니다.

💡 '컨텍스트 재생성'으로 프로젝트 구조와 패턴을
   업데이트할 수 있습니다."
```

---

## 다른 스킬과의 연동

### smart-prompt-enhancer 연동

```
smart-prompt-enhancer가 요청 처리 시:
1. .claude/project-context.md 참조 → 기술 스택 파악
2. .claude/project-patterns.md 참조 → 코드 패턴 적용
3. 구현 방식 선택지 제시
```

### frontend-best-practice 연동

```
project-patterns.md가 없는 경우:
→ frontend-best-practice의 범용 패턴 사용

project-patterns.md가 있는 경우:
→ 프로젝트 맞춤 패턴 우선 사용
```

---

## 사용 예시

### 최초 프로젝트 설정

```
사용자: "프로젝트 컨텍스트 생성해줘"

Claude:
1. package.json 분석 중...
2. 감지된 기술 스택:
   - Next.js 15.0.0
   - TypeScript 5.3.0
   - Tailwind CSS 3.4.0
   - Jotai 2.6.0
   - TanStack Query 5.17.0
   - React Hook Form 7.49.0
   - Zod 3.22.0
   - Supabase 2.39.0

3. 프로젝트 구조 분석 중...
4. 파일 생성 중...

✅ 완료!
   - .claude/project-context.md (프로젝트 메타 정보)
   - .claude/project-patterns.md (코드 패턴 가이드)

이제 코드 작성 시 이 프로젝트에 맞는 패턴이 적용됩니다.
```

### 기술 스택 변경 후 재생성

```
사용자: "컨텍스트 재생성해줘"

Claude:
1. 기존 컨텍스트 파일 삭제...
2. package.json 재분석 중...
3. 변경 감지:
   - 제거됨: Zustand
   - 추가됨: Jotai
4. 파일 재생성 중...

✅ 완료!
   - Zustand 패턴 제거됨
   - Jotai 패턴 추가됨
```

---

## Reference Files

기술별 패턴 템플릿:
- `references/react.md`
- `references/nextjs.md`
- `references/typescript.md`
- `references/tailwind.md`
- `references/jotai.md`
- `references/zustand.md`
- `references/tanstack-query.md`
- `references/react-hook-form.md`
- `references/zod.md`
- `references/supabase.md`
- `references/prisma.md`
