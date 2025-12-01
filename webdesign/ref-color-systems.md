---
title: Modern CSS Color Systems Reference
description: OKLCH, color-mix(), CSS custom properties, and building accessible color palettes
category: webdesign
type: reference
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - color
  - css
  - oklch
  - accessibility
sources:
  - https://evilmartians.com/chronicles/oklch-in-css-why-quit-rgb-hsl
  - https://developer.mozilla.org/en-US/blog/color-palettes-css-color-mix/
  - https://chromacreator.com/blog/css-color-variables-guide
  - https://web.dev/articles/building/a-color-scheme
  - https://www.smashingmagazine.com/2021/11/guide-modern-css-colors/
---

# Modern CSS Color Systems Reference

Building accessible, maintainable color systems with modern CSS.

## Color Space Comparison

| Color Space | Human Readable | Perceptual Uniformity | Wide Gamut | Browser Support |
|-------------|----------------|----------------------|------------|-----------------|
| Hex (#fff) | No | No | No | Universal |
| RGB | No | No | No | Universal |
| HSL | Yes | No (uneven lightness) | No | Universal |
| OKLCH | Yes | Yes | Yes | Modern browsers |
| LCH | Yes | Partial | Yes | Modern browsers |

## OKLCH: The Modern Standard

### Why OKLCH?

1. **Human readable** - You can understand color by looking at values
2. **Perceptual lightness** - 50% lightness actually looks 50% bright
3. **Predictable modifications** - Darken/lighten work as expected
4. **Wide gamut** - Supports P3 colors (30% more colors)

### OKLCH Syntax

```css
color: oklch(L C H);
color: oklch(L C H / alpha);
```

| Component | Range | Description |
|-----------|-------|-------------|
| L (Lightness) | 0-100% | Perceived brightness |
| C (Chroma) | 0-0.4+ | Color intensity/saturation |
| H (Hue) | 0-360 | Color angle on wheel |

### OKLCH Examples

```css
:root {
  /* Primary blue */
  --primary: oklch(55% 0.2 250);

  /* Lighter variant - just increase L */
  --primary-light: oklch(70% 0.2 250);

  /* Darker variant - just decrease L */
  --primary-dark: oklch(40% 0.2 250);

  /* Less saturated - decrease C */
  --primary-muted: oklch(55% 0.1 250);
}
```

## The color-mix() Function

### Basic Syntax

```css
color-mix(in <color-space>, <color1> <percentage>?, <color2> <percentage>?)
```

### Creating Color Variants

```css
:root {
  --brand: oklch(55% 0.2 250);

  /* 20% white mixed in */
  --brand-light: color-mix(in oklch, var(--brand), white 20%);

  /* 30% black mixed in */
  --brand-dark: color-mix(in oklch, var(--brand), black 30%);

  /* 50/50 mix with another color */
  --brand-accent: color-mix(in oklch, var(--brand), var(--secondary) 50%);
}
```

### Transparency with color-mix

```css
:root {
  --brand: oklch(55% 0.2 250);

  /* Semi-transparent version */
  --brand-50: color-mix(in oklch, var(--brand), transparent 50%);
}
```

## Three-Level Color Architecture

### Level 1: Palette (Raw Values)

```css
:root {
  /* Blue palette */
  --blue-50: oklch(97% 0.02 250);
  --blue-100: oklch(93% 0.04 250);
  --blue-200: oklch(85% 0.08 250);
  --blue-300: oklch(75% 0.12 250);
  --blue-400: oklch(65% 0.16 250);
  --blue-500: oklch(55% 0.20 250);
  --blue-600: oklch(45% 0.18 250);
  --blue-700: oklch(38% 0.15 250);
  --blue-800: oklch(30% 0.12 250);
  --blue-900: oklch(22% 0.08 250);

  /* Gray palette */
  --gray-50: oklch(98% 0 0);
  --gray-100: oklch(95% 0 0);
  --gray-200: oklch(90% 0 0);
  --gray-500: oklch(55% 0 0);
  --gray-700: oklch(35% 0 0);
  --gray-900: oklch(15% 0 0);
}
```

### Level 2: Functional (Semantic)

```css
:root {
  /* Brand colors */
  --color-primary: var(--blue-500);
  --color-primary-hover: var(--blue-600);
  --color-secondary: var(--gray-500);

  /* Feedback colors */
  --color-success: oklch(55% 0.18 145);
  --color-warning: oklch(70% 0.18 85);
  --color-error: oklch(55% 0.22 25);
  --color-info: var(--blue-500);

  /* Surface colors */
  --color-background: var(--gray-50);
  --color-surface: white;
  --color-border: var(--gray-200);
}
```

### Level 3: Component (Scoped)

```css
.button {
  --button-bg: var(--color-primary);
  --button-text: white;
  --button-border: transparent;

  background: var(--button-bg);
  color: var(--button-text);
  border-color: var(--button-border);
}

.button:hover {
  --button-bg: var(--color-primary-hover);
}

.button--secondary {
  --button-bg: transparent;
  --button-text: var(--color-primary);
  --button-border: var(--color-primary);
}
```

## Dark Mode Implementation

### CSS Custom Properties Approach

```css
:root {
  /* Light mode (default) */
  --background: oklch(98% 0 0);
  --foreground: oklch(15% 0 0);
  --surface: oklch(100% 0 0);
  --border: oklch(90% 0 0);
  --muted: oklch(55% 0 0);
}

[data-theme="dark"] {
  --background: oklch(15% 0 0);
  --foreground: oklch(95% 0 0);
  --surface: oklch(20% 0 0);
  --border: oklch(30% 0 0);
  --muted: oklch(60% 0 0);
}
```

### System Preference Detection

```css
@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) {
    --background: oklch(15% 0 0);
    --foreground: oklch(95% 0 0);
    /* ... dark mode values */
  }
}
```

## P3 Wide Gamut Colors

### Detection and Fallback

```css
:root {
  /* sRGB fallback */
  --vibrant-red: oklch(55% 0.22 25);

  /* P3 when supported */
  @supports (color: color(display-p3 1 0 0)) {
    --vibrant-red: oklch(55% 0.28 25);
  }
}
```

### Using display-p3

```css
.accent {
  /* Standard gamut fallback */
  background: rgb(255, 0, 100);

  /* Wide gamut for capable displays */
  @supports (color: color(display-p3 1 0 0)) {
    background: color(display-p3 1 0 0.4);
  }
}
```

## Accessibility Guidelines

### Contrast Ratios

| Use Case | Minimum Ratio (WCAG AA) | Enhanced (WCAG AAA) |
|----------|------------------------|---------------------|
| Normal text | 4.5:1 | 7:1 |
| Large text (18px+) | 3:1 | 4.5:1 |
| UI components | 3:1 | 3:1 |

### Building Accessible Palettes

```css
:root {
  /* Ensure proper contrast */
  --text-primary: oklch(20% 0 0);     /* On light: 12:1 contrast */
  --text-secondary: oklch(40% 0 0);   /* On light: 5.5:1 contrast */
  --text-muted: oklch(55% 0 0);       /* On light: 3.5:1 contrast */

  /* Interactive elements need 3:1 minimum */
  --border-focus: oklch(45% 0.2 250); /* High visibility focus */
}
```

### Focus States

```css
:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}
```

## Complete Color System Example

```css
:root {
  /* ===== PALETTE LAYER ===== */

  /* Brand */
  --brand-50: oklch(97% 0.01 250);
  --brand-100: oklch(94% 0.02 250);
  --brand-200: oklch(88% 0.06 250);
  --brand-300: oklch(78% 0.12 250);
  --brand-400: oklch(65% 0.18 250);
  --brand-500: oklch(55% 0.22 250);
  --brand-600: oklch(48% 0.20 250);
  --brand-700: oklch(40% 0.17 250);
  --brand-800: oklch(32% 0.13 250);
  --brand-900: oklch(25% 0.08 250);

  /* Neutral */
  --neutral-0: oklch(100% 0 0);
  --neutral-50: oklch(98% 0.002 250);
  --neutral-100: oklch(96% 0.004 250);
  --neutral-200: oklch(92% 0.006 250);
  --neutral-300: oklch(85% 0.008 250);
  --neutral-400: oklch(70% 0.008 250);
  --neutral-500: oklch(55% 0.008 250);
  --neutral-600: oklch(45% 0.008 250);
  --neutral-700: oklch(35% 0.008 250);
  --neutral-800: oklch(25% 0.008 250);
  --neutral-900: oklch(15% 0.008 250);
  --neutral-950: oklch(10% 0.008 250);

  /* ===== SEMANTIC LAYER ===== */

  /* Backgrounds */
  --bg-primary: var(--neutral-0);
  --bg-secondary: var(--neutral-50);
  --bg-tertiary: var(--neutral-100);

  /* Text */
  --text-primary: var(--neutral-900);
  --text-secondary: var(--neutral-600);
  --text-tertiary: var(--neutral-500);
  --text-inverse: var(--neutral-0);

  /* Borders */
  --border-primary: var(--neutral-200);
  --border-secondary: var(--neutral-100);
  --border-focus: var(--brand-500);

  /* Actions */
  --action-primary: var(--brand-500);
  --action-primary-hover: var(--brand-600);
  --action-secondary: var(--neutral-100);
  --action-secondary-hover: var(--neutral-200);

  /* Feedback */
  --success: oklch(55% 0.18 145);
  --warning: oklch(70% 0.18 85);
  --error: oklch(55% 0.22 25);
  --info: var(--brand-500);
}

/* Dark mode */
[data-theme="dark"] {
  --bg-primary: var(--neutral-950);
  --bg-secondary: var(--neutral-900);
  --bg-tertiary: var(--neutral-800);

  --text-primary: var(--neutral-50);
  --text-secondary: var(--neutral-400);
  --text-tertiary: var(--neutral-500);

  --border-primary: var(--neutral-700);
  --border-secondary: var(--neutral-800);

  --action-primary: var(--brand-400);
  --action-primary-hover: var(--brand-300);
}
```

## Tools and Generators

| Tool | Purpose | URL |
|------|---------|-----|
| OKLCH Picker | Pick OKLCH colors | oklch.com |
| Theme Machine | Generate CSS palettes | keithjgrant.com/posts/2024/06/theme-machine |
| Coolors | Palette generator | coolors.co |
| Realtime Colors | Preview on real UI | realtimecolors.com |

## Browser Support

| Feature | Chrome | Firefox | Safari | Edge |
|---------|--------|---------|--------|------|
| OKLCH | 111+ | 113+ | 15.4+ | 111+ |
| color-mix() | 111+ | 113+ | 16.2+ | 111+ |
| P3 colors | 111+ | 113+ | 15+ | 111+ |

## Related Documents

- @webdesign/ref-tailwind-css.md
- @webdesign/ref-shadcn-ui.md
- @webdesign/howto-css-performance.md
