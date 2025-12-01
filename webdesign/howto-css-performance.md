---
title: How to Optimize CSS Performance
description: Critical CSS, render-blocking elimination, and CSS delivery optimization techniques
category: webdesign
type: howto
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - performance
  - css
  - critical-css
  - optimization
sources:
  - https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Performance/CSS
  - https://kinsta.com/blog/optimize-css/
  - https://wp-rocket.me/blog/critical-css/
  - https://www.speedcurve.com/web-performance-guide/using-critical-css-for-faster-rendering/
  - https://developers.google.com/speed/docs/insights/OptimizeCSSDelivery
---

# How to Optimize CSS Performance

Eliminate render-blocking CSS and improve page load times.

## Why CSS is Render-Blocking

CSS blocks page rendering because:
- Browser must build CSSOM before rendering
- Rules can be overwritten, so parsing must complete
- Content position depends on computed styles

**User impact:** Pages appear blank for 3+ seconds and users abandon them.

## Step 1: Identify Critical CSS

Critical CSS = styles for above-the-fold content visible on initial load.

### Manual Identification

1. Load page in browser
2. Identify elements visible without scrolling
3. Extract only styles used by those elements

### Automated Extraction

Using the Critical npm package:

```bash
npm install critical --save-dev
```

```js
// generate-critical.js
const critical = require('critical');

critical.generate({
  base: 'dist/',
  src: 'index.html',
  target: {
    css: 'critical.css',
    html: 'index-critical.html'
  },
  width: 1300,
  height: 900,
  inline: true
});
```

### Online Tools

- [Sitelocity Critical CSS Generator](https://www.sitelocity.com/critical-path-css-generator)
- [Penthouse](https://github.com/pocketjoso/penthouse)

## Step 2: Inline Critical CSS

Place critical CSS directly in the `<head>`:

```html
<head>
  <style>
    /* Critical CSS - above-the-fold styles */
    body { font-family: sans-serif; margin: 0; }
    .header { background: #fff; padding: 1rem; }
    .hero { min-height: 50vh; display: flex; }
    /* ... only what's visible initially */
  </style>
</head>
```

**Keep it under 14KB** for optimal first-packet delivery.

## Step 3: Defer Non-Critical CSS

Load remaining CSS without blocking:

### Method 1: Media Query Trick

```html
<link rel="stylesheet"
      href="styles.css"
      media="print"
      onload="this.media='all'">
<noscript>
  <link rel="stylesheet" href="styles.css">
</noscript>
```

### Method 2: Preload + JavaScript

```html
<link rel="preload"
      href="styles.css"
      as="style"
      onload="this.onload=null;this.rel='stylesheet'">
<noscript>
  <link rel="stylesheet" href="styles.css">
</noscript>
```

### Method 3: loadCSS Library

```html
<script>
  loadCSS("styles.css");
</script>
```

## Step 4: Split CSS by Media Query

Prevent loading unused CSS:

```html
<!-- Only loads on matching devices -->
<link rel="stylesheet" href="base.css">
<link rel="stylesheet" href="desktop.css" media="(min-width: 1024px)">
<link rel="stylesheet" href="print.css" media="print">
```

The browser downloads all but only blocks rendering for matching queries.

## Step 5: Remove Unused CSS

### Using PurgeCSS

```bash
npm install purgecss --save-dev
```

```js
// purgecss.config.js
module.exports = {
  content: ['./src/**/*.html', './src/**/*.js', './src/**/*.jsx'],
  css: ['./src/css/**/*.css'],
  output: './dist/css'
}
```

### With Tailwind CSS

Tailwind v3+ automatically purges unused styles:

```js
// tailwind.config.js
module.exports = {
  content: [
    './pages/**/*.{js,jsx}',
    './components/**/*.{js,jsx}',
  ],
  // Automatically purges unused styles in production
}
```

### Manual Audit

Use Chrome DevTools Coverage tab:
1. Open DevTools > Coverage
2. Reload page
3. Check CSS coverage (red = unused)

## Step 6: Minify CSS

### Using cssnano

```bash
npm install cssnano postcss-cli --save-dev
```

```js
// postcss.config.js
module.exports = {
  plugins: [
    require('cssnano')({
      preset: 'default',
    }),
  ],
}
```

### Build Script

```bash
npx postcss src/style.css -o dist/style.min.css
```

## Step 7: Enable Compression

### Server Configuration (Apache)

```apache
# .htaccess
<IfModule mod_deflate.c>
  AddOutputFilterByType DEFLATE text/css
</IfModule>
```

### Server Configuration (Nginx)

```nginx
gzip on;
gzip_types text/css;
gzip_min_length 1000;
```

### Brotli (Better Compression)

```nginx
brotli on;
brotli_types text/css;
```

## Step 8: Optimize Selectors

### Avoid Complex Selectors

```css
/* Bad: Complex, slow */
div.container > ul.nav li a.active span { }

/* Good: Simple, fast */
.nav-link-active { }
```

### Avoid Universal Selectors

```css
/* Bad: Matches everything */
* { box-sizing: border-box; }

/* Better: Targeted */
*, *::before, *::after {
  box-sizing: border-box;
}
```

### Use Classes Over Tags

```css
/* Slower: Tag selectors */
header nav ul li a { }

/* Faster: Class selector */
.nav-link { }
```

## Step 9: Optimize Animations

### Use Transform and Opacity

These don't trigger layout recalculations:

```css
/* Good: GPU-accelerated */
.animate {
  transition: transform 0.3s, opacity 0.3s;
}

.animate:hover {
  transform: translateY(-5px);
  opacity: 0.9;
}
```

### Avoid Layout-Triggering Properties

```css
/* Bad: Triggers layout */
.animate:hover {
  width: 110%;      /* Layout */
  margin-left: 5px; /* Layout */
  padding: 2rem;    /* Layout */
}
```

### Use will-change Sparingly

```css
/* Only when animation is about to start */
.will-animate {
  will-change: transform;
}

/* Remove after animation */
.animated {
  will-change: auto;
}
```

## Step 10: Reduce @import Usage

```css
/* Bad: Sequential loading */
@import url('base.css');
@import url('components.css');
@import url('utilities.css');

/* Good: Parallel loading via HTML */
<link rel="stylesheet" href="base.css">
<link rel="stylesheet" href="components.css">
<link rel="stylesheet" href="utilities.css">
```

Or better: bundle into single file.

## Complete Optimization Workflow

### Build Process

```bash
# 1. Compile CSS (Tailwind, Sass, etc.)
npm run build:css

# 2. Extract critical CSS
npm run critical

# 3. Purge unused CSS
npm run purge

# 4. Minify
npm run minify

# 5. Test
npm run lighthouse
```

### package.json Scripts

```json
{
  "scripts": {
    "build:css": "tailwindcss -i src/input.css -o dist/output.css",
    "critical": "node scripts/generate-critical.js",
    "purge": "purgecss --config purgecss.config.js",
    "minify": "postcss dist/*.css -d dist/min/",
    "lighthouse": "lighthouse http://localhost:3000 --output html"
  }
}
```

## WordPress-Specific Optimization

### Using Autoptimize Plugin

```php
// In functions.php or plugin
add_filter('autoptimize_filter_css_defer', '__return_true');
add_filter('autoptimize_filter_css_defer_inline', 'my_critical_css');

function my_critical_css($content) {
    return file_get_contents(get_template_directory() . '/critical.css');
}
```

### Manual Critical CSS in Theme

```php
// header.php
<head>
  <style id="critical-css">
    <?php include get_template_directory() . '/critical.css'; ?>
  </style>

  <link rel="preload"
        href="<?php echo get_stylesheet_uri(); ?>"
        as="style"
        onload="this.onload=null;this.rel='stylesheet'">
</head>
```

## Performance Targets

| Metric | Target | Tool |
|--------|--------|------|
| Total CSS size | < 50KB | DevTools |
| Critical CSS | < 14KB | Manual check |
| Unused CSS | < 10% | Coverage tab |
| Render-blocking time | < 500ms | Lighthouse |

## Checklist

- [ ] Critical CSS identified and extracted
- [ ] Critical CSS inlined in `<head>` (< 14KB)
- [ ] Non-critical CSS deferred
- [ ] Unused CSS removed (PurgeCSS/Tailwind)
- [ ] CSS minified
- [ ] Compression enabled (gzip/Brotli)
- [ ] Complex selectors simplified
- [ ] Animations use transform/opacity
- [ ] No @import statements
- [ ] Lighthouse performance > 90

## Troubleshooting

**Styles flash or break?**
- Critical CSS incomplete
- JavaScript-dependent styles missing
- Increase critical viewport dimensions

**CSS still render-blocking?**
- Check async/defer implementation
- Verify no `@import` in CSS files
- Test with disabled JavaScript

**Large CSS bundle?**
- Review PurgeCSS safelist
- Check for duplicate declarations
- Audit third-party CSS

## Related Documents

- @webdesign/ref-tailwind-css.md
- @webdesign/concept-responsive-design.md
- @reference/ref-performance-targets.md
