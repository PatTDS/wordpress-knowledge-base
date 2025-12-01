---
title: How to Harden WordPress Security
description: Step-by-step guide to hardening WordPress security with server and application configurations
category: security
type: howto
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - security
  - hardening
  - server
  - configuration
audience: intermediate
prerequisites:
  - SSH access to server
  - WordPress admin access
  - WP-CLI installed
sources:
  - https://wp-umbrella.com/blog/wordpress-security-best-practices/
  - https://www.wpbeginner.com/wordpress-security/
  - https://wpsecurityninja.com/wordpress-security-best-practices/
  - https://wp-rocket.me/blog/wordpress-security-best-practices/
  - https://reliqus.com/wordpress-hardening-how-to-secure-and-protect-website/
---

# How to Harden WordPress Security

Step-by-step guide to implement WordPress security hardening measures. In 2024-2025, WordPress sites are attacked every 39 seconds, with 7,966 new vulnerabilities identified in 2024 alone (34% increase from 2023).

## Prerequisites

- SSH or SFTP access to web server
- WordPress admin account
- WP-CLI installed
- Text editor for configuration files

## Step 1: Secure wp-config.php

The wp-config.php file contains database credentials and security keys.

### Move wp-config.php (Optional)

WordPress can read wp-config.php from one directory above the web root:

```bash
# Move config one level up
mv /var/www/html/wp-config.php /var/www/wp-config.php

# Verify WordPress still works
curl -I https://yoursite.com
```

### Add Security Constants

Edit wp-config.php:

```php
<?php
// Disable file editing in admin
define('DISALLOW_FILE_EDIT', true);

// Disable plugin/theme installation from admin (optional - use for production)
define('DISALLOW_FILE_MODS', true);

// Force SSL for admin
define('FORCE_SSL_ADMIN', true);

// Limit post revisions to prevent database bloat
define('WP_POST_REVISIONS', 5);

// Disable debugging in production
define('WP_DEBUG', false);
define('WP_DEBUG_LOG', false);
define('WP_DEBUG_DISPLAY', false);
define('SCRIPT_DEBUG', false);

// Block external HTTP requests (optional - may break some plugins)
// define('WP_HTTP_BLOCK_EXTERNAL', true);
// define('WP_ACCESSIBLE_HOSTS', 'api.wordpress.org,*.github.com');
```

### Regenerate Security Salts

```bash
# Generate new salts via WP-CLI
wp config shuffle-salts

# Or manually get from WordPress:
# https://api.wordpress.org/secret-key/1.1/salt/
```

## Step 2: Secure File Permissions

Set correct file and directory permissions:

```bash
#!/bin/bash
# fix-permissions.sh

WP_ROOT="/var/www/html"

# Set ownership (adjust user:group for your server)
chown -R www-data:www-data $WP_ROOT

# Set directory permissions
find $WP_ROOT -type d -exec chmod 755 {} \;

# Set file permissions
find $WP_ROOT -type f -exec chmod 644 {} \;

# Secure sensitive files
chmod 600 $WP_ROOT/wp-config.php

# Uploads needs write permission
chmod -R 775 $WP_ROOT/wp-content/uploads
```

### Disable PHP in Uploads

Create `/wp-content/uploads/.htaccess`:

```apache
# Disable PHP execution in uploads
<Files *.php>
deny from all
</Files>

# Allow specific file types only
<FilesMatch "\.(jpg|jpeg|png|gif|webp|svg|pdf|doc|docx|xls|xlsx|zip)$">
Order Allow,Deny
Allow from all
</FilesMatch>
```

For Nginx, add to server block:

```nginx
location ~* /wp-content/uploads/.*\.php$ {
    deny all;
}
```

## Step 3: Secure .htaccess (Apache)

Add security rules to root `.htaccess`:

```apache
# Block access to sensitive files
<FilesMatch "\.(env|log|sql|ini|conf|bak|backup|old|orig|save|swp|htaccess|htpasswd)$">
    Order allow,deny
    Deny from all
</FilesMatch>

# Block access to hidden files
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteRule (^\.|/\.) - [F]
</IfModule>

# Protect wp-config.php
<Files wp-config.php>
    Order allow,deny
    Deny from all
</Files>

# Disable XML-RPC
<Files xmlrpc.php>
    Order allow,deny
    Deny from all
</Files>

# Protect wp-includes
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteBase /
    RewriteRule ^wp-admin/includes/ - [F,L]
    RewriteRule !^wp-includes/ - [S=3]
    RewriteRule ^wp-includes/[^/]+\.php$ - [F,L]
    RewriteRule ^wp-includes/js/tinymce/langs/.+\.php - [F,L]
    RewriteRule ^wp-includes/theme-compat/ - [F,L]
</IfModule>

# Disable directory browsing
Options -Indexes

# Block user enumeration
<IfModule mod_rewrite.c>
    RewriteCond %{QUERY_STRING} author=\d
    RewriteRule ^ /? [L,R=301]
</IfModule>
```

## Step 4: Disable XML-RPC

XML-RPC is commonly exploited for brute force attacks.

### Via Plugin

```bash
wp plugin install disable-xml-rpc --activate
```

### Via Functions.php

```php
// Disable XML-RPC completely
add_filter('xmlrpc_enabled', '__return_false');

// Remove XML-RPC headers
remove_action('wp_head', 'rsd_link');
remove_action('wp_head', 'wlwmanifest_link');
```

### Via .htaccess

```apache
<Files xmlrpc.php>
    Order deny,allow
    Deny from all
</Files>
```

## Step 5: Remove Unused Plugins and Themes

```bash
# List all plugins
wp plugin list

# Delete inactive plugins
wp plugin delete $(wp plugin list --status=inactive --field=name)

# List themes
wp theme list

# Keep only active theme and one default
wp theme delete twentytwentythree twentytwentytwo
# Keep twentytwentyfour as fallback
```

## Step 6: Hide WordPress Version

Edit functions.php or create mu-plugin:

```php
<?php
// Remove WordPress version from head
remove_action('wp_head', 'wp_generator');

// Remove version from RSS
add_filter('the_generator', '__return_empty_string');

// Remove version from scripts and styles
function remove_version_strings($src) {
    if (strpos($src, 'ver=')) {
        $src = remove_query_arg('ver', $src);
    }
    return $src;
}
add_filter('style_loader_src', 'remove_version_strings', 9999);
add_filter('script_loader_src', 'remove_version_strings', 9999);
```

## Step 7: Install Security Plugin

Choose one security plugin:

### Option A: Wordfence

```bash
wp plugin install wordfence --activate

# Configure via WP-CLI
wp option update wordfence_hide_version 1
wp option update wordfence_disable_xmlrpc 1
wp option update wordfence_block_author_scan 1
```

### Option B: Shield Security

```bash
wp plugin install wp-simple-firewall --activate
```

### Option C: Sucuri

```bash
wp plugin install sucuri-scanner --activate
```

## Step 8: Configure Login Protection

### Limit Login Attempts

```bash
wp plugin install limit-login-attempts-reloaded --activate
```

### Change Login URL (Optional)

```bash
wp plugin install wps-hide-login --activate
wp option update whl_page 'my-secret-login'
```

### Add Login Captcha

```bash
wp plugin install login-recaptcha --activate
```

## Step 9: Set Up Automatic Updates

```php
// In wp-config.php or functions.php

// Enable auto-updates for core
define('WP_AUTO_UPDATE_CORE', true);

// Auto-update plugins
add_filter('auto_update_plugin', '__return_true');

// Auto-update themes
add_filter('auto_update_theme', '__return_true');
```

Or use WP-CLI to enable:

```bash
# Enable auto-updates for all plugins
wp plugin auto-updates enable --all

# Enable auto-updates for all themes
wp theme auto-updates enable --all
```

## Step 10: Database Security

### Change Table Prefix

For new installations only:

```bash
# In wp-config.php before installation
$table_prefix = 'wp_s3cur3_';
```

### Secure Database User

```sql
-- Create restricted user (MySQL)
CREATE USER 'wordpress_user'@'localhost' IDENTIFIED BY 'strong_password';
GRANT SELECT, INSERT, UPDATE, DELETE ON wordpress_db.* TO 'wordpress_user'@'localhost';
-- Only add CREATE, ALTER, DROP if needed for updates
FLUSH PRIVILEGES;
```

## Verification

After hardening, verify security:

```bash
# Verify file integrity
wp core verify-checksums
wp plugin verify-checksums --all

# Check security status
wp doctor check

# Test site loads correctly
curl -I https://yoursite.com

# Check security headers
curl -I https://yoursite.com | grep -E "(X-|Content-Security|Strict-Transport)"
```

## Troubleshooting

### Site Broken After Hardening

1. Check PHP error logs: `tail -f /var/log/php-errors.log`
2. Enable WP_DEBUG temporarily
3. Disable recently added rules one by one
4. Restore .htaccess from backup

### Plugin Conflicts

1. Deactivate all security plugins
2. Enable one at a time
3. Check for conflicts with existing plugins

### Login Issues

1. Access via SFTP to edit files
2. Rename security plugin folder temporarily
3. Reset login URL if using hide login plugin

## Related Documents

- ref-security-checklist.md - Security audit checklist
- howto-authentication-2fa.md - Two-factor authentication setup
- howto-ssl-https.md - SSL/HTTPS configuration
- howto-security-headers.md - HTTP security headers
