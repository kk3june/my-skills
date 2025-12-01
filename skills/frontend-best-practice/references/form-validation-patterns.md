# React Hook Form + Zod Best Practice Patterns

> React Hook Form: https://react-hook-form.com/docs
> Zod: https://zod.dev

## Zod 스키마 설계

### 기본 스키마 패턴

```typescript
import { z } from 'zod'

// 기본 필드 타입
export const userSchema = z.object({
  // 문자열
  name: z.string().min(2, '이름은 2자 이상이어야 합니다'),

  // 이메일
  email: z.string().email('올바른 이메일 형식이 아닙니다'),

  // 비밀번호 (복합 조건)
  password: z
    .string()
    .min(8, '비밀번호는 8자 이상이어야 합니다')
    .regex(/[A-Z]/, '대문자를 포함해야 합니다')
    .regex(/[a-z]/, '소문자를 포함해야 합니다')
    .regex(/[0-9]/, '숫자를 포함해야 합니다')
    .regex(/[^A-Za-z0-9]/, '특수문자를 포함해야 합니다'),

  // 숫자
  age: z.number().min(18, '18세 이상만 가입 가능합니다').max(120),

  // 선택 필드
  nickname: z.string().optional(),

  // nullable
  bio: z.string().nullable(),

  // 기본값
  role: z.enum(['user', 'admin']).default('user'),

  // 날짜
  birthDate: z.coerce.date(),

  // URL
  website: z.string().url().optional().or(z.literal('')),

  // 전화번호
  phone: z.string().regex(/^01[0-9]-[0-9]{4}-[0-9]{4}$/, '올바른 전화번호 형식이 아닙니다'),
})

export type User = z.infer<typeof userSchema>
```

### 스키마 재사용 및 확장

```typescript
// 기본 주소 스키마
const addressSchema = z.object({
  street: z.string().min(1, '주소를 입력해주세요'),
  city: z.string().min(1, '도시를 입력해주세요'),
  zipCode: z.string().regex(/^\d{5}$/, '우편번호는 5자리 숫자입니다'),
})

// 배송 주소 (확장)
const shippingAddressSchema = addressSchema.extend({
  recipientName: z.string().min(1, '수령인 이름을 입력해주세요'),
  phone: z.string().min(1, '전화번호를 입력해주세요'),
  deliveryNote: z.string().optional(),
})

// 일부 필드만 선택
const partialAddressSchema = addressSchema.pick({
  city: true,
  zipCode: true,
})

// 일부 필드 제외
const addressWithoutZipSchema = addressSchema.omit({
  zipCode: true,
})

// 모든 필드 optional
const optionalAddressSchema = addressSchema.partial()

// 특정 필드만 required
const requiredStreetSchema = addressSchema.partial().required({
  street: true,
})
```

### 조건부 검증

```typescript
// 비밀번호 확인
const passwordFormSchema = z
  .object({
    password: z.string().min(8),
    confirmPassword: z.string(),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: '비밀번호가 일치하지 않습니다',
    path: ['confirmPassword'],
  })

// 조건부 필수 필드
const orderSchema = z
  .object({
    paymentMethod: z.enum(['card', 'bank', 'cash']),
    cardNumber: z.string().optional(),
    bankAccount: z.string().optional(),
  })
  .refine(
    (data) => {
      if (data.paymentMethod === 'card') {
        return !!data.cardNumber
      }
      if (data.paymentMethod === 'bank') {
        return !!data.bankAccount
      }
      return true
    },
    {
      message: '결제 정보를 입력해주세요',
      path: ['cardNumber'], // 또는 동적으로 설정
    }
  )

// 복잡한 조건부 검증 (superRefine)
const formSchema = z
  .object({
    type: z.enum(['individual', 'business']),
    name: z.string().min(1),
    businessNumber: z.string().optional(),
    representative: z.string().optional(),
  })
  .superRefine((data, ctx) => {
    if (data.type === 'business') {
      if (!data.businessNumber) {
        ctx.addIssue({
          code: z.ZodIssueCode.custom,
          message: '사업자등록번호를 입력해주세요',
          path: ['businessNumber'],
        })
      }
      if (!data.representative) {
        ctx.addIssue({
          code: z.ZodIssueCode.custom,
          message: '대표자명을 입력해주세요',
          path: ['representative'],
        })
      }
    }
  })
```

### Transform & Preprocess

```typescript
// 값 변환
const userInputSchema = z.object({
  // 공백 제거
  email: z.string().trim().email(),

  // 소문자 변환
  username: z.string().toLowerCase(),

  // 숫자로 변환
  age: z.string().transform((val) => parseInt(val, 10)),

  // 날짜로 변환 (coerce 사용)
  birthDate: z.coerce.date(),

  // 커스텀 변환
  phone: z.string().transform((val) => val.replace(/-/g, '')),
})

// Preprocess (검증 전 처리)
const priceSchema = z.preprocess(
  (val) => {
    if (typeof val === 'string') {
      return parseFloat(val.replace(/,/g, ''))
    }
    return val
  },
  z.number().positive()
)

// 빈 문자열을 undefined로
const optionalString = z.preprocess(
  (val) => (val === '' ? undefined : val),
  z.string().optional()
)
```

## React Hook Form 통합

### 기본 폼 패턴

```typescript
'use client'

import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'

const formSchema = z.object({
  email: z.string().email('올바른 이메일을 입력해주세요'),
  password: z.string().min(8, '8자 이상 입력해주세요'),
})

type FormData = z.infer<typeof formSchema>

export function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting, isValid, isDirty },
    reset,
    setError,
    clearErrors,
  } = useForm<FormData>({
    resolver: zodResolver(formSchema),
    defaultValues: {
      email: '',
      password: '',
    },
    mode: 'onBlur', // 'onChange' | 'onBlur' | 'onSubmit' | 'onTouched' | 'all'
  })

  const onSubmit = async (data: FormData) => {
    try {
      const response = await loginApi(data)
      // 성공 처리
    } catch (error) {
      // 서버 에러를 폼에 표시
      setError('root', {
        message: '로그인에 실패했습니다',
      })
      // 또는 특정 필드에
      setError('email', {
        message: '이미 사용 중인 이메일입니다',
      })
    }
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      {errors.root && (
        <div className="text-red-500 text-sm">{errors.root.message}</div>
      )}

      <div>
        <Input
          {...register('email')}
          type="email"
          placeholder="이메일"
          className={errors.email ? 'border-red-500' : ''}
        />
        {errors.email && (
          <p className="text-red-500 text-sm mt-1">{errors.email.message}</p>
        )}
      </div>

      <div>
        <Input
          {...register('password')}
          type="password"
          placeholder="비밀번호"
          className={errors.password ? 'border-red-500' : ''}
        />
        {errors.password && (
          <p className="text-red-500 text-sm mt-1">{errors.password.message}</p>
        )}
      </div>

      <Button type="submit" disabled={isSubmitting || !isDirty}>
        {isSubmitting ? '로그인 중...' : '로그인'}
      </Button>
    </form>
  )
}
```

### Shadcn Form 통합

```typescript
'use client'

import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'
import {
  Form,
  FormControl,
  FormDescription,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from '@/components/ui/form'
import { Input } from '@/components/ui/input'
import { Button } from '@/components/ui/button'
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select'

const formSchema = z.object({
  username: z.string().min(2).max(50),
  email: z.string().email(),
  role: z.enum(['admin', 'user', 'guest']),
})

export function ProfileForm() {
  const form = useForm<z.infer<typeof formSchema>>({
    resolver: zodResolver(formSchema),
    defaultValues: {
      username: '',
      email: '',
      role: 'user',
    },
  })

  function onSubmit(values: z.infer<typeof formSchema>) {
    console.log(values)
  }

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-6">
        <FormField
          control={form.control}
          name="username"
          render={({ field }) => (
            <FormItem>
              <FormLabel>사용자명</FormLabel>
              <FormControl>
                <Input placeholder="username" {...field} />
              </FormControl>
              <FormDescription>
                공개적으로 표시되는 이름입니다.
              </FormDescription>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name="email"
          render={({ field }) => (
            <FormItem>
              <FormLabel>이메일</FormLabel>
              <FormControl>
                <Input placeholder="email@example.com" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name="role"
          render={({ field }) => (
            <FormItem>
              <FormLabel>역할</FormLabel>
              <Select onValueChange={field.onChange} defaultValue={field.value}>
                <FormControl>
                  <SelectTrigger>
                    <SelectValue placeholder="역할 선택" />
                  </SelectTrigger>
                </FormControl>
                <SelectContent>
                  <SelectItem value="admin">관리자</SelectItem>
                  <SelectItem value="user">사용자</SelectItem>
                  <SelectItem value="guest">게스트</SelectItem>
                </SelectContent>
              </Select>
              <FormMessage />
            </FormItem>
          )}
        />

        <Button type="submit">저장</Button>
      </form>
    </Form>
  )
}
```

### 동적 필드 (useFieldArray)

```typescript
'use client'

import { useForm, useFieldArray } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'
import { Plus, Trash2 } from 'lucide-react'

const itemSchema = z.object({
  name: z.string().min(1, '상품명을 입력해주세요'),
  quantity: z.number().min(1, '1개 이상 입력해주세요'),
  price: z.number().min(0, '가격은 0 이상이어야 합니다'),
})

const orderSchema = z.object({
  customerName: z.string().min(1),
  items: z.array(itemSchema).min(1, '최소 1개의 상품을 추가해주세요'),
})

type OrderForm = z.infer<typeof orderSchema>

export function OrderForm() {
  const {
    control,
    register,
    handleSubmit,
    formState: { errors },
    watch,
  } = useForm<OrderForm>({
    resolver: zodResolver(orderSchema),
    defaultValues: {
      customerName: '',
      items: [{ name: '', quantity: 1, price: 0 }],
    },
  })

  const { fields, append, remove } = useFieldArray({
    control,
    name: 'items',
  })

  // 총 금액 계산
  const items = watch('items')
  const total = items.reduce((sum, item) => sum + item.quantity * item.price, 0)

  const onSubmit = (data: OrderForm) => {
    console.log(data)
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-6">
      <div>
        <label>고객명</label>
        <Input {...register('customerName')} />
        {errors.customerName && (
          <p className="text-red-500">{errors.customerName.message}</p>
        )}
      </div>

      <div className="space-y-4">
        <div className="flex justify-between items-center">
          <h3>상품 목록</h3>
          <Button
            type="button"
            variant="outline"
            size="sm"
            onClick={() => append({ name: '', quantity: 1, price: 0 })}
          >
            <Plus className="h-4 w-4 mr-1" /> 상품 추가
          </Button>
        </div>

        {fields.map((field, index) => (
          <div key={field.id} className="flex gap-2 items-start">
            <div className="flex-1">
              <Input
                {...register(`items.${index}.name`)}
                placeholder="상품명"
              />
              {errors.items?.[index]?.name && (
                <p className="text-red-500 text-sm">
                  {errors.items[index]?.name?.message}
                </p>
              )}
            </div>

            <div className="w-24">
              <Input
                {...register(`items.${index}.quantity`, { valueAsNumber: true })}
                type="number"
                placeholder="수량"
              />
            </div>

            <div className="w-32">
              <Input
                {...register(`items.${index}.price`, { valueAsNumber: true })}
                type="number"
                placeholder="가격"
              />
            </div>

            <Button
              type="button"
              variant="ghost"
              size="icon"
              onClick={() => remove(index)}
              disabled={fields.length === 1}
            >
              <Trash2 className="h-4 w-4" />
            </Button>
          </div>
        ))}

        {errors.items?.root && (
          <p className="text-red-500">{errors.items.root.message}</p>
        )}
      </div>

      <div className="flex justify-between items-center">
        <span className="text-lg font-semibold">
          총 금액: {total.toLocaleString()}원
        </span>
        <Button type="submit">주문하기</Button>
      </div>
    </form>
  )
}
```

### 다단계 폼 (Multi-step Form)

```typescript
'use client'

import { useState } from 'react'
import { useForm, FormProvider } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'

// 각 단계별 스키마
const step1Schema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
})

const step2Schema = z.object({
  name: z.string().min(2),
  phone: z.string().min(10),
})

const step3Schema = z.object({
  address: z.string().min(5),
  zipCode: z.string().length(5),
})

// 전체 스키마
const fullSchema = step1Schema.merge(step2Schema).merge(step3Schema)

type FormData = z.infer<typeof fullSchema>

const steps = [
  { schema: step1Schema, fields: ['email', 'password'] },
  { schema: step2Schema, fields: ['name', 'phone'] },
  { schema: step3Schema, fields: ['address', 'zipCode'] },
]

export function MultiStepForm() {
  const [currentStep, setCurrentStep] = useState(0)

  const methods = useForm<FormData>({
    resolver: zodResolver(fullSchema),
    mode: 'onChange',
    defaultValues: {
      email: '',
      password: '',
      name: '',
      phone: '',
      address: '',
      zipCode: '',
    },
  })

  const { trigger, handleSubmit, formState: { isSubmitting } } = methods

  const nextStep = async () => {
    const fields = steps[currentStep].fields as (keyof FormData)[]
    const isValid = await trigger(fields)

    if (isValid) {
      setCurrentStep((prev) => Math.min(prev + 1, steps.length - 1))
    }
  }

  const prevStep = () => {
    setCurrentStep((prev) => Math.max(prev - 1, 0))
  }

  const onSubmit = async (data: FormData) => {
    console.log('Final data:', data)
  }

  return (
    <FormProvider {...methods}>
      <form onSubmit={handleSubmit(onSubmit)}>
        {/* Progress indicator */}
        <div className="flex justify-center mb-8">
          {steps.map((_, index) => (
            <div
              key={index}
              className={`w-8 h-8 rounded-full flex items-center justify-center mx-2 ${
                index <= currentStep ? 'bg-primary text-white' : 'bg-gray-200'
              }`}
            >
              {index + 1}
            </div>
          ))}
        </div>

        {/* Step content */}
        {currentStep === 0 && <Step1 />}
        {currentStep === 1 && <Step2 />}
        {currentStep === 2 && <Step3 />}

        {/* Navigation */}
        <div className="flex justify-between mt-8">
          <Button
            type="button"
            variant="outline"
            onClick={prevStep}
            disabled={currentStep === 0}
          >
            이전
          </Button>

          {currentStep < steps.length - 1 ? (
            <Button type="button" onClick={nextStep}>
              다음
            </Button>
          ) : (
            <Button type="submit" disabled={isSubmitting}>
              {isSubmitting ? '제출 중...' : '제출'}
            </Button>
          )}
        </div>
      </form>
    </FormProvider>
  )
}

// Step 컴포넌트들
function Step1() {
  const { register, formState: { errors } } = useFormContext<FormData>()

  return (
    <div className="space-y-4">
      <h2>계정 정보</h2>
      <Input {...register('email')} placeholder="이메일" />
      {errors.email && <p className="text-red-500">{errors.email.message}</p>}

      <Input {...register('password')} type="password" placeholder="비밀번호" />
      {errors.password && <p className="text-red-500">{errors.password.message}</p>}
    </div>
  )
}

// Step2, Step3도 유사하게 구현
```

### 비동기 검증

```typescript
const usernameSchema = z.string().min(3).refine(
  async (username) => {
    // 서버에서 중복 체크
    const response = await fetch(`/api/check-username?username=${username}`)
    const data = await response.json()
    return data.available
  },
  { message: '이미 사용 중인 사용자명입니다' }
)

// 또는 useForm의 resolver에서 직접 처리
const formSchema = z.object({
  username: z.string().min(3),
  email: z.string().email(),
})

export function SignupForm() {
  const form = useForm({
    resolver: zodResolver(formSchema),
  })

  // 실시간 검증 (debounced)
  const username = form.watch('username')

  useEffect(() => {
    const timer = setTimeout(async () => {
      if (username && username.length >= 3) {
        const isAvailable = await checkUsername(username)
        if (!isAvailable) {
          form.setError('username', {
            message: '이미 사용 중인 사용자명입니다',
          })
        } else {
          form.clearErrors('username')
        }
      }
    }, 500)

    return () => clearTimeout(timer)
  }, [username])

  // ...
}
```

## 공통 패턴

### 재사용 가능한 입력 컴포넌트

```typescript
// components/forms/FormInput.tsx
import { UseFormRegisterReturn } from 'react-hook-form'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'
import { cn } from '@/lib/utils'

interface FormInputProps {
  label: string
  error?: string
  registration: UseFormRegisterReturn
  type?: 'text' | 'email' | 'password' | 'number'
  placeholder?: string
  description?: string
  className?: string
}

export function FormInput({
  label,
  error,
  registration,
  type = 'text',
  placeholder,
  description,
  className,
}: FormInputProps) {
  return (
    <div className={cn('space-y-2', className)}>
      <Label htmlFor={registration.name}>{label}</Label>
      <Input
        id={registration.name}
        type={type}
        placeholder={placeholder}
        {...registration}
        className={cn(error && 'border-destructive')}
        aria-invalid={!!error}
        aria-describedby={error ? `${registration.name}-error` : undefined}
      />
      {description && (
        <p className="text-sm text-muted-foreground">{description}</p>
      )}
      {error && (
        <p
          id={`${registration.name}-error`}
          className="text-sm text-destructive"
          role="alert"
        >
          {error}
        </p>
      )}
    </div>
  )
}

// 사용
<FormInput
  label="이메일"
  error={errors.email?.message}
  registration={register('email')}
  type="email"
  placeholder="email@example.com"
/>
```

### 폼 상태 관리 유틸리티

```typescript
// lib/form-utils.ts
import { UseFormReturn } from 'react-hook-form'

export function getFormErrorMessage<T extends Record<string, any>>(
  form: UseFormReturn<T>,
  fieldName: keyof T
): string | undefined {
  return form.formState.errors[fieldName]?.message as string | undefined
}

export function isFieldDirty<T extends Record<string, any>>(
  form: UseFormReturn<T>,
  fieldName: keyof T
): boolean {
  return !!form.formState.dirtyFields[fieldName as string]
}

export function hasAnyError<T extends Record<string, any>>(
  form: UseFormReturn<T>
): boolean {
  return Object.keys(form.formState.errors).length > 0
}

// 폼 데이터 정리 (undefined/null 제거)
export function cleanFormData<T extends Record<string, any>>(data: T): T {
  return Object.fromEntries(
    Object.entries(data).filter(([_, value]) => value != null && value !== '')
  ) as T
}
```
