---
title: How to Configure SSL/HTTPS for WordPress
description: Complete guide to SSL certificate installation, HTTPS migration, and TLS configuration
category: security
type: howto
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - security
  - ssl
  - https
  - tls
  - encryption
audience: intermediate
prerequisites:
  - Domain pointing to server
  - SSH access
  - WordPress admin access
sources:
  - https://developer.wordpress.org/advanced-administration/security/https/
  - https://melapress.com/ssl-tls-https-guide-wordpress-administrators/
  - https://www.wpbeginner.com/wp-tutorials/how-to-add-ssl-and-https-in-wordpress/
  - https://getshieldsecurity.com/blog/wordpress-https/
  - https://wpengine.com/support/ssl/
---

# How to Configure SSL/HTTPS for WordPress

This guide covers obtaining SSL certificates, configuring HTTPS, and ensuring secure connections for WordPress sites. WordPress fully supports HTTPS when a TLS/SSL certificate is properly configured.

## Prerequisites

- Domain name pointing to your server
- SSH access to server (for Let's Encrypt)
- WordPress admin access
- Access to server configuration (Apache/Nginx)

## Step 1: Obtain SSL Certificate

### Option A: Let's Encrypt (Free)

Let's Encrypt provides free SSL certificates backed by AWS, Cisco, Mozilla, and others.

#### Using Certbot (Recommended)

```bash
# Install Certbot (Ubuntu/Debian)
sudo apt update
sudo apt install certbot

# For Apache
sudo apt install python3-certbot-apache
sudo certbot --apache -d example.com -d www.example.com

# For Nginx
sudo apt install python3-certbot-nginx
sudo certbot --nginx -d example.com -d www.example.com
```

#### Verify Installation

```bash
# Check certificate
sudo certbot certificates

# Test renewal
sudo certbot renew --dry-run
```

#### Auto-Renewal (Cron)

Certbot installs auto-renewal automatically. Verify:

```bash
# Check systemd timer
sudo systemctl status certbot.timer

# Or check cron
sudo cat /etc/cron.d/certbot
```

### Option B: Commercial SSL

For extended validation (EV) or organization validation (OV):

1. Generate CSR:
```bash
openssl req -new -newkey rsa:2048 -nodes \
  -keyout domain.key \
  -out domain.csr
```

2. Submit CSR to certificate authority
3. Install certificate per CA instructions

### Option C: Hosting Provider

Many hosts provide free SSL:
- Check hosting control panel
- Enable "Force HTTPS" option
- Certificates auto-renew

## Step 2: Configure WordPress

### Update WordPress URLs

Method 1 - WP-CLI (Recommended):

```bash
# Update site URLs
wp option update home 'https://example.com'
wp option update siteurl 'https://example.com'

# Search and replace old URLs
wp search-replace 'http://example.com' 'https://example.com' --skip-columns=guid
```

Method 2 - wp-config.php:

```php
define('WP_HOME', 'https://example.com');
define('WP_SITEURL', 'https://example.com');
```

Method 3 - Admin Panel:

1. Go to **Settings > General**
2. Update **WordPress Address (URL)** to https://
3. Update **Site Address (URL)** to https://

### Force SSL for Admin

Add to wp-config.php:

```php
define('FORCE_SSL_ADMIN', true);
```

### Handle Reverse Proxy/Load Balancer

If WordPress is behind a reverse proxy:

```php
// In wp-config.php
if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) &&
    $_SERVER['HTTP_X_FORWARDED_PROTO'] === 'https') {
    $_SERVER['HTTPS'] = 'on';
}
```

For Cloudflare specifically:

```php
if (isset($_SERVER['HTTP_CF_VISITOR'])) {
    $visitor = json_decode($_SERVER['HTTP_CF_VISITOR']);
    if ($visitor->scheme === 'https') {
        $_SERVER['HTTPS'] = 'on';
    }
}
```

## Step 3: Configure HTTP to HTTPS Redirect

### Apache (.htaccess)

```apache
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
</IfModule>
```

### Nginx

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # ... rest of configuration
}
```

## Step 4: Enable Modern TLS

### Recommended TLS Configuration

Support TLS 1.2 and TLS 1.3 only:

**Apache:**

```apache
SSLProtocol -all +TLSv1.2 +TLSv1.3
SSLCipherSuite ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384
SSLHonorCipherOrder off
SSLSessionTickets off
```

**Nginx:**

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
ssl_prefer_server_ciphers off;
ssl_session_timeout 1d;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off;
```

### Enable HSTS

Strict Transport Security tells browsers to always use HTTPS:

**Apache:**

```apache
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
```

**Nginx:**

```nginx
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
```

**Important HSTS Notes:**
- Start with short max-age (300) for testing
- Increase to 31536000 (1 year) after confirming it works
- `includeSubDomains` affects all subdomains
- `preload` allows browser preloading (permanent - use carefully)

## Step 5: Fix Mixed Content

Mixed content occurs when HTTPS pages load HTTP resources.

### Find Mixed Content

```bash
# Search database for http URLs
wp search-replace 'http://example.com' 'https://example.com' --dry-run

# Check specific tables
wp db query "SELECT * FROM wp_posts WHERE post_content LIKE '%http://example.com%'"
```

### Fix in Database

```bash
# Replace all instances
wp search-replace 'http://example.com' 'https://example.com' --skip-columns=guid
```

### Fix in Theme/Plugins

Search for hardcoded HTTP URLs:

```bash
grep -r "http://" wp-content/themes/your-theme/
grep -r "http://" wp-content/plugins/your-plugin/
```

### Use Plugin for Automatic Fixes

```bash
wp plugin install really-simple-ssl --activate
```

Really Simple SSL:
- Detects SSL certificate
- Configures WordPress settings
- Fixes mixed content dynamically
- Enables HTTPS redirect

## Step 6: Enable HTTP/2

HTTP/2 improves performance over HTTPS.

### Apache

```apache
# Enable HTTP/2 module
sudo a2enmod http2

# In virtual host
Protocols h2 http/1.1
```

### Nginx

```nginx
listen 443 ssl http2;
```

### Verify HTTP/2

```bash
curl -I --http2 https://example.com
# Look for: HTTP/2 200
```

## Step 7: Configure SFTP

Use SFTP instead of FTP for file transfers:

```php
// In wp-config.php
define('FTP_SSL', true);
define('FS_METHOD', 'direct');
```

Connect with SFTP client (not FTP):
- Host: sftp://example.com
- Port: 22
- Protocol: SFTP (SSH File Transfer)

## Verification

### Check SSL Configuration

```bash
# Test SSL Labs grade
curl https://www.ssllabs.com/ssltest/analyze.html?d=example.com

# Test certificate
openssl s_client -connect example.com:443 -servername example.com

# Check expiration
echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null | openssl x509 -noout -dates
```

### WordPress Verification

```bash
# Verify WordPress URLs
wp option get home
wp option get siteurl

# Should both return https://example.com

# Check for mixed content
wp db query "SELECT COUNT(*) FROM wp_posts WHERE post_content LIKE '%http://example.com%'"
```

### Browser Verification

1. Visit site in browser
2. Check for padlock icon
3. Click padlock for certificate details
4. Open DevTools Console - check for mixed content warnings

## SSL Certificate Monitoring

### Expiration Alerts

Create monitoring script:

```bash
#!/bin/bash
# check-ssl-expiry.sh

DOMAIN="example.com"
DAYS_WARN=30

EXPIRY=$(echo | openssl s_client -connect $DOMAIN:443 -servername $DOMAIN 2>/dev/null | openssl x509 -noout -enddate | cut -d= -f2)
EXPIRY_EPOCH=$(date -d "$EXPIRY" +%s)
NOW_EPOCH=$(date +%s)
DAYS_LEFT=$(( ($EXPIRY_EPOCH - $NOW_EPOCH) / 86400 ))

if [ $DAYS_LEFT -lt $DAYS_WARN ]; then
    echo "SSL certificate for $DOMAIN expires in $DAYS_LEFT days!"
    # Send alert email or notification
fi
```

Add to cron for daily check:

```bash
0 9 * * * /path/to/check-ssl-expiry.sh
```

## Troubleshooting

### Redirect Loop

If you get infinite redirects:

1. Check for conflicting redirects in .htaccess
2. Verify reverse proxy settings
3. Check plugin conflicts (disable caching plugins)
4. Add to wp-config.php for proxy setups:

```php
if ($_SERVER['HTTP_X_FORWARDED_PROTO'] === 'https') {
    $_SERVER['HTTPS'] = 'on';
}
```

### Certificate Errors

| Error | Solution |
|-------|----------|
| ERR_CERT_DATE_INVALID | Check server time; renew certificate |
| ERR_CERT_COMMON_NAME_INVALID | Certificate doesn't match domain |
| ERR_CERT_AUTHORITY_INVALID | Missing intermediate certificate |
| ERR_CERT_REVOKED | Certificate revoked; obtain new one |

### Mixed Content After Migration

1. Clear all caches (browser, CDN, WordPress)
2. Regenerate sitemaps
3. Update CDN URLs to HTTPS
4. Check hardcoded URLs in widgets/customizer

### Let's Encrypt Rate Limits

If hitting rate limits:

```bash
# Use staging for testing
sudo certbot --staging -d example.com

# Check rate limit status
# https://crt.sh/?q=example.com
```

## Related Documents

- ref-security-checklist.md - Security audit checklist
- howto-security-headers.md - HTTP security headers
- howto-security-hardening.md - General hardening
