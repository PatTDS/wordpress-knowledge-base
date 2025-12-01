---
title: WP-CLI Command Reference
category: tools
type: reference
updated: 2025-12-01
version: 1.0.0
tags:
  - wp-cli
  - command-line
  - automation
  - wordpress-management
sources:
  - https://wp-cli.org/
  - https://developer.wordpress.org/cli/commands/
  - https://malcure.com/blog/utilities/wp-cli-cheatsheet/
  - https://wpclimastery.com/blog/wp-cli-config-files-best-practices-and-advanced-configuration-2025/
---

# WP-CLI Command Reference

WP-CLI is the official command-line interface for WordPress. Current stable version: **2.12.0**.

## Quick Reference

### Core Management

| Command | Description |
|---------|-------------|
| `wp core version` | Display WordPress version |
| `wp core update` | Update WordPress core |
| `wp core download` | Download WordPress files |
| `wp core install` | Install WordPress |
| `wp core verify-checksums` | Verify core file integrity |

### Plugin Management

```bash
# List all plugins
wp plugin list

# Install plugin
wp plugin install plugin-name --activate

# Update all plugins
wp plugin update --all

# Deactivate all plugins (troubleshooting)
wp plugin deactivate --all

# Verify plugin checksums
wp plugin verify-checksums --all

# Install with dry-run
wp plugin install plugin-name --dry-run
```

### Theme Management

```bash
# List themes
wp theme list

# Activate theme
wp theme activate theme-name

# Update all themes
wp theme update --all

# Install and activate
wp theme install theme-name --activate

# Switch to default (troubleshooting)
wp theme activate twentytwentyfour
```

### Database Operations

```bash
# Export database
wp db export backup.sql

# Import database
wp db import backup.sql

# Search and replace URLs
wp search-replace 'old-url.com' 'new-url.com' --skip-columns=guid

# Optimize tables
wp db optimize

# Check database
wp db check

# Repair database
wp db repair

# Run SQL query
wp db query "SELECT * FROM wp_options LIMIT 10"
```

### User Management

```bash
# List users
wp user list

# Create user
wp user create username email@example.com --role=editor

# Update password
wp user update admin --user_pass='NewSecurePassword'

# Delete user
wp user delete username --reassign=admin

# Generate users (testing)
wp user generate --count=10
```

### Content Management

```bash
# List posts
wp post list --post_type=post

# Create post
wp post create --post_title='Title' --post_status=publish

# Delete post
wp post delete 123 --force

# List pages
wp post list --post_type=page

# Generate posts (testing)
wp post generate --count=10
```

### Cache Management

```bash
# Flush all caches
wp cache flush

# Delete transients
wp transient delete --all

# Delete expired transients
wp transient delete --expired

# Clear rewrite rules
wp rewrite flush
```

### Media Operations

```bash
# Regenerate thumbnails
wp media regenerate

# Regenerate missing only
wp media regenerate --only-missing

# Import media
wp media import /path/to/image.jpg

# List image sizes
wp media image-size
```

### Cron Management

```bash
# List scheduled events
wp cron event list

# Run due events
wp cron event run --due-now

# Run specific event
wp cron event run wp_version_check

# Test cron spawning
wp cron test
```

### Maintenance Mode

```bash
# Enable maintenance mode
wp maintenance-mode activate

# Disable maintenance mode
wp maintenance-mode deactivate

# Check status
wp maintenance-mode status
```

## Configuration Best Practices (2025)

### Global Config File

Create `~/.wp-cli/config.yml`:

```yaml
# Global WP-CLI configuration
path: /var/www/html
url: https://example.com
user: admin

# Development defaults
core update:
  minor: true

plugin update:
  all: true

db export:
  add-drop-table: true
```

### Project Config File

Create `wp-cli.yml` in project root:

```yaml
# Project-specific WP-CLI configuration
path: wp
url: https://staging.example.com

# Environment-specific
@development:
  url: http://localhost:8080

@staging:
  url: https://staging.example.com
  ssh: deploy@staging.example.com

@production:
  url: https://example.com
  ssh: deploy@production.example.com
```

### Use Aliases

```bash
# Run on staging
wp @staging plugin list

# Run on production
wp @production cache flush

# Run on all environments
wp @all core version
```

## Essential Workflows

### Fresh Installation

```bash
# Download WordPress
wp core download

# Create config
wp config create --dbname=wordpress --dbuser=root --dbpass=password

# Install WordPress
wp core install --url=example.com --title="Site" --admin_user=admin --admin_password=password --admin_email=admin@example.com

# Install essential plugins
wp plugin install query-monitor autoptimize --activate

# Set permalinks
wp rewrite structure '/%postname%/'
wp rewrite flush
```

### Database Migration

```bash
# Export with search-replace for staging
wp db export - | wp search-replace 'https://production.com' 'https://staging.com' --skip-columns=guid - > staging.sql

# Import on target
wp @staging db import staging.sql
wp @staging cache flush
wp @staging rewrite flush
```

### Backup Workflow

```bash
# Full export
wp db export backup-$(date +%Y%m%d-%H%M%S).sql

# Export specific tables
wp db export --tables=wp_posts,wp_postmeta posts-backup.sql

# Compress
wp db export - | gzip > backup.sql.gz
```

### Troubleshooting

```bash
# Enable debug mode
wp config set WP_DEBUG true --raw
wp config set WP_DEBUG_LOG true --raw
wp config set WP_DEBUG_DISPLAY false --raw

# Check PHP version
wp eval "echo phpversion();"

# Memory usage
wp eval 'echo memory_get_usage(true) / 1024 / 1024 . " MB";'

# Verify file integrity
wp core verify-checksums
wp plugin verify-checksums --all
```

### Security Hardening

```bash
# Disable file editing
wp config set DISALLOW_FILE_EDIT true --raw

# Shuffle salts
wp config shuffle-salts

# Hide version
wp eval 'remove_action("wp_head", "wp_generator");'

# List admin users
wp user list --role=administrator
```

## Dry Run Mode

Always test critical commands:

```bash
# Test plugin installation
wp plugin install plugin-name --dry-run

# Test search-replace
wp search-replace 'old' 'new' --dry-run

# Preview what will be deleted
wp post list --post_type=revision --format=count
```

## Performance Optimization

```bash
# Delete revisions
wp post delete $(wp post list --post_type='revision' --format=ids) --force

# Clean orphaned metadata
wp db query "DELETE pm FROM wp_postmeta pm LEFT JOIN wp_posts p ON pm.post_id = p.ID WHERE p.ID IS NULL"

# Optimize autoloaded options
wp db query "SELECT SUM(LENGTH(option_value)) FROM wp_options WHERE autoload='yes'"

# Profile WordPress load
wp profile stage --all
```

## WP-CLI Packages

```bash
# List installed packages
wp package list

# Install package
wp package install wp-cli/profile-command

# Update packages
wp package update

# Useful packages
wp package install wp-cli/doctor-command
wp package install wp-cli/dist-archive-command
```

## Safety Guidelines

1. **Always backup first**: `wp db export` before major changes
2. **Use staging**: Test commands on staging before production
3. **Verify directory**: Run `pwd` before executing commands
4. **Use dry-run**: Test with `--dry-run` when available
5. **Enable maintenance mode**: For updates and migrations

## Related Documents

- @howto-deployment-automation.md
- @howto-development-environment.md
- @ref-imagemagick-processing.md
