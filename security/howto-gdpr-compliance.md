---
title: How to Achieve GDPR Compliance for WordPress
description: Complete guide to implementing GDPR requirements including consent, data handling, and privacy
category: security
type: howto
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - security
  - gdpr
  - privacy
  - compliance
  - cookies
audience: intermediate
prerequisites:
  - WordPress admin access
  - Understanding of GDPR basics
sources:
  - https://loginpress.pro/wordpress-gdpr-plugins/
  - https://analytify.io/how-to-make-wordpress-website-gdpr-compliant/
  - https://www.wpbeginner.com/beginners-guide/the-ultimate-guide-to-wordpress-and-gdpr-compliance-everything-you-need-to-know/
  - https://www.siteground.com/kb/gdpr-compliant-wordpress/
  - https://www.10degrees.uk/2024/06/25/gdpr-compliance-and-wordpress/
---

# How to Achieve GDPR Compliance for WordPress

GDPR (General Data Protection Regulation) applies to any WordPress site collecting data from EU residents. This guide covers practical implementation of GDPR requirements.

## GDPR Overview

GDPR applies if you:
- Are based in the EU
- Offer goods/services to EU residents
- Monitor behavior of EU residents
- Collect any personal data from EU individuals

Non-compliance can result in fines up to 4% of annual revenue or 20 million EUR.

## Core GDPR Requirements

| Requirement | Description |
|-------------|-------------|
| Lawful Basis | Legal reason for data processing |
| Consent | Explicit, informed consent before collection |
| Right to Access | Users can request their data |
| Right to Erasure | Users can request data deletion |
| Data Portability | Users can export their data |
| Breach Notification | Report breaches within 72 hours |
| Data Protection | Implement security measures |
| Privacy by Design | Build privacy into systems |

## Step 1: Privacy Policy Page

WordPress has built-in privacy policy tools.

### Create Privacy Policy

1. Go to **Settings > Privacy**
2. Click **Create New Page** or select existing
3. Use WordPress privacy policy template as starting point
4. Customize for your site

### Privacy Policy Content

Include these sections:

```markdown
## Privacy Policy for [Your Site]

### Who We Are
[Company info and contact details]

### What Personal Data We Collect

**Comments**: Name, email, IP address, browser user agent
**Contact Forms**: [List fields collected]
**Account Registration**: [List data collected]
**Analytics**: [Describe tracking]
**Cookies**: [List cookies used]

### Why We Collect Data
[Explain purposes - e.g., respond to inquiries, improve services]

### How Long We Retain Data
- Comments: Until deleted
- Analytics: [X] months
- Account data: Until account deleted

### What Rights You Have
- Access your data
- Request correction
- Request deletion
- Object to processing
- Data portability

### How to Exercise Your Rights
Contact us at: [privacy email]

### Where We Send Your Data
[List third parties - analytics, email services, etc.]

### Contact Information
Data Protection Officer: [name]
Email: [email]

Last updated: [date]
```

## Step 2: Cookie Consent

EU law requires consent before setting non-essential cookies.

### Install Cookie Consent Plugin

Option 1: Complianz

```bash
wp plugin install complianz-gdpr --activate
```

Features:
- Cookie scanner
- Consent management
- GDPR, CCPA, PECR compliance
- Regional consent policies

Option 2: Cookie Notice

```bash
wp plugin install cookie-notice --activate
```

Features:
- Simple cookie banner
- Consent logging
- Cookie blocking

Option 3: WPConsent

```bash
wp plugin install wp-consent-api --activate
```

### Cookie Categories

Configure these cookie categories:

| Category | Examples | Consent Required |
|----------|----------|------------------|
| Necessary | Session, security | No |
| Functional | Preferences, language | Yes |
| Analytics | Google Analytics | Yes |
| Marketing | Ads, tracking | Yes |

### Block Scripts Until Consent

```php
<?php
// Example: Block Google Analytics until consent
add_action('wp_head', function() {
    if (!isset($_COOKIE['cookie_consent_analytics']) ||
        $_COOKIE['cookie_consent_analytics'] !== 'accepted') {
        return; // Don't load analytics
    }
    ?>
    <!-- Google Analytics code here -->
    <?php
});
```

### Consent Logging

Store consent records:

```php
<?php
// Log consent (simplified example)
function log_gdpr_consent($user_id, $consent_type) {
    global $wpdb;

    $wpdb->insert(
        $wpdb->prefix . 'gdpr_consent_log',
        [
            'user_id' => $user_id,
            'consent_type' => $consent_type,
            'ip_address' => wp_privacy_anonymize_ip($_SERVER['REMOTE_ADDR']),
            'user_agent' => sanitize_text_field($_SERVER['HTTP_USER_AGENT']),
            'timestamp' => current_time('mysql')
        ]
    );
}
```

## Step 3: Comment Consent

WordPress has built-in comment consent checkbox.

### Enable Comment Cookie Consent

1. Go to **Settings > Discussion**
2. Enable "Show comments cookies opt-in checkbox"

Or via code:

```php
// Ensure comment consent checkbox
add_action('comment_form_after_fields', function() {
    ?>
    <p class="comment-form-consent">
        <input id="wp-comment-cookies-consent" name="wp-comment-cookies-consent" type="checkbox" value="yes">
        <label for="wp-comment-cookies-consent">
            Save my name, email, and website in this browser for the next time I comment.
        </label>
    </p>
    <?php
});
```

## Step 4: Contact Form Consent

Add consent checkboxes to forms.

### Contact Form 7

```html
[acceptance gdpr-consent]
I consent to having this website store my submitted information
so they can respond to my inquiry.
[See our privacy policy](/privacy-policy/)
[/acceptance]

[submit "Submit"]
```

### WPForms

1. Add checkbox field
2. Set as required
3. Add privacy policy link in description

### Gravity Forms

```php
// Add GDPR consent field type
add_action('gform_add_field_buttons', function($field_groups) {
    $field_groups[0]['fields'][] = [
        'class' => 'button',
        'value' => __('GDPR Consent', 'gravityforms'),
        'data-type' => 'consent'
    ];
    return $field_groups;
});
```

## Step 5: Data Access and Export

WordPress has built-in data export tools.

### Export User Data

1. Go to **Tools > Export Personal Data**
2. Enter user's email
3. Click **Send Request**
4. User confirms via email
5. Download link generated

### Via WP-CLI

```bash
# Generate export
wp privacy export user@example.com

# Check pending requests
wp privacy list --status=pending
```

### Custom Data Export

Add custom plugin data to exports:

```php
<?php
// Register custom data exporter
add_filter('wp_privacy_personal_data_exporters', function($exporters) {
    $exporters['my-plugin'] = [
        'exporter_friendly_name' => 'My Plugin Data',
        'callback' => 'my_plugin_data_exporter',
    ];
    return $exporters;
});

function my_plugin_data_exporter($email_address, $page = 1) {
    $user = get_user_by('email', $email_address);

    if (!$user) {
        return ['data' => [], 'done' => true];
    }

    $data = [
        [
            'group_id' => 'my-plugin',
            'group_label' => 'My Plugin Data',
            'item_id' => 'custom-data-' . $user->ID,
            'data' => [
                ['name' => 'Setting 1', 'value' => get_user_meta($user->ID, '_my_setting_1', true)],
                ['name' => 'Setting 2', 'value' => get_user_meta($user->ID, '_my_setting_2', true)],
            ],
        ],
    ];

    return ['data' => $data, 'done' => true];
}
```

## Step 6: Data Erasure

Implement right to deletion.

### Delete User Data

1. Go to **Tools > Erase Personal Data**
2. Enter user's email
3. Click **Send Request**
4. User confirms via email
5. Data erased

### Via WP-CLI

```bash
# Request erasure
wp privacy erase user@example.com

# Process pending erasures
wp privacy process --type=erase
```

### Custom Data Erasure

```php
<?php
// Register custom data eraser
add_filter('wp_privacy_personal_data_erasers', function($erasers) {
    $erasers['my-plugin'] = [
        'eraser_friendly_name' => 'My Plugin Data',
        'callback' => 'my_plugin_data_eraser',
    ];
    return $erasers;
});

function my_plugin_data_eraser($email_address, $page = 1) {
    $user = get_user_by('email', $email_address);

    if (!$user) {
        return ['items_removed' => false, 'done' => true];
    }

    // Delete user's custom data
    delete_user_meta($user->ID, '_my_setting_1');
    delete_user_meta($user->ID, '_my_setting_2');

    return [
        'items_removed' => true,
        'items_retained' => false,
        'messages' => ['My Plugin data erased'],
        'done' => true
    ];
}
```

## Step 7: Google Analytics GDPR

GA4 has built-in privacy features.

### Configure GA4 for GDPR

1. In GA4, go to **Admin > Data Settings > Data Collection**
2. Enable **IP Anonymization** (default in GA4)
3. Set **Data Retention** to minimum needed
4. Disable **Google Signals** if not needed

### Load GA4 Only With Consent

```html
<!-- Only load GA4 after consent -->
<script>
document.addEventListener('cookie_consent_given', function(e) {
    if (e.detail.analytics) {
        // Load GA4
        var script = document.createElement('script');
        script.src = 'https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXX';
        document.head.appendChild(script);

        window.dataLayer = window.dataLayer || [];
        function gtag(){dataLayer.push(arguments);}
        gtag('js', new Date());
        gtag('config', 'G-XXXXXXXX', {
            'anonymize_ip': true
        });
    }
});
</script>
```

### Consent Mode v2

```html
<script>
// Default to denied
gtag('consent', 'default', {
    'analytics_storage': 'denied',
    'ad_storage': 'denied',
    'wait_for_update': 500
});

// Update after consent
function updateConsent(analytics, ads) {
    gtag('consent', 'update', {
        'analytics_storage': analytics ? 'granted' : 'denied',
        'ad_storage': ads ? 'granted' : 'denied'
    });
}
</script>
```

## Step 8: Data Security

GDPR requires appropriate security measures.

### Implement Security Measures

```php
<?php
// Add to wp-config.php

// Force HTTPS
define('FORCE_SSL_ADMIN', true);

// Disable file editing
define('DISALLOW_FILE_EDIT', true);

// Limit login attempts (via plugin)
```

### Secure Personal Data

```php
<?php
// Encrypt sensitive data
function encrypt_personal_data($data) {
    $key = defined('LOGGED_IN_KEY') ? LOGGED_IN_KEY : 'fallback-key';
    return openssl_encrypt($data, 'AES-256-CBC', $key, 0, substr($key, 0, 16));
}

function decrypt_personal_data($encrypted) {
    $key = defined('LOGGED_IN_KEY') ? LOGGED_IN_KEY : 'fallback-key';
    return openssl_decrypt($encrypted, 'AES-256-CBC', $key, 0, substr($key, 0, 16));
}
```

### Access Logging

```php
<?php
// Log access to personal data
function log_data_access($user_id, $data_type) {
    global $wpdb;

    $wpdb->insert(
        $wpdb->prefix . 'gdpr_access_log',
        [
            'accessed_user_id' => $user_id,
            'accessor_user_id' => get_current_user_id(),
            'data_type' => $data_type,
            'timestamp' => current_time('mysql'),
            'ip_address' => wp_privacy_anonymize_ip($_SERVER['REMOTE_ADDR'])
        ]
    );
}
```

## Step 9: Third-Party Integrations

Audit and document third-party services.

### Common Third Parties

| Service | Data Shared | DPA Required |
|---------|-------------|--------------|
| Google Analytics | IP, browsing | Yes |
| Mailchimp | Email, name | Yes |
| Stripe | Payment info | Yes (built-in) |
| Cloudflare | IP, browsing | Yes (built-in) |
| Facebook Pixel | Browsing | Yes |

### Data Processing Agreements

Ensure you have DPAs with:
- Hosting provider
- Email service provider
- Analytics providers
- Payment processors
- CDN providers

### Document Data Flows

```markdown
## Data Flow Documentation

### Contact Form Submissions
1. User submits form
2. Data stored in WordPress database
3. Email sent via [provider]
4. Stored for [X] days
5. Deleted automatically after [X] days

### Newsletter Signups
1. User enters email
2. Sent to Mailchimp via API
3. Stored until unsubscribed
4. Synced with WordPress user (if exists)
```

## Step 10: UK GDPR Compliance

Post-Brexit UK considerations.

### UK-Specific Requirements

After Brexit, UK has separate but similar GDPR:
- UK GDPR + Data Protection Act 2018 for UK residents
- EU GDPR for EU citizens

### Implementation

```php
<?php
// Detect user location for appropriate policy
function get_applicable_privacy_policy() {
    $country = get_user_country(); // Implement geo-detection

    if (in_array($country, ['GB', 'UK'])) {
        return 'UK GDPR applies';
    } elseif (is_eu_country($country)) {
        return 'EU GDPR applies';
    }

    return 'General privacy policy applies';
}
```

## Compliance Checklist

### Essential Items

- [ ] Privacy policy published and linked
- [ ] Cookie consent banner implemented
- [ ] Consent logging in place
- [ ] Data export capability working
- [ ] Data deletion capability working
- [ ] Contact forms have consent checkbox
- [ ] Comment consent enabled
- [ ] Third-party DPAs in place
- [ ] Security measures implemented
- [ ] Staff trained on data handling

### Recommended Items

- [ ] Privacy policy reviewed by legal
- [ ] Data processing records maintained
- [ ] Breach response plan documented
- [ ] Regular privacy audits scheduled
- [ ] DPO appointed (if required)

## Recommended Plugins

| Plugin | Features | Price |
|--------|----------|-------|
| Complianz | Full compliance suite | Free/Premium |
| WP GDPR Compliance | Data requests, consent | Free |
| Cookie Notice | Simple consent banner | Free |
| WP Activity Log | Access logging | Free/Premium |
| GDPR Cookie Compliance | Cookie management | Free |

## Troubleshooting

### Cookie Banner Not Showing

1. Check plugin is activated
2. Clear caching plugins
3. Check for JavaScript errors
4. Verify cookie settings

### Data Export Empty

1. Check user email exists
2. Verify export process completed
3. Check error logs for issues
4. Test with admin account

### Consent Not Logging

1. Check database table exists
2. Verify JavaScript loading
3. Test in incognito mode
4. Check for plugin conflicts

## Related Documents

- ref-security-checklist.md - Security audit checklist
- howto-security-hardening.md - Security measures
- concept-owasp-wordpress.md - Security concepts
