# React 19 Best Practice Patterns

> 공식 문서: https://react.dev/reference

## React 19 새로운 기능

### use() Hook

```typescript
// ✅ Promise 직접 읽기
import { use } from 'react'

function UserProfile({ userPromise }: { userPromise: Promise<User> }) {
  const user = use(userPromise) // Suspense 필요
  return <div>{user.name}</div>
}

// 사용
<Suspense fallback={<Skeleton />}>
  <UserProfile userPromise={fetchUser(id)} />
</Suspense>

// ✅ Context 읽기 (useContext 대체)
function ThemeButton() {
  const theme = use(ThemeContext)
  return <button style={{ background: theme.primary }}>Click</button>
}

// 조건부 use (useContext와 달리 가능!)
function ConditionalComponent({ shouldUseTheme }: { shouldUseTheme: boolean }) {
  if (shouldUseTheme) {
    const theme = use(ThemeContext)
    return <div style={{ color: theme.text }}>Themed</div>
  }
  return <div>Not themed</div>
}
```

### Actions & useActionState

```typescript
// ✅ Form Actions (Next.js Server Actions와 함께)
'use client'

import { useActionState } from 'react'

function LoginForm() {
  const [state, formAction, isPending] = useActionState(
    async (prevState: FormState, formData: FormData) => {
      const email = formData.get('email')
      const password = formData.get('password')

      try {
        await login(email, password)
        return { success: true, error: null }
      } catch (e) {
        return { success: false, error: 'Invalid credentials' }
      }
    },
    { success: false, error: null }
  )

  return (
    <form action={formAction}>
      {state.error && <p className="text-red-500">{state.error}</p>}
      <input name="email" type="email" disabled={isPending} />
      <input name="password" type="password" disabled={isPending} />
      <button type="submit" disabled={isPending}>
        {isPending ? 'Logging in...' : 'Login'}
      </button>
    </form>
  )
}
```

### useOptimistic

```typescript
'use client'

import { useOptimistic, useTransition } from 'react'

interface Todo {
  id: string
  text: string
  completed: boolean
}

function TodoList({ todos }: { todos: Todo[] }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo: Todo) => [...state, newTodo]
  )
  const [isPending, startTransition] = useTransition()

  async function addTodo(formData: FormData) {
    const text = formData.get('text') as string
    const optimisticTodo: Todo = {
      id: crypto.randomUUID(),
      text,
      completed: false,
    }

    startTransition(async () => {
      addOptimisticTodo(optimisticTodo)
      await saveTodo(text) // 서버 저장
    })
  }

  return (
    <div>
      <form action={addTodo}>
        <input name="text" />
        <button type="submit">Add</button>
      </form>

      <ul>
        {optimisticTodos.map((todo) => (
          <li key={todo.id} style={{ opacity: todo.id.startsWith('temp-') ? 0.5 : 1 }}>
            {todo.text}
          </li>
        ))}
      </ul>
    </div>
  )
}
```

## 컴포넌트 설계 패턴

### Compound Components

```typescript
// 복합 컴포넌트 패턴
import { createContext, useContext, useState, ReactNode } from 'react'

interface TabsContextType {
  activeTab: string
  setActiveTab: (tab: string) => void
}

const TabsContext = createContext<TabsContextType | null>(null)

function useTabs() {
  const context = useContext(TabsContext)
  if (!context) throw new Error('useTabs must be used within Tabs')
  return context
}

interface TabsProps {
  defaultTab: string
  children: ReactNode
}

function Tabs({ defaultTab, children }: TabsProps) {
  const [activeTab, setActiveTab] = useState(defaultTab)

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  )
}

function TabList({ children }: { children: ReactNode }) {
  return <div className="tab-list flex gap-2">{children}</div>
}

function Tab({ value, children }: { value: string; children: ReactNode }) {
  const { activeTab, setActiveTab } = useTabs()

  return (
    <button
      className={`tab ${activeTab === value ? 'active' : ''}`}
      onClick={() => setActiveTab(value)}
    >
      {children}
    </button>
  )
}

function TabPanel({ value, children }: { value: string; children: ReactNode }) {
  const { activeTab } = useTabs()
  if (activeTab !== value) return null
  return <div className="tab-panel">{children}</div>
}

// 네임스페이스 할당
Tabs.List = TabList
Tabs.Tab = Tab
Tabs.Panel = TabPanel

// 사용
<Tabs defaultTab="profile">
  <Tabs.List>
    <Tabs.Tab value="profile">프로필</Tabs.Tab>
    <Tabs.Tab value="settings">설정</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panel value="profile">프로필 내용</Tabs.Panel>
  <Tabs.Panel value="settings">설정 내용</Tabs.Panel>
</Tabs>
```

### Render Props

```typescript
// 렌더 프롭스 패턴
interface MousePosition {
  x: number
  y: number
}

interface MouseTrackerProps {
  children: (position: MousePosition) => ReactNode
}

function MouseTracker({ children }: MouseTrackerProps) {
  const [position, setPosition] = useState({ x: 0, y: 0 })

  useEffect(() => {
    const handleMove = (e: MouseEvent) => {
      setPosition({ x: e.clientX, y: e.clientY })
    }
    window.addEventListener('mousemove', handleMove)
    return () => window.removeEventListener('mousemove', handleMove)
  }, [])

  return <>{children(position)}</>
}

// 사용
<MouseTracker>
  {({ x, y }) => (
    <div>
      마우스 위치: {x}, {y}
    </div>
  )}
</MouseTracker>
```

### Controlled vs Uncontrolled

```typescript
// ✅ Controlled: 부모가 상태 관리
interface ControlledInputProps {
  value: string
  onChange: (value: string) => void
}

function ControlledInput({ value, onChange }: ControlledInputProps) {
  return (
    <input
      value={value}
      onChange={(e) => onChange(e.target.value)}
    />
  )
}

// ✅ Uncontrolled: 컴포넌트가 내부 상태 관리
interface UncontrolledInputProps {
  defaultValue?: string
  onSubmit?: (value: string) => void
}

function UncontrolledInput({ defaultValue, onSubmit }: UncontrolledInputProps) {
  const inputRef = useRef<HTMLInputElement>(null)

  const handleSubmit = () => {
    if (inputRef.current && onSubmit) {
      onSubmit(inputRef.current.value)
    }
  }

  return (
    <div>
      <input ref={inputRef} defaultValue={defaultValue} />
      <button onClick={handleSubmit}>Submit</button>
    </div>
  )
}

// ✅ 하이브리드: 둘 다 지원
interface HybridInputProps {
  value?: string
  defaultValue?: string
  onChange?: (value: string) => void
}

function HybridInput({ value, defaultValue, onChange }: HybridInputProps) {
  const [internalValue, setInternalValue] = useState(defaultValue ?? '')
  const isControlled = value !== undefined

  const currentValue = isControlled ? value : internalValue

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const newValue = e.target.value
    if (!isControlled) {
      setInternalValue(newValue)
    }
    onChange?.(newValue)
  }

  return <input value={currentValue} onChange={handleChange} />
}
```

## Custom Hooks 패턴

### 상태 추상화

```typescript
// useToggle
function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue)

  const toggle = useCallback(() => setValue((v) => !v), [])
  const setTrue = useCallback(() => setValue(true), [])
  const setFalse = useCallback(() => setValue(false), [])

  return { value, toggle, setTrue, setFalse } as const
}

// 사용
const { value: isOpen, toggle, setFalse: close } = useToggle()

// useLocalStorage
function useLocalStorage<T>(key: string, initialValue: T) {
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

// 사용
const [theme, setTheme] = useLocalStorage('theme', 'light')
```

### 비동기 상태

```typescript
// useAsync
interface AsyncState<T> {
  data: T | null
  loading: boolean
  error: Error | null
}

function useAsync<T>(asyncFunction: () => Promise<T>, deps: DependencyList = []) {
  const [state, setState] = useState<AsyncState<T>>({
    data: null,
    loading: true,
    error: null,
  })

  useEffect(() => {
    let mounted = true

    setState((prev) => ({ ...prev, loading: true }))

    asyncFunction()
      .then((data) => {
        if (mounted) {
          setState({ data, loading: false, error: null })
        }
      })
      .catch((error) => {
        if (mounted) {
          setState({ data: null, loading: false, error })
        }
      })

    return () => {
      mounted = false
    }
  }, deps)

  return state
}

// 사용
const { data: user, loading, error } = useAsync(() => fetchUser(id), [id])

// useFetch (with abort)
function useFetch<T>(url: string, options?: RequestInit) {
  const [state, setState] = useState<AsyncState<T>>({
    data: null,
    loading: true,
    error: null,
  })

  useEffect(() => {
    const controller = new AbortController()

    setState((prev) => ({ ...prev, loading: true }))

    fetch(url, { ...options, signal: controller.signal })
      .then((res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`)
        return res.json()
      })
      .then((data) => setState({ data, loading: false, error: null }))
      .catch((error) => {
        if (error.name !== 'AbortError') {
          setState({ data: null, loading: false, error })
        }
      })

    return () => controller.abort()
  }, [url])

  return state
}
```

### UI 유틸리티

```typescript
// useDebounce
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value)

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay)
    return () => clearTimeout(timer)
  }, [value, delay])

  return debouncedValue
}

// useMediaQuery
function useMediaQuery(query: string): boolean {
  const [matches, setMatches] = useState(false)

  useEffect(() => {
    const media = window.matchMedia(query)
    setMatches(media.matches)

    const listener = (e: MediaQueryListEvent) => setMatches(e.matches)
    media.addEventListener('change', listener)
    return () => media.removeEventListener('change', listener)
  }, [query])

  return matches
}

// 사용
const isMobile = useMediaQuery('(max-width: 768px)')

// useOnClickOutside
function useOnClickOutside<T extends HTMLElement>(
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

// 사용
const modalRef = useRef<HTMLDivElement>(null)
useOnClickOutside(modalRef, () => setIsOpen(false))
```

### 복잡한 상태 관리

```typescript
// useReducer 패턴
interface State {
  count: number
  step: number
}

type Action =
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' }
  | { type: 'SET_STEP'; payload: number }
  | { type: 'RESET' }

const initialState: State = { count: 0, step: 1 }

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + state.step }
    case 'DECREMENT':
      return { ...state, count: state.count - state.step }
    case 'SET_STEP':
      return { ...state, step: action.payload }
    case 'RESET':
      return initialState
    default:
      return state
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState)

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
      <input
        type="number"
        value={state.step}
        onChange={(e) =>
          dispatch({ type: 'SET_STEP', payload: Number(e.target.value) })
        }
      />
      <button onClick={() => dispatch({ type: 'RESET' })}>Reset</button>
    </div>
  )
}
```

## 성능 최적화

### React.memo (React 19 이전)

```typescript
// React 19부터는 대부분 불필요 (React Compiler)
// 하지만 아직 필요한 경우:

// 기본 사용
const ExpensiveList = memo(function ExpensiveList({ items }: { items: Item[] }) {
  return (
    <ul>
      {items.map((item) => (
        <ExpensiveItem key={item.id} item={item} />
      ))}
    </ul>
  )
})

// 커스텀 비교 함수
const UserCard = memo(
  function UserCard({ user }: { user: User }) {
    return <div>{user.name}</div>
  },
  (prevProps, nextProps) => {
    // true 반환 시 리렌더링 스킵
    return prevProps.user.id === nextProps.user.id
  }
)
```

### Suspense 경계

```typescript
// 최적화된 Suspense 배치
function Dashboard() {
  return (
    <div className="grid grid-cols-3 gap-4">
      {/* 개별 Suspense - 독립적으로 로딩 */}
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

// 중첩 Suspense
function App() {
  return (
    <Suspense fallback={<FullPageLoader />}>
      <Layout>
        <Suspense fallback={<ContentSkeleton />}>
          <Content />
        </Suspense>
      </Layout>
    </Suspense>
  )
}
```

### useTransition

```typescript
function SearchPage() {
  const [query, setQuery] = useState('')
  const [isPending, startTransition] = useTransition()
  const [results, setResults] = useState<SearchResult[]>([])

  const handleSearch = (e: React.ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value
    setQuery(value) // 즉시 업데이트 (긴급)

    startTransition(async () => {
      // 낮은 우선순위로 실행
      const data = await searchApi(value)
      setResults(data)
    })
  }

  return (
    <div>
      <input value={query} onChange={handleSearch} />
      {isPending && <Spinner />}
      <ResultList results={results} />
    </div>
  )
}
```

### useDeferredValue

```typescript
function SearchResults({ query }: { query: string }) {
  const deferredQuery = useDeferredValue(query)
  const isStale = query !== deferredQuery

  const results = useMemo(
    () => filterHugeList(deferredQuery),
    [deferredQuery]
  )

  return (
    <div style={{ opacity: isStale ? 0.5 : 1 }}>
      {results.map((item) => (
        <div key={item.id}>{item.name}</div>
      ))}
    </div>
  )
}
```

## 에러 처리

### Error Boundary with useErrorBoundary

```typescript
// react-error-boundary 라이브러리 사용 권장
import { ErrorBoundary, useErrorBoundary } from 'react-error-boundary'

function ErrorFallback({ error, resetErrorBoundary }: FallbackProps) {
  return (
    <div className="p-4 bg-red-50 rounded-lg">
      <h2 className="text-red-800 font-bold">오류가 발생했습니다</h2>
      <p className="text-red-600">{error.message}</p>
      <button
        onClick={resetErrorBoundary}
        className="mt-2 px-4 py-2 bg-red-600 text-white rounded"
      >
        다시 시도
      </button>
    </div>
  )
}

function App() {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onReset={() => {
        // 리셋 시 수행할 작업
      }}
    >
      <Dashboard />
    </ErrorBoundary>
  )
}

// 명시적 에러 throw
function DataComponent() {
  const { showBoundary } = useErrorBoundary()

  const handleClick = async () => {
    try {
      await riskyOperation()
    } catch (error) {
      showBoundary(error)
    }
  }

  return <button onClick={handleClick}>위험한 작업</button>
}
```

## 접근성

### Focus 관리

```typescript
function Modal({ isOpen, onClose, children }: ModalProps) {
  const closeButtonRef = useRef<HTMLButtonElement>(null)
  const modalRef = useRef<HTMLDivElement>(null)

  // 모달 열릴 때 포커스 이동
  useEffect(() => {
    if (isOpen) {
      closeButtonRef.current?.focus()
    }
  }, [isOpen])

  // 포커스 트랩
  useEffect(() => {
    if (!isOpen) return

    const modal = modalRef.current
    if (!modal) return

    const focusableElements = modal.querySelectorAll(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    )
    const firstElement = focusableElements[0] as HTMLElement
    const lastElement = focusableElements[focusableElements.length - 1] as HTMLElement

    const handleTab = (e: KeyboardEvent) => {
      if (e.key !== 'Tab') return

      if (e.shiftKey && document.activeElement === firstElement) {
        e.preventDefault()
        lastElement.focus()
      } else if (!e.shiftKey && document.activeElement === lastElement) {
        e.preventDefault()
        firstElement.focus()
      }
    }

    document.addEventListener('keydown', handleTab)
    return () => document.removeEventListener('keydown', handleTab)
  }, [isOpen])

  if (!isOpen) return null

  return (
    <div
      ref={modalRef}
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
    >
      <button ref={closeButtonRef} onClick={onClose} aria-label="닫기">
        X
      </button>
      <h2 id="modal-title">제목</h2>
      {children}
    </div>
  )
}
```

### ARIA 패턴

```typescript
// 아코디언
function Accordion({ items }: { items: AccordionItem[] }) {
  const [openIndex, setOpenIndex] = useState<number | null>(null)

  return (
    <div>
      {items.map((item, index) => (
        <div key={item.id}>
          <button
            id={`accordion-header-${item.id}`}
            aria-expanded={openIndex === index}
            aria-controls={`accordion-panel-${item.id}`}
            onClick={() => setOpenIndex(openIndex === index ? null : index)}
          >
            {item.title}
          </button>
          <div
            id={`accordion-panel-${item.id}`}
            role="region"
            aria-labelledby={`accordion-header-${item.id}`}
            hidden={openIndex !== index}
          >
            {item.content}
          </div>
        </div>
      ))}
    </div>
  )
}

// 알림
function Toast({ message, type }: ToastProps) {
  return (
    <div
      role="alert"
      aria-live="polite"
      className={`toast toast-${type}`}
    >
      {message}
    </div>
  )
}
```
