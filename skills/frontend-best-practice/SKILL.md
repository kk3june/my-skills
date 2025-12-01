---
name: frontend-best-practice
description: 프론트엔드 개발 시 공식 문서 기반 Best Practice를 적용하여 높은 코드 퀄리티를 보장합니다. React, Next.js, TypeScript, Tailwind, Shadcn, react-hook-form, zod 스택에 특화되어 있습니다.
version: "1.0.0"
---

# Frontend Best Practice Skill

프론트엔드 코드 작성, 리뷰, 리팩토링 시 공식 문서 기반의 Best Practice를 적용하여 프로덕션 레벨의 코드 퀄리티를 보장합니다.

## 핵심 원칙

### 1. 공식 문서 우선 (Documentation First)
- 모든 패턴은 공식 문서에서 권장하는 방식을 따름
- 추측이나 관습보다 문서화된 Best Practice 우선
- 버전별 차이가 있을 경우 최신 권장 사항 적용

### 2. 타입 안전성 (Type Safety)
- `any` 사용 금지, 불가피한 경우 `unknown` + 타입 가드
- 명시적 타입 정의로 런타임 에러 방지
- Generic을 활용한 재사용 가능한 타입 설계

### 3. 관심사 분리 (Separation of Concerns)
- UI 로직과 비즈니스 로직 분리
- Custom Hook을 통한 로직 추출
- 컴포넌트는 렌더링에만 집중

---

## 공식 문서 레퍼런스

코드 작성 시 항상 아래 공식 문서를 참조하세요:

### Core Stack
| 기술 | 공식 문서 | Best Practice |
|------|----------|---------------|
| **Next.js 15** | https://nextjs.org/docs | App Router 사용, RSC 우선 |
| **React 19** | https://react.dev/reference | 새 use() API, Server Components |
| **TypeScript 5** | https://www.typescriptlang.org/docs | strict 모드 필수 |

### UI & Styling
| 기술 | 공식 문서 | Best Practice |
|------|----------|---------------|
| **Tailwind CSS 4** | https://tailwindcss.com/docs | @apply 최소화, 컴포넌트 추출 |
| **Shadcn/ui** | https://ui.shadcn.com/docs | 컴포넌트 복사 후 커스텀 |
| **Lucide React** | https://lucide.dev/guide | Tree-shaking 위해 개별 import |

### Forms & Validation
| 기술 | 공식 문서 | Best Practice |
|------|----------|---------------|
| **React Hook Form** | https://react-hook-form.com/docs | Controller 대신 register 우선 |
| **Zod** | https://zod.dev | 스키마 재사용, transform 활용 |

### State & Data
| 기술 | 공식 문서 | Best Practice |
|------|----------|---------------|
| **TanStack Query** | https://tanstack.com/query/latest/docs | staleTime 설정, 캐시 전략 |
| **Zustand** | https://docs.pmnd.rs/zustand | slice 패턴, persist middleware |
| **Supabase** | https://supabase.com/docs | Row Level Security 필수 |

---

## 자동 적용 규칙

코드 작성 시 아래 규칙을 자동으로 적용합니다:

### Next.js App Router

```typescript
// ✅ CORRECT: Server Component (기본값)
// app/users/page.tsx
async function UsersPage() {
  const users = await fetchUsers() // 서버에서 직접 데이터 fetching
  return <UserList users={users} />
}

// ✅ CORRECT: Client Component (필요한 경우만)
// components/counter.tsx
'use client'
import { useState } from 'react'

export function Counter() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>
}
```

**적용 기준:**
- `useState`, `useEffect`, `onClick` 등 사용 → `'use client'`
- 데이터 fetching만 → Server Component 유지
- 혼합 시 → Client 부분만 분리하여 최소화

### TypeScript 필수 패턴

```typescript
// ✅ CORRECT: Props 타입 정의
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'ghost'
  size?: 'sm' | 'md' | 'lg'
  children: React.ReactNode
  onClick?: () => void
  disabled?: boolean
  className?: string
}

// ✅ CORRECT: Generic 컴포넌트
interface ListProps<T> {
  items: T[]
  renderItem: (item: T, index: number) => React.ReactNode
  keyExtractor: (item: T) => string
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={keyExtractor(item)}>{renderItem(item, index)}</li>
      ))}
    </ul>
  )
}

// ❌ WRONG: any 사용
function BadComponent(props: any) { ... }

// ✅ CORRECT: unknown + 타입 가드
function parseJson(text: string): unknown {
  return JSON.parse(text)
}

function isUser(data: unknown): data is User {
  return (
    typeof data === 'object' &&
    data !== null &&
    'id' in data &&
    'email' in data
  )
}
```

### Shadcn + Tailwind 패턴

```typescript
// ✅ CORRECT: cn() 유틸리티로 조건부 클래스
import { cn } from '@/lib/utils'

interface CardProps {
  className?: string
  variant?: 'default' | 'destructive'
}

function Card({ className, variant = 'default' }: CardProps) {
  return (
    <div
      className={cn(
        'rounded-lg border p-4',
        variant === 'destructive' && 'border-destructive bg-destructive/10',
        className
      )}
    />
  )
}

// ✅ CORRECT: Lucide 아이콘 개별 import (tree-shaking)
import { ChevronRight, User, Settings } from 'lucide-react'

// ❌ WRONG: 전체 import
import * as Icons from 'lucide-react'
```

### React Hook Form + Zod 패턴

```typescript
// ✅ CORRECT: 스키마 정의
import { z } from 'zod'

const loginSchema = z.object({
  email: z
    .string()
    .min(1, '이메일을 입력해주세요')
    .email('올바른 이메일 형식이 아닙니다'),
  password: z
    .string()
    .min(8, '비밀번호는 8자 이상이어야 합니다')
    .regex(/[A-Z]/, '대문자를 포함해야 합니다')
    .regex(/[0-9]/, '숫자를 포함해야 합니다'),
})

type LoginForm = z.infer<typeof loginSchema>

// ✅ CORRECT: Form 컴포넌트
'use client'
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'

export function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<LoginForm>({
    resolver: zodResolver(loginSchema),
    defaultValues: {
      email: '',
      password: '',
    },
  })

  const onSubmit = async (data: LoginForm) => {
    // API 호출
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <Input
        {...register('email')}
        error={errors.email?.message}
        disabled={isSubmitting}
      />
      <Input
        {...register('password')}
        type="password"
        error={errors.password?.message}
        disabled={isSubmitting}
      />
      <Button type="submit" disabled={isSubmitting}>
        {isSubmitting ? '로그인 중...' : '로그인'}
      </Button>
    </form>
  )
}
```

### TanStack Query 패턴

```typescript
// ✅ CORRECT: Query Key Factory
export const userKeys = {
  all: ['users'] as const,
  lists: () => [...userKeys.all, 'list'] as const,
  list: (filters: UserFilters) => [...userKeys.lists(), filters] as const,
  details: () => [...userKeys.all, 'detail'] as const,
  detail: (id: string) => [...userKeys.details(), id] as const,
}

// ✅ CORRECT: Custom Query Hook
export function useUsers(filters: UserFilters) {
  return useQuery({
    queryKey: userKeys.list(filters),
    queryFn: () => fetchUsers(filters),
    staleTime: 5 * 60 * 1000, // 5분
    gcTime: 30 * 60 * 1000, // 30분
  })
}

// ✅ CORRECT: Mutation with Optimistic Update
export function useUpdateUser() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: updateUser,
    onMutate: async (newUser) => {
      await queryClient.cancelQueries({ queryKey: userKeys.detail(newUser.id) })
      const previous = queryClient.getQueryData(userKeys.detail(newUser.id))
      queryClient.setQueryData(userKeys.detail(newUser.id), newUser)
      return { previous }
    },
    onError: (err, newUser, context) => {
      queryClient.setQueryData(userKeys.detail(newUser.id), context?.previous)
    },
    onSettled: (data, error, variables) => {
      queryClient.invalidateQueries({ queryKey: userKeys.detail(variables.id) })
    },
  })
}
```

### Zustand 패턴

```typescript
// ✅ CORRECT: Slice 패턴 with TypeScript
import { create } from 'zustand'
import { persist, createJSONStorage } from 'zustand/middleware'
import { immer } from 'zustand/middleware/immer'

interface AuthSlice {
  user: User | null
  token: string | null
  login: (user: User, token: string) => void
  logout: () => void
}

interface CartSlice {
  items: CartItem[]
  addItem: (item: CartItem) => void
  removeItem: (id: string) => void
  clearCart: () => void
}

type Store = AuthSlice & CartSlice

export const useStore = create<Store>()(
  persist(
    immer((set) => ({
      // Auth Slice
      user: null,
      token: null,
      login: (user, token) =>
        set((state) => {
          state.user = user
          state.token = token
        }),
      logout: () =>
        set((state) => {
          state.user = null
          state.token = null
        }),

      // Cart Slice
      items: [],
      addItem: (item) =>
        set((state) => {
          const existing = state.items.find((i) => i.id === item.id)
          if (existing) {
            existing.quantity += item.quantity
          } else {
            state.items.push(item)
          }
        }),
      removeItem: (id) =>
        set((state) => {
          state.items = state.items.filter((i) => i.id !== id)
        }),
      clearCart: () =>
        set((state) => {
          state.items = []
        }),
    })),
    {
      name: 'app-storage',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({ items: state.items }), // cart만 persist
    }
  )
)
```

---

## 코드 리뷰 체크리스트

코드 리뷰 요청 시 아래 체크리스트를 자동 적용합니다:

### 필수 체크 (MUST)

- [ ] **타입 안전성**: `any` 사용 없음, 모든 props/state 타입 정의
- [ ] **Server/Client 분리**: `'use client'` 최소화, RSC 우선
- [ ] **에러 처리**: try-catch, Error Boundary, 사용자 피드백
- [ ] **로딩 상태**: Suspense 또는 isLoading 처리
- [ ] **접근성**: 키보드 네비게이션, ARIA 레이블, 시맨틱 HTML
- [ ] **보안**: XSS 방지, 입력값 검증, 환경변수 노출 금지

### 권장 체크 (SHOULD)

- [ ] **성능**: 불필요한 리렌더링 방지, 메모이제이션 적절히 사용
- [ ] **코드 분할**: dynamic import로 번들 크기 최적화
- [ ] **재사용성**: 3회 이상 중복 시 컴포넌트/훅 추출
- [ ] **일관성**: 네이밍 컨벤션, 파일 구조 통일
- [ ] **테스트**: 핵심 비즈니스 로직 테스트 커버리지

### 선택 체크 (MAY)

- [ ] **문서화**: 복잡한 로직에 JSDoc 주석
- [ ] **국제화**: 하드코딩된 문자열 분리
- [ ] **애니메이션**: 60fps 유지, GPU 가속 활용

---

## 리팩토링 가이드

리팩토링 요청 시 아래 패턴을 적용합니다:

### 1. 컴포넌트 분리

**Before (거대한 컴포넌트):**
```typescript
function UserDashboard() {
  const [user, setUser] = useState(null)
  const [posts, setPosts] = useState([])
  const [isLoading, setIsLoading] = useState(true)
  // ... 200줄의 로직

  return (
    <div>
      {/* 헤더 50줄 */}
      {/* 사이드바 80줄 */}
      {/* 콘텐츠 100줄 */}
      {/* 푸터 30줄 */}
    </div>
  )
}
```

**After (관심사 분리):**
```typescript
// hooks/useUserData.ts
export function useUserData(userId: string) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  })
}

// components/dashboard/UserHeader.tsx
export function UserHeader({ user }: { user: User }) {
  return <header>...</header>
}

// components/dashboard/UserSidebar.tsx
export function UserSidebar({ user }: { user: User }) {
  return <aside>...</aside>
}

// app/dashboard/page.tsx
async function UserDashboard() {
  const user = await fetchUser()

  return (
    <div className="flex">
      <UserSidebar user={user} />
      <main>
        <UserHeader user={user} />
        <Suspense fallback={<PostsSkeleton />}>
          <UserPosts userId={user.id} />
        </Suspense>
      </main>
    </div>
  )
}
```

### 2. Custom Hook 추출

**Before:**
```typescript
function SearchComponent() {
  const [query, setQuery] = useState('')
  const [results, setResults] = useState([])
  const [isLoading, setIsLoading] = useState(false)
  const [error, setError] = useState(null)

  useEffect(() => {
    if (!query) return

    const controller = new AbortController()
    setIsLoading(true)

    fetch(`/api/search?q=${query}`, { signal: controller.signal })
      .then(res => res.json())
      .then(data => {
        setResults(data)
        setIsLoading(false)
      })
      .catch(err => {
        if (err.name !== 'AbortError') {
          setError(err)
          setIsLoading(false)
        }
      })

    return () => controller.abort()
  }, [query])

  // ... 렌더링 로직
}
```

**After:**
```typescript
// hooks/useSearch.ts
export function useSearch<T>(endpoint: string) {
  const [query, setQuery] = useState('')
  const debouncedQuery = useDebounce(query, 300)

  const { data, isLoading, error } = useQuery({
    queryKey: ['search', endpoint, debouncedQuery],
    queryFn: () => fetch(`${endpoint}?q=${debouncedQuery}`).then(r => r.json()),
    enabled: debouncedQuery.length > 0,
  })

  return { query, setQuery, results: data as T[], isLoading, error }
}

// components/SearchComponent.tsx
function SearchComponent() {
  const { query, setQuery, results, isLoading } = useSearch<Product>('/api/search')

  return (
    <div>
      <Input value={query} onChange={e => setQuery(e.target.value)} />
      {isLoading ? <Skeleton /> : <ResultList items={results} />}
    </div>
  )
}
```

### 3. 조건부 렌더링 정리

**Before:**
```typescript
return (
  <div>
    {isLoading ? (
      <Spinner />
    ) : error ? (
      <Error message={error.message} />
    ) : data && data.length > 0 ? (
      <List items={data} />
    ) : (
      <Empty message="데이터가 없습니다" />
    )}
  </div>
)
```

**After:**
```typescript
// 패턴 1: Early Return
function DataDisplay({ isLoading, error, data }: Props) {
  if (isLoading) return <Spinner />
  if (error) return <Error message={error.message} />
  if (!data?.length) return <Empty message="데이터가 없습니다" />

  return <List items={data} />
}

// 패턴 2: 상태 객체
const states = {
  loading: <Spinner />,
  error: <Error message={error?.message} />,
  empty: <Empty message="데이터가 없습니다" />,
  success: <List items={data} />,
}

const currentState = isLoading ? 'loading'
  : error ? 'error'
  : !data?.length ? 'empty'
  : 'success'

return states[currentState]
```

---

## 파일/폴더 구조

Next.js App Router 기준 권장 구조:

```
src/
├── app/                      # App Router 페이지
│   ├── (auth)/              # Route Group (URL에 미포함)
│   │   ├── login/
│   │   └── register/
│   ├── (main)/
│   │   ├── dashboard/
│   │   └── settings/
│   ├── api/                 # API Routes
│   ├── layout.tsx
│   ├── page.tsx
│   └── globals.css
│
├── components/              # 공유 컴포넌트
│   ├── ui/                  # Shadcn 기본 컴포넌트
│   │   ├── button.tsx
│   │   ├── input.tsx
│   │   └── card.tsx
│   ├── forms/               # 폼 컴포넌트
│   │   ├── LoginForm.tsx
│   │   └── RegisterForm.tsx
│   └── layouts/             # 레이아웃 컴포넌트
│       ├── Header.tsx
│       └── Sidebar.tsx
│
├── hooks/                   # Custom Hooks
│   ├── useAuth.ts
│   ├── useDebounce.ts
│   └── useMediaQuery.ts
│
├── lib/                     # 유틸리티
│   ├── utils.ts             # cn() 등 헬퍼
│   ├── api.ts               # API 클라이언트
│   └── validations.ts       # Zod 스키마
│
├── stores/                  # Zustand 스토어
│   ├── useAuthStore.ts
│   └── useCartStore.ts
│
├── types/                   # 타입 정의
│   ├── api.ts
│   └── models.ts
│
└── constants/               # 상수
    ├── routes.ts
    └── config.ts
```

---

## 에러 처리 패턴

### API 에러 처리

```typescript
// lib/api.ts
class ApiError extends Error {
  constructor(
    message: string,
    public status: number,
    public code?: string
  ) {
    super(message)
    this.name = 'ApiError'
  }
}

async function fetchApi<T>(url: string, options?: RequestInit): Promise<T> {
  const response = await fetch(url, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      ...options?.headers,
    },
  })

  if (!response.ok) {
    const error = await response.json().catch(() => ({}))
    throw new ApiError(
      error.message || '요청 처리 중 오류가 발생했습니다',
      response.status,
      error.code
    )
  }

  return response.json()
}

// 사용
try {
  const user = await fetchApi<User>('/api/user')
} catch (error) {
  if (error instanceof ApiError) {
    if (error.status === 401) {
      redirect('/login')
    }
    toast.error(error.message)
  }
}
```

### Error Boundary

```typescript
// components/ErrorBoundary.tsx
'use client'

import { Component, ReactNode } from 'react'

interface Props {
  children: ReactNode
  fallback?: ReactNode
}

interface State {
  hasError: boolean
  error?: Error
}

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error }
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Error caught:', error, errorInfo)
    // Sentry 등 에러 트래킹 서비스에 전송
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div className="flex flex-col items-center justify-center p-8">
          <h2 className="text-lg font-semibold">문제가 발생했습니다</h2>
          <p className="text-muted-foreground">{this.state.error?.message}</p>
          <Button onClick={() => this.setState({ hasError: false })}>
            다시 시도
          </Button>
        </div>
      )
    }

    return this.props.children
  }
}

// Next.js 15 error.tsx
// app/dashboard/error.tsx
'use client'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div className="flex flex-col items-center justify-center min-h-[400px]">
      <h2 className="text-lg font-semibold">문제가 발생했습니다</h2>
      <p className="text-muted-foreground mb-4">{error.message}</p>
      <Button onClick={reset}>다시 시도</Button>
    </div>
  )
}
```

---

## 성능 최적화

### React Compiler (React 19)

```typescript
// React 19부터는 자동 메모이제이션
// useMemo, useCallback, memo 대부분 불필요

// ❌ 더 이상 필요 없음 (React 19+)
const memoizedValue = useMemo(() => computeExpensive(a, b), [a, b])
const memoizedCallback = useCallback(() => doSomething(a), [a])

// ✅ 그냥 작성 (React Compiler가 최적화)
const value = computeExpensive(a, b)
const callback = () => doSomething(a)
```

### Dynamic Import

```typescript
// ✅ 조건부 로딩
import dynamic from 'next/dynamic'

const HeavyChart = dynamic(() => import('@/components/HeavyChart'), {
  loading: () => <ChartSkeleton />,
  ssr: false, // 클라이언트에서만 렌더링
})

// ✅ 모달 등 사용자 인터랙션 후 로딩
const SettingsModal = dynamic(() => import('@/components/SettingsModal'))
```

### Image 최적화

```typescript
import Image from 'next/image'

// ✅ 반응형 이미지
<Image
  src="/hero.jpg"
  alt="Hero"
  fill
  sizes="(max-width: 768px) 100vw, 50vw"
  priority // LCP 이미지는 priority 추가
  className="object-cover"
/>

// ✅ 고정 크기 이미지
<Image
  src="/avatar.png"
  alt="Avatar"
  width={48}
  height={48}
  className="rounded-full"
/>
```

---

## 활용 방법

### 기능 구현 요청 시

```
"로그인 폼 만들어줘"
→ React Hook Form + Zod 패턴으로 구현
→ 에러 처리, 로딩 상태, 접근성 포함
→ 공식 문서 기반 Best Practice 적용
```

### 코드 리뷰 요청 시

```
"이 코드 리뷰해줘"
→ 체크리스트 기반 리뷰
→ 개선점 구체적으로 제안
→ 리팩토링 예시 코드 제공
```

### 리팩토링 요청 시

```
"이 컴포넌트 리팩토링해줘"
→ 관심사 분리 패턴 적용
→ Custom Hook 추출
→ 타입 안전성 강화
```

---

## Reference Files

상세 가이드:
- **Next.js 패턴**: references/nextjs-patterns.md
- **React 패턴**: references/react-patterns.md
- **TypeScript 패턴**: references/typescript-patterns.md
- **스타일링 패턴**: references/styling-patterns.md
- **폼/검증 패턴**: references/form-validation-patterns.md
