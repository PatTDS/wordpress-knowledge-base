---
title: How to Set Up WordPress Caching
description: Step-by-step guide to implementing page caching, object caching with Redis/Memcached, and Varnish reverse proxy for WordPress
category: performance
type: howto
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - caching
  - redis
  - varnish
  - object-cache
  - performance
prerequisites:
  - SSH access to server (for Redis/Varnish)
  - wp-cli installed
  - Basic understanding of caching concepts
audience: intermediate
sources:
  - https://wp-rocket.me/wordpress-cache/redis-object-caching/
  - https://wordpress.org/plugins/redis-cache/
  - https://objectcache.pro/
  - https://geekworkers.ch/en/blog/wordpress-server-cache-optimization/
  - https://wetopi.com/redis-object-cache-for-wordpress/
related:
  - concept-core-web-vitals.md
  - howto-cdn-setup.md
---

# How to Set Up WordPress Caching

## Understanding Cache Layers

WordPress caching works in multiple layers. Optimal performance requires configuring each layer appropriately.

### Cache Layer Hierarchy

```
Browser Cache (closest to user)
    ↓
CDN Cache (edge servers)
    ↓
Page Cache (full HTML)
    ↓
Object Cache (database queries)
    ↓
OPcache (PHP bytecode)
    ↓
Database (origin)
```

### Cache Types Explained

| Cache Type | What It Stores | Speed Impact |
|------------|---------------|--------------|
| Browser | Static assets | Eliminates requests |
| Page/Full | Complete HTML | Skips PHP entirely |
| Object | Database query results | Reduces DB queries |
| OPcache | Compiled PHP | Faster PHP execution |

## Part 1: Page Caching

Page caching stores complete HTML output, skipping PHP and database for repeat visits.

### Option A: WP Super Cache (Free)

```bash
# Install via WP-CLI
wp plugin install wp-super-cache --activate

# Enable caching
wp option update wpsupercache_enabled 1
```

**Configuration (wp-admin → Settings → WP Super Cache):**

1. Enable Caching: ON
2. Caching Mode: Expert (mod_rewrite)
3. Miscellaneous:
   - Cache Restrictions: Disable for logged-in users
   - Enable 304 browser caching
   - Don't cache pages with GET parameters

### Option B: W3 Total Cache (Free, More Options)

```bash
wp plugin install w3-total-cache --activate
```

**Recommended Settings:**

```php
// Page Cache
'pgcache.enabled' => true,
'pgcache.engine' => 'file_generic',  // or 'redis' if available

// Browser Cache
'browsercache.enabled' => true,
'browsercache.cssjs.lifetime' => 31536000,  // 1 year
'browsercache.html.lifetime' => 3600,
'browsercache.other.lifetime' => 31536000,
```

### Option C: WP Rocket (Premium, Recommended)

WP Rocket combines page cache, browser cache, and optimization features.

```bash
# After purchase, upload and activate
wp plugin activate wp-rocket
```

**Key Settings to Enable:**
- Cache → Enable caching for mobile devices
- Cache → Separate cache files for mobile
- File Optimization → Minify CSS/JS
- Media → LazyLoad images
- Preload → Activate Preloading

## Part 2: Object Caching with Redis

Object caching stores database query results in memory, dramatically reducing database load.

### Prerequisites

```bash
# Check if Redis is available
redis-cli ping
# Should return: PONG

# If not installed (Ubuntu/Debian)
sudo apt update
sudo apt install redis-server
sudo systemctl enable redis-server
sudo systemctl start redis-server
```

### Install Redis Object Cache Plugin

```bash
# Install the plugin
wp plugin install redis-cache --activate

# Enable Redis object cache
wp redis enable
```

### Configure wp-config.php

Add before `/* That's all, stop editing! */`:

```php
// Redis configuration
define('WP_REDIS_HOST', '127.0.0.1');
define('WP_REDIS_PORT', 6379);
define('WP_REDIS_DATABASE', 0);
define('WP_REDIS_TIMEOUT', 1);
define('WP_REDIS_READ_TIMEOUT', 1);

// Optional: Add prefix for multi-site
define('WP_REDIS_PREFIX', 'mysite_');

// Enable key compression for memory efficiency
define('WP_REDIS_IGBINARY', true);
```

### Verify Redis Connection

```bash
# Check plugin status
wp redis status

# Expected output:
# Status: Connected
# Client: PhpRedis (5.x.x)
# Ping: 1.23ms
# Errors: None

# Check Redis memory usage
redis-cli info memory | grep used_memory_human
```

### Object Cache Pro (Premium Alternative)

For high-traffic sites, Object Cache Pro offers better performance:

```bash
# After purchase
composer require rhubarbgroup/object-cache-pro

# Configure in wp-config.php
define('WP_REDIS_CONFIG', [
    'host' => '127.0.0.1',
    'port' => 6379,
    'database' => 0,
    'timeout' => 1.0,
    'read_timeout' => 1.0,
    'async_flush' => true,
    'compression' => 'zstd',
    'serializer' => 'igbinary',
    'prefetch' => true,
]);
```

## Part 3: Memcached Alternative

If Redis isn't available, Memcached is a solid alternative.

### Install Memcached

```bash
# Install Memcached server
sudo apt install memcached php-memcached

# Restart PHP
sudo systemctl restart php8.0-fpm  # adjust for your version
```

### WordPress Configuration

```bash
# Install plugin
wp plugin install memcached --activate
```

Add to wp-config.php:

```php
$memcached_servers = array(
    'default' => array(
        '127.0.0.1:11211'
    )
);
```

## Part 4: Varnish Reverse Proxy

Varnish sits in front of WordPress, serving cached pages without touching PHP.

### Install Varnish

```bash
# Ubuntu/Debian
sudo apt install varnish

# Configure Varnish to listen on port 80
sudo nano /etc/varnish/default.vcl
```

### Basic Varnish Configuration

```vcl
vcl 4.0;

backend default {
    .host = "127.0.0.1";
    .port = "8080";  # WordPress runs here
}

sub vcl_recv {
    # Remove cookies for static files
    if (req.url ~ "\.(css|js|jpg|jpeg|png|gif|ico|webp|woff2?)$") {
        unset req.http.Cookie;
        return (hash);
    }

    # Don't cache admin or login
    if (req.url ~ "^/wp-(admin|login)") {
        return (pass);
    }

    # Don't cache logged in users
    if (req.http.Cookie ~ "wordpress_logged_in") {
        return (pass);
    }

    # Don't cache POST requests
    if (req.method == "POST") {
        return (pass);
    }
}

sub vcl_backend_response {
    # Cache for 1 hour by default
    set beresp.ttl = 1h;

    # Cache static files longer
    if (bereq.url ~ "\.(css|js|jpg|jpeg|png|gif|ico|webp|woff2?)$") {
        set beresp.ttl = 7d;
    }
}

sub vcl_deliver {
    # Add debug header
    if (obj.hits > 0) {
        set resp.http.X-Cache = "HIT";
    } else {
        set resp.http.X-Cache = "MISS";
    }
}
```

### Varnish + WordPress Integration

```bash
# Install Varnish purge plugin
wp plugin install varnish-http-purge --activate
```

Configure purge settings in wp-admin → Settings → Varnish HTTP Purge.

## Part 5: OPcache Configuration

OPcache caches compiled PHP bytecode.

### Recommended Settings

Edit php.ini or create `/etc/php/8.0/mods-available/opcache-custom.ini`:

```ini
[opcache]
opcache.enable=1
opcache.enable_cli=0
opcache.memory_consumption=256
opcache.interned_strings_buffer=16
opcache.max_accelerated_files=20000
opcache.max_wasted_percentage=10
opcache.validate_timestamps=1
opcache.revalidate_freq=60
opcache.fast_shutdown=1
opcache.save_comments=1
```

### Verify OPcache

```php
<?php
// Create opcache-status.php in web root
var_export(opcache_get_status());
```

## Part 6: Combined Stack Setup

### Recommended Stack (High Performance)

```
User Request
    ↓
Cloudflare (CDN + WAF)
    ↓
Varnish (Full page cache)
    ↓
Nginx (Web server)
    ↓
PHP-FPM + OPcache
    ↓
WordPress + Redis (Object cache)
    ↓
MySQL
```

### Recommended Stack (Simpler)

```
User Request
    ↓
Cloudflare (CDN)
    ↓
Nginx/Apache
    ↓
PHP-FPM + OPcache
    ↓
WordPress + WP Rocket (Page cache)
    ↓
Redis (Object cache)
    ↓
MySQL
```

## Cache Warming

Pre-populate caches after clearing:

```bash
# Basic sitemap-based warming
curl -s https://example.com/sitemap.xml | \
  grep -oP '<loc>\K[^<]+' | \
  xargs -I {} curl -s -o /dev/null {}

# With WP-CLI and WP Rocket
wp rocket preload

# With W3TC
wp w3-total-cache pgcache_prime
```

## Cache Invalidation

### Clear All Caches

```bash
# WordPress cache
wp cache flush

# WP Rocket
wp rocket clean --confirm

# W3 Total Cache
wp w3-total-cache flush all

# Redis
redis-cli FLUSHDB

# Varnish
varnishadm "ban req.url ~ /"

# OPcache (via PHP)
php -r "opcache_reset();"
```

### Selective Cache Clearing

```bash
# Clear single post cache (WP Rocket)
wp rocket clean --post-id=123

# Clear object cache key
wp cache delete my_key my_group
```

## Troubleshooting

### Cache Not Working

1. **Check headers:**
   ```bash
   curl -I https://example.com | grep -i cache
   ```

2. **Verify bypass conditions:**
   - Not logged in
   - No query strings
   - Not POST request

3. **Check for conflicting plugins**

### Redis Connection Issues

```bash
# Test connection
redis-cli ping

# Check WordPress connection
wp redis status

# View Redis logs
sudo tail -f /var/log/redis/redis-server.log
```

### High Memory Usage

```bash
# Redis memory analysis
redis-cli --bigkeys

# Clear specific prefix
redis-cli KEYS "mysite_*" | xargs redis-cli DEL
```

## Performance Verification

### Expected Cache Hit Ratio

| Cache Layer | Target Hit Ratio |
|-------------|-----------------|
| Browser | N/A (no requests) |
| CDN | 70-90% |
| Varnish | 80-95% |
| Object Cache | 90-99% |

### Measure Impact

```bash
# Before/after TTFB
curl -o /dev/null -s -w "TTFB: %{time_starttransfer}s\n" https://example.com

# Query Monitor plugin shows DB queries
# Target: < 50 queries per page load with object cache
```
