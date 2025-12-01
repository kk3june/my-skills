# Next.js 15 Best Practice Patterns

> 공식 문서: https://nextjs.org/docs

## Server Components vs Client Components

### 결정 기준

| 사용 사례 | Server Component | Client Component |
|----------|-----------------|------------------|
| 데이터 fetching | ✅ | - |
| 백엔드 리소스 직접 접근 | ✅ | - |
| 민감한 정보 (API keys, tokens) | ✅ | - |
| 큰 의존성 유지 (서버에서만) | ✅ | - |
| 인터랙티비티 (onClick, onChange) | - | ✅ |
| 상태 관리 (useState, useReducer) | - | ✅ |
| 생명주기 (useEffect) | - | ✅ |
| 브라우저 전용 API | - | ✅ |
| Custom Hooks (상태 의존) | - | ✅ |

### 패턴: 클라이언트 컴포넌트 최소화

```typescript
// ❌ WRONG: 전체를 Client Component로
'use client'

export default function ProductPage() {
  const [cart, setCart] = useState([])
  const product = await fetchProduct() // 에러! async 사용 불가

  return (
    <div>
      <ProductInfo product={product} />
      <AddToCartButton onClick={() => setCart([...cart, product])} />
    </div>
  )
}

// ✅ CORRECT: 인터랙티브 부분만 Client Component
// app/products/[id]/page.tsx (Server Component)
export default async function ProductPage({ params }: { params: { id: string } }) {
  const product = await fetchProduct(params.id)

  return (
    <div>
      <ProductInfo product={product} />
      <AddToCartButton product={product} /> {/* 이것만 Client */}
    </div>
  )
}

// components/AddToCartButton.tsx
'use client'

import { useCart } from '@/hooks/useCart'

export function AddToCartButton({ product }: { product: Product }) {
  const { addItem } = useCart()
  return <Button onClick={() => addItem(product)}>장바구니 담기</Button>
}
```

## Data Fetching

### Server Component에서 직접 Fetching

```typescript
// app/users/page.tsx
async function UsersPage() {
  // 서버에서 직접 fetch - 자동으로 캐싱됨
  const users = await fetch('https://api.example.com/users', {
    next: { revalidate: 3600 }, // 1시간마다 재검증
  }).then(res => res.json())

  return <UserList users={users} />
}

// 또는 직접 DB 접근
import { db } from '@/lib/db'

async function UsersPage() {
  const users = await db.user.findMany()
  return <UserList users={users} />
}
```

### 캐싱 전략

```typescript
// 1. 정적 데이터 (빌드 시 캐시)
fetch('https://api.example.com/posts', { cache: 'force-cache' })

// 2. 동적 데이터 (매 요청마다)
fetch('https://api.example.com/posts', { cache: 'no-store' })

// 3. 시간 기반 재검증
fetch('https://api.example.com/posts', {
  next: { revalidate: 3600 }, // 1시간
})

// 4. 태그 기반 재검증
fetch('https://api.example.com/posts', {
  next: { tags: ['posts'] },
})

// 재검증 트리거 (Server Action)
import { revalidateTag } from 'next/cache'

export async function createPost(data: PostData) {
  await db.post.create({ data })
  revalidateTag('posts')
}
```

## Streaming & Suspense

### Loading UI

```typescript
// app/dashboard/loading.tsx
export default function Loading() {
  return <DashboardSkeleton />
}

// 또는 Suspense로 세분화
// app/dashboard/page.tsx
export default function DashboardPage() {
  return (
    <div className="grid grid-cols-3 gap-4">
      <Suspense fallback={<StatsSkeleton />}>
        <Stats />
      </Suspense>

      <Suspense fallback={<ChartSkeleton />}>
        <RevenueChart />
      </Suspense>

      <Suspense fallback={<TableSkeleton />}>
        <RecentOrders />
      </Suspense>
    </div>
  )
}
```

### Parallel Data Fetching

```typescript
// ✅ 병렬 fetching
async function Dashboard() {
  // 동시에 시작
  const statsPromise = fetchStats()
  const revenuePromise = fetchRevenue()
  const ordersPromise = fetchOrders()

  // Promise.all로 대기
  const [stats, revenue, orders] = await Promise.all([
    statsPromise,
    revenuePromise,
    ordersPromise,
  ])

  return (
    <>
      <Stats data={stats} />
      <RevenueChart data={revenue} />
      <RecentOrders data={orders} />
    </>
  )
}
```

## Server Actions

### 기본 사용법

```typescript
// app/actions.ts
'use server'

import { revalidatePath } from 'next/cache'
import { redirect } from 'next/navigation'
import { z } from 'zod'

const createPostSchema = z.object({
  title: z.string().min(1),
  content: z.string().min(10),
})

export async function createPost(formData: FormData) {
  const validatedFields = createPostSchema.safeParse({
    title: formData.get('title'),
    content: formData.get('content'),
  })

  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
    }
  }

  const post = await db.post.create({
    data: validatedFields.data,
  })

  revalidatePath('/posts')
  redirect(`/posts/${post.id}`)
}

// 사용
// components/CreatePostForm.tsx
'use client'

import { useFormState } from 'react-dom'
import { createPost } from '@/app/actions'

const initialState = { errors: {} }

export function CreatePostForm() {
  const [state, formAction] = useFormState(createPost, initialState)

  return (
    <form action={formAction}>
      <input name="title" />
      {state.errors?.title && <p className="text-red-500">{state.errors.title}</p>}

      <textarea name="content" />
      {state.errors?.content && <p className="text-red-500">{state.errors.content}</p>}

      <SubmitButton />
    </form>
  )
}

function SubmitButton() {
  const { pending } = useFormStatus()
  return (
    <Button type="submit" disabled={pending}>
      {pending ? '저장 중...' : '저장'}
    </Button>
  )
}
```

## Routing Patterns

### Route Groups

```
app/
├── (auth)/                 # 인증 관련 (레이아웃 공유)
│   ├── layout.tsx          # 인증 전용 레이아웃
│   ├── login/
│   └── register/
├── (main)/                 # 메인 앱 (다른 레이아웃)
│   ├── layout.tsx          # 메인 레이아웃 (헤더, 사이드바)
│   ├── dashboard/
│   └── settings/
└── layout.tsx              # 루트 레이아웃
```

### Parallel Routes

```typescript
// app/dashboard/@analytics/page.tsx
export default function Analytics() {
  return <AnalyticsChart />
}

// app/dashboard/@team/page.tsx
export default function Team() {
  return <TeamList />
}

// app/dashboard/layout.tsx
export default function Layout({
  children,
  analytics,
  team,
}: {
  children: React.ReactNode
  analytics: React.ReactNode
  team: React.ReactNode
}) {
  return (
    <div className="grid grid-cols-2 gap-4">
      <div>{children}</div>
      <div>{analytics}</div>
      <div>{team}</div>
    </div>
  )
}
```

### Intercepting Routes

```
app/
├── feed/
│   └── page.tsx
├── photo/
│   └── [id]/
│       └── page.tsx        # /photo/123 직접 접근 시
└── @modal/
    └── (.)photo/           # (.) = 같은 레벨 인터셉트
        └── [id]/
            └── page.tsx    # /feed에서 /photo/123 클릭 시 모달로
```

## Middleware

```typescript
// middleware.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  const token = request.cookies.get('token')?.value

  // 인증 필요한 경로
  if (request.nextUrl.pathname.startsWith('/dashboard')) {
    if (!token) {
      return NextResponse.redirect(new URL('/login', request.url))
    }
  }

  // 이미 로그인된 사용자가 로그인 페이지 접근
  if (request.nextUrl.pathname.startsWith('/login')) {
    if (token) {
      return NextResponse.redirect(new URL('/dashboard', request.url))
    }
  }

  return NextResponse.next()
}

export const config = {
  matcher: ['/dashboard/:path*', '/login', '/register'],
}
```

## Metadata & SEO

```typescript
// app/layout.tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: {
    default: 'My App',
    template: '%s | My App',
  },
  description: 'My awesome application',
  openGraph: {
    title: 'My App',
    description: 'My awesome application',
    url: 'https://myapp.com',
    siteName: 'My App',
    images: [
      {
        url: 'https://myapp.com/og.jpg',
        width: 1200,
        height: 630,
      },
    ],
    locale: 'ko_KR',
    type: 'website',
  },
  robots: {
    index: true,
    follow: true,
  },
}

// app/posts/[id]/page.tsx
export async function generateMetadata({
  params,
}: {
  params: { id: string }
}): Promise<Metadata> {
  const post = await fetchPost(params.id)

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [post.coverImage],
    },
  }
}
```

## Environment Variables

```typescript
// 서버 전용 (NEXT_PUBLIC_ 접두사 없음)
const API_KEY = process.env.API_KEY

// 클라이언트에서도 사용 (NEXT_PUBLIC_ 접두사)
const ANALYTICS_ID = process.env.NEXT_PUBLIC_ANALYTICS_ID

// 타입 안전성
// env.mjs (with @t3-oss/env-nextjs)
import { createEnv } from '@t3-oss/env-nextjs'
import { z } from 'zod'

export const env = createEnv({
  server: {
    DATABASE_URL: z.string().url(),
    API_KEY: z.string().min(1),
  },
  client: {
    NEXT_PUBLIC_APP_URL: z.string().url(),
  },
  runtimeEnv: {
    DATABASE_URL: process.env.DATABASE_URL,
    API_KEY: process.env.API_KEY,
    NEXT_PUBLIC_APP_URL: process.env.NEXT_PUBLIC_APP_URL,
  },
})

// 사용
import { env } from '@/env.mjs'
console.log(env.DATABASE_URL) // 타입 안전!
```
