# React Hook Form 패턴

> 공식 문서: https://react-hook-form.com/docs

## 기본 사용

```typescript
import { useForm } from 'react-hook-form'

interface FormData {
  email: string
  password: string
}

function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<FormData>()

  const onSubmit = async (data: FormData) => {
    await login(data)
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register('email', { required: '이메일을 입력하세요' })}
        type="email"
      />
      {errors.email && <span>{errors.email.message}</span>}

      <input
        {...register('password', { required: '비밀번호를 입력하세요' })}
        type="password"
      />
      {errors.password && <span>{errors.password.message}</span>}

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? '로그인 중...' : '로그인'}
      </button>
    </form>
  )
}
```

## Zod 통합

```typescript
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'

const schema = z.object({
  email: z.string().email('올바른 이메일을 입력하세요'),
  password: z.string().min(8, '비밀번호는 8자 이상이어야 합니다'),
})

type FormData = z.infer<typeof schema>

function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<FormData>({
    resolver: zodResolver(schema),
  })

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* ... */}
    </form>
  )
}
```

## 기본값 설정

```typescript
const { register, reset } = useForm<FormData>({
  defaultValues: {
    email: '',
    name: '',
  },
})

// 또는 비동기 기본값
const { register } = useForm<FormData>({
  defaultValues: async () => {
    const user = await fetchUser()
    return { email: user.email, name: user.name }
  },
})

// 외부에서 값 설정
useEffect(() => {
  if (userData) {
    reset(userData)
  }
}, [userData, reset])
```

## Controller (제어 컴포넌트)

```typescript
import { useForm, Controller } from 'react-hook-form'
import { Select } from '@/components/ui/select'

function Form() {
  const { control, handleSubmit } = useForm<FormData>()

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <Controller
        name="category"
        control={control}
        rules={{ required: '카테고리를 선택하세요' }}
        render={({ field, fieldState: { error } }) => (
          <>
            <Select
              value={field.value}
              onValueChange={field.onChange}
              options={categories}
            />
            {error && <span>{error.message}</span>}
          </>
        )}
      />
    </form>
  )
}
```

## Watch & Conditional Fields

```typescript
function Form() {
  const { register, watch } = useForm<FormData>()

  const watchShowAge = watch('showAge', false)
  const allFields = watch() // 모든 필드 감시

  return (
    <form>
      <input type="checkbox" {...register('showAge')} />
      {watchShowAge && (
        <input type="number" {...register('age')} />
      )}
    </form>
  )
}
```

## Field Array

```typescript
import { useForm, useFieldArray } from 'react-hook-form'

interface FormData {
  items: { name: string; quantity: number }[]
}

function OrderForm() {
  const { control, register } = useForm<FormData>({
    defaultValues: { items: [{ name: '', quantity: 1 }] },
  })

  const { fields, append, remove } = useFieldArray({
    control,
    name: 'items',
  })

  return (
    <form>
      {fields.map((field, index) => (
        <div key={field.id}>
          <input {...register(`items.${index}.name`)} />
          <input type="number" {...register(`items.${index}.quantity`)} />
          <button type="button" onClick={() => remove(index)}>삭제</button>
        </div>
      ))}
      <button type="button" onClick={() => append({ name: '', quantity: 1 })}>
        추가
      </button>
    </form>
  )
}
```

## Form State 활용

```typescript
const {
  formState: {
    isDirty,        // 값이 변경되었는지
    isValid,        // 유효성 검사 통과
    isSubmitting,   // 제출 중
    isSubmitted,    // 제출됨
    submitCount,    // 제출 횟수
    errors,         // 에러 객체
    dirtyFields,    // 변경된 필드들
    touchedFields,  // 터치된 필드들
  },
} = useForm<FormData>({
  mode: 'onChange',  // 실시간 유효성 검사
})
```

## 에러 처리

```typescript
const {
  setError,
  clearErrors,
  formState: { errors },
} = useForm<FormData>()

// 서버 에러 설정
const onSubmit = async (data: FormData) => {
  try {
    await submitForm(data)
  } catch (error) {
    setError('root.serverError', {
      type: 'server',
      message: '서버 오류가 발생했습니다',
    })
  }
}

// 특정 필드 에러 설정
setError('email', {
  type: 'manual',
  message: '이미 사용 중인 이메일입니다',
})

// 에러 클리어
clearErrors('email')
clearErrors() // 모든 에러 클리어
```
