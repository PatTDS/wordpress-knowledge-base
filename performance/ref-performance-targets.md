---
title: WordPress Performance Targets Reference
description: Definitive reference for WordPress performance metrics, thresholds, and benchmarks based on 2024-2025 industry standards
category: performance
type: reference
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - performance
  - metrics
  - benchmarks
  - lighthouse
  - core-web-vitals
audience: intermediate
sources:
  - https://developers.google.com/search/docs/appearance/core-web-vitals
  - https://web.dev/performance/
  - https://nitropack.io/blog/post/core-web-vitals
  - https://wp-rocket.me/lighthouse-performance-score-wordpress/
related:
  - concept-core-web-vitals.md
  - tutorial-lighthouse-optimization.md
---

# WordPress Performance Targets Reference

## Core Web Vitals Thresholds

### Official Google Thresholds (2024-2025)

| Metric | Good | Needs Improvement | Poor |
|--------|------|-------------------|------|
| LCP (Largest Contentful Paint) | ≤ 2.5s | 2.5s - 4.0s | > 4.0s |
| INP (Interaction to Next Paint) | ≤ 200ms | 200ms - 500ms | > 500ms |
| CLS (Cumulative Layout Shift) | ≤ 0.1 | 0.1 - 0.25 | > 0.25 |

### Lighthouse Performance Metrics

| Metric | Weight | Good Threshold |
|--------|--------|----------------|
| First Contentful Paint (FCP) | 10% | ≤ 1.8s |
| Largest Contentful Paint (LCP) | 25% | ≤ 2.5s |
| Total Blocking Time (TBT) | 30% | ≤ 200ms |
| Cumulative Layout Shift (CLS) | 25% | ≤ 0.1 |
| Speed Index | 10% | ≤ 3.4s |

**Note:** Time to Interactive (TTI) was removed in Lighthouse 10.

## Lighthouse Score Ranges

| Score | Rating | Description |
|-------|--------|-------------|
| 90-100 | Good (Green) | Minimal optimization needed |
| 50-89 | Needs Improvement (Orange) | Optimization recommended |
| 0-49 | Poor (Red) | Critical optimization required |

### WordPress-Specific Targets

| Scenario | Minimum Target | Recommended Target |
|----------|----------------|-------------------|
| Simple brochure site | 70 | 85+ |
| Blog with images | 65 | 80+ |
| WooCommerce store | 50 | 70+ |
| Complex dynamic site | 45 | 65+ |

## Server Performance Metrics

### Time to First Byte (TTFB)

| Rating | Threshold |
|--------|-----------|
| Good | ≤ 200ms |
| Acceptable | 200ms - 500ms |
| Slow | 500ms - 1000ms |
| Poor | > 1000ms |

### Server Response Time Benchmarks

| Hosting Type | Expected TTFB |
|--------------|---------------|
| Static CDN | 50-100ms |
| Managed WordPress (premium) | 100-200ms |
| Quality shared hosting | 200-400ms |
| Budget shared hosting | 400-1000ms |
| Unoptimized VPS | 500-2000ms |

## Page Weight Targets

### Total Page Size

| Type | Target | Maximum |
|------|--------|---------|
| Homepage | < 1.5 MB | 3 MB |
| Blog post | < 1 MB | 2 MB |
| Product page | < 2 MB | 4 MB |
| Portfolio/gallery | < 2.5 MB | 5 MB |

### Asset Budgets

| Asset Type | Budget per Page |
|------------|-----------------|
| HTML | < 50 KB |
| CSS (total) | < 100 KB |
| JavaScript (total) | < 300 KB |
| Images (total) | < 1 MB |
| Fonts (total) | < 100 KB |

### Critical Resource Limits

| Resource | Target |
|----------|--------|
| Critical CSS | < 14 KB (fits in first TCP roundtrip) |
| Above-fold images | < 200 KB combined |
| Hero image | < 150 KB (after optimization) |

## Request Count Targets

| Type | Target | Maximum |
|------|--------|---------|
| Total HTTP requests | < 50 | 100 |
| JavaScript files | < 10 | 20 |
| CSS files | < 5 | 10 |
| Image requests | < 30 | 50 |
| Third-party requests | < 10 | 20 |

## Image Optimization Targets

### File Size by Dimension

| Image Width | Max File Size (WebP) | Max File Size (JPEG) |
|-------------|---------------------|----------------------|
| Thumbnail (150px) | 5 KB | 10 KB |
| Medium (300px) | 15 KB | 30 KB |
| Large (1024px) | 50 KB | 100 KB |
| Full HD (1920px) | 100 KB | 200 KB |
| 4K (3840px) | 200 KB | 400 KB |

### Compression Targets

| Format | Quality Setting | Use Case |
|--------|-----------------|----------|
| WebP | 80-85% | General images |
| AVIF | 75-80% | Modern browsers |
| JPEG | 82-85% | Fallback |
| PNG-8 | N/A | Simple graphics, icons |

### Format Comparison

| Format | Compression vs JPEG |
|--------|---------------------|
| JPEG | Baseline |
| WebP | 25-34% smaller |
| AVIF | ~50% smaller |

## Caching Targets

### Browser Cache Duration

| Resource Type | Recommended TTL |
|---------------|-----------------|
| Static assets (CSS/JS) | 1 year (with versioning) |
| Images | 1 year |
| Fonts | 1 year |
| HTML pages | 0 or short (for dynamic) |

### Object Cache Hit Ratio

| Rating | Hit Ratio |
|--------|-----------|
| Excellent | > 95% |
| Good | 85-95% |
| Acceptable | 70-85% |
| Poor | < 70% |

### CDN Cache Hit Ratio

| Provider Type | Expected Hit Ratio |
|---------------|-------------------|
| Cloudflare | 50-60% |
| BunnyCDN | 70-90% |
| Premium CDN | 80-95% |

## JavaScript Performance Targets

### Main Thread Time

| Metric | Good | Poor |
|--------|------|------|
| Total Blocking Time | < 200ms | > 600ms |
| Max Potential FID | < 100ms | > 300ms |
| Long Tasks | 0 tasks > 50ms | Any task > 150ms |

### Script Evaluation

| Metric | Target |
|--------|--------|
| JavaScript execution time | < 2s |
| Script parse time | < 500ms |
| Third-party JS | < 30% of total JS time |

## WordPress-Specific Benchmarks

### Plugin Performance Impact

| Plugin Type | Acceptable Overhead |
|-------------|---------------------|
| Security | +100-200ms TTFB |
| Caching | -200ms to -500ms TTFB |
| SEO | +50-100ms |
| Page builder | +200-500ms |
| WooCommerce | +100-300ms |
| Forms | +50-100ms |

### Database Query Targets

| Metric | Target | Warning | Critical |
|--------|--------|---------|----------|
| Total queries per page | < 50 | 50-100 | > 100 |
| Slow queries (>100ms) | 0 | 1-3 | > 3 |
| Duplicate queries | 0 | 1-5 | > 5 |

## Mobile vs Desktop Targets

### Mobile-First Metrics

| Metric | Mobile Target | Desktop Target |
|--------|---------------|----------------|
| LCP | < 2.5s | < 2.0s |
| INP | < 200ms | < 100ms |
| CLS | < 0.1 | < 0.1 |
| Speed Index | < 4.0s | < 3.0s |

### Network Simulation

| Connection | Download | Latency |
|------------|----------|---------|
| Fast 3G | 1.6 Mbps | 150ms |
| Slow 4G | 3 Mbps | 100ms |
| 4G | 9 Mbps | 50ms |
| WiFi | 30 Mbps | 20ms |

## Monitoring Thresholds

### Alerting Recommendations

| Metric | Warning | Critical |
|--------|---------|----------|
| Lighthouse Performance | < 60 | < 40 |
| LCP | > 3.0s | > 4.0s |
| CLS | > 0.15 | > 0.25 |
| TTFB | > 500ms | > 1000ms |
| Uptime | < 99.5% | < 99% |

### Regression Detection

| Metric | Significant Change |
|--------|-------------------|
| Lighthouse score | ± 5 points |
| LCP | ± 500ms |
| CLS | ± 0.05 |
| Page weight | ± 200 KB |
| Request count | ± 10 requests |

## Industry Benchmarks (2024-2025)

### WordPress Site Statistics

| Metric | Average | Top 25% |
|--------|---------|---------|
| CWV pass rate (mobile) | ~50% | >75% |
| Lighthouse Performance | 35-45 | 70+ |
| Page weight | 2.5 MB | < 1.5 MB |
| Load time | 4-6s | < 2.5s |

### Traffic Impact Correlation

| CWV Status | Observed Traffic Change |
|------------|------------------------|
| All metrics pass | +10-40% traffic lift |
| LCP improvement 0.5s | +8% conversion rate |
| Bounce rate per 1s delay | +10-20% increase |

## Testing Methodology

### Recommended Test Conditions

| Setting | Value |
|---------|-------|
| CPU throttling | 4x slowdown |
| Network | Fast 3G or Slow 4G |
| Device | Moto G4 (emulated) |
| Sample size | Minimum 3 runs |
| Time of day | Off-peak hours |

### Consistency Requirements

| Metric | Acceptable Variance |
|--------|---------------------|
| Lighthouse score | ± 5 points |
| LCP | ± 300ms |
| CLS | ± 0.02 |
| TTFB | ± 50ms |
