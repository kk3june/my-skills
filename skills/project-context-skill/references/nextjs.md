# Next.js 패턴

> 공식 문서: https://nextjs.org/docs

## Server/Client 컴포넌트

### Server Component (기본값)

```typescript
// app/users/page.tsx
async function UsersPage() {
  const users = await fetchUsers() // 서버에서 직접 fetching
  return <UserList users={users} />
}
```

### Client Component

```typescript
// components/Counter.tsx
'use client'

import { useState } from 'react'

export function Counter() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>
}
```

**규칙**: `useState`, `useEffect`, `onClick` → `'use client'` 필요

## Data Fetching

### 캐싱 전략

```typescript
// 정적 (빌드 시)
fetch(url, { cache: 'force-cache' })

// 동적 (매 요청)
fetch(url, { cache: 'no-store' })

// 시간 기반 재검증
fetch(url, { next: { revalidate: 3600 } })

// 태그 기반 재검증
fetch(url, { next: { tags: ['posts'] } })
```

### 재검증 트리거

```typescript
import { revalidateTag, revalidatePath } from 'next/cache'

export async function createPost(data: PostData) {
  await db.post.create({ data })
  revalidateTag('posts')    // 태그 기반
  revalidatePath('/posts')  // 경로 기반
}
```

## Server Actions

```typescript
// app/actions.ts
'use server'

import { revalidatePath } from 'next/cache'
import { redirect } from 'next/navigation'

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string
  await db.post.create({ data: { title } })
  revalidatePath('/posts')
  redirect('/posts')
}
```

```typescript
// 사용
<form action={createPost}>
  <input name="title" />
  <button type="submit">저장</button>
</form>
```

## App Router 구조

```
app/
├── (auth)/                 # Route Group
│   ├── login/page.tsx
│   └── register/page.tsx
├── (main)/
│   ├── layout.tsx          # 공유 레이아웃
│   ├── dashboard/page.tsx
│   └── settings/page.tsx
├── api/                    # API Routes
├── layout.tsx              # 루트 레이아웃
├── page.tsx
├── error.tsx               # 에러 UI
├── loading.tsx             # 로딩 UI
└── not-found.tsx           # 404 UI
```

## Metadata

```typescript
// 정적
export const metadata: Metadata = {
  title: 'My App',
  description: 'Description',
}

// 동적
export async function generateMetadata({ params }): Promise<Metadata> {
  const post = await fetchPost(params.id)
  return { title: post.title }
}
```

## Image 최적화

```typescript
import Image from 'next/image'

<Image
  src="/hero.jpg"
  alt="Hero"
  fill
  sizes="(max-width: 768px) 100vw, 50vw"
  priority  // LCP 이미지
  className="object-cover"
/>
```
