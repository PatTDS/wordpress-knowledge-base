---
title: WordPress Lighthouse Performance Optimization Tutorial
description: Step-by-step tutorial to improve your WordPress Lighthouse performance score from poor to good (70+)
category: performance
type: tutorial
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - lighthouse
  - performance
  - core-web-vitals
  - tutorial
  - optimization
prerequisites:
  - WordPress site running
  - Admin access
  - Basic understanding of web performance
audience: beginner
sources:
  - https://wp-rocket.me/lighthouse-performance-score-wordpress/
  - https://wpdeveloper.com/google-lighthouse-how-to-achieve-highest-score/
  - https://10web.io/blog/what-is-google-lighthouse-and-how-to-use-it/
  - https://appcosoftware.com/how-to-enhance-the-lighthouse-performance-score-and-metrics-on-wordpress/
related:
  - concept-core-web-vitals.md
  - ref-performance-targets.md
  - howto-image-optimization.md
  - howto-caching-setup.md
---

# WordPress Lighthouse Performance Optimization Tutorial

This tutorial guides you through improving your WordPress site's Lighthouse performance score from poor (<50) to good (70+) using a systematic 4-week approach.

## Before You Begin

### Baseline Measurement

1. Open Chrome DevTools (F12)
2. Go to Lighthouse tab
3. Select:
   - Mode: Navigation
   - Device: Mobile
   - Categories: Performance only
4. Click "Analyze page load"
5. Record your scores:
   - Performance: ___
   - LCP: ___ seconds
   - TBT: ___ ms
   - CLS: ___
   - FCP: ___ seconds
   - Speed Index: ___ seconds

### Understanding the Metrics

| Metric | Weight | Your Score | Target |
|--------|--------|------------|--------|
| First Contentful Paint | 10% | | <1.8s |
| Largest Contentful Paint | 25% | | <2.5s |
| Total Blocking Time | 30% | | <200ms |
| Cumulative Layout Shift | 25% | | <0.1 |
| Speed Index | 10% | | <3.4s |

## Week 1: Quick Wins (20-30% Improvement)

### Day 1-2: Image Optimization

Images typically cause 80% of LCP problems.

**Step 1: Install Image Optimization Plugin**

```bash
wp plugin install shortpixel-image-optimiser --activate
```

**Step 2: Configure ShortPixel**

1. Get free API key from shortpixel.com
2. Settings → ShortPixel:
   - Compression: Lossy
   - WebP creation: Yes
   - Remove EXIF: Yes
   - Resize large images: Max 2560px

**Step 3: Bulk Optimize Existing Images**

1. Media → Bulk ShortPixel
2. Start optimization
3. Wait for completion (may take hours for large libraries)

**Step 4: Verify**

Run Lighthouse again. Check "Properly size images" and "Serve images in next-gen formats" diagnostics.

### Day 3-4: Enable Caching

**Step 1: Install Caching Plugin**

```bash
# Free option
wp plugin install w3-total-cache --activate

# Or premium (recommended)
# Upload and activate WP Rocket
```

**Step 2: Configure Page Caching**

For W3 Total Cache:
1. Performance → General Settings
2. Enable Page Cache
3. Page Cache Method: Disk (Enhanced)

For WP Rocket:
- Caching enabled by default

**Step 3: Enable Browser Caching**

For W3TC:
1. Performance → Browser Cache
2. Enable all options
3. Set Expires header: 31536000 (1 year)

**Step 4: Verify Cache Working**

```bash
curl -I https://yoursite.com
# Look for: Cache-Control, Expires headers
```

### Day 5-7: Basic CDN Setup

**Step 1: Sign Up for Cloudflare**

1. cloudflare.com → Sign up
2. Add your site
3. Update nameservers at registrar

**Step 2: Basic Configuration**

1. SSL/TLS → Full (strict)
2. Speed → Optimization:
   - Auto Minify: HTML, CSS, JS
   - Brotli: On
3. Caching → Browser Cache TTL: 1 year

**Step 3: Verify CDN Active**

```bash
curl -I https://yoursite.com
# Look for: cf-cache-status, cf-ray headers
```

**Week 1 Checkpoint:**
- [ ] Images optimized (WebP served)
- [ ] Page caching enabled
- [ ] Browser caching configured
- [ ] CDN active
- Expected improvement: 20-30 points

Run Lighthouse again and record new scores.

## Week 2: LCP Optimization

LCP is weighted 25% of performance score.

### Identify LCP Element

1. Run Lighthouse
2. Scroll to "Largest Contentful Paint element"
3. Note what element it is (usually hero image or heading)

### Optimize Hero Images

**Step 1: Prioritize LCP Image**

Add to your hero image:
```html
<img src="hero.webp"
     fetchpriority="high"
     width="1920"
     height="1080"
     alt="Hero">
```

**Step 2: Preload LCP Image**

Add to `<head>` via functions.php:

```php
function preload_lcp_image() {
    if (is_front_page()) {
        echo '<link rel="preload" as="image" href="' .
             get_template_directory_uri() . '/images/hero.webp" ' .
             'type="image/webp">';
    }
}
add_action('wp_head', 'preload_lcp_image', 1);
```

**Step 3: Remove Lazy Loading from LCP**

LCP images should NOT be lazy loaded:

```php
function skip_lazy_load_hero($value, $image, $context) {
    if (strpos($image, 'hero') !== false) {
        return false;
    }
    return $value;
}
add_filter('wp_img_tag_add_loading_attr', 'skip_lazy_load_hero', 10, 3);
```

### Reduce Server Response Time

**Step 1: Check Current TTFB**

```bash
curl -o /dev/null -s -w "TTFB: %{time_starttransfer}\n" https://yoursite.com
```

Target: <200ms

**Step 2: Enable Object Caching**

If Redis available:
```bash
wp plugin install redis-cache --activate
wp redis enable
```

**Step 3: Optimize Database**

```bash
# Delete revisions
wp post delete $(wp post list --post_type='revision' --format=ids) --force

# Delete transients
wp transient delete --expired

# Optimize tables
wp db optimize
```

### Eliminate Render-Blocking Resources

**Step 1: Defer JavaScript**

In WP Rocket: File Optimization → Load JavaScript deferred

Or manually:
```php
function defer_js($tag, $handle, $src) {
    if (is_admin()) return $tag;
    return str_replace(' src', ' defer src', $tag);
}
add_filter('script_loader_tag', 'defer_js', 10, 3);
```

**Step 2: Generate Critical CSS**

In WP Rocket: File Optimization → Optimize CSS Delivery

Or with Jetpack Boost:
```bash
wp plugin install jetpack-boost --activate
```
Enable "Optimize CSS Loading"

**Week 2 Checkpoint:**
- [ ] LCP element identified and optimized
- [ ] Hero image preloaded with fetchpriority="high"
- [ ] TTFB < 500ms
- [ ] JavaScript deferred
- [ ] Critical CSS generated
- Expected LCP: <2.5s

## Week 3: INP and JavaScript Optimization

TBT (proxy for INP) is weighted 30%.

### Audit JavaScript Usage

**Step 1: Coverage Report**

1. Chrome DevTools → More tools → Coverage
2. Reload page
3. Check JS files with high unused %

**Step 2: Remove Unused Scripts**

Add to functions.php:
```php
function remove_unused_scripts() {
    // Remove emoji
    remove_action('wp_head', 'print_emoji_detection_script', 7);
    remove_action('wp_print_styles', 'print_emoji_styles');

    // Remove embed
    wp_dequeue_script('wp-embed');

    // Remove jQuery migrate if not needed
    wp_dequeue_script('jquery-migrate');
}
add_action('wp_enqueue_scripts', 'remove_unused_scripts', 100);
```

### Delay Non-Critical JavaScript

**Step 1: Using WP Rocket**

File Optimization → Delay JavaScript execution: ON

Add to delay list:
```
/gtag/js
/analytics.js
/fbevents.js
/hotjar
/intercom
```

**Step 2: Manual Delay**

```php
function delay_non_critical_js() {
    ?>
    <script>
    const delayScripts = () => {
        document.querySelectorAll('script[data-delay]').forEach(s => {
            const ns = document.createElement('script');
            ns.src = s.getAttribute('data-delay');
            document.body.appendChild(s);
        });
    };
    ['mouseover','click','keydown','touchstart','scroll'].forEach(e =>
        window.addEventListener(e, delayScripts, {once:true, passive:true}));
    setTimeout(delayScripts, 5000);
    </script>
    <?php
}
add_action('wp_head', 'delay_non_critical_js');
```

### Conditional Script Loading

**Install Asset CleanUp:**
```bash
wp plugin install wp-asset-clean-up --activate
```

Configure per page:
1. Edit page
2. Scroll to "Asset CleanUp" metabox
3. Disable scripts not needed on that page

### Optimize Third-Party Scripts

**YouTube Embeds:**
```bash
wp plugin install starter-starter --activate
# Or use Lite YouTube Embed
```

**Chat Widgets:**
Only load on scroll:
```javascript
window.addEventListener('scroll', function() {
    if (window.scrollY > 200 && !window.chatLoaded) {
        // Load chat widget
        window.chatLoaded = true;
    }
}, {passive: true});
```

**Week 3 Checkpoint:**
- [ ] Unused scripts removed
- [ ] Third-party JS delayed
- [ ] Per-page script loading configured
- [ ] YouTube embeds use facades
- Expected TBT: <300ms

## Week 4: CLS and Final Refinement

CLS is weighted 25%.

### Fix Layout Shifts

**Step 1: Add Dimensions to All Images**

```php
// WordPress 5.5+ does this automatically
// Verify in your theme:
add_filter('wp_calculate_image_srcset_meta', '__return_true');
```

Check all images have width/height:
```html
<img src="image.jpg" width="800" height="600" alt="Description">
```

**Step 2: Reserve Space for Ads/Embeds**

```css
.ad-container {
    min-height: 250px;
    background: #f0f0f0;
}

.video-container {
    aspect-ratio: 16/9;
    width: 100%;
}
```

**Step 3: Optimize Font Loading**

```css
@font-face {
    font-family: 'CustomFont';
    src: url('font.woff2') format('woff2');
    font-display: swap;
}
```

Preload critical fonts:
```html
<link rel="preload" as="font" type="font/woff2"
      href="/fonts/main.woff2" crossorigin>
```

**Step 4: Avoid Dynamic Content Injection**

Don't inject content above existing content. If needed:
```css
.notification-bar {
    position: fixed;
    top: 0;
    /* Doesn't shift content */
}
```

### Final Optimizations

**Enable Preconnect:**
```php
function add_preconnect() {
    echo '<link rel="preconnect" href="https://fonts.googleapis.com">';
    echo '<link rel="preconnect" href="https://cdn.yoursite.com" crossorigin>';
}
add_action('wp_head', 'add_preconnect', 1);
```

**Enable DNS Prefetch:**
```php
function add_dns_prefetch() {
    echo '<link rel="dns-prefetch" href="//www.google-analytics.com">';
    echo '<link rel="dns-prefetch" href="//www.googletagmanager.com">';
}
add_action('wp_head', 'add_dns_prefetch', 1);
```

**Final Database Cleanup:**
```bash
wp transient delete --all
wp cache flush
wp db optimize
```

### Verification

**Run Final Lighthouse:**
1. Incognito window
2. Lighthouse → Mobile → Performance
3. Run 3 times, take average

**Target Scores:**
| Metric | Target |
|--------|--------|
| Performance | 70+ |
| LCP | <2.5s |
| TBT | <200ms |
| CLS | <0.1 |

**Week 4 Checkpoint:**
- [ ] All images have dimensions
- [ ] Ad/embed space reserved
- [ ] Fonts optimized with font-display: swap
- [ ] Preconnect/prefetch configured
- [ ] CLS < 0.1
- Expected score: 70+

## Ongoing Maintenance

### Weekly Tasks

```bash
# Clear expired transients
wp transient delete --expired

# Check performance
lighthouse https://yoursite.com --preset=desktop
```

### Monthly Tasks

```bash
# Database optimization
wp db optimize

# Plugin audit
wp plugin list

# Update everything
wp core update
wp plugin update --all
wp theme update --all
```

### Per-Deployment

- Run Lighthouse in CI/CD
- Compare scores to baseline
- Alert on regression >5 points

## Troubleshooting Common Issues

### Score Fluctuates Significantly

**Cause:** Lighthouse variability, third-party scripts
**Fix:** Run 3+ times, take median

### LCP Still Slow After Optimization

**Check:**
1. Is LCP element preloaded?
2. Is server response fast (<200ms)?
3. Are fonts blocking rendering?

### TBT/INP Still High

**Check:**
1. Third-party scripts delayed?
2. Main thread not blocked?
3. Long tasks in Performance tab?

### CLS Score Won't Improve

**Check:**
1. All images have dimensions?
2. Fonts using font-display: swap?
3. Dynamic content inserting above fold?

## Summary Checklist

### Quick Wins (Week 1)
- [ ] Image optimization plugin active
- [ ] WebP being served
- [ ] Page caching enabled
- [ ] Browser caching configured
- [ ] CDN active

### LCP (Week 2)
- [ ] Hero image <150KB
- [ ] fetchpriority="high" on LCP element
- [ ] LCP image preloaded
- [ ] No lazy loading on LCP
- [ ] TTFB <200ms

### JavaScript (Week 3)
- [ ] All JS deferred or delayed
- [ ] Unused scripts removed
- [ ] Third-party JS delayed
- [ ] Per-page script loading
- [ ] TBT <200ms

### CLS (Week 4)
- [ ] All images have dimensions
- [ ] Fonts use font-display: swap
- [ ] Space reserved for dynamic content
- [ ] CLS <0.1

## Tools Reference

| Tool | Purpose |
|------|---------|
| PageSpeed Insights | Quick performance check |
| Lighthouse (Chrome) | Detailed analysis |
| WebPageTest | Waterfall analysis |
| Chrome DevTools Coverage | Unused CSS/JS |
| Query Monitor | Database/PHP analysis |
| GTmetrix | Historical tracking |
