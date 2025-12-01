---
title: Understanding OWASP Top 10 for WordPress
description: Explanation of OWASP Top 10 vulnerabilities and how they apply to WordPress development
category: security
type: concept
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - security
  - owasp
  - vulnerabilities
  - concepts
audience: intermediate
sources:
  - https://owasp.org/www-project-top-ten/
  - https://melapress.com/owasp-wordpress-security-top-10/
  - https://blogvault.net/owasp-top-10/
  - https://gridpane.com/kb/the-owasp-top-10-and-gridpane-wordpress-security-options/
  - https://learn.wordpress.org/tutorial/extending-wordpress-common-security-vulnerabilities/
---

# Understanding OWASP Top 10 for WordPress

The OWASP (Open Worldwide Application Security Project) Top 10 represents the most critical security risks for web applications. This document explains each vulnerability type and how it specifically applies to WordPress.

## What is OWASP?

OWASP is a nonprofit foundation that works to improve software security. The OWASP Top 10 is updated periodically based on data from hundreds of organizations worldwide.

**Current Version**: 2021 (next update expected 2025)

## The OWASP Top 10 (2021)

| Rank | Category | WordPress Relevance |
|------|----------|---------------------|
| A01 | Broken Access Control | High |
| A02 | Cryptographic Failures | Medium |
| A03 | Injection | High |
| A04 | Insecure Design | Medium |
| A05 | Security Misconfiguration | High |
| A06 | Vulnerable and Outdated Components | Critical |
| A07 | Identification and Authentication Failures | High |
| A08 | Software and Data Integrity Failures | Medium |
| A09 | Security Logging and Monitoring Failures | Medium |
| A10 | Server-Side Request Forgery (SSRF) | Low |

## A01: Broken Access Control

**Definition**: Failures that allow users to act outside their intended permissions.

### WordPress Context

Access control is managed through WordPress roles and capabilities. Broken access control occurs when:
- Plugins don't check user capabilities before performing actions
- Direct object references allow access to other users' data
- Admin functions accessible to non-admin users

### WordPress Examples

**Vulnerable Code:**
```php
// BAD: No capability check
function delete_user_data() {
    $user_id = $_GET['user_id'];
    delete_user_meta($user_id, 'sensitive_data');
}
add_action('wp_ajax_delete_data', 'delete_user_data');
```

**Secure Code:**
```php
// GOOD: Proper capability and ownership check
function delete_user_data() {
    // Check nonce
    if (!wp_verify_nonce($_GET['_wpnonce'], 'delete_data')) {
        wp_die('Security check failed');
    }

    // Check capability
    if (!current_user_can('edit_users')) {
        wp_die('Unauthorized access');
    }

    $user_id = intval($_GET['user_id']);

    // Additional ownership check
    if (get_current_user_id() !== $user_id && !current_user_can('manage_options')) {
        wp_die('You can only delete your own data');
    }

    delete_user_meta($user_id, 'sensitive_data');
}
add_action('wp_ajax_delete_data', 'delete_user_data');
```

### WordPress Protections

- Use `current_user_can()` before sensitive operations
- Always verify nonces for form submissions
- Use capability-based checks, not role-based
- Implement ownership verification for user data

## A02: Cryptographic Failures

**Definition**: Failures related to cryptography that expose sensitive data.

### WordPress Context

Cryptographic failures in WordPress include:
- Transmitting data over HTTP instead of HTTPS
- Weak password hashing
- Exposing sensitive data in URLs or logs
- Improper storage of API keys/secrets

### WordPress Examples

**Vulnerable:**
```php
// BAD: Storing API key in plain text option
update_option('my_api_key', $api_key);

// BAD: Logging sensitive data
error_log('User password: ' . $password);
```

**Secure:**
```php
// GOOD: Encrypt sensitive data
function encrypt_api_key($key) {
    $salt = defined('LOGGED_IN_KEY') ? LOGGED_IN_KEY : 'fallback';
    return openssl_encrypt($key, 'AES-256-CBC', $salt, 0, substr($salt, 0, 16));
}

update_option('my_api_key', encrypt_api_key($api_key));

// GOOD: Never log passwords
error_log('User login attempt for: ' . $username);
```

### WordPress Protections

- Always use HTTPS (`FORCE_SSL_ADMIN`)
- Store secrets in wp-config.php, not database
- Use WordPress password hashing functions
- Never log sensitive information

## A03: Injection

**Definition**: Untrusted data sent to an interpreter as part of a command or query.

### WordPress Context

Common injection types in WordPress:
- **SQL Injection**: Malicious SQL in database queries
- **XSS (Cross-Site Scripting)**: Malicious scripts in output
- **Command Injection**: OS commands in server-side code

### WordPress Examples

**SQL Injection - Vulnerable:**
```php
// BAD: Direct variable in query
global $wpdb;
$results = $wpdb->get_results(
    "SELECT * FROM {$wpdb->posts} WHERE post_title = '{$_GET['title']}'"
);
```

**SQL Injection - Secure:**
```php
// GOOD: Use prepared statements
global $wpdb;
$results = $wpdb->get_results(
    $wpdb->prepare(
        "SELECT * FROM {$wpdb->posts} WHERE post_title = %s",
        sanitize_text_field($_GET['title'])
    )
);
```

**XSS - Vulnerable:**
```php
// BAD: Unescaped output
echo '<h1>' . $_GET['name'] . '</h1>';
```

**XSS - Secure:**
```php
// GOOD: Escape output
echo '<h1>' . esc_html($_GET['name']) . '</h1>';

// GOOD: For URLs
echo '<a href="' . esc_url($url) . '">Link</a>';

// GOOD: For attributes
echo '<div data-value="' . esc_attr($value) . '">';
```

### WordPress Protections

- **Input**: Use `sanitize_*()` functions
- **Output**: Use `esc_*()` functions
- **Database**: Use `$wpdb->prepare()`
- **Files**: Validate file types and extensions

## A04: Insecure Design

**Definition**: Missing or ineffective security controls in the design phase.

### WordPress Context

Insecure design in WordPress plugins/themes includes:
- Trusting client-side validation alone
- Missing rate limiting on sensitive operations
- No separation between user and admin functionality

### Design Principles for WordPress

```php
// GOOD: Defense in depth
function process_form() {
    // 1. Verify nonce (CSRF protection)
    if (!wp_verify_nonce($_POST['nonce'], 'form_action')) {
        return;
    }

    // 2. Check capability (authorization)
    if (!current_user_can('edit_posts')) {
        return;
    }

    // 3. Validate input (data validation)
    $email = sanitize_email($_POST['email']);
    if (!is_email($email)) {
        return;
    }

    // 4. Rate limiting (abuse prevention)
    if (get_transient('form_limit_' . get_current_user_id())) {
        return;
    }
    set_transient('form_limit_' . get_current_user_id(), true, 60);

    // 5. Process with minimal privileges
    process_data($email);
}
```

## A05: Security Misconfiguration

**Definition**: Missing or incorrect security settings across the application stack.

### WordPress Context

Security misconfigurations are extremely common:
- Default "admin" username
- Debug mode enabled in production
- Directory listing enabled
- Exposed version numbers
- Unnecessary features enabled (XML-RPC)

### WordPress Hardening

```php
// wp-config.php settings
define('WP_DEBUG', false);
define('WP_DEBUG_LOG', false);
define('WP_DEBUG_DISPLAY', false);
define('DISALLOW_FILE_EDIT', true);
define('DISALLOW_FILE_MODS', true);
```

```apache
# .htaccess hardening
Options -Indexes
<Files wp-config.php>
    Order allow,deny
    Deny from all
</Files>
```

### Security Configuration Checklist

- Disable file editing
- Remove default admin account
- Hide WordPress version
- Disable XML-RPC
- Secure directory permissions
- Enable security headers

## A06: Vulnerable and Outdated Components

**Definition**: Using components with known vulnerabilities.

### WordPress Context

This is the **most critical risk for WordPress**. In 2024:
- Plugin vulnerabilities = 68% of all WordPress security issues
- 92% of vulnerabilities come from outdated plugins
- 50% of admins don't enable auto-updates

### WordPress Mitigation

```bash
# Check for updates
wp core check-update
wp plugin list --update=available
wp theme list --update=available

# Update everything
wp core update
wp plugin update --all
wp theme update --all

# Verify integrity
wp core verify-checksums
wp plugin verify-checksums --all
```

### Best Practices

- Enable auto-updates for plugins and themes
- Remove unused plugins (delete, don't just deactivate)
- Use plugins from reputable sources
- Monitor security advisories (WPScan, Wordfence)
- Maintain inventory of all components

## A07: Identification and Authentication Failures

**Definition**: Weaknesses in authentication mechanisms.

### WordPress Context

Authentication failures include:
- Weak passwords allowed
- No brute force protection
- Missing 2FA
- Session management issues

### WordPress Protections

```php
// Enforce strong passwords
add_filter('wp_is_password_compromised', '__return_true', 10, 2);

// Custom password policy
add_filter('registration_errors', function($errors, $login, $email) {
    $password = $_POST['user_pass'];

    if (strlen($password) < 12) {
        $errors->add('password_length', 'Password must be at least 12 characters');
    }

    if (!preg_match('/[A-Z]/', $password)) {
        $errors->add('password_uppercase', 'Password must contain uppercase letter');
    }

    return $errors;
}, 10, 3);
```

### Authentication Hardening

- Implement 2FA for all admin users
- Use login attempt limiting
- Disable user enumeration
- Implement proper session timeouts
- Consider passwordless authentication

## A08: Software and Data Integrity Failures

**Definition**: Failures to verify software updates and data integrity.

### WordPress Context

Integrity failures in WordPress:
- Installing plugins from untrusted sources
- Not verifying plugin/theme code
- Insecure update mechanisms
- Missing file integrity monitoring

### WordPress Protections

```bash
# Verify core file integrity
wp core verify-checksums

# Verify plugin integrity
wp plugin verify-checksums --all

# Verify theme integrity
wp theme verify-checksums --all
```

```php
// Only install from trusted sources
// In wp-config.php
define('DISALLOW_FILE_MODS', true);  // Prevent any installations via admin
```

### Integrity Monitoring

- Use security plugins for file integrity monitoring
- Implement hash verification for custom code
- Monitor for unauthorized file changes
- Use signed commits in version control

## A09: Security Logging and Monitoring Failures

**Definition**: Insufficient logging and monitoring to detect attacks.

### WordPress Context

Many WordPress sites have no activity logging:
- Failed login attempts not logged
- File changes not monitored
- No alerts for suspicious activity

### WordPress Logging

```php
// Log failed login attempts
add_action('wp_login_failed', function($username) {
    error_log(sprintf(
        '[LOGIN FAILED] Username: %s, IP: %s, Time: %s',
        $username,
        $_SERVER['REMOTE_ADDR'],
        current_time('mysql')
    ));
});

// Log successful logins
add_action('wp_login', function($username, $user) {
    error_log(sprintf(
        '[LOGIN SUCCESS] Username: %s, IP: %s, Time: %s',
        $username,
        $_SERVER['REMOTE_ADDR'],
        current_time('mysql')
    ));
}, 10, 2);
```

### Monitoring Setup

- Install WP Activity Log plugin
- Configure email alerts for suspicious activity
- Set up external log aggregation
- Monitor for:
  - Multiple failed logins
  - User privilege changes
  - Plugin installations
  - File modifications

## A10: Server-Side Request Forgery (SSRF)

**Definition**: Fetching remote resources without validating user-supplied URLs.

### WordPress Context

SSRF can occur in WordPress when:
- Plugins fetch URLs from user input
- Importing content from external sources
- Processing webhooks or callbacks

### WordPress Examples

**Vulnerable:**
```php
// BAD: Fetching any URL user provides
$response = wp_remote_get($_POST['url']);
```

**Secure:**
```php
// GOOD: Validate and restrict URLs
function safe_fetch($url) {
    // Parse URL
    $parsed = wp_parse_url($url);

    // Block internal IPs
    $host = gethostbyname($parsed['host']);
    if (filter_var($host, FILTER_VALIDATE_IP, FILTER_FLAG_NO_PRIV_RANGE | FILTER_FLAG_NO_RES_RANGE) === false) {
        return false;
    }

    // Allow only specific domains
    $allowed_domains = ['api.example.com', 'cdn.example.com'];
    if (!in_array($parsed['host'], $allowed_domains)) {
        return false;
    }

    // Only allow HTTPS
    if ($parsed['scheme'] !== 'https') {
        return false;
    }

    return wp_remote_get($url);
}
```

## Summary: WordPress Security Priorities

Based on OWASP Top 10, prioritize these areas for WordPress:

1. **Keep Everything Updated** (A06)
2. **Implement Proper Access Controls** (A01)
3. **Sanitize Input, Escape Output** (A03)
4. **Secure Configuration** (A05)
5. **Strong Authentication** (A07)
6. **Enable Logging** (A09)

## Development Checklist

For every WordPress plugin/theme:

- [ ] Verify nonces on all forms
- [ ] Check capabilities before actions
- [ ] Use $wpdb->prepare() for queries
- [ ] Escape all output with esc_*()
- [ ] Sanitize all input with sanitize_*()
- [ ] Log security-relevant events
- [ ] Document security considerations

## Related Documents

- ref-security-checklist.md - Security audit checklist
- ref-vulnerabilities-2024.md - Current vulnerability landscape
- howto-security-hardening.md - Hardening procedures
