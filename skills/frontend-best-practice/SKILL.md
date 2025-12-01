---
name: frontend-best-practice
description: React 기반 프론트엔드 개발 시 공식 문서 기반 Best Practice를 적용합니다. React, TypeScript, Tailwind, Shadcn, react-hook-form, zod 스택에 특화되어 있으며, Next.js 프로젝트에서는 추가 패턴이 자동 적용됩니다.
version: "1.1.0"
---

# Frontend Best Practice Skill

React 기반 프론트엔드 코드 작성, 리뷰, 리팩토링 시 공식 문서 기반의 Best Practice를 적용하여 프로덕션 레벨의 코드 퀄리티를 보장합니다.

## 프로젝트 감지 및 적용

이 스킬은 프로젝트 타입을 자동 감지하여 적절한 패턴을 적용합니다:

| 프로젝트 타입 | 감지 방법 | 적용 패턴 |
|-------------|----------|----------|
| **React SPA** | Vite, CRA, package.json | Core 패턴 |
| **Next.js** | next.config.js, package.json | Core + Next.js 확장 |
| **React Native** | react-native in deps | Core + RN 패턴 |

---

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

### Core (항상 적용)

| 기술 | 공식 문서 | Best Practice |
|------|----------|---------------|
| **React 19** | https://react.dev/reference | use() API, Hooks 패턴 |
| **TypeScript 5** | https://www.typescriptlang.org/docs | strict 모드 필수 |
| **Tailwind CSS 4** | https://tailwindcss.com/docs | @apply 최소화, 컴포넌트 추출 |
| **Shadcn/ui** | https://ui.shadcn.com/docs | 컴포넌트 복사 후 커스텀 |
| **Lucide React** | https://lucide.dev/guide | Tree-shaking 위해 개별 import |

### Forms & Validation

| 기술 | 공식 문서 | Best Practice |
|------|----------|---------------|
| **React Hook Form** | https://react-hook-form.com/docs | Controller 대신 register 우선 |
| **Zod** | https://zod.dev | 스키마 재사용, transform 활용 |

### State & Data Fetching

| 기술 | 공식 문서 | Best Practice |
|------|----------|---------------|
| **TanStack Query** | https://tanstack.com/query/latest/docs | staleTime 설정, 캐시 전략 |
| **Zustand** | https://docs.pmnd.rs/zustand | slice 패턴, persist middleware |
| **Supabase** | https://supabase.com/docs | Row Level Security 필수 |

### Framework Extensions (프로젝트에 따라 적용)

| 기술 | 공식 문서 | 적용 조건 |
|------|----------|----------|
| **Next.js 15** | https://nextjs.org/docs | next.config 감지 시 |
| **Vite** | https://vitejs.dev/guide | vite.config 감지 시 |
| **React Router** | https://reactrouter.com/en/main | react-router-dom 감지 시 |

---

## Core 패턴 (모든 React 프로젝트)

### React 컴포넌트 패턴

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

export function Button({
  variant = 'primary',
  size = 'md',
  children,
  className,
  ...props
}: ButtonProps) {
  return (
    <button
      className={cn(
        'inline-flex items-center justify-center rounded-md font-medium',
        variants[variant],
        sizes[size],
        className
      )}
      {...props}
    >
      {children}
    </button>
  )
}

// ✅ CORRECT: Generic 컴포넌트
interface ListProps<T> {
  items: T[]
  renderItem: (item: T, index: number) => React.ReactNode
  keyExtractor: (item: T) => string
  emptyMessage?: string
}

function List<T>({
  items,
  renderItem,
  keyExtractor,
  emptyMessage = '항목이 없습니다',
}: ListProps<T>) {
  if (items.length === 0) {
    return <p className="text-muted-foreground">{emptyMessage}</p>
  }

  return (
    <ul className="space-y-2">
      {items.map((item, index) => (
        <li key={keyExtractor(item)}>{renderItem(item, index)}</li>
      ))}
    </ul>
  )
}

// ❌ WRONG: any 사용
function BadComponent(props: any) { ... }

// ✅ CORRECT: unknown + 타입 가드
function isUser(data: unknown): data is User {
  return (
    typeof data === 'object' &&
    data !== null &&
    'id' in data &&
    'email' in data
  )
}
```

### Custom Hook 패턴

```typescript
// ✅ useToggle
export function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue)

  const toggle = useCallback(() => setValue((v) => !v), [])
  const setTrue = useCallback(() => setValue(true), [])
  const setFalse = useCallback(() => setValue(false), [])

  return { value, toggle, setTrue, setFalse } as const
}

// ✅ useDebounce
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value)

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay)
    return () => clearTimeout(timer)
  }, [value, delay])

  return debouncedValue
}

// ✅ useLocalStorage
export function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') return initialValue
    try {
      const item = window.localStorage.getItem(key)
      return item ? JSON.parse(item) : initialValue
    } catch {
      return initialValue
    }
  })

  const setValue = useCallback(
    (value: T | ((val: T) => T)) => {
      setStoredValue((prev) => {
        const valueToStore = value instanceof Function ? value(prev) : value
        window.localStorage.setItem(key, JSON.stringify(valueToStore))
        return valueToStore
      })
    },
    [key]
  )

  return [storedValue, setValue] as const
}

// ✅ useOnClickOutside
export function useOnClickOutside<T extends HTMLElement>(
  ref: RefObject<T>,
  handler: (event: MouseEvent | TouchEvent) => void
) {
  useEffect(() => {
    const listener = (event: MouseEvent | TouchEvent) => {
      if (!ref.current || ref.current.contains(event.target as Node)) {
        return
      }
      handler(event)
    }

    document.addEventListener('mousedown', listener)
    document.addEventListener('touchstart', listener)

    return () => {
      document.removeEventListener('mousedown', listener)
      document.removeEventListener('touchstart', listener)
    }
  }, [ref, handler])
}
```

### Shadcn + Tailwind 패턴

```typescript
// ✅ cn() 유틸리티 (필수 설정)
// lib/utils.ts
import { type ClassValue, clsx } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}

// ✅ 조건부 클래스
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

// ✅ Lucide 아이콘 개별 import (tree-shaking)
import { ChevronRight, User, Settings } from 'lucide-react'

// ❌ WRONG: 전체 import
import * as Icons from 'lucide-react'
```

### React Hook Form + Zod 패턴

```typescript
// ✅ 스키마 정의
import { z } from 'zod'

export const loginSchema = z.object({
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

export type LoginFormData = z.infer<typeof loginSchema>

// ✅ Form 컴포넌트
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'

export function LoginForm({ onSubmit }: { onSubmit: (data: LoginFormData) => Promise<void> }) {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
    setError,
  } = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
    defaultValues: { email: '', password: '' },
  })

  const handleFormSubmit = async (data: LoginFormData) => {
    try {
      await onSubmit(data)
    } catch (error) {
      setError('root', { message: '로그인에 실패했습니다' })
    }
  }

  return (
    <form onSubmit={handleSubmit(handleFormSubmit)} className="space-y-4">
      {errors.root && (
        <p className="text-sm text-destructive">{errors.root.message}</p>
      )}

      <div>
        <Input
          {...register('email')}
          type="email"
          placeholder="이메일"
          className={cn(errors.email && 'border-destructive')}
        />
        {errors.email && (
          <p className="text-sm text-destructive mt-1">{errors.email.message}</p>
        )}
      </div>

      <div>
        <Input
          {...register('password')}
          type="password"
          placeholder="비밀번호"
          className={cn(errors.password && 'border-destructive')}
        />
        {errors.password && (
          <p className="text-sm text-destructive mt-1">{errors.password.message}</p>
        )}
      </div>

      <Button type="submit" disabled={isSubmitting} className="w-full">
        {isSubmitting ? '로그인 중...' : '로그인'}
      </Button>
    </form>
  )
}
```

### TanStack Query 패턴

```typescript
// ✅ Query Key Factory
export const userKeys = {
  all: ['users'] as const,
  lists: () => [...userKeys.all, 'list'] as const,
  list: (filters: UserFilters) => [...userKeys.lists(), filters] as const,
  details: () => [...userKeys.all, 'detail'] as const,
  detail: (id: string) => [...userKeys.details(), id] as const,
}

// ✅ Custom Query Hook
export function useUsers(filters: UserFilters) {
  return useQuery({
    queryKey: userKeys.list(filters),
    queryFn: () => fetchUsers(filters),
    staleTime: 5 * 60 * 1000, // 5분
    gcTime: 30 * 60 * 1000, // 30분
  })
}

export function useUser(id: string) {
  return useQuery({
    queryKey: userKeys.detail(id),
    queryFn: () => fetchUser(id),
    enabled: !!id,
  })
}

// ✅ Mutation with Optimistic Update
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
// ✅ Slice 패턴 with TypeScript
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
  totalPrice: () => number
}

type Store = AuthSlice & CartSlice

export const useStore = create<Store>()(
  persist(
    immer((set, get) => ({
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
      totalPrice: () =>
        get().items.reduce((sum, item) => sum + item.price * item.quantity, 0),
    })),
    {
      name: 'app-storage',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({ items: state.items }),
    }
  )
)
```

### 에러 처리 패턴

```typescript
// ✅ API Error Class
export class ApiError extends Error {
  constructor(
    message: string,
    public status: number,
    public code?: string
  ) {
    super(message)
    this.name = 'ApiError'
  }
}

// ✅ Fetch Wrapper
export async function fetchApi<T>(url: string, options?: RequestInit): Promise<T> {
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

// ✅ Error Boundary (Class Component)
interface Props {
  children: React.ReactNode
  fallback?: React.ReactNode
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
```

---

## 파일/폴더 구조

### React SPA (Vite/CRA)

```
src/
├── components/              # 공유 컴포넌트
│   ├── ui/                  # Shadcn 기본 컴포넌트
│   │   ├── button.tsx
│   │   ├── input.tsx
│   │   └── card.tsx
│   ├── forms/               # 폼 컴포넌트
│   └── layouts/             # 레이아웃 컴포넌트
│
├── pages/                   # 페이지 컴포넌트
│   ├── HomePage.tsx
│   ├── LoginPage.tsx
│   └── DashboardPage.tsx
│
├── hooks/                   # Custom Hooks
│   ├── useAuth.ts
│   ├── useDebounce.ts
│   └── useLocalStorage.ts
│
├── lib/                     # 유틸리티
│   ├── utils.ts             # cn() 등 헬퍼
│   ├── api.ts               # API 클라이언트
│   └── validations.ts       # Zod 스키마
│
├── stores/                  # Zustand 스토어
│   └── useStore.ts
│
├── types/                   # 타입 정의
│   ├── api.ts
│   └── models.ts
│
├── constants/               # 상수
│   └── config.ts
│
├── App.tsx
├── main.tsx
└── index.css
```

---

## 코드 리뷰 체크리스트

### 필수 (MUST)

- [ ] **타입 안전성**: `any` 사용 없음, 모든 props/state 타입 정의
- [ ] **에러 처리**: try-catch, Error Boundary, 사용자 피드백
- [ ] **로딩 상태**: isLoading 또는 Suspense 처리
- [ ] **접근성**: 키보드 네비게이션, ARIA 레이블, 시맨틱 HTML
- [ ] **보안**: XSS 방지, 입력값 검증

### 권장 (SHOULD)

- [ ] **성능**: 불필요한 리렌더링 방지
- [ ] **재사용성**: 3회 이상 중복 시 컴포넌트/훅 추출
- [ ] **일관성**: 네이밍 컨벤션, 파일 구조 통일
- [ ] **테스트**: 핵심 비즈니스 로직 테스트

### 선택 (MAY)

- [ ] **문서화**: 복잡한 로직에 JSDoc 주석
- [ ] **국제화**: 하드코딩된 문자열 분리

---

## 리팩토링 가이드

### 1. Custom Hook 추출

**Before:**
```typescript
function SearchComponent() {
  const [query, setQuery] = useState('')
  const [results, setResults] = useState([])
  const [isLoading, setIsLoading] = useState(false)

  useEffect(() => {
    if (!query) return
    setIsLoading(true)
    fetch(`/api/search?q=${query}`)
      .then(res => res.json())
      .then(data => {
        setResults(data)
        setIsLoading(false)
      })
  }, [query])

  return (/* 렌더링 */)
}
```

**After:**
```typescript
// hooks/useSearch.ts
export function useSearch<T>(endpoint: string) {
  const [query, setQuery] = useState('')
  const debouncedQuery = useDebounce(query, 300)

  const { data, isLoading } = useQuery({
    queryKey: ['search', endpoint, debouncedQuery],
    queryFn: () => fetchApi<T[]>(`${endpoint}?q=${debouncedQuery}`),
    enabled: debouncedQuery.length > 0,
  })

  return { query, setQuery, results: data ?? [], isLoading }
}

// components/SearchComponent.tsx
function SearchComponent() {
  const { query, setQuery, results, isLoading } = useSearch<Product>('/api/search')
  return (/* 렌더링 */)
}
```

### 2. 조건부 렌더링 정리

**Before:**
```typescript
{isLoading ? <Spinner /> : error ? <Error /> : data ? <List /> : <Empty />}
```

**After:**
```typescript
function DataDisplay({ isLoading, error, data }: Props) {
  if (isLoading) return <Spinner />
  if (error) return <Error message={error.message} />
  if (!data?.length) return <Empty />
  return <List items={data} />
}
```

---

## Next.js 프로젝트 확장

> **적용 조건**: `next.config.js` 또는 `next.config.mjs` 감지 시 자동 적용

Next.js 프로젝트에서는 위 Core 패턴에 추가로 아래 패턴이 적용됩니다:

### Server/Client 컴포넌트 분리

```typescript
// ✅ Server Component (기본값) - 데이터 fetching
// app/users/page.tsx
async function UsersPage() {
  const users = await fetchUsers() // 서버에서 직접
  return <UserList users={users} />
}

// ✅ Client Component - 인터랙션 필요 시만
// components/Counter.tsx
'use client'

import { useState } from 'react'

export function Counter() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>
}
```

**적용 기준:**
- `useState`, `useEffect`, `onClick` 등 → `'use client'`
- 데이터 fetching만 → Server Component 유지
- 혼합 시 → Client 부분만 분리하여 최소화

### App Router 파일 구조

```
src/
├── app/                      # App Router
│   ├── (auth)/              # Route Group
│   │   ├── login/page.tsx
│   │   └── register/page.tsx
│   ├── (main)/
│   │   ├── layout.tsx       # 공유 레이아웃
│   │   ├── dashboard/page.tsx
│   │   └── settings/page.tsx
│   ├── api/                 # API Routes
│   ├── layout.tsx
│   ├── page.tsx
│   ├── error.tsx            # 에러 UI
│   ├── loading.tsx          # 로딩 UI
│   └── globals.css
│
├── components/              # (Core 구조와 동일)
├── hooks/
├── lib/
└── ...
```

### Server Actions

```typescript
// app/actions.ts
'use server'

import { revalidatePath } from 'next/cache'

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string
  const content = formData.get('content') as string

  await db.post.create({ data: { title, content } })
  revalidatePath('/posts')
}

// 사용
<form action={createPost}>
  <input name="title" />
  <textarea name="content" />
  <button type="submit">저장</button>
</form>
```

### Next.js 전용 체크리스트 (추가)

- [ ] **Server/Client 분리**: `'use client'` 최소화
- [ ] **Metadata**: 각 페이지에 SEO 메타데이터 설정
- [ ] **Image 최적화**: next/image 사용, sizes 속성 설정
- [ ] **Loading/Error UI**: 각 라우트에 loading.tsx, error.tsx

상세 패턴은 `references/nextjs-patterns.md` 참조

---

## Reference Files

상세 가이드:
- **React 패턴**: references/react-patterns.md
- **TypeScript 패턴**: references/typescript-patterns.md
- **스타일링 패턴**: references/styling-patterns.md
- **폼/검증 패턴**: references/form-validation-patterns.md
- **Next.js 패턴**: references/nextjs-patterns.md (Next.js 프로젝트용)
