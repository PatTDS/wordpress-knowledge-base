---
title: How to Generate and Implement Critical CSS
description: Step-by-step guide to extracting, inlining, and managing critical CSS for above-the-fold content in WordPress
category: performance
type: howto
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - critical-css
  - css
  - fcp
  - lcp
  - render-blocking
  - performance
prerequisites:
  - Node.js 18+ (for CLI tools)
  - Understanding of CSS and above-the-fold concept
audience: intermediate
sources:
  - https://wp-rocket.me/blog/critical-css/
  - https://nitropack.io/blog/post/critical-css
  - https://10web.io/blog/inline-critical-css-defer-unused-css-wordpress/
  - https://jetpack.com/resources/wordpress-critical-css/
  - https://w3speedup.com/how-to-generate-critical-css-in-wordpress/
related:
  - concept-core-web-vitals.md
  - howto-javascript-optimization.md
---

# How to Generate and Implement Critical CSS

Critical CSS is the minimum CSS required to render above-the-fold content. Inlining it eliminates render-blocking CSS requests and dramatically improves First Contentful Paint (FCP) and Largest Contentful Paint (LCP).

## Understanding Critical CSS

### What Gets Included

Critical CSS should contain styles for:
- Page layout structure visible above fold
- Typography (fonts, sizes, colors)
- Header and navigation
- Hero section/banner
- Any visible buttons or forms
- Basic color scheme

### What to Exclude

- Footer styles
- Below-fold sections
- Modal/popup styles
- Animation keyframes (unless immediate)
- Print styles
- Hover/focus states (optional)

### Target Size

**Keep critical CSS under 14KB** (the size of one TCP roundtrip packet). Larger critical CSS negates the benefit by increasing HTML size.

## Part 1: Plugin-Based Solutions

### Option A: WP Rocket (Premium, Automated)

WP Rocket automatically generates critical CSS per page template.

```bash
wp plugin activate wp-rocket
```

**Enable in Settings → WP Rocket → File Optimization:**
1. Enable "Optimize CSS delivery"
2. Select "Remove Unused CSS" (generates critical CSS automatically)
3. Wait for CSS to be generated (async process)

**Regenerate Critical CSS:**
```bash
wp rocket regenerate-critical-css
```

### Option B: Jetpack Boost (Free)

```bash
wp plugin install jetpack-boost --activate
```

**Enable Critical CSS:**
1. Go to Jetpack → Boost
2. Enable "Optimize CSS Loading"
3. Click "Regenerate" after major changes

### Option C: Autoptimize + Critical CSS

```bash
wp plugin install autoptimize --activate
```

**Configuration:**
1. Enable "Optimize CSS Code"
2. Enable "Inline and Defer CSS"
3. Add manually generated critical CSS in the provided field

### Option D: Critical CSS For WP

```bash
wp plugin install critical-css-for-wp --activate
```

Free plugin that generates critical CSS using external API.

## Part 2: Manual Generation with CLI Tools

### Using critical (Node.js)

```bash
# Install globally
npm install -g critical

# Generate critical CSS for a page
critical https://yoursite.com --base ./ --inline --minify \
  --width 1920 --height 1080 \
  --css wp-content/themes/yourtheme/style.css \
  > critical.css

# Generate for mobile
critical https://yoursite.com --base ./ --minify \
  --width 375 --height 667 \
  --css wp-content/themes/yourtheme/style.css \
  > critical-mobile.css
```

### Using Penthouse

```bash
npm install -g penthouse

# Generate critical CSS
penthouse https://yoursite.com \
  --css wp-content/themes/yourtheme/style.css \
  --width 1920 --height 1080 \
  --out critical.css
```

### Using criticalCSS (API-based)

```javascript
// generate-critical.js
const criticalcss = require("criticalcss");
const fs = require("fs");

criticalcss.getRules("https://yoursite.com", {
  width: 1920,
  height: 1080
}, function(err, output) {
  fs.writeFileSync("critical.css", output);
});
```

## Part 3: Implementing Critical CSS in WordPress

### Method 1: Inline in Header (Recommended)

Add to `functions.php`:

```php
function inline_critical_css() {
    // Path to critical CSS file
    $critical_css_file = get_template_directory() . '/assets/css/critical.css';

    if (file_exists($critical_css_file)) {
        echo '<style id="critical-css">';
        echo file_get_contents($critical_css_file);
        echo '</style>';
    }
}
add_action('wp_head', 'inline_critical_css', 1);
```

### Method 2: Conditional Critical CSS

Different templates need different critical CSS:

```php
function conditional_critical_css() {
    $critical_file = get_template_directory() . '/assets/css/critical/';

    // Determine which critical CSS to use
    if (is_front_page()) {
        $critical_file .= 'home.css';
    } elseif (is_singular('post')) {
        $critical_file .= 'post.css';
    } elseif (is_archive()) {
        $critical_file .= 'archive.css';
    } elseif (is_page()) {
        $critical_file .= 'page.css';
    } else {
        $critical_file .= 'default.css';
    }

    if (file_exists($critical_file)) {
        echo '<style id="critical-css">';
        echo file_get_contents($critical_file);
        echo '</style>';
    }
}
add_action('wp_head', 'conditional_critical_css', 1);
```

### Method 3: Mobile/Desktop Variants

```php
function responsive_critical_css() {
    $base_path = get_template_directory() . '/assets/css/critical/';

    // Inline both, let media queries handle it
    if (file_exists($base_path . 'critical-mobile.css')) {
        echo '<style id="critical-mobile" media="(max-width: 768px)">';
        echo file_get_contents($base_path . 'critical-mobile.css');
        echo '</style>';
    }

    if (file_exists($base_path . 'critical-desktop.css')) {
        echo '<style id="critical-desktop" media="(min-width: 769px)">';
        echo file_get_contents($base_path . 'critical-desktop.css');
        echo '</style>';
    }
}
add_action('wp_head', 'responsive_critical_css', 1);
```

## Part 4: Deferring Non-Critical CSS

### Method 1: Using rel="preload"

```php
function defer_non_critical_css($html, $handle, $href, $media) {
    // Skip critical CSS and admin styles
    if (is_admin() || $handle === 'critical-css') {
        return $html;
    }

    // Convert to preload
    return sprintf(
        '<link rel="preload" href="%s" as="style" onload="this.onload=null;this.rel=\'stylesheet\'" media="%s">
        <noscript><link rel="stylesheet" href="%s" media="%s"></noscript>',
        $href,
        $media,
        $href,
        $media
    );
}
add_filter('style_loader_tag', 'defer_non_critical_css', 10, 4);
```

### Method 2: Using JavaScript Fallback

```html
<!-- In wp_footer -->
<script>
(function() {
    var links = document.querySelectorAll('link[rel="preload"][as="style"]');
    links.forEach(function(link) {
        link.rel = 'stylesheet';
    });
})();
</script>
```

### Method 3: loadCSS Polyfill

For older browser support:

```php
function enqueue_loadcss_polyfill() {
    ?>
    <script>
    /*! loadCSS. [c]2017 Filament Group, Inc. MIT License */
    (function(w){"use strict";var loadCSS=function(href,before,media){...};}(typeof global!=="undefined"?global:this));
    </script>
    <?php
}
add_action('wp_head', 'enqueue_loadcss_polyfill', 2);
```

## Part 5: Build Process Integration

### Gulp Task

```javascript
// gulpfile.js
const critical = require('critical');
const gulp = require('gulp');

gulp.task('critical-css', function() {
    return critical.generate({
        base: './',
        src: 'https://yoursite.com',
        css: ['wp-content/themes/yourtheme/style.css'],
        dimensions: [
            { width: 375, height: 667 },   // Mobile
            { width: 1920, height: 1080 }  // Desktop
        ],
        dest: 'wp-content/themes/yourtheme/assets/css/critical.css',
        minify: true,
        extract: false
    });
});
```

### npm Script

```json
{
  "scripts": {
    "critical": "critical https://yoursite.com --base ./ --minify --width 1920 --height 1080 > assets/css/critical.css",
    "critical:mobile": "critical https://yoursite.com --base ./ --minify --width 375 --height 667 > assets/css/critical-mobile.css"
  }
}
```

### GitHub Actions Integration

```yaml
# .github/workflows/critical-css.yml
name: Generate Critical CSS

on:
  push:
    paths:
      - '**.css'
      - '**.php'

jobs:
  critical:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: '20'

      - name: Install critical
        run: npm install -g critical

      - name: Generate Critical CSS
        run: |
          critical ${{ secrets.SITE_URL }} \
            --base ./ --minify \
            --width 1920 --height 1080 \
            > wp-content/themes/mytheme/assets/css/critical.css

      - name: Commit changes
        run: |
          git config --local user.email "bot@github.com"
          git config --local user.name "GitHub Bot"
          git add -A
          git commit -m "chore: regenerate critical CSS" || exit 0
          git push
```

## Part 6: Handling Dynamic Content

### Pages with Varied Content

For pages where above-fold content varies significantly:

```php
function dynamic_critical_css() {
    // Generate page-specific key
    $page_key = 'critical_css_' . get_the_ID();
    $critical_css = get_transient($page_key);

    if (false === $critical_css) {
        // Generate or fetch critical CSS
        $critical_css = generate_critical_css_for_page(get_permalink());
        set_transient($page_key, $critical_css, WEEK_IN_SECONDS);
    }

    echo '<style id="critical-css">' . $critical_css . '</style>';
}
```

### WooCommerce Considerations

WooCommerce pages need specific critical CSS:

```php
function wc_critical_css() {
    $css_file = get_template_directory() . '/assets/css/critical/';

    if (is_shop() || is_product_category()) {
        $css_file .= 'shop.css';
    } elseif (is_product()) {
        $css_file .= 'product.css';
    } elseif (is_cart()) {
        $css_file .= 'cart.css';
    } elseif (is_checkout()) {
        $css_file .= 'checkout.css';
    }

    // ... load appropriate file
}
```

## Part 7: Testing and Validation

### Lighthouse Audit

Check for "Eliminate render-blocking resources" in Lighthouse:
- Before: CSS files listed as render-blocking
- After: No CSS listed (or significantly reduced)

### Coverage Report

Chrome DevTools → Coverage tab:
1. Record page load
2. Check CSS coverage percentage
3. Unused CSS should be in non-critical files

### Visual Regression Test

Ensure critical CSS renders page correctly:

```bash
# Using BackstopJS
backstop test

# Or Percy
npx percy snapshot
```

### Performance Comparison

```bash
# Test FCP before and after
lighthouse https://yoursite.com --only-categories=performance \
  --output=json --output-path=./report.json

# Check FCP and LCP metrics in report
```

## Troubleshooting

### FOUC (Flash of Unstyled Content)

**Cause:** Critical CSS missing essential styles.

**Fix:**
1. Regenerate critical CSS with larger viewport
2. Manually add missing styles to critical CSS
3. Increase critical CSS scope (include more elements)

### Styles Look Wrong Above Fold

**Cause:** Critical CSS extraction missed interactive states or JS-dependent styles.

**Fix:**
1. Add specific selectors manually:
   ```css
   /* Add to critical CSS */
   .nav-open .menu { display: block; }
   ```
2. Ensure critical CSS includes all visible elements

### Critical CSS Too Large (>14KB)

**Causes:**
- Too much CSS included
- Not minified
- Unused selectors

**Fixes:**
1. Reduce viewport height in generation
2. Use PurgeCSS on source CSS first
3. Remove animation keyframes
4. Minify output

### Different on Mobile vs Desktop

**Solution:** Generate separate critical CSS files:

```bash
# Mobile
critical https://site.com --width 375 --height 667 > critical-mobile.css

# Desktop
critical https://site.com --width 1920 --height 1080 > critical-desktop.css
```

Serve conditionally with media queries or user-agent detection.

## Maintenance

### When to Regenerate

- After theme updates
- After major CSS changes
- After layout modifications
- After adding new above-fold components

### Automation

Set up CI/CD to regenerate on CSS changes (see GitHub Actions example above).

### Monitoring

- Track FCP/LCP in Search Console
- Set up alerts for regression
- Monitor CSS file sizes
