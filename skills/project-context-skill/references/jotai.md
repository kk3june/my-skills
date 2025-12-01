# Jotai 패턴

> 공식 문서: https://jotai.org/docs

## 기본 Atom

### Primitive Atom

```typescript
import { atom, useAtom, useAtomValue, useSetAtom } from 'jotai'

// 정의
const countAtom = atom(0)
const textAtom = atom('')

// 사용
function Counter() {
  const [count, setCount] = useAtom(countAtom)      // 읽기 + 쓰기
  const count = useAtomValue(countAtom)              // 읽기만
  const setCount = useSetAtom(countAtom)             // 쓰기만

  return <button onClick={() => setCount(c => c + 1)}>{count}</button>
}
```

### Derived Atom (읽기 전용)

```typescript
const todosAtom = atom<Todo[]>([])

// 파생 atom
const completedTodosAtom = atom(
  (get) => get(todosAtom).filter(todo => todo.completed)
)

const todoCountAtom = atom(
  (get) => get(todosAtom).length
)
```

### Writable Derived Atom

```typescript
const todosAtom = atom<Todo[]>([])

// 읽기 + 쓰기 가능한 파생 atom
const todoAtom = atom(
  (get) => get(todosAtom),
  (get, set, newTodo: Todo) => {
    set(todosAtom, [...get(todosAtom), newTodo])
  }
)
```

## Async Atom

```typescript
// 비동기 읽기
const userAtom = atom(async () => {
  const response = await fetch('/api/user')
  return response.json()
})

// 사용 (Suspense 필요)
function UserProfile() {
  const user = useAtomValue(userAtom)
  return <div>{user.name}</div>
}

// Suspense로 감싸기
<Suspense fallback={<Loading />}>
  <UserProfile />
</Suspense>
```

## Atom with Storage (Persistence)

```typescript
import { atomWithStorage } from 'jotai/utils'

// localStorage 자동 동기화
const themeAtom = atomWithStorage('theme', 'light')
const userPrefsAtom = atomWithStorage('prefs', { notifications: true })

// sessionStorage
const sessionAtom = atomWithStorage('session', null, sessionStorage)
```

## Reset Atom

```typescript
import { atomWithReset, useResetAtom, RESET } from 'jotai/utils'

const formAtom = atomWithReset({
  name: '',
  email: '',
})

function Form() {
  const [form, setForm] = useAtom(formAtom)
  const resetForm = useResetAtom(formAtom)

  return (
    <form>
      <input value={form.name} onChange={e => setForm(f => ({ ...f, name: e.target.value }))} />
      <button type="button" onClick={resetForm}>초기화</button>
    </form>
  )
}
```

## Atom Family (동적 Atom)

```typescript
import { atomFamily } from 'jotai/utils'

// ID별로 다른 atom 생성
const todoAtomFamily = atomFamily((id: string) =>
  atom(async () => {
    const response = await fetch(`/api/todos/${id}`)
    return response.json()
  })
)

// 사용
function TodoItem({ id }: { id: string }) {
  const todo = useAtomValue(todoAtomFamily(id))
  return <div>{todo.title}</div>
}
```

## 상태 분리 패턴

```typescript
// atoms/auth.ts
export const userAtom = atom<User | null>(null)
export const isAuthenticatedAtom = atom((get) => get(userAtom) !== null)

// atoms/cart.ts
export const cartItemsAtom = atom<CartItem[]>([])
export const cartTotalAtom = atom((get) =>
  get(cartItemsAtom).reduce((sum, item) => sum + item.price * item.quantity, 0)
)

// atoms/ui.ts
export const sidebarOpenAtom = atom(false)
export const modalAtom = atom<{ type: string; data?: unknown } | null>(null)
```

## DevTools

```typescript
import { useAtomsDebugValue } from 'jotai-devtools'

function DebugAtoms() {
  useAtomsDebugValue()
  return null
}

// App에 추가
<>
  <DebugAtoms />
  <App />
</>
```

## Jotai + TanStack Query

```typescript
import { atomWithQuery, atomWithMutation } from 'jotai-tanstack-query'

const userQueryAtom = atomWithQuery(() => ({
  queryKey: ['user'],
  queryFn: fetchUser,
}))

const updateUserMutation = atomWithMutation(() => ({
  mutationFn: updateUser,
}))
```
