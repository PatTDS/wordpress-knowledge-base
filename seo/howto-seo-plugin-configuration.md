---
title: How to Configure WordPress SEO Plugins
description: Step-by-step guide to setting up and configuring Rank Math, Yoast SEO, and AIOSEO
category: seo
type: howto
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - seo-plugins
  - rank-math
  - yoast-seo
  - aioseo
  - configuration
audience: beginner
sources:
  - https://zapier.com/blog/rank-math-vs-yoast/
  - https://aioseo.com/rank-math-vs-yoast/
  - https://onlinemediamasters.com/rank-math-vs-yoast/
  - https://kinsta.com/blog/rank-math-vs-yoast/
  - https://rankmath.com/kb/why-rank-math-is-better-than-yoast/
---

# How to Configure WordPress SEO Plugins

## Overview

This guide covers the installation and configuration of the three major WordPress SEO plugins: Rank Math, Yoast SEO, and All in One SEO (AIOSEO). Each section provides optimal settings for 2024-2025 SEO best practices.

## Plugin Comparison Quick Reference

| Feature | Rank Math (Free) | Yoast SEO (Free) | AIOSEO (Free) |
|---------|------------------|------------------|---------------|
| Focus keywords per post | 5 | 1 | Unlimited |
| Schema types included | 20+ | 6 | 10+ |
| Internal linking suggestions | Yes | No (Premium) | No |
| Redirections | Yes | No (Premium) | No |
| 404 monitoring | Yes | No | Yes |
| Google Search Console integration | Yes | No | Yes |
| Performance overhead | Low (~0.2-0.5s) | Medium | Low |
| Active installs | 3+ million | 10+ million | 3+ million |

## Rank Math Configuration

### Step 1: Installation

```bash
# Via WP-CLI
wp plugin install seo-by-rank-math --activate

# Or manually: Plugins > Add New > Search "Rank Math"
```

### Step 2: Setup Wizard

1. Navigate to Rank Math > Setup Wizard
2. Select **Advanced** mode for full control
3. Connect your Rank Math account (free)
4. Import settings from existing SEO plugin (if migrating)

### Step 3: General Settings

**Dashboard > Modules to Enable:**
- 404 Monitor
- Analytics (if using Search Console)
- Local SEO (for local businesses)
- Redirections
- Schema (Rich Snippets)
- SEO Analysis
- Sitemap

### Step 4: Titles & Meta Configuration

Navigate to Rank Math > Titles & Meta:

**Homepage:**
```
Title: %sitename% %sep% %tagline%
Description: Your 150-160 character meta description
```

**Posts:**
```
Title: %title% %sep% %sitename%
Description: %excerpt%
```

**Pages:**
```
Title: %title% %sep% %sitename%
Description: %excerpt%
```

### Step 5: Sitemap Settings

Navigate to Rank Math > Sitemap Settings:

| Setting | Recommended Value |
|---------|-------------------|
| Include Images | Yes |
| Include Featured Images | Yes |
| Ping Search Engines | Yes |
| Posts per sitemap | 200 |
| Include Posts | Yes |
| Include Pages | Yes |
| Include Categories | Yes (if content-rich) |
| Include Tags | No |
| Include Author Archives | No |
| Include Date Archives | No |

### Step 6: Schema Markup

Navigate to Rank Math > Titles & Meta > Post Type:

**Default Schema Types:**
- Blog Posts: Article
- Pages: WebPage
- Products (WooCommerce): Product
- Services: Service

### Step 7: Optimal Rank Math Settings via WP-CLI

```bash
# Enable essential modules
wp option update rank_math_modules '["404-monitor","analytics","local-seo","redirections","rich-snippet","seo-analysis","sitemap"]' --format=json

# Set default schema type
wp option update rank_math_default_schema_type 'Article'

# Enable breadcrumbs
wp option update rank_math_breadcrumbs_enable 'on'

# Configure title separator
wp option update rank_math_title_separator '-'
```

## Yoast SEO Configuration

### Step 1: Installation

```bash
# Via WP-CLI
wp plugin install wordpress-seo --activate

# Or manually: Plugins > Add New > Search "Yoast SEO"
```

### Step 2: Configuration Wizard

1. Navigate to Yoast SEO > First-time configuration
2. Complete the 12-step setup wizard
3. Configure site representation (organization or person)
4. Set social profiles

### Step 3: Search Appearance Settings

Navigate to Yoast SEO > Search Appearance:

**Content Types (Posts):**
- Show Posts in search results: Yes
- SEO title: `%%title%% %%sep%% %%sitename%%`
- Meta description: Leave empty for per-post control
- Schema: Article

**Content Types (Pages):**
- Show Pages in search results: Yes
- SEO title: `%%title%% %%sep%% %%sitename%%`
- Schema: WebPage

**Taxonomies:**
- Show Categories in search results: Yes (if content-rich)
- Show Tags in search results: No (usually)

**Archives:**
- Author archives: Disable if single author
- Date archives: Disable

### Step 4: XML Sitemaps

Navigate to Yoast SEO > General > Features:

- XML Sitemaps: Enabled
- Sitemap URL: `yourdomain.com/sitemap_index.xml`

**Exclude from sitemap (via Search Appearance):**
- Tags (usually)
- Author archives (if single author)
- Date archives
- Format archives

### Step 5: Social Settings

Navigate to Yoast SEO > Social:

**Facebook/Open Graph:**
- Enable Open Graph meta data
- Default image for sharing

**Twitter:**
- Enable Twitter card meta data
- Default card type: Summary with large image

### Step 6: Integration

Navigate to Yoast SEO > General > Integrations:

- Semrush integration: Optional
- Wincher integration: Optional
- Ryte integration: Optional
- Elementor integration: Enable if using Elementor

### Step 7: Premium Features (Yoast SEO Premium)

If using premium:
- Enable internal linking suggestions
- Configure redirects manager
- Enable multiple focus keywords
- Set up 24/7 indexability monitoring

## All in One SEO Configuration

### Step 1: Installation

```bash
# Via WP-CLI
wp plugin install all-in-one-seo-pack --activate

# Or manually: Plugins > Add New > Search "All in One SEO"
```

### Step 2: Setup Wizard

1. Navigate to All in One SEO > Dashboard
2. Click "Launch Setup Wizard"
3. Select your category (blog, online store, etc.)
4. Configure site representation
5. Enable SEO features

### Step 3: Search Appearance

Navigate to All in One SEO > Search Appearance:

**Global Settings:**
- Title Separator: `-`
- Knowledge Graph: Organization or Person

**Content Types:**
- Posts: Show in search, default title `#post_title #separator_sa #site_title`
- Pages: Show in search
- Custom Post Types: Configure per type

### Step 4: Sitemap Configuration

Navigate to All in One SEO > Sitemaps:

**General Sitemap:**
- Enable Sitemap: Yes
- Enable Sitemap Indexes: Yes
- Links Per Sitemap: 1000

**Sitemap Priority:**
- Posts: 0.8
- Pages: 0.6
- Taxonomies: 0.4

### Step 5: Social Networks

Navigate to All in One SEO > Social Networks:

- Facebook: Enable Open Graph
- Twitter: Enable Twitter Cards
- Pinterest: Verify if needed
- LinkedIn: Add company URL

### Step 6: Local SEO (Business Sites)

Navigate to All in One SEO > Local SEO:

- Business Name
- Address (NAP consistency crucial)
- Phone Number
- Business Hours
- Payment Methods Accepted

## Post-Installation Verification

### Verification Checklist

After configuring your chosen SEO plugin:

- [ ] Submit sitemap to Google Search Console
- [ ] Submit sitemap to Bing Webmaster Tools
- [ ] Verify Open Graph tags with Facebook Debug Tool
- [ ] Test Twitter Cards with Twitter Card Validator
- [ ] Check schema with Google Rich Results Test
- [ ] Run a Lighthouse audit
- [ ] Check robots.txt is correctly generated

### Testing Commands

```bash
# Check sitemap accessibility
curl -I https://yourdomain.com/sitemap_index.xml

# Verify robots.txt
curl https://yourdomain.com/robots.txt

# Check SEO meta tags (requires xmllint)
curl -s https://yourdomain.com | grep -E '<meta (name="description"|property="og:|name="twitter:)'
```

## Migration Between Plugins

### Migrating to Rank Math

1. Install Rank Math (don't deactivate current plugin yet)
2. Go to Rank Math > Status & Tools > Import & Export
3. Select your current plugin
4. Click "Import"
5. Verify settings transferred correctly
6. Deactivate and delete old plugin

### Migrating to Yoast SEO

1. Install Yoast SEO
2. Use SEO Data Transporter plugin
3. Or manually export/import settings
4. Verify redirects transferred

### Migrating to AIOSEO

1. Install AIOSEO
2. Go to AIOSEO > Tools > Import/Export
3. Import from supported plugins
4. Verify and test

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| Sitemap not generating | Check permalink settings, flush cache |
| Schema errors in Search Console | Validate with Rich Results Test, check conflicting plugins |
| Meta descriptions not showing | Clear cache, check for conflicting plugins |
| 404 redirects not working | Check .htaccess permissions, verify redirect plugin is enabled |
| Social images not displaying | Verify image dimensions (1200x630 for OG), clear social media cache |

### Plugin Conflicts

Avoid running multiple SEO plugins simultaneously. Common conflict symptoms:
- Duplicate meta tags
- Multiple sitemaps
- Conflicting schema markup
- Performance degradation

## Performance Comparison

Based on 2024-2025 testing:

| Plugin | Load Time Impact | Memory Usage |
|--------|-----------------|--------------|
| Rank Math | +0.2-0.5s | ~2MB |
| Yoast SEO | +0.5-1.0s | ~3MB |
| AIOSEO | +0.3-0.6s | ~2.5MB |

**Note:** Results vary based on enabled modules and server configuration.

## Recommendations by Use Case

| Use Case | Recommended Plugin |
|----------|-------------------|
| Beginners | Yoast SEO (simplest setup) |
| Multi-site owners | Rank Math (unlimited site license) |
| Local businesses | Rank Math or AIOSEO (better local SEO) |
| E-commerce | AIOSEO or Rank Math (WooCommerce integration) |
| Budget-conscious | Rank Math (most features for free) |
| Enterprise | Yoast SEO Premium or AIOSEO Pro |

## Related Documents

- @seo/ref-technical-seo.md - Technical SEO reference
- @seo/ref-schema-markup.md - Schema markup details
- @howtos/howto-plugin-configuration.md - General plugin setup
