# Tailwind CSS + Shadcn/ui Best Practice Patterns

> Tailwind CSS: https://tailwindcss.com/docs
> Shadcn/ui: https://ui.shadcn.com/docs

## Tailwind CSS 핵심 원칙

### 1. 유틸리티 우선 (Utility-First)

```typescript
// ✅ CORRECT: 유틸리티 클래스 직접 사용
<button className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">
  버튼
</button>

// ❌ WRONG: @apply 남발 (CSS로 돌아가는 것)
// styles.css
.btn-primary {
  @apply bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded;
}
```

**@apply 사용 시점:**
- 매우 긴 클래스 목록이 반복될 때
- 서드파티 라이브러리 스타일 오버라이드 시
- 기본 HTML 요소 (prose 컨텐츠) 스타일링 시

### 2. 반응형 디자인 (Mobile-First)

```typescript
// ✅ Mobile-first 접근
<div className="
  flex flex-col       // 기본: 모바일
  md:flex-row        // 768px 이상
  lg:gap-8           // 1024px 이상
">

// 브레이크포인트
// sm: 640px
// md: 768px
// lg: 1024px
// xl: 1280px
// 2xl: 1536px

// 반응형 그리드
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4">
  {items.map(item => <Card key={item.id} {...item} />)}
</div>

// 반응형 타이포그래피
<h1 className="text-2xl sm:text-3xl md:text-4xl lg:text-5xl font-bold">
  제목
</h1>
```

### 3. 상태 변형 (State Variants)

```typescript
// 호버, 포커스, 액티브
<button className="
  bg-blue-500
  hover:bg-blue-600
  focus:ring-2 focus:ring-blue-500 focus:ring-offset-2
  active:bg-blue-700
  disabled:opacity-50 disabled:cursor-not-allowed
">

// 그룹 호버
<div className="group p-4 hover:bg-slate-100 rounded-lg">
  <h3 className="group-hover:text-blue-500">제목</h3>
  <p className="group-hover:text-slate-600">설명</p>
</div>

// 피어 상태 (인풋 검증)
<div>
  <input className="peer" type="email" required />
  <p className="hidden peer-invalid:block text-red-500">
    올바른 이메일을 입력하세요
  </p>
</div>

// 다크모드
<div className="bg-white dark:bg-slate-800 text-black dark:text-white">
```

## cn() 유틸리티

### clsx + tailwind-merge 조합

```typescript
// lib/utils.ts
import { type ClassValue, clsx } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}

// 사용 예시
import { cn } from '@/lib/utils'

interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'ghost'
  size?: 'sm' | 'md' | 'lg'
  className?: string
  children: React.ReactNode
}

function Button({ variant = 'primary', size = 'md', className, children }: ButtonProps) {
  return (
    <button
      className={cn(
        // 기본 스타일
        'inline-flex items-center justify-center rounded-md font-medium transition-colors',
        'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2',
        'disabled:pointer-events-none disabled:opacity-50',

        // variant
        {
          primary: 'bg-primary text-primary-foreground hover:bg-primary/90',
          secondary: 'bg-secondary text-secondary-foreground hover:bg-secondary/80',
          ghost: 'hover:bg-accent hover:text-accent-foreground',
        }[variant],

        // size
        {
          sm: 'h-8 px-3 text-sm',
          md: 'h-10 px-4',
          lg: 'h-12 px-6 text-lg',
        }[size],

        // 외부에서 전달된 클래스 (오버라이드 가능)
        className
      )}
    >
      {children}
    </button>
  )
}

// 사용: 필요시 스타일 오버라이드
<Button variant="primary" className="w-full">전체 너비 버튼</Button>
```

## Shadcn/ui 패턴

### 컴포넌트 커스터마이징

```typescript
// ✅ 올바른 방식: 컴포넌트 확장
// components/ui/button.tsx (Shadcn에서 복사한 파일)

// 기본 variants 확장
const buttonVariants = cva(
  'inline-flex items-center justify-center ...',
  {
    variants: {
      variant: {
        default: '...',
        destructive: '...',
        // 커스텀 variant 추가
        success: 'bg-green-500 text-white hover:bg-green-600',
        warning: 'bg-yellow-500 text-white hover:bg-yellow-600',
      },
      size: {
        default: '...',
        // 커스텀 사이즈 추가
        xs: 'h-7 px-2 text-xs',
        icon: 'h-10 w-10',
      },
    },
  }
)

// 사용
<Button variant="success">성공</Button>
<Button size="icon"><Settings /></Button>
```

### 컴포넌트 조합

```typescript
// 복합 컴포넌트 패턴
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle,
} from '@/components/ui/card'
import { Button } from '@/components/ui/button'
import { Badge } from '@/components/ui/badge'

interface ProductCardProps {
  product: Product
  onAddToCart: () => void
}

export function ProductCard({ product, onAddToCart }: ProductCardProps) {
  return (
    <Card className="overflow-hidden">
      <div className="aspect-square relative">
        <Image
          src={product.image}
          alt={product.name}
          fill
          className="object-cover"
        />
        {product.isNew && (
          <Badge className="absolute top-2 right-2">NEW</Badge>
        )}
      </div>

      <CardHeader>
        <CardTitle className="line-clamp-1">{product.name}</CardTitle>
        <CardDescription className="line-clamp-2">
          {product.description}
        </CardDescription>
      </CardHeader>

      <CardContent>
        <div className="flex items-center justify-between">
          <span className="text-2xl font-bold">
            {product.price.toLocaleString()}원
          </span>
          {product.originalPrice && (
            <span className="text-sm text-muted-foreground line-through">
              {product.originalPrice.toLocaleString()}원
            </span>
          )}
        </div>
      </CardContent>

      <CardFooter>
        <Button onClick={onAddToCart} className="w-full">
          장바구니 담기
        </Button>
      </CardFooter>
    </Card>
  )
}
```

## 레이아웃 패턴

### Flexbox 패턴

```typescript
// 수평 중앙 정렬
<div className="flex items-center justify-center">

// 양쪽 정렬
<div className="flex items-center justify-between">

// 수직 스택
<div className="flex flex-col gap-4">

// 수평 스택 (wrap)
<div className="flex flex-wrap gap-4">

// 균등 분할
<div className="flex">
  <div className="flex-1">1</div>
  <div className="flex-1">2</div>
  <div className="flex-1">3</div>
</div>
```

### Grid 패턴

```typescript
// 기본 그리드
<div className="grid grid-cols-3 gap-4">

// 반응형 그리드
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">

// 자동 맞춤 그리드 (auto-fill)
<div className="grid grid-cols-[repeat(auto-fill,minmax(250px,1fr))] gap-4">

// 12-column 그리드
<div className="grid grid-cols-12 gap-4">
  <div className="col-span-8">메인 콘텐츠</div>
  <div className="col-span-4">사이드바</div>
</div>

// 복잡한 그리드
<div className="grid grid-cols-4 grid-rows-3 gap-4">
  <div className="col-span-2 row-span-2">큰 아이템</div>
  <div>작은 아이템 1</div>
  <div>작은 아이템 2</div>
  <div className="col-span-2">중간 아이템</div>
</div>
```

### 컨테이너 패턴

```typescript
// 기본 컨테이너
<div className="container mx-auto px-4">

// 최대 너비 제한
<div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">

// 전체 화면 레이아웃
<div className="min-h-screen flex flex-col">
  <header className="h-16 border-b">헤더</header>
  <main className="flex-1">메인</main>
  <footer className="h-20 border-t">푸터</footer>
</div>

// 사이드바 레이아웃
<div className="flex min-h-screen">
  <aside className="w-64 border-r">사이드바</aside>
  <main className="flex-1 p-6">메인</main>
</div>
```

## 애니메이션 패턴

### Tailwind 기본 애니메이션

```typescript
// 트랜지션
<button className="transition-colors duration-200 hover:bg-blue-600">

// 스케일
<div className="transition-transform hover:scale-105">

// 여러 속성
<div className="transition-all duration-300 hover:shadow-lg hover:-translate-y-1">

// 내장 애니메이션
<div className="animate-spin">로딩</div>
<div className="animate-pulse">스켈레톤</div>
<div className="animate-bounce">바운스</div>
```

### 커스텀 애니메이션

```typescript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      keyframes: {
        'fade-in': {
          '0%': { opacity: '0', transform: 'translateY(-10px)' },
          '100%': { opacity: '1', transform: 'translateY(0)' },
        },
        'slide-in-right': {
          '0%': { transform: 'translateX(100%)' },
          '100%': { transform: 'translateX(0)' },
        },
      },
      animation: {
        'fade-in': 'fade-in 0.3s ease-out',
        'slide-in-right': 'slide-in-right 0.3s ease-out',
      },
    },
  },
}

// 사용
<div className="animate-fade-in">페이드 인</div>
<div className="animate-slide-in-right">슬라이드 인</div>
```

## 아이콘 패턴 (Lucide React)

### 기본 사용

```typescript
// ✅ 개별 import (tree-shaking)
import { ChevronRight, User, Settings, X, Check } from 'lucide-react'

// ❌ 전체 import 금지
import * as Icons from 'lucide-react'

// 기본 사용
<ChevronRight className="h-4 w-4" />

// 색상
<User className="h-5 w-5 text-muted-foreground" />

// 버튼과 함께
<Button>
  <Settings className="h-4 w-4 mr-2" />
  설정
</Button>

// 아이콘만 있는 버튼
<Button variant="ghost" size="icon">
  <X className="h-4 w-4" />
  <span className="sr-only">닫기</span>
</Button>
```

### 동적 아이콘

```typescript
import { LucideIcon, Home, User, Settings, Bell } from 'lucide-react'

const iconMap: Record<string, LucideIcon> = {
  home: Home,
  user: User,
  settings: Settings,
  notifications: Bell,
}

interface NavItemProps {
  icon: keyof typeof iconMap
  label: string
  href: string
}

function NavItem({ icon, label, href }: NavItemProps) {
  const Icon = iconMap[icon]

  return (
    <Link href={href} className="flex items-center gap-2">
      <Icon className="h-5 w-5" />
      <span>{label}</span>
    </Link>
  )
}
```

## 다크모드

### CSS 변수 기반 (Shadcn 방식)

```css
/* globals.css */
@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --primary: 221.2 83.2% 53.3%;
    --primary-foreground: 210 40% 98%;
    /* ... */
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --primary: 217.2 91.2% 59.8%;
    --primary-foreground: 222.2 47.4% 11.2%;
    /* ... */
  }
}
```

```typescript
// 테마 토글
'use client'

import { useTheme } from 'next-themes'
import { Moon, Sun } from 'lucide-react'

export function ThemeToggle() {
  const { theme, setTheme } = useTheme()

  return (
    <Button
      variant="ghost"
      size="icon"
      onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}
    >
      <Sun className="h-5 w-5 rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0" />
      <Moon className="absolute h-5 w-5 rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100" />
      <span className="sr-only">테마 전환</span>
    </Button>
  )
}
```

## 접근성

### 포커스 스타일

```typescript
// 기본 포커스 링
<button className="focus:outline-none focus:ring-2 focus:ring-primary focus:ring-offset-2">

// 키보드 포커스만
<button className="focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-primary">

// 포커스 내부
<div className="focus-within:ring-2 focus-within:ring-primary">
  <label>이메일</label>
  <input type="email" />
</div>
```

### 스크린 리더

```typescript
// 시각적으로 숨기고 스크린 리더에만
<span className="sr-only">메뉴 열기</span>

// 또는 aria-label 사용
<button aria-label="메뉴 열기">
  <Menu className="h-5 w-5" />
</button>

// 장식용 요소
<div aria-hidden="true">...</div>
```

## 성능 최적화

### PurgeCSS (자동)

```javascript
// tailwind.config.js
module.exports = {
  content: [
    './app/**/*.{js,ts,jsx,tsx}',
    './components/**/*.{js,ts,jsx,tsx}',
  ],
  // 사용하지 않는 클래스 자동 제거
}
```

### 동적 클래스 주의

```typescript
// ❌ WRONG: 동적 문자열 - PurgeCSS가 감지 못함
const color = 'blue'
<div className={`text-${color}-500`}>

// ✅ CORRECT: 완전한 클래스명
const colorClasses = {
  blue: 'text-blue-500',
  red: 'text-red-500',
  green: 'text-green-500',
}
<div className={colorClasses[color]}>

// ✅ CORRECT: safelist 사용 (tailwind.config.js)
module.exports = {
  safelist: [
    'text-blue-500',
    'text-red-500',
    'text-green-500',
  ],
}
```
