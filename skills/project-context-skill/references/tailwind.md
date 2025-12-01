# Tailwind CSS 패턴

> 공식 문서: https://tailwindcss.com/docs

## 레이아웃

### Flexbox

```tsx
// 중앙 정렬
<div className="flex items-center justify-center">

// 세로 배치
<div className="flex flex-col gap-4">

// 양쪽 정렬
<div className="flex items-center justify-between">

// Wrap
<div className="flex flex-wrap gap-2">
```

### Grid

```tsx
// 기본 그리드
<div className="grid grid-cols-3 gap-4">

// 반응형 그리드
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">

// Auto-fit
<div className="grid grid-cols-[repeat(auto-fit,minmax(250px,1fr))] gap-4">
```

## 반응형 디자인

```tsx
// 모바일 퍼스트
<div className="
  w-full           // 기본 (mobile)
  md:w-1/2         // 768px 이상
  lg:w-1/3         // 1024px 이상
  xl:w-1/4         // 1280px 이상
">

// 숨김/표시
<div className="hidden md:block">  // 모바일에서 숨김
<div className="block md:hidden">  // 데스크톱에서 숨김
```

## 상태 스타일링

```tsx
// Hover, Focus, Active
<button className="
  bg-blue-500
  hover:bg-blue-600
  focus:ring-2 focus:ring-blue-500 focus:outline-none
  active:bg-blue-700
">

// Disabled
<button className="
  bg-blue-500
  disabled:bg-gray-300 disabled:cursor-not-allowed
">

// Group hover
<div className="group">
  <span className="group-hover:text-blue-500">Hover parent</span>
</div>

// Peer (sibling)
<input className="peer" />
<span className="peer-invalid:text-red-500">Error</span>
```

## 다크 모드

```tsx
// 시스템 설정 따름
<div className="bg-white dark:bg-gray-900 text-gray-900 dark:text-white">

// 수동 제어 (class 전략)
// tailwind.config.js: darkMode: 'class'
<html className="dark">
  <div className="bg-white dark:bg-gray-900">
</html>
```

## 컴포넌트 패턴

### Button

```tsx
const buttonVariants = {
  primary: 'bg-blue-500 text-white hover:bg-blue-600',
  secondary: 'bg-gray-200 text-gray-800 hover:bg-gray-300',
  outline: 'border border-gray-300 hover:bg-gray-50',
}

const buttonSizes = {
  sm: 'px-3 py-1.5 text-sm',
  md: 'px-4 py-2 text-base',
  lg: 'px-6 py-3 text-lg',
}

<button className={`
  ${buttonVariants[variant]}
  ${buttonSizes[size]}
  rounded-lg font-medium transition-colors
  focus:outline-none focus:ring-2 focus:ring-offset-2
  disabled:opacity-50 disabled:cursor-not-allowed
`}>
```

### Card

```tsx
<div className="
  bg-white dark:bg-gray-800
  rounded-lg shadow-md
  p-6
  border border-gray-200 dark:border-gray-700
">
```

### Input

```tsx
<input className="
  w-full px-4 py-2
  border border-gray-300 rounded-lg
  focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent
  disabled:bg-gray-100 disabled:cursor-not-allowed
  placeholder:text-gray-400
"/>
```

## cn() 유틸리티

```typescript
// lib/utils.ts
import { clsx, type ClassValue } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}

// 사용
<div className={cn(
  'px-4 py-2 rounded',
  isActive && 'bg-blue-500',
  className  // props로 받은 클래스
)}>
```

## 애니메이션

```tsx
// Transition
<div className="transition-all duration-300 ease-in-out">

// Transform
<div className="hover:scale-105 transition-transform">

// 커스텀 애니메이션
// tailwind.config.js
animation: {
  'fade-in': 'fadeIn 0.5s ease-out',
}
keyframes: {
  fadeIn: {
    '0%': { opacity: '0' },
    '100%': { opacity: '1' },
  }
}

<div className="animate-fade-in">
```

## 자주 쓰는 조합

```tsx
// Truncate (말줄임)
<p className="truncate">  // 한 줄
<p className="line-clamp-2">  // 여러 줄

// Aspect Ratio
<div className="aspect-video">  // 16:9
<div className="aspect-square">  // 1:1

// Container
<div className="container mx-auto px-4">

// Sticky Header
<header className="sticky top-0 z-50 bg-white/80 backdrop-blur">

// Scrollbar 숨김
<div className="overflow-auto scrollbar-hide">
```
