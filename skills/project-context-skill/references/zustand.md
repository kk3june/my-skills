# Zustand 패턴

> 공식 문서: https://docs.pmnd.rs/zustand

## 기본 Store

```typescript
import { create } from 'zustand'

interface CounterStore {
  count: number
  increment: () => void
  decrement: () => void
  reset: () => void
}

const useCounterStore = create<CounterStore>((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
}))

// 사용
function Counter() {
  const { count, increment } = useCounterStore()
  return <button onClick={increment}>{count}</button>
}
```

## Selector로 최적화

```typescript
// 전체 구독 (리렌더링 많음)
const { count, user } = useStore()

// 선택적 구독 (권장)
const count = useStore((state) => state.count)
const user = useStore((state) => state.user)

// 여러 값 선택 (shallow 비교)
import { shallow } from 'zustand/shallow'

const { count, user } = useStore(
  (state) => ({ count: state.count, user: state.user }),
  shallow
)
```

## Immer Middleware

```typescript
import { create } from 'zustand'
import { immer } from 'zustand/middleware/immer'

interface TodoStore {
  todos: Todo[]
  addTodo: (todo: Todo) => void
  toggleTodo: (id: string) => void
}

const useTodoStore = create<TodoStore>()(
  immer((set) => ({
    todos: [],
    addTodo: (todo) =>
      set((state) => {
        state.todos.push(todo)  // 직접 수정 가능
      }),
    toggleTodo: (id) =>
      set((state) => {
        const todo = state.todos.find((t) => t.id === id)
        if (todo) todo.completed = !todo.completed
      }),
  }))
)
```

## Persist Middleware

```typescript
import { create } from 'zustand'
import { persist, createJSONStorage } from 'zustand/middleware'

const useStore = create<Store>()(
  persist(
    (set) => ({
      // state & actions
    }),
    {
      name: 'app-storage',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({ user: state.user }),  // 일부만 저장
    }
  )
)
```

## Slice 패턴 (대규모 스토어)

```typescript
// slices/authSlice.ts
export interface AuthSlice {
  user: User | null
  token: string | null
  login: (user: User, token: string) => void
  logout: () => void
}

export const createAuthSlice = (set): AuthSlice => ({
  user: null,
  token: null,
  login: (user, token) => set({ user, token }),
  logout: () => set({ user: null, token: null }),
})

// slices/cartSlice.ts
export interface CartSlice {
  items: CartItem[]
  addItem: (item: CartItem) => void
  clearCart: () => void
}

export const createCartSlice = (set, get): CartSlice => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
  clearCart: () => set({ items: [] }),
})

// store.ts
import { create } from 'zustand'
import { createAuthSlice, AuthSlice } from './slices/authSlice'
import { createCartSlice, CartSlice } from './slices/cartSlice'

type Store = AuthSlice & CartSlice

export const useStore = create<Store>()((...args) => ({
  ...createAuthSlice(...args),
  ...createCartSlice(...args),
}))
```

## 비동기 Actions

```typescript
const useStore = create<Store>((set, get) => ({
  users: [],
  isLoading: false,
  error: null,

  fetchUsers: async () => {
    set({ isLoading: true, error: null })
    try {
      const users = await api.getUsers()
      set({ users, isLoading: false })
    } catch (error) {
      set({ error: error.message, isLoading: false })
    }
  },
}))
```

## Store 외부에서 접근

```typescript
// 컴포넌트 외부에서 상태 읽기
const count = useStore.getState().count

// 컴포넌트 외부에서 상태 변경
useStore.setState({ count: 10 })

// 구독
const unsubscribe = useStore.subscribe(
  (state) => console.log('State changed:', state)
)
```

## DevTools

```typescript
import { devtools } from 'zustand/middleware'

const useStore = create<Store>()(
  devtools(
    (set) => ({
      // state & actions
    }),
    { name: 'MyStore' }
  )
)
```
