# TypeScript Best Practice Patterns

> 공식 문서: https://www.typescriptlang.org/docs

## 필수 설정

### tsconfig.json

```json
{
  "compilerOptions": {
    // 필수: strict 모드
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true,

    // 추가 안전성
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,

    // 모던 설정
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "esModuleInterop": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "isolatedModules": true
  }
}
```

## 타입 정의 패턴

### 기본 타입

```typescript
// 원시 타입
const name: string = 'Alice'
const age: number = 30
const isActive: boolean = true
const nothing: null = null
const notDefined: undefined = undefined

// 배열
const numbers: number[] = [1, 2, 3]
const names: Array<string> = ['a', 'b', 'c'] // 동일

// 튜플
const point: [number, number] = [10, 20]
const namedTuple: [x: number, y: number] = [10, 20]

// 객체
const user: { name: string; age: number } = { name: 'Alice', age: 30 }

// 유니온
const id: string | number = 'abc123'

// 리터럴
const direction: 'up' | 'down' | 'left' | 'right' = 'up'
```

### Interface vs Type

```typescript
// ✅ Interface: 객체 타입, 확장 가능
interface User {
  id: string
  name: string
  email: string
}

// 확장
interface Admin extends User {
  role: 'admin'
  permissions: string[]
}

// 선언 병합 (라이브러리 확장에 유용)
interface Window {
  customProperty: string
}

// ✅ Type: 유니온, 조건부 타입, 복잡한 타입
type Status = 'pending' | 'approved' | 'rejected'

type Response<T> = {
  data: T
  error: string | null
  loading: boolean
}

// 조건부 타입
type NonNullable<T> = T extends null | undefined ? never : T

// 매핑 타입
type Readonly<T> = { readonly [K in keyof T]: T[K] }
```

### React 컴포넌트 타입

```typescript
// Props 타입
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'ghost'
  size?: 'sm' | 'md' | 'lg'
  disabled?: boolean
  children: React.ReactNode
  onClick?: () => void
  className?: string
}

// ✅ 함수 컴포넌트
function Button({ variant = 'primary', children, ...props }: ButtonProps) {
  return <button {...props}>{children}</button>
}

// children을 받는 컴포넌트
interface LayoutProps {
  children: React.ReactNode
  sidebar?: React.ReactNode
}

// 이벤트 핸들러
interface FormProps {
  onSubmit: (data: FormData) => void
  onChange: React.ChangeEventHandler<HTMLInputElement>
  onClick: React.MouseEventHandler<HTMLButtonElement>
}

// HTML 요소 props 확장
interface CustomInputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string
  error?: string
}

function CustomInput({ label, error, ...props }: CustomInputProps) {
  return (
    <div>
      <label>{label}</label>
      <input {...props} />
      {error && <span>{error}</span>}
    </div>
  )
}

// forwardRef 컴포넌트
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string
}

const Input = React.forwardRef<HTMLInputElement, InputProps>(
  ({ label, ...props }, ref) => (
    <div>
      <label>{label}</label>
      <input ref={ref} {...props} />
    </div>
  )
)
Input.displayName = 'Input'
```

## 제네릭 패턴

### 기본 제네릭

```typescript
// 함수 제네릭
function identity<T>(value: T): T {
  return value
}

// 배열 유틸리티
function first<T>(arr: T[]): T | undefined {
  return arr[0]
}

function last<T>(arr: T[]): T | undefined {
  return arr[arr.length - 1]
}

// 여러 제네릭 파라미터
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second]
}

// 기본값
function createArray<T = string>(length: number, value: T): T[] {
  return Array(length).fill(value)
}
```

### 제네릭 제약

```typescript
// extends로 타입 제한
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key]
}

// 객체 타입 제약
function merge<T extends object, U extends object>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 }
}

// 생성자 제약
function createInstance<T>(ctor: new () => T): T {
  return new ctor()
}

// 함수 제약
function pipe<T, U, V>(
  value: T,
  fn1: (input: T) => U,
  fn2: (input: U) => V
): V {
  return fn2(fn1(value))
}
```

### 제네릭 컴포넌트

```typescript
// 제네릭 리스트 컴포넌트
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
    return <p>{emptyMessage}</p>
  }

  return (
    <ul>
      {items.map((item, index) => (
        <li key={keyExtractor(item)}>{renderItem(item, index)}</li>
      ))}
    </ul>
  )
}

// 사용
<List
  items={users}
  renderItem={(user) => <span>{user.name}</span>}
  keyExtractor={(user) => user.id}
/>

// 제네릭 Select 컴포넌트
interface SelectProps<T> {
  options: T[]
  value: T | null
  onChange: (value: T) => void
  getLabel: (option: T) => string
  getValue: (option: T) => string
}

function Select<T>({ options, value, onChange, getLabel, getValue }: SelectProps<T>) {
  return (
    <select
      value={value ? getValue(value) : ''}
      onChange={(e) => {
        const selected = options.find((opt) => getValue(opt) === e.target.value)
        if (selected) onChange(selected)
      }}
    >
      <option value="">선택하세요</option>
      {options.map((opt) => (
        <option key={getValue(opt)} value={getValue(opt)}>
          {getLabel(opt)}
        </option>
      ))}
    </select>
  )
}
```

## 타입 가드

### 기본 타입 가드

```typescript
// typeof 가드
function processValue(value: string | number) {
  if (typeof value === 'string') {
    return value.toUpperCase() // string으로 추론
  }
  return value.toFixed(2) // number로 추론
}

// instanceof 가드
class Dog {
  bark() { console.log('Woof!') }
}

class Cat {
  meow() { console.log('Meow!') }
}

function makeSound(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark()
  } else {
    animal.meow()
  }
}

// in 연산자
interface Bird {
  fly(): void
  layEggs(): void
}

interface Fish {
  swim(): void
  layEggs(): void
}

function move(animal: Bird | Fish) {
  if ('swim' in animal) {
    animal.swim()
  } else {
    animal.fly()
  }
}
```

### 커스텀 타입 가드

```typescript
// is 키워드 사용
interface User {
  type: 'user'
  name: string
  email: string
}

interface Admin {
  type: 'admin'
  name: string
  permissions: string[]
}

function isAdmin(person: User | Admin): person is Admin {
  return person.type === 'admin'
}

function greet(person: User | Admin) {
  if (isAdmin(person)) {
    console.log(`Admin ${person.name} with ${person.permissions.length} permissions`)
  } else {
    console.log(`User ${person.name} (${person.email})`)
  }
}

// null 체크 가드
function isNotNull<T>(value: T | null | undefined): value is T {
  return value != null
}

const values = [1, null, 2, undefined, 3]
const filtered = values.filter(isNotNull) // number[]

// API 응답 가드
interface SuccessResponse<T> {
  success: true
  data: T
}

interface ErrorResponse {
  success: false
  error: string
}

type ApiResponse<T> = SuccessResponse<T> | ErrorResponse

function isSuccess<T>(response: ApiResponse<T>): response is SuccessResponse<T> {
  return response.success === true
}

async function fetchUser(id: string) {
  const response: ApiResponse<User> = await api.get(`/users/${id}`)

  if (isSuccess(response)) {
    return response.data // User 타입
  }

  throw new Error(response.error)
}
```

### Discriminated Union

```typescript
// ✅ 판별 유니온 (권장 패턴)
type LoadingState = { status: 'loading' }
type SuccessState<T> = { status: 'success'; data: T }
type ErrorState = { status: 'error'; error: Error }

type AsyncState<T> = LoadingState | SuccessState<T> | ErrorState

function renderState<T>(state: AsyncState<T>, renderData: (data: T) => React.ReactNode) {
  switch (state.status) {
    case 'loading':
      return <Spinner />
    case 'success':
      return renderData(state.data)
    case 'error':
      return <Error message={state.error.message} />
  }
}

// 더 복잡한 예시
type Action =
  | { type: 'SET_USER'; payload: User }
  | { type: 'LOGOUT' }
  | { type: 'SET_ERROR'; payload: string }
  | { type: 'SET_LOADING'; payload: boolean }

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'SET_USER':
      return { ...state, user: action.payload, error: null }
    case 'LOGOUT':
      return { ...state, user: null }
    case 'SET_ERROR':
      return { ...state, error: action.payload }
    case 'SET_LOADING':
      return { ...state, loading: action.payload }
  }
}
```

## 유틸리티 타입

### 내장 유틸리티

```typescript
interface User {
  id: string
  name: string
  email: string
  age: number
  createdAt: Date
}

// Partial: 모든 속성을 optional로
type PartialUser = Partial<User>
// { id?: string; name?: string; ... }

// Required: 모든 속성을 필수로
type RequiredUser = Required<PartialUser>

// Pick: 특정 속성만 선택
type UserPreview = Pick<User, 'id' | 'name'>
// { id: string; name: string }

// Omit: 특정 속성 제외
type UserWithoutId = Omit<User, 'id'>
// { name: string; email: string; age: number; createdAt: Date }

// Record: 키-값 맵
type UserRoles = Record<string, 'admin' | 'user' | 'guest'>
// { [key: string]: 'admin' | 'user' | 'guest' }

// Exclude: 유니온에서 특정 타입 제외
type Status = 'pending' | 'approved' | 'rejected'
type ActiveStatus = Exclude<Status, 'rejected'>
// 'pending' | 'approved'

// Extract: 유니온에서 특정 타입만 추출
type StringStatus = Extract<string | number | boolean, string>
// string

// NonNullable: null, undefined 제거
type MaybeString = string | null | undefined
type DefinitelyString = NonNullable<MaybeString>
// string

// ReturnType: 함수 반환 타입 추출
function getUser() {
  return { id: '1', name: 'Alice' }
}
type UserReturn = ReturnType<typeof getUser>
// { id: string; name: string }

// Parameters: 함수 파라미터 타입 추출
function createUser(name: string, age: number) { /* ... */ }
type CreateUserParams = Parameters<typeof createUser>
// [string, number]

// Awaited: Promise 내부 타입 추출
type PromiseUser = Promise<User>
type ResolvedUser = Awaited<PromiseUser>
// User
```

### 커스텀 유틸리티

```typescript
// DeepPartial
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P]
}

// DeepReadonly
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P]
}

// Optional 특정 키만
type OptionalKeys<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>

type UserWithOptionalEmail = OptionalKeys<User, 'email'>
// { id: string; name: string; age: number; email?: string }

// Required 특정 키만
type RequiredKeys<T, K extends keyof T> = Omit<T, K> & Required<Pick<T, K>>

// Nullable
type Nullable<T> = T | null

// NonNullableFields: 모든 필드에서 null 제거
type NonNullableFields<T> = {
  [P in keyof T]: NonNullable<T[P]>
}

// ValueOf: 객체 값 타입 추출
type ValueOf<T> = T[keyof T]

const statusMap = {
  pending: 1,
  approved: 2,
  rejected: 3,
} as const

type StatusValue = ValueOf<typeof statusMap>
// 1 | 2 | 3
```

## as const와 불변성

### const assertion

```typescript
// as const로 리터럴 타입 고정
const routes = {
  home: '/',
  about: '/about',
  users: '/users',
} as const

type Route = (typeof routes)[keyof typeof routes]
// '/' | '/about' | '/users'

// 배열을 튜플로
const colors = ['red', 'green', 'blue'] as const
type Color = (typeof colors)[number]
// 'red' | 'green' | 'blue'

// enum 대체
const Status = {
  PENDING: 'pending',
  APPROVED: 'approved',
  REJECTED: 'rejected',
} as const

type StatusType = (typeof Status)[keyof typeof Status]
// 'pending' | 'approved' | 'rejected'
```

### satisfies 연산자 (TS 4.9+)

```typescript
// ✅ satisfies: 타입 체크 + 리터럴 타입 유지
const config = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retries: 3,
} satisfies Record<string, string | number>

// config.apiUrl은 string (리터럴 'https://...'이 아닌)
// 하지만 타입 체크는 됨

// 복잡한 예시
type ColorConfig = Record<string, [number, number, number] | string>

const palette = {
  primary: [0, 122, 255],
  secondary: '#666666',
  danger: [255, 0, 0],
} satisfies ColorConfig

// palette.primary는 [number, number, number]로 추론
// 튜플 메서드 사용 가능
palette.primary[0] // OK
```

## 에러 처리 타입

### Result 패턴

```typescript
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E }

function parseJson<T>(text: string): Result<T> {
  try {
    return { success: true, data: JSON.parse(text) }
  } catch (e) {
    return { success: false, error: e instanceof Error ? e : new Error(String(e)) }
  }
}

const result = parseJson<User>('{"name": "Alice"}')
if (result.success) {
  console.log(result.data.name) // User 타입
} else {
  console.error(result.error.message) // Error 타입
}
```

### try-catch 타입 안전성

```typescript
// ❌ error는 unknown
try {
  throw new Error('fail')
} catch (error) {
  // error는 unknown
  console.log(error.message) // 에러!
}

// ✅ 타입 가드 사용
try {
  throw new Error('fail')
} catch (error) {
  if (error instanceof Error) {
    console.log(error.message) // OK
  }
}

// ✅ 또는 unknown 타입 유틸리티
function getErrorMessage(error: unknown): string {
  if (error instanceof Error) return error.message
  if (typeof error === 'string') return error
  return 'Unknown error'
}

try {
  throw new Error('fail')
} catch (error) {
  console.log(getErrorMessage(error))
}
```

## 모듈 타입

### 타입 전용 import/export

```typescript
// ✅ 타입만 import (런타임에 제거됨)
import type { User, Post } from './types'

// 혼합 import
import { fetchUser, type User } from './api'

// ✅ 타입만 export
export type { User, Post }

// 또는 interface를 re-export
export type { UserResponse } from './api'
```

### 모듈 확장 (Declaration Merging)

```typescript
// next-auth 타입 확장
// types/next-auth.d.ts
import 'next-auth'

declare module 'next-auth' {
  interface Session {
    user: {
      id: string
      role: 'admin' | 'user'
    } & DefaultSession['user']
  }

  interface User {
    role: 'admin' | 'user'
  }
}

// Window 객체 확장
declare global {
  interface Window {
    gtag: (...args: unknown[]) => void
    dataLayer: unknown[]
  }
}

export {} // 모듈로 만들기 위해 필요
```
