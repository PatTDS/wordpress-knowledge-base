---
title: Technical SEO Reference for WordPress
description: Comprehensive reference for WordPress technical SEO elements, settings, and best practices
category: seo
type: reference
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - technical-seo
  - wordpress
  - performance
  - crawling
  - indexing
audience: intermediate
sources:
  - https://wp-rocket.me/blog/technical-seo-wordpress/
  - https://aioseo.com/technical-seo/
  - https://www.searchenginejournal.com/wordpress-seo/wordpress-seo-checklist-get-ready-for-site-launch/
  - https://belovdigital.agency/blog/top-12-wordpress-seo-techniques-for-2025/
  - https://wordpress.com/blog/2025/03/14/wordpress-seo/
---

# Technical SEO Reference for WordPress

## Overview

Technical SEO encompasses the server and site-level optimizations that help search engines crawl, index, and render your WordPress site effectively. This reference covers essential elements, configurations, and best practices for 2024-2025.

## Core Technical SEO Checklist

| Element | Status | Priority |
|---------|--------|----------|
| SSL/HTTPS enabled | Required | Critical |
| XML sitemap submitted | Required | High |
| robots.txt configured | Required | High |
| Core Web Vitals passing | Required | High |
| Mobile-friendly design | Required | Critical |
| Clean URL structure | Required | High |
| Canonical tags implemented | Required | Medium |
| Structured data added | Recommended | Medium |

## URL Structure & Permalinks

### Recommended Permalink Settings

WordPress Settings > Permalinks:
- **Recommended**: Post name (`/%postname%/`)
- **Avoid**: Plain (`?p=123`) or Numeric

```
Good: example.com/wordpress-seo-guide/
Bad:  example.com/?p=123
Bad:  example.com/2025/12/01/post-title/
```

### URL Best Practices

| Do | Don't |
|----|----|
| Use hyphens as separators | Use underscores or spaces |
| Keep URLs short (3-5 words) | Exceed 75 characters |
| Use lowercase letters | Mix cases |
| Include target keyword | Stuff multiple keywords |
| Use static URLs | Use dynamic parameters |

## XML Sitemaps

### WordPress Native Sitemaps

Since WordPress 5.5, core includes built-in XML sitemaps at `/wp-sitemap.xml`.

**Included by Default:**
- Posts
- Pages
- Custom post types
- Author archives
- Taxonomies

### SEO Plugin Sitemaps

Both Rank Math and Yoast SEO provide enhanced sitemaps with more control:

```
# Yoast SEO sitemap
example.com/sitemap_index.xml

# Rank Math sitemap
example.com/sitemap_index.xml
```

### Sitemap Optimization

| Setting | Recommendation |
|---------|---------------|
| Include posts | Yes |
| Include pages | Yes |
| Include categories | Depends on content |
| Include tags | Usually no |
| Include author archives | Usually no |
| Include date archives | No |
| Include media attachments | No |

## robots.txt Configuration

### Basic robots.txt Template

```
User-agent: *
Allow: /

# Block WordPress system directories
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php

# Block plugin/theme assets paths (optional)
Disallow: /wp-includes/

# Block search results
Disallow: /?s=
Disallow: /search/

# Block trackbacks and pingbacks
Disallow: /trackback/
Disallow: */trackback/

# Block author archives (if not useful)
Disallow: /author/

# XML Sitemap location
Sitemap: https://example.com/sitemap_index.xml
```

### Common robots.txt Mistakes

| Mistake | Impact |
|---------|--------|
| Blocking CSS/JS files | Prevents rendering |
| Blocking /wp-content/ | Blocks images |
| Forgetting sitemap reference | Slower indexing |
| Blocking wp-admin/admin-ajax.php | Breaks plugin functions |

## Canonical Tags

### Implementation

SEO plugins handle canonical tags automatically. For custom implementations:

```php
// In theme's functions.php
function add_canonical_link() {
    if (is_singular()) {
        echo '<link rel="canonical" href="' . esc_url(get_permalink()) . '" />';
    }
}
add_action('wp_head', 'add_canonical_link');
```

### Common Canonical Issues

| Issue | Solution |
|-------|----------|
| HTTP vs HTTPS duplicate | Force HTTPS in .htaccess |
| www vs non-www duplicate | Set preferred in Search Console |
| Trailing slash inconsistency | Configure in WordPress settings |
| Pagination duplicates | Use rel="next/prev" or canonical to page 1 |
| Parameter duplicates | Canonical to clean URL |

## HTTPS/SSL Requirements

### WordPress HTTPS Configuration

```php
// wp-config.php
define('FORCE_SSL_ADMIN', true);

// Force HTTPS on frontend (add to functions.php)
if (!is_ssl()) {
    wp_redirect('https://' . $_SERVER['HTTP_HOST'] . $_SERVER['REQUEST_URI'], 301);
    exit();
}
```

### .htaccess HTTPS Redirect

```apache
# Force HTTPS
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
```

## Mobile Optimization

### Required Elements

| Element | Implementation |
|---------|---------------|
| Viewport meta tag | `<meta name="viewport" content="width=device-width, initial-scale=1">` |
| Responsive images | Use srcset attribute |
| Touch-friendly buttons | Minimum 48x48px touch targets |
| Readable font size | Minimum 16px base font |
| No horizontal scroll | Proper responsive CSS |

### Testing Tools

- Google Mobile-Friendly Test
- Chrome DevTools Device Mode
- PageSpeed Insights (mobile tab)

## Crawl Budget Optimization

### Factors Affecting Crawl Budget

| Factor | Impact |
|--------|--------|
| Server response time | High |
| Duplicate content | High |
| Low-value pages | Medium |
| Redirect chains | Medium |
| Orphan pages | Low |

### Optimization Strategies

```php
// Block low-value pages from indexing
function noindex_low_value_pages() {
    if (is_search() || is_date() || is_attachment()) {
        echo '<meta name="robots" content="noindex, follow" />';
    }
}
add_action('wp_head', 'noindex_low_value_pages');
```

## Internal Linking

### Best Practices

| Practice | Benefit |
|----------|---------|
| Contextual links in content | Passes PageRank effectively |
| Descriptive anchor text | Signals relevance |
| Hierarchical structure | Clear site architecture |
| Breadcrumb navigation | User experience + SEO |
| Related posts sections | Distributes link equity |

### Internal Linking Plugins

- **Link Whisper**: AI-powered suggestions
- **Yoast SEO Premium**: Internal linking suggestions
- **Rank Math**: Built-in link suggestions

## Redirect Management

### HTTP Status Codes

| Code | Use Case |
|------|----------|
| 301 | Permanent redirect (passes ~90% link equity) |
| 302 | Temporary redirect (use sparingly) |
| 307 | Temporary redirect (HTTP/1.1) |
| 308 | Permanent redirect (HTTP/1.1) |
| 410 | Content permanently deleted |

### Redirect Implementation

```php
// In theme's functions.php or plugin
function custom_redirects() {
    if (is_page('old-page-slug')) {
        wp_redirect(home_url('/new-page-slug/'), 301);
        exit();
    }
}
add_action('template_redirect', 'custom_redirects');
```

### .htaccess Redirects

```apache
# Single page redirect
Redirect 301 /old-page/ https://example.com/new-page/

# Regex redirect
RedirectMatch 301 ^/category/(.*)$ https://example.com/topics/$1
```

## Performance Requirements

### Core Web Vitals Thresholds (2024)

| Metric | Good | Needs Improvement | Poor |
|--------|------|-------------------|------|
| LCP | ≤ 2.5s | 2.5s - 4.0s | > 4.0s |
| INP | ≤ 200ms | 200ms - 500ms | > 500ms |
| CLS | ≤ 0.1 | 0.1 - 0.25 | > 0.25 |

### WordPress Performance Stack

| Layer | Solution |
|-------|----------|
| Hosting | Managed WordPress (Kinsta, WP Engine, SiteGround) |
| Page caching | WP Rocket, LiteSpeed Cache, or host-level |
| Image optimization | ShortPixel, Imagify, or Smush |
| CDN | Cloudflare, Fastly, or host CDN |
| Database optimization | WP-Optimize or Advanced Database Cleaner |

## Security Headers

### Recommended Headers

```apache
# .htaccess security headers
Header set X-Content-Type-Options "nosniff"
Header set X-Frame-Options "SAMEORIGIN"
Header set X-XSS-Protection "1; mode=block"
Header set Referrer-Policy "strict-origin-when-cross-origin"
Header set Permissions-Policy "geolocation=(), microphone=(), camera=()"
```

### Content Security Policy

```apache
Header set Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.google-analytics.com https://www.googletagmanager.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com; img-src 'self' data: https:; connect-src 'self' https://www.google-analytics.com"
```

## Hreflang for Multilingual Sites

### Implementation

```html
<link rel="alternate" hreflang="en" href="https://example.com/page/" />
<link rel="alternate" hreflang="es" href="https://example.com/es/page/" />
<link rel="alternate" hreflang="x-default" href="https://example.com/page/" />
```

### WordPress Multilingual Plugins

| Plugin | Hreflang Support |
|--------|-----------------|
| WPML | Automatic |
| Polylang | Automatic |
| TranslatePress | Automatic |
| MultilingualPress | Automatic |

## SEO Audit Checklist

### Monthly Audit Items

- [ ] Check Search Console for crawl errors
- [ ] Review Core Web Vitals report
- [ ] Scan for broken links (404s)
- [ ] Verify sitemap accuracy
- [ ] Check mobile usability issues
- [ ] Review index coverage report
- [ ] Monitor page experience signals

### Tools for Auditing

| Tool | Purpose |
|------|---------|
| Google Search Console | Index coverage, CWV, manual actions |
| Screaming Frog | Technical crawl analysis |
| Ahrefs Site Audit | Comprehensive SEO audit |
| GTmetrix | Performance analysis |
| Chrome Lighthouse | CWV and accessibility |

## Related Documents

- @seo/concept-core-web-vitals-seo.md - Understanding CWV
- @seo/howto-seo-plugin-configuration.md - Plugin setup
- @reference/ref-performance-targets.md - Performance benchmarks
