---
title: How to Set Up a CDN for WordPress
description: Step-by-step guide to configuring Cloudflare, Bunny CDN, and other CDNs for optimal WordPress performance
category: performance
type: howto
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - cdn
  - cloudflare
  - bunny
  - performance
  - caching
prerequisites:
  - Domain with DNS access
  - WordPress site operational
audience: beginner
sources:
  - https://onlinemediamasters.com/cloudflare-settings-wordpress/
  - https://onlinemediamasters.com/bunnycdn-review/
  - https://bunny.net/blog/new-bunnynet-plugin-changes-the-wordpress-performance-game/
  - https://rovity.io/cloudflare-settings-to-boost-website-performance/
  - https://kinsta.com/docs/wordpress-hosting/wordpress-cdn/bunny/
related:
  - howto-caching-setup.md
  - concept-core-web-vitals.md
---

# How to Set Up a CDN for WordPress

A Content Delivery Network (CDN) caches your content on edge servers worldwide, reducing latency for visitors regardless of location. CDN setup is one of the highest-impact performance improvements.

## Understanding CDN Types

### Reverse Proxy CDN (Cloudflare)

- All traffic routes through CDN
- Includes security features (WAF, DDoS protection)
- Manages DNS
- Can cache HTML (APO)

### Pull/Origin CDN (Bunny CDN)

- Pulls assets from your origin server
- Serves static assets via CDN URL
- Origin server handles HTML
- Better cache hit ratios for assets

### When to Use Both

Cloudflare + Bunny CDN is a powerful combination:
- **Cloudflare:** DNS, security, APO for HTML
- **Bunny CDN:** Static assets with higher cache hit ratio (70-90% vs 50-60%)

## Part 1: Cloudflare Setup

### Step 1: Create Cloudflare Account

1. Go to cloudflare.com and sign up
2. Add your site
3. Select Free plan (or Pro for more features)

### Step 2: Update Nameservers

Cloudflare provides two nameservers. Update at your registrar:

```
ns1.cloudflare.com
ns2.cloudflare.com
```

Wait for propagation (up to 48 hours, usually faster).

### Step 3: Configure DNS Records

In Cloudflare DNS dashboard:

| Type | Name | Content | Proxy |
|------|------|---------|-------|
| A | @ | Your server IP | Proxied (orange cloud) |
| A | www | Your server IP | Proxied |
| CNAME | * | yourdomain.com | Proxied |

**Important:** Enable proxy (orange cloud) to route traffic through Cloudflare.

### Step 4: SSL Configuration

1. Go to SSL/TLS → Overview
2. Select "Full (strict)" mode
3. Enable "Always Use HTTPS"
4. Enable "Automatic HTTPS Rewrites"

### Step 5: Performance Settings

**Speed → Optimization:**

```
Auto Minify: HTML, CSS, JavaScript ✓
Brotli: On ✓
Early Hints: On ✓
Rocket Loader: Test carefully (can break JS)
```

**Caching → Configuration:**

```
Caching Level: Standard
Browser Cache TTL: 1 year
Always Online: On (for downtime protection)
```

### Step 6: Page Rules (3 free rules)

**Rule 1: Cache Static Assets**
```
URL: *yourdomain.com/wp-content/*
Settings:
  - Cache Level: Cache Everything
  - Edge Cache TTL: 1 month
  - Browser Cache TTL: 1 year
```

**Rule 2: Bypass Admin**
```
URL: *yourdomain.com/wp-admin/*
Settings:
  - Cache Level: Bypass
  - Disable Apps: On
  - Disable Performance: On
```

**Rule 3: Bypass Login**
```
URL: *yourdomain.com/wp-login.php*
Settings:
  - Cache Level: Bypass
```

### Step 7: Install WordPress Plugin

```bash
wp plugin install cloudflare --activate
```

Configure in Settings → Cloudflare with API key.

## Part 2: Cloudflare APO (Recommended)

Automatic Platform Optimization caches full HTML pages at edge.

### Enable APO

1. Subscribe to APO ($5/month or free with Pro plan)
2. Install Cloudflare plugin if not installed
3. Enable APO in plugin settings

### APO Benefits

- Full page caching at edge
- Automatic cache invalidation on post updates
- Works with logged-in user bypass
- ~3x faster TTFB

### Verify APO Working

```bash
curl -I https://yourdomain.com

# Look for headers:
# cf-cache-status: HIT
# cf-apo-via: origin
```

## Part 3: Bunny CDN Setup

### Step 1: Create Bunny Account

1. Sign up at bunny.net
2. Add payment method (pay-as-you-go)

### Step 2: Create Pull Zone

1. Go to CDN → Pull Zones
2. Click "Add Pull Zone"
3. Configure:
   - Name: yoursite-cdn
   - Origin URL: https://yourdomain.com
   - Enable all regions (or select specific)

### Step 3: Configure Pull Zone Settings

**General:**
```
Type: Standard Tier
Origin: https://yourdomain.com
Use Stale: On (serve stale on origin error)
```

**Caching:**
```
Override cache control: Yes
Browser cache: 365 days
Edge cache: 365 days
Strip cookies: Enable
Disable Cookies: On (for static assets)
```

**Security:**
```
Block root path: Off
Linked hostname: yourdomain.b-cdn.net
```

### Step 4: WordPress Integration

**Option A: Bunny.net Plugin (Recommended)**

```bash
wp plugin install bunnycdn --activate
```

Configure in Settings → Bunny.net:
- Enter API key
- Select Pull Zone
- Enable CDN Rewriter

**Option B: CDN Enabler Plugin**

```bash
wp plugin install cdn-enabler --activate
```

Configure:
- CDN URL: https://yourzone.b-cdn.net
- Included directories: wp-content,wp-includes

**Option C: WP Rocket Integration**

In WP Rocket → CDN:
- Enable CDN
- CDN URL: https://yourzone.b-cdn.net

### Step 5: Custom CDN Hostname (Optional)

Use your own subdomain (cdn.yourdomain.com):

1. In Bunny → Pull Zone → Hostnames
2. Add: cdn.yourdomain.com
3. Add CNAME record in DNS:
   ```
   cdn.yourdomain.com → yourzone.b-cdn.net
   ```
4. Enable SSL in Bunny
5. Update WordPress CDN URL to cdn.yourdomain.com

## Part 4: Bunny Optimizer

Bunny Optimizer provides automatic image optimization.

### Enable Optimizer

1. In Pull Zone settings → Optimizer
2. Enable "Bunny Optimizer"
3. Configure:
   ```
   WebP Conversion: On
   AVIF Conversion: On (for modern browsers)
   Quality: 85
   Sharpening: Light
   ```

### Optimizer Parameters

Control optimization via URL:
```
# Original
https://cdn.site.com/image.jpg

# Resize to width
https://cdn.site.com/image.jpg?width=800

# WebP with quality
https://cdn.site.com/image.jpg?quality=80

# Combined
https://cdn.site.com/image.jpg?width=800&quality=80&sharpen=true
```

## Part 5: Using Cloudflare + Bunny Together

### Recommended Architecture

```
User Request
    ↓
Cloudflare (DNS + Security + APO for HTML)
    ↓
Your Origin Server (WordPress)
    ↓
Static Assets (CSS, JS, Images)
    ↓
Bunny CDN (High cache hit ratio)
```

### Configuration

1. Keep Cloudflare as primary DNS
2. Create Bunny Pull Zone with origin = your domain (through Cloudflare)
3. Rewrite only static asset URLs to Bunny:
   - /wp-content/uploads/* → Bunny
   - /wp-content/themes/*/assets/* → Bunny
   - /wp-includes/* → Bunny (optional)

### WordPress Code

```php
function hybrid_cdn_urls($url) {
    // Static assets go to Bunny
    $bunny_url = 'https://yourzone.b-cdn.net';
    $static_paths = array('/wp-content/uploads/', '/wp-content/themes/', '/wp-includes/');

    foreach ($static_paths as $path) {
        if (strpos($url, $path) !== false) {
            return str_replace(home_url(), $bunny_url, $url);
        }
    }

    // Everything else stays on origin (Cloudflare handles HTML)
    return $url;
}
add_filter('wp_get_attachment_url', 'hybrid_cdn_urls');
add_filter('script_loader_src', 'hybrid_cdn_urls');
add_filter('style_loader_src', 'hybrid_cdn_urls');
```

## Part 6: Other CDN Options

### KeyCDN

```bash
wp plugin install cdn-enabler --activate
```

- Good performance
- Pay-as-you-go pricing
- WordPress plugin available

### StackPath (MaxCDN)

- Edge computing capabilities
- DDoS protection
- Real-time analytics

### AWS CloudFront

```bash
# Complex setup, best for AWS-hosted sites
# Use W3 Total Cache for integration
wp plugin install w3-total-cache --activate
```

## Part 7: Cache Invalidation

### Cloudflare

```bash
# Purge everything
curl -X POST "https://api.cloudflare.com/client/v4/zones/ZONE_ID/purge_cache" \
     -H "Authorization: Bearer API_TOKEN" \
     -H "Content-Type: application/json" \
     --data '{"purge_everything":true}'

# Purge specific URLs
curl -X POST "https://api.cloudflare.com/client/v4/zones/ZONE_ID/purge_cache" \
     -H "Authorization: Bearer API_TOKEN" \
     -H "Content-Type: application/json" \
     --data '{"files":["https://yourdomain.com/page-to-purge/"]}'
```

### Bunny CDN

```bash
# Via API
curl -X POST "https://api.bunny.net/pullzone/ZONE_ID/purgeCache" \
     -H "AccessKey: API_KEY"

# Via plugin - automatic on post update
```

### Automatic Purge on Updates

Most plugins handle this automatically. Manual trigger:

```php
// Purge CDN on post save
add_action('save_post', function($post_id) {
    // Cloudflare
    do_action('cloudflare_purge_by_url', get_permalink($post_id));

    // Or Bunny
    // Bunny plugin handles automatically
});
```

## Part 8: Verification and Testing

### Check CDN Headers

```bash
# Cloudflare
curl -I https://yourdomain.com
# cf-cache-status: HIT
# cf-ray: xxxxx

# Bunny CDN
curl -I https://cdn.yourdomain.com/image.jpg
# cdn-cache: HIT
# x-bunny-cache-status: HIT
```

### Cache Hit Ratio

**Cloudflare:** Dashboard → Analytics → Cache

Target: 50-60% for full site (higher with APO)

**Bunny CDN:** Dashboard → Statistics

Target: 70-90% for static assets

### Performance Testing

```bash
# Test from different locations
curl -w "DNS: %{time_namelookup}s\nConnect: %{time_connect}s\nTTFB: %{time_starttransfer}s\nTotal: %{time_total}s\n" \
     -o /dev/null -s https://yourdomain.com

# Use WebPageTest with multiple locations
# https://www.webpagetest.org
```

## Troubleshooting

### Mixed Content Errors

After enabling CDN:
```php
// Force HTTPS for CDN URLs
add_filter('wp_get_attachment_url', function($url) {
    return str_replace('http://', 'https://', $url);
});
```

### Images Not Loading from CDN

1. Check CDN URL is correct
2. Verify CORS headers:
   ```
   Access-Control-Allow-Origin: *
   ```
3. Check origin is accessible to CDN

### Stale Content

1. Purge CDN cache
2. Check cache headers from origin
3. Verify cache invalidation on updates

### SSL Certificate Issues

1. Ensure origin has valid SSL
2. Use "Full (strict)" in Cloudflare
3. For Bunny custom hostname, generate SSL in dashboard

## Cost Optimization

### Cloudflare Free Tier

- Sufficient for most sites
- Consider Pro ($20/mo) for:
  - APO included
  - Better support
  - Image optimization (Polish)

### Bunny CDN Pricing

- Pay per GB transferred
- Average cost: $0.01-0.03/GB
- Example: 100GB/month = ~$1-3

### Reducing CDN Costs

1. Set long cache TTLs
2. Optimize images before CDN
3. Use appropriate image sizes
4. Enable Brotli compression
