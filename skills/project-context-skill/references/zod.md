# Zod 패턴

> 공식 문서: https://zod.dev

## 기본 스키마

```typescript
import { z } from 'zod'

// Primitive 타입
const stringSchema = z.string()
const numberSchema = z.number()
const booleanSchema = z.boolean()
const dateSchema = z.date()

// 검증
stringSchema.parse('hello')      // 성공: 'hello'
stringSchema.parse(123)          // 에러 발생

// 안전한 검증
const result = stringSchema.safeParse('hello')
if (result.success) {
  console.log(result.data)
} else {
  console.log(result.error.issues)
}
```

## 문자열 검증

```typescript
const schema = z.string()
  .min(1, '필수 입력입니다')
  .max(100, '100자 이하로 입력하세요')
  .email('올바른 이메일을 입력하세요')
  .url('올바른 URL을 입력하세요')
  .regex(/^[a-z]+$/, '소문자만 입력 가능합니다')
  .trim()
  .toLowerCase()

// 커스텀 에러 메시지
const email = z.string({
  required_error: '이메일을 입력하세요',
  invalid_type_error: '문자열을 입력하세요',
}).email('올바른 이메일 형식이 아닙니다')
```

## 숫자 검증

```typescript
const schema = z.number()
  .min(0, '0 이상이어야 합니다')
  .max(100, '100 이하여야 합니다')
  .int('정수를 입력하세요')
  .positive('양수를 입력하세요')
  .nonnegative('0 이상의 수를 입력하세요')

// 문자열 → 숫자 변환
const stringToNumber = z.string().transform(Number).pipe(z.number().positive())
```

## 객체 스키마

```typescript
const userSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  name: z.string().min(2),
  age: z.number().int().positive().optional(),
  role: z.enum(['admin', 'user', 'guest']),
  createdAt: z.date(),
})

// 타입 추출
type User = z.infer<typeof userSchema>

// Partial / Required
const partialUser = userSchema.partial()           // 모든 필드 optional
const requiredUser = userSchema.required()         // 모든 필드 required
const pickUser = userSchema.pick({ email: true })  // email만
const omitUser = userSchema.omit({ id: true })     // id 제외
```

## 배열 스키마

```typescript
const stringArraySchema = z.array(z.string())
const userArraySchema = z.array(userSchema)

// 배열 검증
const schema = z.array(z.string())
  .min(1, '최소 1개 이상 선택하세요')
  .max(5, '최대 5개까지 선택 가능합니다')
  .nonempty('빈 배열은 허용되지 않습니다')
```

## Union & Discriminated Union

```typescript
// Union
const stringOrNumber = z.union([z.string(), z.number()])
// 또는
const stringOrNumber2 = z.string().or(z.number())

// Discriminated Union (권장)
const eventSchema = z.discriminatedUnion('type', [
  z.object({ type: z.literal('click'), x: z.number(), y: z.number() }),
  z.object({ type: z.literal('scroll'), offset: z.number() }),
  z.object({ type: z.literal('keypress'), key: z.string() }),
])

type Event = z.infer<typeof eventSchema>
```

## Transform & Refine

```typescript
// 값 변환
const schema = z.string()
  .transform((val) => val.trim().toLowerCase())

// 커스텀 검증
const passwordSchema = z.string()
  .min(8)
  .refine((val) => /[A-Z]/.test(val), '대문자를 포함해야 합니다')
  .refine((val) => /[0-9]/.test(val), '숫자를 포함해야 합니다')

// 여러 필드 검증
const formSchema = z.object({
  password: z.string().min(8),
  confirmPassword: z.string(),
}).refine((data) => data.password === data.confirmPassword, {
  message: '비밀번호가 일치하지 않습니다',
  path: ['confirmPassword'],
})
```

## API 응답 검증

```typescript
const apiResponseSchema = z.object({
  success: z.boolean(),
  data: z.object({
    users: z.array(userSchema),
    total: z.number(),
  }),
  error: z.string().nullable(),
})

// API 호출 시 검증
async function fetchUsers() {
  const response = await fetch('/api/users')
  const json = await response.json()

  const result = apiResponseSchema.safeParse(json)
  if (!result.success) {
    throw new Error('Invalid API response')
  }

  return result.data
}
```

## Default 값

```typescript
const schema = z.object({
  name: z.string(),
  role: z.enum(['admin', 'user']).default('user'),
  settings: z.object({
    theme: z.enum(['light', 'dark']).default('light'),
    notifications: z.boolean().default(true),
  }).default({}),
})

// 기본값 적용
schema.parse({ name: 'John' })
// { name: 'John', role: 'user', settings: { theme: 'light', notifications: true } }
```

## Coerce (자동 변환)

```typescript
// 문자열 → 숫자 자동 변환
const schema = z.coerce.number()
schema.parse('42')  // 42

// 폼 데이터 처리에 유용
const formSchema = z.object({
  age: z.coerce.number().int().positive(),
  isActive: z.coerce.boolean(),
  createdAt: z.coerce.date(),
})
```
