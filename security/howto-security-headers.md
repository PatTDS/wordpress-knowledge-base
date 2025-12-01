---
title: How to Implement HTTP Security Headers in WordPress
description: Complete guide to configuring security headers including CSP, X-Frame-Options, and more
category: security
type: howto
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - security
  - headers
  - csp
  - xss
  - clickjacking
audience: intermediate
prerequisites:
  - WordPress admin access
  - Server configuration access
sources:
  - https://patchstack.com/articles/wordpress-security-headers/
  - https://patchstack.com/articles/configure-the-x-frame-options-header-in-wordpress/
  - https://getshieldsecurity.com/blog/wordpress-content-security-policy/
  - https://www.wpbeginner.com/beginners-guide/how-to-add-http-security-headers-in-wordpress/
  - https://melapress.com/wordpress-content-security-policy/
  - https://servebolt.com/articles/how-to-add-http-security-headers-in-wordpress/
---

# How to Implement HTTP Security Headers in WordPress

HTTP security headers protect against common web attacks like clickjacking, XSS, and content injection. This guide covers all essential headers with WordPress-specific considerations.

## Essential Security Headers

| Header | Purpose | Priority |
|--------|---------|----------|
| Strict-Transport-Security | Force HTTPS | Critical |
| X-Content-Type-Options | Prevent MIME sniffing | Critical |
| X-Frame-Options | Prevent clickjacking | High |
| Content-Security-Policy | Control resource loading | High |
| Referrer-Policy | Control referer information | Medium |
| Permissions-Policy | Control browser features | Medium |
| X-XSS-Protection | Legacy XSS filter | Low (deprecated) |

## Method 1: .htaccess (Apache)

Add to your root `.htaccess` file:

```apache
<IfModule mod_headers.c>
    # Strict Transport Security (HSTS)
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" env=HTTPS

    # Prevent MIME type sniffing
    Header always set X-Content-Type-Options "nosniff"

    # Clickjacking protection
    Header always set X-Frame-Options "SAMEORIGIN"

    # XSS Protection (legacy browsers)
    Header always set X-XSS-Protection "1; mode=block"

    # Referrer Policy
    Header always set Referrer-Policy "strict-origin-when-cross-origin"

    # Permissions Policy (formerly Feature Policy)
    Header always set Permissions-Policy "accelerometer=(), camera=(), geolocation=(), gyroscope=(), magnetometer=(), microphone=(), payment=(), usb=()"

    # Content Security Policy (basic)
    Header always set Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' data:; connect-src 'self'; frame-ancestors 'self'"
</IfModule>
```

## Method 2: Nginx Configuration

Add to your server block:

```nginx
# Security Headers
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "accelerometer=(), camera=(), geolocation=(), gyroscope=(), magnetometer=(), microphone=(), payment=(), usb=()" always;

# Content Security Policy
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' data:; connect-src 'self'; frame-ancestors 'self'" always;
```

## Method 3: WordPress Plugin

### Using Shield Security

```bash
wp plugin install wp-simple-firewall --activate
```

Navigate to **Shield Security > Config > HTTP Headers** to enable headers.

### Using HTTP Headers Plugin

```bash
wp plugin install http-headers --activate
```

Configure each header in **Settings > HTTP Headers**.

### Using Security Headers Plugin

```bash
wp plugin install developer-api-for-security-headers --activate
```

## Method 4: PHP (functions.php or mu-plugin)

Create `wp-content/mu-plugins/security-headers.php`:

```php
<?php
/**
 * Security Headers
 * Add HTTP security headers via WordPress
 */

add_action('send_headers', function() {
    // Don't set headers in admin or for logged-in users' admin bar
    if (is_admin()) {
        return;
    }

    // HSTS - Force HTTPS
    header('Strict-Transport-Security: max-age=31536000; includeSubDomains; preload');

    // Prevent MIME sniffing
    header('X-Content-Type-Options: nosniff');

    // Clickjacking protection
    header('X-Frame-Options: SAMEORIGIN');

    // XSS Protection (legacy)
    header('X-XSS-Protection: 1; mode=block');

    // Referrer Policy
    header('Referrer-Policy: strict-origin-when-cross-origin');

    // Permissions Policy
    header('Permissions-Policy: accelerometer=(), camera=(), geolocation=(), gyroscope=(), magnetometer=(), microphone=(), payment=(), usb=()');
});
```

## Content Security Policy (CSP) for WordPress

CSP is complex for WordPress due to inline scripts in admin and plugins.

### Understanding CSP Directives

| Directive | Controls |
|-----------|----------|
| default-src | Fallback for all resources |
| script-src | JavaScript sources |
| style-src | CSS sources |
| img-src | Image sources |
| font-src | Font file sources |
| connect-src | AJAX, WebSocket, fetch |
| frame-src | iframe sources |
| frame-ancestors | Who can embed this page |
| form-action | Form submission targets |
| base-uri | Base URL restrictions |

### WordPress-Compatible CSP

WordPress requires `unsafe-inline` and `unsafe-eval` for core functionality:

```apache
Content-Security-Policy:
    default-src 'self';
    script-src 'self' 'unsafe-inline' 'unsafe-eval';
    style-src 'self' 'unsafe-inline';
    img-src 'self' data: https:;
    font-src 'self' data:;
    connect-src 'self';
    frame-ancestors 'self';
    form-action 'self';
    base-uri 'self';
```

### CSP for Common Services

Add these to your CSP as needed:

```apache
# Google Analytics
script-src https://www.googletagmanager.com https://www.google-analytics.com;
img-src https://www.google-analytics.com;
connect-src https://www.google-analytics.com;

# Google Fonts
style-src https://fonts.googleapis.com;
font-src https://fonts.gstatic.com;

# YouTube Embeds
frame-src https://www.youtube.com https://www.youtube-nocookie.com;

# Vimeo Embeds
frame-src https://player.vimeo.com;

# Gravatar
img-src https://secure.gravatar.com;

# CDN (example: Cloudflare)
script-src https://cdnjs.cloudflare.com;
style-src https://cdnjs.cloudflare.com;
```

### Report-Only Mode

Test CSP without breaking site:

```apache
Content-Security-Policy-Report-Only: default-src 'self'; report-uri /csp-report-endpoint/
```

View violations in browser console or set up reporting endpoint.

### CSP Reporting

```php
// Add to mu-plugin for CSP violation logging
add_action('rest_api_init', function() {
    register_rest_route('security/v1', '/csp-report', [
        'methods' => 'POST',
        'callback' => function($request) {
            $report = $request->get_json_params();
            error_log('CSP Violation: ' . json_encode($report));
            return new WP_REST_Response(null, 204);
        },
        'permission_callback' => '__return_true'
    ]);
});
```

## X-Frame-Options vs frame-ancestors

X-Frame-Options is deprecated in favor of CSP frame-ancestors:

| X-Frame-Options | CSP Equivalent |
|-----------------|----------------|
| DENY | frame-ancestors 'none' |
| SAMEORIGIN | frame-ancestors 'self' |
| ALLOW-FROM uri | frame-ancestors uri |

**Recommendation**: Use both for maximum browser compatibility:

```apache
Header always set X-Frame-Options "SAMEORIGIN"
Header always set Content-Security-Policy "frame-ancestors 'self'"
```

## Permissions-Policy Configuration

Control browser features:

```apache
Permissions-Policy:
    accelerometer=(),
    camera=(),
    geolocation=(),
    gyroscope=(),
    magnetometer=(),
    microphone=(),
    payment=(),
    usb=(),
    interest-cohort=()
```

For sites that need specific features:

```apache
# Allow geolocation for maps
Permissions-Policy: geolocation=(self "https://maps.google.com")

# Allow camera for video chat
Permissions-Policy: camera=(self)
```

## CORS Headers

For sites serving APIs or fonts cross-origin:

```apache
<IfModule mod_headers.c>
    # Allow specific origins
    Header set Access-Control-Allow-Origin "https://example.com"

    # Allow credentials
    Header set Access-Control-Allow-Credentials "true"

    # Allow methods
    Header set Access-Control-Allow-Methods "GET, POST, OPTIONS"

    # Allow headers
    Header set Access-Control-Allow-Headers "Content-Type, Authorization"
</IfModule>
```

## Admin-Specific Headers

Different headers for admin vs frontend:

```php
<?php
add_action('send_headers', function() {
    if (is_admin()) {
        // Relaxed CSP for admin
        header("Content-Security-Policy: default-src 'self' 'unsafe-inline' 'unsafe-eval'; img-src 'self' data: https:;");
    } else {
        // Stricter CSP for frontend
        header("Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline';");
    }
});
```

## Verification

### Test Headers

```bash
# Check all headers
curl -I https://example.com

# Check specific headers
curl -I https://example.com | grep -E "(Strict-Transport|X-Frame|Content-Security|X-Content-Type)"
```

### Online Tools

- [SecurityHeaders.com](https://securityheaders.com/)
- [Mozilla Observatory](https://observatory.mozilla.org/)
- [CSP Evaluator](https://csp-evaluator.withgoogle.com/)

### Browser DevTools

1. Open Developer Tools (F12)
2. Go to Network tab
3. Reload page
4. Click on main document request
5. Check Response Headers section

## Troubleshooting

### Site Broken After Adding CSP

1. Enable CSP in Report-Only mode first
2. Check browser console for violations
3. Add required sources incrementally
4. Common issues:
   - Inline scripts need `'unsafe-inline'`
   - Google Fonts need specific domains
   - CDN resources need their domains

### WordPress Admin Not Working

WordPress admin requires inline scripts and eval:

```apache
# For admin area only
<If "%{REQUEST_URI} =~ m#^/wp-admin#">
    Header set Content-Security-Policy "script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline';"
</If>
```

### Embedded Content Not Loading

For iframes (YouTube, Vimeo, etc.):

```apache
Content-Security-Policy: frame-src https://www.youtube.com https://player.vimeo.com;
```

### Google Fonts Not Loading

```apache
Content-Security-Policy:
    style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
    font-src 'self' https://fonts.gstatic.com;
```

### Plugin Compatibility Issues

Some plugins inject inline scripts. Options:

1. Add plugin's CDN domains to CSP
2. Use nonce-based CSP (advanced)
3. Accept `'unsafe-inline'` for scripts (less secure)

## Starter Templates

### Basic Security (Most Compatible)

```apache
<IfModule mod_headers.c>
    Header always set X-Content-Type-Options "nosniff"
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
</IfModule>
```

### Standard Security

```apache
<IfModule mod_headers.c>
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains" env=HTTPS
    Header always set X-Content-Type-Options "nosniff"
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set X-XSS-Protection "1; mode=block"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    Header always set Content-Security-Policy "frame-ancestors 'self'"
</IfModule>
```

### Strict Security

```apache
<IfModule mod_headers.c>
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" env=HTTPS
    Header always set X-Content-Type-Options "nosniff"
    Header always set X-Frame-Options "DENY"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    Header always set Permissions-Policy "accelerometer=(), camera=(), geolocation=(), gyroscope=(), magnetometer=(), microphone=(), payment=(), usb=(), interest-cohort=()"
    Header always set Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' data:; connect-src 'self'; frame-ancestors 'none'; form-action 'self'; base-uri 'self'"
</IfModule>
```

## Related Documents

- ref-security-checklist.md - Security audit checklist
- howto-ssl-https.md - SSL/HTTPS configuration
- howto-security-hardening.md - General hardening
- concept-owasp-wordpress.md - OWASP vulnerabilities
