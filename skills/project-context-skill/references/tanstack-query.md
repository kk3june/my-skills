# TanStack Query 패턴

> 공식 문서: https://tanstack.com/query/latest/docs

## 기본 Query

```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'

// 기본 조회
function UserProfile({ userId }: { userId: string }) {
  const { data, isLoading, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  })

  if (isLoading) return <Spinner />
  if (error) return <Error message={error.message} />
  return <div>{data.name}</div>
}
```

## Query Key 컨벤션

```typescript
// 계층적 구조 권장
['users']                    // 전체 목록
['users', userId]            // 단일 항목
['users', userId, 'posts']   // 관계 데이터
['users', { status, page }]  // 필터링된 목록

// queryKeys 팩토리 패턴
export const userKeys = {
  all: ['users'] as const,
  lists: () => [...userKeys.all, 'list'] as const,
  list: (filters: Filters) => [...userKeys.lists(), filters] as const,
  details: () => [...userKeys.all, 'detail'] as const,
  detail: (id: string) => [...userKeys.details(), id] as const,
}

// 사용
useQuery({
  queryKey: userKeys.detail(userId),
  queryFn: () => fetchUser(userId),
})
```

## Query Options

```typescript
const { data } = useQuery({
  queryKey: ['user', userId],
  queryFn: () => fetchUser(userId),
  staleTime: 5 * 60 * 1000,       // 5분간 fresh
  gcTime: 30 * 60 * 1000,         // 30분간 캐시 유지
  retry: 3,                        // 실패 시 재시도
  refetchOnWindowFocus: false,     // 포커스 시 재요청 비활성화
  enabled: !!userId,               // 조건부 실행
})
```

## Mutation

```typescript
function CreatePost() {
  const queryClient = useQueryClient()

  const mutation = useMutation({
    mutationFn: (newPost: NewPost) => createPost(newPost),
    onSuccess: () => {
      // 캐시 무효화
      queryClient.invalidateQueries({ queryKey: ['posts'] })
    },
  })

  return (
    <button
      onClick={() => mutation.mutate({ title: 'New Post' })}
      disabled={mutation.isPending}
    >
      {mutation.isPending ? '저장 중...' : '저장'}
    </button>
  )
}
```

## Optimistic Update

```typescript
const mutation = useMutation({
  mutationFn: updateTodo,
  onMutate: async (newTodo) => {
    // 진행 중인 refetch 취소
    await queryClient.cancelQueries({ queryKey: ['todos', newTodo.id] })

    // 이전 값 저장
    const previousTodo = queryClient.getQueryData(['todos', newTodo.id])

    // 낙관적 업데이트
    queryClient.setQueryData(['todos', newTodo.id], newTodo)

    return { previousTodo }
  },
  onError: (err, newTodo, context) => {
    // 롤백
    queryClient.setQueryData(['todos', newTodo.id], context?.previousTodo)
  },
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ['todos'] })
  },
})
```

## Infinite Query

```typescript
const {
  data,
  fetchNextPage,
  hasNextPage,
  isFetchingNextPage,
} = useInfiniteQuery({
  queryKey: ['posts'],
  queryFn: ({ pageParam }) => fetchPosts(pageParam),
  initialPageParam: 0,
  getNextPageParam: (lastPage) => lastPage.nextCursor,
})

// 데이터 접근
const allPosts = data?.pages.flatMap(page => page.items) ?? []
```

## Prefetching

```typescript
// 마우스 호버 시 prefetch
function PostLink({ postId }: { postId: string }) {
  const queryClient = useQueryClient()

  const prefetchPost = () => {
    queryClient.prefetchQuery({
      queryKey: ['post', postId],
      queryFn: () => fetchPost(postId),
      staleTime: 60 * 1000,
    })
  }

  return (
    <Link href={`/posts/${postId}`} onMouseEnter={prefetchPost}>
      게시글 보기
    </Link>
  )
}
```

## Suspense 모드

```typescript
import { useSuspenseQuery } from '@tanstack/react-query'

function UserProfile({ userId }: { userId: string }) {
  // Suspense boundary 필요
  const { data } = useSuspenseQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  })

  return <div>{data.name}</div>
}

// 사용
<Suspense fallback={<Loading />}>
  <UserProfile userId="1" />
</Suspense>
```

## Provider 설정

```typescript
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60 * 1000,
      retry: 1,
    },
  },
})

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Children />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  )
}
```
