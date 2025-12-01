---
title: Tailwind CSS Best Practices Reference
description: Production optimization and best practices for Tailwind CSS v3/v4 in 2024-2025
category: webdesign
type: reference
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - tailwind
  - css
  - performance
  - optimization
sources:
  - https://v3.tailwindcss.com/docs/optimizing-for-production
  - https://tailgrids.com/blog/tailwind-css-best-practices-and-performance-optimization
  - https://www.uxpin.com/studio/blog/tailwind-best-practices/
  - https://medium.com/@sureshdotariya/tailwind-css-4-best-practices-for-enterprise-scale-projects-2025-playbook-bf2910402581
  - https://www.wisp.blog/blog/best-practices-for-using-tailwind-css-in-large-projects
---

# Tailwind CSS Best Practices Reference

Production optimization and best practices for Tailwind CSS in 2024-2025.

## Performance Optimization

### Production Build Optimization

For the smallest possible production build:

1. **Enable PurgeCSS** - Automatically removes unused styles
2. **Minify CSS** - Use cssnano for minification
3. **Compress with Brotli** - Better compression than gzip

```bash
# Production build command
npm run build
```

### JIT (Just-In-Time) Compiler

JIT mode generates styles on-demand during development:

- Reduces initial file size
- Enables rapid prototyping
- Generates only required styles

```js
// tailwind.config.js (v3)
module.exports = {
  mode: 'jit',
  content: ['./src/**/*.{js,jsx,ts,tsx,html}'],
}
```

### Content Configuration

Specify all template paths to ensure proper purging:

```js
// tailwind.config.js
module.exports = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx}',
    './components/**/*.{js,ts,jsx,tsx}',
    './app/**/*.{js,ts,jsx,tsx}',
  ],
}
```

## Code Organization

### Class Order Convention

Adopt a standard order for utility classes:

1. Layout (display, position, grid/flex)
2. Sizing (width, height)
3. Spacing (margin, padding)
4. Typography (font, text)
5. Visual (colors, borders)
6. Interactive (hover, focus)

### Automatic Class Sorting

Use the official Prettier plugin:

```bash
npm install -D prettier-plugin-tailwindcss
```

```json
// .prettierrc
{
  "plugins": ["prettier-plugin-tailwindcss"]
}
```

### Component Extraction

For repeated patterns, extract to components rather than `@apply`:

```jsx
// Good: Component extraction
function Button({ children, variant = 'primary' }) {
  const variants = {
    primary: 'bg-blue-500 hover:bg-blue-600 text-white',
    secondary: 'bg-gray-200 hover:bg-gray-300 text-gray-800',
  }
  return (
    <button className={`px-4 py-2 rounded ${variants[variant]}`}>
      {children}
    </button>
  )
}
```

## Using @apply

### When to Use

- Creating base styles for third-party components
- Defining styles in CSS files where class markup isn't accessible

### When to Avoid

- General component styling (use component extraction)
- Complex patterns (harder to maintain)
- When it increases CSS bundle size

```css
/* Acceptable: Base form styles */
@layer components {
  .form-input {
    @apply w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-blue-500;
  }
}
```

## Tailwind CSS v4 (2025)

### CSS-First Token Model

Define design tokens with `@theme`:

```css
@theme {
  --color-primary: oklch(0.7 0.15 200);
  --color-secondary: oklch(0.6 0.1 280);
  --spacing-base: 1rem;
  --font-sans: 'Inter', system-ui, sans-serif;
}
```

### Multi-Brand UI Support

Ship different brand themes without rebuilds using CSS custom properties.

## Essential Plugins

### Official Plugins

| Plugin | Purpose |
|--------|---------|
| @tailwindcss/forms | Better form element styling |
| @tailwindcss/typography | Rich text content styling |
| @tailwindcss/aspect-ratio | Aspect ratio utilities |
| @tailwindcss/container-queries | Container query support |

```bash
npm install @tailwindcss/forms @tailwindcss/typography
```

```js
// tailwind.config.js
module.exports = {
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography'),
  ],
}
```

## Next.js Integration

### App Router Compatibility

Tailwind works fully with React Server Components:

- Styles applied through static class names
- No runtime JavaScript for styling
- Compatible with streaming and Suspense

### Configuration

```js
// tailwind.config.js for Next.js
module.exports = {
  content: [
    './app/**/*.{js,ts,jsx,tsx,mdx}',
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
  ],
}
```

## Dark Mode

### Class-Based Strategy

```js
// tailwind.config.js
module.exports = {
  darkMode: 'class',
}
```

```html
<div class="bg-white dark:bg-gray-900">
  <p class="text-gray-900 dark:text-white">Content</p>
</div>
```

### System Preference Strategy

```js
// tailwind.config.js
module.exports = {
  darkMode: 'media',
}
```

## Performance Checklist

- [ ] Content paths configured correctly
- [ ] Production build generates minimal CSS
- [ ] Prettier plugin installed for class sorting
- [ ] Unused utilities not manually added
- [ ] @apply used sparingly
- [ ] Component extraction for repeated patterns
- [ ] Official plugins for common patterns

## File Size Targets

| Stage | Expected Size |
|-------|---------------|
| Development | 3-5 MB (includes all utilities) |
| Production (purged) | 10-50 KB |
| Production (gzipped) | 5-15 KB |

## Related Documents

- @webdesign/howto-css-performance.md
- @webdesign/ref-layout-patterns.md
- @webdesign/ref-color-systems.md
