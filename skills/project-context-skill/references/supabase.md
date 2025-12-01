# Supabase 패턴

> 공식 문서: https://supabase.com/docs

## 클라이언트 설정

```typescript
// lib/supabase/client.ts (브라우저용)
import { createBrowserClient } from '@supabase/ssr'

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}

// lib/supabase/server.ts (서버용 - Next.js App Router)
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'

export async function createClient() {
  const cookieStore = await cookies()

  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll()
        },
        setAll(cookiesToSet) {
          cookiesToSet.forEach(({ name, value, options }) =>
            cookieStore.set(name, value, options)
          )
        },
      },
    }
  )
}
```

## 인증

### 회원가입 / 로그인

```typescript
const supabase = createClient()

// 이메일 회원가입
const { data, error } = await supabase.auth.signUp({
  email: 'user@example.com',
  password: 'password123',
})

// 이메일 로그인
const { data, error } = await supabase.auth.signInWithPassword({
  email: 'user@example.com',
  password: 'password123',
})

// OAuth 로그인
const { data, error } = await supabase.auth.signInWithOAuth({
  provider: 'google',
  options: {
    redirectTo: `${window.location.origin}/auth/callback`,
  },
})

// 로그아웃
await supabase.auth.signOut()
```

### 세션 확인

```typescript
// 현재 사용자
const { data: { user } } = await supabase.auth.getUser()

// 세션 가져오기
const { data: { session } } = await supabase.auth.getSession()

// 인증 상태 변경 구독
supabase.auth.onAuthStateChange((event, session) => {
  if (event === 'SIGNED_IN') {
    // 로그인됨
  } else if (event === 'SIGNED_OUT') {
    // 로그아웃됨
  }
})
```

## 데이터베이스 쿼리

### 기본 CRUD

```typescript
// 조회
const { data, error } = await supabase
  .from('posts')
  .select('*')

// 조건 조회
const { data, error } = await supabase
  .from('posts')
  .select('*')
  .eq('status', 'published')
  .order('created_at', { ascending: false })
  .limit(10)

// 삽입
const { data, error } = await supabase
  .from('posts')
  .insert({ title: 'New Post', content: 'Content' })
  .select()
  .single()

// 수정
const { data, error } = await supabase
  .from('posts')
  .update({ title: 'Updated Title' })
  .eq('id', postId)
  .select()
  .single()

// 삭제
const { error } = await supabase
  .from('posts')
  .delete()
  .eq('id', postId)
```

### 관계 데이터 조회

```typescript
// 1:N 관계 (posts → comments)
const { data } = await supabase
  .from('posts')
  .select(`
    id,
    title,
    comments (
      id,
      content,
      user:users (name)
    )
  `)
  .eq('id', postId)
  .single()

// N:1 관계 (posts → author)
const { data } = await supabase
  .from('posts')
  .select(`
    *,
    author:users!author_id (
      id,
      name,
      avatar_url
    )
  `)
```

### 필터링

```typescript
const { data } = await supabase
  .from('posts')
  .select('*')
  .eq('status', 'published')      // =
  .neq('type', 'draft')           // !=
  .gt('views', 100)               // >
  .gte('likes', 10)               // >=
  .lt('price', 1000)              // <
  .in('category', ['tech', 'dev']) // IN
  .like('title', '%Next%')        // LIKE
  .ilike('title', '%next%')       // ILIKE (대소문자 무시)
  .is('deleted_at', null)         // IS NULL
  .or('status.eq.published,featured.eq.true') // OR
```

## 실시간 구독

```typescript
// 변경 사항 구독
const channel = supabase
  .channel('posts-changes')
  .on(
    'postgres_changes',
    {
      event: '*',  // INSERT, UPDATE, DELETE, *
      schema: 'public',
      table: 'posts',
      filter: 'status=eq.published',
    },
    (payload) => {
      console.log('Change:', payload)
    }
  )
  .subscribe()

// 구독 해제
supabase.removeChannel(channel)
```

## 스토리지

```typescript
// 파일 업로드
const { data, error } = await supabase.storage
  .from('avatars')
  .upload(`${userId}/avatar.png`, file, {
    cacheControl: '3600',
    upsert: true,
  })

// 공개 URL 가져오기
const { data } = supabase.storage
  .from('avatars')
  .getPublicUrl(`${userId}/avatar.png`)

// 서명된 URL (비공개 버킷용)
const { data, error } = await supabase.storage
  .from('private')
  .createSignedUrl('path/to/file.pdf', 3600) // 1시간

// 파일 삭제
const { error } = await supabase.storage
  .from('avatars')
  .remove([`${userId}/avatar.png`])
```

## 타입 생성

```bash
# CLI로 타입 생성
npx supabase gen types typescript --project-id your-project-id > types/supabase.ts
```

```typescript
// 타입 적용
import { Database } from '@/types/supabase'

const supabase = createClient<Database>()

// 자동 완성 지원
const { data } = await supabase
  .from('posts')  // 테이블 자동 완성
  .select('title, content')  // 컬럼 자동 완성
```

## RPC (서버 함수 호출)

```typescript
// PostgreSQL 함수 호출
const { data, error } = await supabase.rpc('get_user_stats', {
  user_id: userId,
})
```
