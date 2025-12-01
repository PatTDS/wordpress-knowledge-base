---
title: shadcn/ui Component Patterns Reference
description: Best practices and patterns for building with shadcn/ui components
category: webdesign
type: reference
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - shadcn
  - react
  - components
  - tailwind
sources:
  - https://ui.shadcn.com/docs
  - https://manupa.dev/blog/anatomy-of-shadcn-ui
  - https://cursorrules.org/article/shadcn-cursor-mdc-file
  - https://www.shadcn.io/patterns
  - https://github.com/birobirobiro/awesome-shadcn-ui
---

# shadcn/ui Component Patterns Reference

Best practices and patterns for building with shadcn/ui in 2024.

## Core Philosophy

"This is not a component library. It is how you build your component library."

### Key Principles

| Principle | Description |
|-----------|-------------|
| Open Code | Top layer of component code is open for modification |
| Composition | Every component uses common, composable interface |
| Separation | Design separate from implementation |

## Architecture Patterns

### Single Responsibility

Each component has one job:

```tsx
// Badge renders with styles based on variant
// Delegates style management to variants object
const Badge = ({ variant = "default", children }) => {
  return (
    <span className={badgeVariants({ variant })}>
      {children}
    </span>
  )
}
```

### Open/Closed Principle

Add new variants without modifying existing code:

```ts
// cva (class-variance-authority) pattern
const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-md",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground",
        destructive: "bg-destructive text-destructive-foreground",
        outline: "border border-input bg-background",
        // Add new variants here without modifying above
        success: "bg-green-500 text-white",
      },
      size: {
        default: "h-10 px-4 py-2",
        sm: "h-9 rounded-md px-3",
        lg: "h-11 rounded-md px-8",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
)
```

## Code Organization

### Directory Structure

```
components/
├── ui/                    # Base shadcn components
│   ├── button.tsx
│   ├── card.tsx
│   └── dialog.tsx
├── forms/                 # Form-related components
│   ├── login-form.tsx
│   └── signup-form.tsx
├── layouts/               # Layout components
│   ├── header.tsx
│   └── sidebar.tsx
└── composite/             # Compound components
    └── data-table.tsx
```

### File Naming

- One component per file
- File named after component: `Button.tsx`
- Use PascalCase for components
- Use kebab-case for utilities

## The cn() Utility

Manage CSS class names effectively:

```ts
import { clsx, type ClassValue } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

Usage:

```tsx
<Button className={cn(
  "base-styles",
  isActive && "active-styles",
  className
)}>
```

## Composition Patterns

### Compound Components

Build complex UI from simple pieces:

```tsx
<Card>
  <CardHeader>
    <CardTitle>Title</CardTitle>
    <CardDescription>Description</CardDescription>
  </CardHeader>
  <CardContent>
    <p>Content here</p>
  </CardContent>
  <CardFooter>
    <Button>Action</Button>
  </CardFooter>
</Card>
```

### Combining Components

Create flexible patterns:

```tsx
<Dialog>
  <DialogTrigger asChild>
    <Button variant="outline">Open</Button>
  </DialogTrigger>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>Dialog Title</DialogTitle>
    </DialogHeader>
    <Form>
      {/* Form content */}
    </Form>
  </DialogContent>
</Dialog>
```

## Performance Optimization

### Lazy Loading

```tsx
import { lazy, Suspense } from 'react'

const DataTable = lazy(() => import('@/components/data-table'))

function Page() {
  return (
    <Suspense fallback={<TableSkeleton />}>
      <DataTable />
    </Suspense>
  )
}
```

### Memoization

```tsx
import { memo, useCallback } from 'react'

const ListItem = memo(function ListItem({ item, onSelect }) {
  return (
    <div onClick={() => onSelect(item.id)}>
      {item.name}
    </div>
  )
})

function List({ items }) {
  const handleSelect = useCallback((id) => {
    // Handle selection
  }, [])

  return items.map(item => (
    <ListItem key={item.id} item={item} onSelect={handleSelect} />
  ))
}
```

## Anti-Patterns to Avoid

### Direct Component Modification

```tsx
// Bad: Modifying shadcn component source
// components/ui/button.tsx
export const Button = () => {
  // Don't add business logic here
}

// Good: Create wrapper component
// components/custom-button.tsx
import { Button } from '@/components/ui/button'

export const CustomButton = ({ ...props }) => {
  // Add custom logic here
  return <Button {...props} />
}
```

### Overusing Custom CSS

```tsx
// Bad: Fighting Tailwind with custom CSS
<Button style={{ backgroundColor: 'red' }}>

// Good: Use variant system
<Button variant="destructive">
```

### Neglecting Accessibility

```tsx
// Bad: Missing accessibility
<button onClick={handleClick}>X</button>

// Good: Proper accessibility
<Button
  onClick={handleClick}
  aria-label="Close dialog"
>
  <X className="h-4 w-4" />
</Button>
```

## Customization Patterns

### CSS Variables

```css
/* globals.css */
@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --primary: 222.2 47.4% 11.2%;
    --primary-foreground: 210 40% 98%;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --primary: 210 40% 98%;
    --primary-foreground: 222.2 47.4% 11.2%;
  }
}
```

### Extending Components

```tsx
import { Button, ButtonProps } from '@/components/ui/button'
import { Loader2 } from 'lucide-react'

interface LoadingButtonProps extends ButtonProps {
  loading?: boolean
}

export function LoadingButton({
  loading,
  children,
  disabled,
  ...props
}: LoadingButtonProps) {
  return (
    <Button disabled={disabled || loading} {...props}>
      {loading && <Loader2 className="mr-2 h-4 w-4 animate-spin" />}
      {children}
    </Button>
  )
}
```

## Component Libraries Built on shadcn/ui

| Library | Focus |
|---------|-------|
| Reui | Extended component set |
| Motion Primitives | Animation components |
| Cult UI | Creative components |
| Magic UI | Visual effects |

## Installation

```bash
# Initialize shadcn/ui
npx shadcn@latest init

# Add components
npx shadcn@latest add button card dialog

# Add multiple components
npx shadcn@latest add button card dialog form input
```

## Best Practices Checklist

- [ ] Use cn() for class merging
- [ ] Keep components in logical directories
- [ ] Use composition over inheritance
- [ ] Implement lazy loading for large components
- [ ] Don't modify base shadcn components directly
- [ ] Maintain accessibility in custom components
- [ ] Use CSS variables for theming
- [ ] Follow single responsibility principle

## Related Documents

- @webdesign/ref-tailwind-css.md
- @webdesign/ref-color-systems.md
- @webdesign/concept-responsive-design.md
