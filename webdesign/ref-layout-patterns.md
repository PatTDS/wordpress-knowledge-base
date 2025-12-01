---
title: Modern CSS Layout Patterns Reference
description: CSS Grid, Flexbox, auto-fit/minmax patterns, and intrinsic layout techniques
category: webdesign
type: reference
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - grid
  - flexbox
  - layout
  - responsive
sources:
  - https://css-tricks.com/snippets/css/complete-guide-grid/
  - https://blog.pixelfreestudio.com/ultimate-guide-to-css-grid-and-flexbox-layouts-in-2024/
  - https://mayashavin.com/articles/auto-fit-layout-css-flex-vs-grid
  - https://codingchefs.com/articles/modern-css-layouts-mastering-grid-flexbox-and-subgrid-in-2025
  - https://medium.com/@orami98/15-modern-css-grid-techniques-that-will-replace-your-flexbox-layouts-in-2025-fef21d2e1479
---

# Modern CSS Layout Patterns Reference

Essential Grid and Flexbox patterns for responsive layouts.

## When to Use Grid vs Flexbox

| Use Case | Recommended | Reason |
|----------|-------------|--------|
| Page layout | Grid | Two-dimensional control |
| Card grids | Grid | Equal sizing, auto-fit |
| Navigation | Flexbox | One-dimensional, variable items |
| Component interior | Flexbox | Alignment and distribution |
| Overlapping elements | Grid | Grid areas, z-index |
| Unknown item count | Grid (auto-fit) | Automatic columns |

**Rule of thumb:** Grid for layout structure, Flexbox for component internals.

## The Most Famous CSS Grid Pattern

```css
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
}
```

### How It Works

| Part | Function |
|------|----------|
| `repeat()` | Creates multiple columns |
| `auto-fit` | Fits columns to container width |
| `minmax(250px, 1fr)` | Min 250px, max 1 fraction |
| `gap` | Space between items |

### Result

- Columns automatically adjust to fit container
- Each column minimum 250px wide
- Columns expand to fill available space
- Responsive without media queries

## auto-fit vs auto-fill

```css
/* auto-fit: Expands items to fill space */
grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));

/* auto-fill: Keeps empty column slots */
grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
```

| Property | Behavior with Few Items |
|----------|------------------------|
| auto-fit | Items expand to fill row |
| auto-fill | Empty column spaces preserved |

## Essential Grid Patterns

### 1. Responsive Card Grid

```css
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(300px, 100%), 1fr));
  gap: 1.5rem;
}
```

The `min(300px, 100%)` prevents overflow on small screens.

### 2. Holy Grail Layout

```css
.layout {
  display: grid;
  grid-template-rows: auto 1fr auto;
  grid-template-columns: 200px 1fr 200px;
  min-height: 100vh;
}

.header { grid-column: 1 / -1; }
.sidebar-left { grid-column: 1; }
.main { grid-column: 2; }
.sidebar-right { grid-column: 3; }
.footer { grid-column: 1 / -1; }

@media (max-width: 768px) {
  .layout {
    grid-template-columns: 1fr;
  }
  .sidebar-left,
  .sidebar-right {
    grid-column: 1;
  }
}
```

### 3. Sticky Footer

```css
.page {
  display: grid;
  grid-template-rows: auto 1fr auto;
  min-height: 100vh;
}

.header { /* auto height */ }
.main { /* takes remaining space */ }
.footer { /* auto height, always at bottom */ }
```

### 4. Centered Content with Max Width

```css
.container {
  display: grid;
  grid-template-columns:
    minmax(1rem, 1fr)
    minmax(0, 75ch)
    minmax(1rem, 1fr);
}

.container > * {
  grid-column: 2;
}

.container > .full-width {
  grid-column: 1 / -1;
}
```

### 5. Masonry-Style (CSS Grid)

```css
.masonry {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  grid-auto-rows: 10px;
  gap: 1rem;
}

.masonry-item {
  /* Set row span based on content height */
  grid-row-end: span var(--rows, 20);
}
```

Note: True CSS masonry is coming. Current approaches require JavaScript for dynamic heights.

### 6. Subgrid

```css
.parent {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
}

.child {
  display: grid;
  grid-column: span 2;
  grid-template-columns: subgrid;
}
```

Subgrid aligns nested items to parent grid lines.

## Essential Flexbox Patterns

### 1. Center Everything

```css
.center {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

### 2. Space Between Navigation

```css
.nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.nav-links {
  display: flex;
  gap: 1rem;
}
```

### 3. Push Item to End

```css
.header {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.header .user-menu {
  margin-left: auto; /* Pushes to end */
}
```

### 4. Equal Height Cards

```css
.cards {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

.card {
  flex: 1 1 300px;
  display: flex;
  flex-direction: column;
}

.card-content {
  flex: 1; /* Takes remaining space */
}

.card-footer {
  margin-top: auto; /* Always at bottom */
}
```

### 5. Responsive Flex Items

```css
.container {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

.item {
  flex: 1 1 max(200px, 30%);
}
```

### 6. Vertical Stack with Gap

```css
.stack {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
```

## Combining Grid and Flexbox

### Layout Pattern: Grid Structure + Flex Details

```css
/* Grid for page structure */
.page {
  display: grid;
  grid-template-columns: 250px 1fr;
  grid-template-rows: auto 1fr auto;
  min-height: 100vh;
}

/* Flexbox for component internals */
.sidebar {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.card {
  display: flex;
  flex-direction: column;
}

.card-actions {
  display: flex;
  gap: 0.5rem;
  margin-top: auto;
}
```

## Intrinsic Sizing Patterns

### 1. Fit Content

```css
.button {
  width: fit-content;
  min-width: min-content;
  max-width: max-content;
}
```

### 2. Clamp for Containers

```css
.container {
  width: clamp(300px, 90%, 1200px);
  margin-inline: auto;
}
```

### 3. Min/Max Pattern

```css
.sidebar {
  width: min(300px, 100%);
}

.hero {
  height: max(400px, 50vh);
}
```

## Tailwind CSS Equivalents

| CSS Pattern | Tailwind Classes |
|-------------|------------------|
| `grid-template-columns: repeat(auto-fit, minmax(250px, 1fr))` | `grid grid-cols-[repeat(auto-fit,minmax(250px,1fr))]` |
| `display: flex; gap: 1rem` | `flex gap-4` |
| `justify-content: space-between` | `justify-between` |
| `flex: 1 1 300px` | `flex-1 basis-[300px]` or custom |
| `margin-left: auto` | `ml-auto` |
| `width: fit-content` | `w-fit` |

### Tailwind Grid

```html
<!-- Auto-fit grid -->
<div class="grid grid-cols-[repeat(auto-fit,minmax(250px,1fr))] gap-4">
  <div>Item 1</div>
  <div>Item 2</div>
  <div>Item 3</div>
</div>

<!-- Responsive with breakpoints -->
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  <div>Item 1</div>
  <div>Item 2</div>
  <div>Item 3</div>
</div>
```

## Common Gotchas

### 1. Grid Items Overflow

```css
/* Problem: Long content overflows */
.grid {
  grid-template-columns: repeat(3, 1fr);
}

/* Solution: Use minmax with 0 min */
.grid {
  grid-template-columns: repeat(3, minmax(0, 1fr));
}
```

### 2. Flex Items Don't Wrap

```css
/* Problem: Items overflow container */
.flex-container {
  display: flex;
}

/* Solution: Add flex-wrap */
.flex-container {
  display: flex;
  flex-wrap: wrap;
}
```

### 3. Gap Not Working in Safari

Gap in Flexbox supported in Safari 14.1+. For older support:

```css
/* Fallback for older browsers */
.flex-container > * + * {
  margin-left: 1rem;
}

/* Modern browsers */
@supports (gap: 1rem) {
  .flex-container {
    gap: 1rem;
  }
  .flex-container > * + * {
    margin-left: 0;
  }
}
```

### 4. min-content Breaks Layout

```css
/* Problem: min(250px, 100%) fails on small screens */
grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));

/* Solution: Use min() function */
grid-template-columns: repeat(auto-fit, minmax(min(250px, 100%), 1fr));
```

## Browser Support

| Feature | Chrome | Firefox | Safari | Edge |
|---------|--------|---------|--------|------|
| CSS Grid | 57+ | 52+ | 10.1+ | 16+ |
| Flexbox Gap | 84+ | 63+ | 14.1+ | 84+ |
| Subgrid | 117+ | 71+ | 16+ | 117+ |
| Container Queries | 105+ | 110+ | 16+ | 105+ |

## Quick Reference

### Grid Properties

```css
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: auto;
  gap: 1rem;
  place-items: center;
}
```

### Flexbox Properties

```css
.flex {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 1rem;
}
```

## Related Documents

- @webdesign/concept-responsive-design.md
- @webdesign/ref-tailwind-css.md
- @webdesign/howto-css-performance.md
