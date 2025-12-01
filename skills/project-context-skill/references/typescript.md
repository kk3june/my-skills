# TypeScript 패턴

> 공식 문서: https://www.typescriptlang.org/docs

## 타입 정의

### Interface vs Type

```typescript
// Interface - 확장 가능, 선언 병합
interface User {
  id: string
  name: string
}

interface User {  // 선언 병합
  email: string
}

// Type - 유니온, 교차 타입에 적합
type Status = 'pending' | 'success' | 'error'
type UserWithRole = User & { role: Role }
```

### 권장 사용법

```typescript
// 객체 형태 → interface
interface ButtonProps {
  variant: 'primary' | 'secondary'
  children: React.ReactNode
}

// 유니온, 튜플, 기본 타입 별칭 → type
type ID = string | number
type Coordinates = [number, number]
type EventHandler = () => void
```

## Utility Types

```typescript
interface User {
  id: string
  name: string
  email: string
  age?: number
}

// Partial - 모든 속성 optional
type PartialUser = Partial<User>

// Required - 모든 속성 required
type RequiredUser = Required<User>

// Pick - 특정 속성만 선택
type UserCredentials = Pick<User, 'email' | 'name'>

// Omit - 특정 속성 제외
type UserWithoutId = Omit<User, 'id'>

// Record - 키-값 매핑
type UserRoles = Record<string, 'admin' | 'user'>

// Readonly - 읽기 전용
type ReadonlyUser = Readonly<User>
```

## Generic 패턴

```typescript
// 기본 Generic
function identity<T>(arg: T): T {
  return arg
}

// 제약 조건
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key]
}

// 기본값
interface ApiResponse<T = unknown> {
  data: T
  status: number
}

// 여러 Generic
function merge<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 }
}
```

## Type Guard

```typescript
// typeof guard
function padLeft(value: string, padding: string | number) {
  if (typeof padding === 'number') {
    return ' '.repeat(padding) + value
  }
  return padding + value
}

// in guard
interface Bird { fly(): void }
interface Fish { swim(): void }

function move(animal: Bird | Fish) {
  if ('fly' in animal) {
    animal.fly()
  } else {
    animal.swim()
  }
}

// 커스텀 Type Guard
function isString(value: unknown): value is string {
  return typeof value === 'string'
}

// Discriminated Union
type Result<T> =
  | { success: true; data: T }
  | { success: false; error: string }

function handleResult<T>(result: Result<T>) {
  if (result.success) {
    console.log(result.data)  // T로 추론
  } else {
    console.log(result.error)  // string으로 추론
  }
}
```

## Mapped Types

```typescript
// 모든 속성을 optional로
type Optional<T> = {
  [K in keyof T]?: T[K]
}

// 모든 속성을 readonly로
type Immutable<T> = {
  readonly [K in keyof T]: T[K]
}

// 특정 키만 required
type RequireKeys<T, K extends keyof T> = T & Required<Pick<T, K>>

// 사용 예
type UserWithRequiredEmail = RequireKeys<User, 'email'>
```

## Template Literal Types

```typescript
type EventName = 'click' | 'focus' | 'blur'
type EventHandler = `on${Capitalize<EventName>}`
// 'onClick' | 'onFocus' | 'onBlur'

type HTTPMethod = 'GET' | 'POST' | 'PUT' | 'DELETE'
type Endpoint = `/api/${string}`
type Route = `${HTTPMethod} ${Endpoint}`
```

## as const

```typescript
// 리터럴 타입으로 좁히기
const config = {
  endpoint: '/api/users',
  method: 'GET',
} as const

// type: { readonly endpoint: "/api/users"; readonly method: "GET" }

// 배열을 튜플로
const tuple = [1, 2, 3] as const
// type: readonly [1, 2, 3]

// enum 대체
const Status = {
  Pending: 'pending',
  Success: 'success',
  Error: 'error',
} as const

type StatusType = typeof Status[keyof typeof Status]
// 'pending' | 'success' | 'error'
```

## 타입 추론 활용

```typescript
// ReturnType
function createUser() {
  return { id: '1', name: 'John' }
}
type User = ReturnType<typeof createUser>

// Parameters
function greet(name: string, age: number) {}
type GreetParams = Parameters<typeof greet>
// [string, number]

// Awaited (Promise unwrap)
type UserPromise = Promise<User>
type ResolvedUser = Awaited<UserPromise>
// User
```

## 함수 오버로딩

```typescript
// 오버로드 시그니처
function parseInput(input: string): string[]
function parseInput(input: number): number[]
function parseInput(input: string | number): string[] | number[] {
  if (typeof input === 'string') {
    return input.split(',')
  }
  return [input]
}
```
