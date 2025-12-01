---
title: How to Backup and Recover WordPress Sites
description: Complete guide to WordPress backup strategies, tools, and disaster recovery procedures
category: security
type: howto
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - security
  - backup
  - recovery
  - disaster-recovery
  - restoration
audience: intermediate
prerequisites:
  - WordPress admin access
  - SSH access (for manual backups)
  - Cloud storage account (recommended)
sources:
  - https://developer.wordpress.org/advanced-administration/security/backup/
  - https://docs.aws.amazon.com/whitepapers/latest/best-practices-wordpress/appendix-c-backup-and-recovery.html
  - https://blogvault.net/321-backup-wordpress/
  - https://thrivewp.com/wordpress-backup-and-recovery/
  - https://www.wpbeginner.com/beginners-guide/how-to-backup-your-wordpress-site/
  - https://belovdigital.agency/blog/wordpress-backup-and-restore-agency-best-practices/
---

# How to Backup and Recover WordPress Sites

Backups are your first defense against data loss, hacking, and failed updates. This guide covers the 3-2-1 backup strategy, automation, and recovery procedures.

## The 3-2-1 Backup Rule

The proven backup strategy:
- **3** copies of your data
- **2** different storage types
- **1** copy stored offsite

Example implementation:
1. Primary: Live website
2. Local: Server backup
3. Offsite: Cloud storage (AWS S3, Google Drive, Dropbox)

## What to Backup

WordPress sites consist of two parts:

| Component | Contains | Location |
|-----------|----------|----------|
| **Database** | Posts, pages, comments, settings, users | MySQL/MariaDB |
| **Files** | Themes, plugins, uploads, configuration | wp-content/, wp-config.php |

### Critical Files

```
/
├── wp-config.php          # Configuration (must backup)
├── .htaccess              # Permalinks, security rules
└── wp-content/
    ├── themes/            # Custom themes
    ├── plugins/           # Custom plugins
    ├── uploads/           # Media files
    └── mu-plugins/        # Must-use plugins
```

## Backup Frequency

| Site Type | Database | Full Files |
|-----------|----------|------------|
| Low traffic blog | Weekly | Weekly |
| Business site | Daily | Weekly |
| E-commerce | Hourly | Daily |
| High-traffic/WooCommerce | Real-time | Daily |

## Method 1: WP-CLI Backup

### Database Backup

```bash
# Export database
wp db export backup-$(date +%Y%m%d-%H%M%S).sql

# Export with gzip compression
wp db export - | gzip > backup-$(date +%Y%m%d-%H%M%S).sql.gz

# Export specific tables
wp db export --tables=wp_posts,wp_postmeta backup.sql
```

### File Backup

```bash
# Create compressed archive of wp-content
tar -czvf wp-content-$(date +%Y%m%d).tar.gz wp-content/

# Include wp-config.php and .htaccess
tar -czvf full-backup-$(date +%Y%m%d).tar.gz \
  wp-content/ \
  wp-config.php \
  .htaccess
```

### Complete Backup Script

```bash
#!/bin/bash
# backup-wordpress.sh

# Configuration
WP_PATH="/var/www/html"
BACKUP_PATH="/backups"
DATE=$(date +%Y%m%d-%H%M%S)
RETENTION_DAYS=30

# Create backup directory
mkdir -p $BACKUP_PATH/$DATE

# Export database
cd $WP_PATH
wp db export $BACKUP_PATH/$DATE/database.sql

# Compress database
gzip $BACKUP_PATH/$DATE/database.sql

# Backup files
tar -czf $BACKUP_PATH/$DATE/files.tar.gz \
  wp-content/ \
  wp-config.php \
  .htaccess \
  2>/dev/null

# Create checksum
cd $BACKUP_PATH/$DATE
sha256sum * > checksums.txt

# Cleanup old backups
find $BACKUP_PATH -type d -mtime +$RETENTION_DAYS -exec rm -rf {} \;

echo "Backup complete: $BACKUP_PATH/$DATE"
```

### Schedule with Cron

```bash
# Edit crontab
crontab -e

# Daily backup at 3 AM
0 3 * * * /path/to/backup-wordpress.sh >> /var/log/wp-backup.log 2>&1

# Weekly full backup on Sunday
0 4 * * 0 /path/to/full-backup.sh >> /var/log/wp-backup.log 2>&1
```

## Method 2: UpdraftPlus (Free Plugin)

UpdraftPlus is the most popular WordPress backup plugin with 3+ million installations.

### Installation

```bash
wp plugin install updraftplus --activate
```

### Configuration

1. Go to **Settings > UpdraftPlus Backups**
2. Click **Settings** tab
3. Configure schedule:
   - Files: Weekly
   - Database: Daily
4. Choose remote storage:
   - Dropbox
   - Google Drive
   - Amazon S3
   - FTP/SFTP
5. Set retention: Keep 3-5 backups

### Manual Backup

1. Click **Backup Now**
2. Select components:
   - [x] Include database
   - [x] Include files
3. Click **Backup Now**

### Restore from UpdraftPlus

1. Go to **Settings > UpdraftPlus Backups**
2. Click **Existing Backups** tab
3. Find backup to restore
4. Click **Restore**
5. Select components to restore
6. Follow restoration wizard

## Method 3: Duplicator Pro

Enterprise-grade backup and migration.

### Installation

```bash
wp plugin install duplicator --activate
```

### Create Package

1. Go to **Duplicator > Packages**
2. Click **Create New**
3. Configure archive settings
4. Run scanner
5. Build package
6. Download installer.php + archive

### Restore from Duplicator

1. Upload installer.php and archive to new location
2. Access installer.php in browser
3. Enter database credentials
4. Complete restoration
5. Delete installer files

## Method 4: Cloud Storage Sync

### AWS S3 Sync

```bash
#!/bin/bash
# sync-to-s3.sh

WP_PATH="/var/www/html"
S3_BUCKET="s3://your-bucket/wordpress-backups"
DATE=$(date +%Y%m%d)

# Database export
cd $WP_PATH
wp db export - | gzip | aws s3 cp - $S3_BUCKET/$DATE/database.sql.gz

# Sync wp-content
aws s3 sync wp-content/ $S3_BUCKET/$DATE/wp-content/ \
  --exclude "cache/*" \
  --exclude "*.log"

# Backup config files
aws s3 cp wp-config.php $S3_BUCKET/$DATE/
aws s3 cp .htaccess $S3_BUCKET/$DATE/

# Lifecycle policy handles retention in S3
```

### Google Cloud Storage

```bash
# Using gsutil
gsutil -m rsync -r wp-content/ gs://bucket/wp-content/
gsutil cp database.sql.gz gs://bucket/backups/
```

### Dropbox CLI

```bash
# Using rclone
rclone sync /backups/wordpress remote:wordpress-backups
```

## Method 5: Server-Level Snapshots

### Docker Volume Backup

```bash
# Stop containers
docker-compose stop

# Backup volumes
docker run --rm \
  -v wordpress_data:/data \
  -v $(pwd)/backup:/backup \
  alpine tar czf /backup/wordpress-data.tar.gz /data

# Backup database volume
docker run --rm \
  -v mysql_data:/data \
  -v $(pwd)/backup:/backup \
  alpine tar czf /backup/mysql-data.tar.gz /data

# Restart containers
docker-compose start
```

### VPS Snapshots

Most VPS providers offer snapshot backups:
- DigitalOcean: Droplet Snapshots
- AWS: EC2 Snapshots/AMIs
- Linode: Linode Backups
- Vultr: Snapshots

## Backup Verification

### Test Backup Integrity

```bash
#!/bin/bash
# verify-backup.sh

BACKUP_PATH="/backups/latest"

# Verify database can be read
gunzip -t $BACKUP_PATH/database.sql.gz && echo "Database: OK" || echo "Database: CORRUPTED"

# Verify archive integrity
tar -tzf $BACKUP_PATH/files.tar.gz > /dev/null && echo "Files: OK" || echo "Files: CORRUPTED"

# Verify checksums
cd $BACKUP_PATH
sha256sum -c checksums.txt
```

### Restoration Testing

Schedule monthly restoration tests:

1. Create staging environment
2. Restore backup to staging
3. Verify site functionality
4. Document any issues
5. Update procedures as needed

## Recovery Procedures

### Full Site Recovery

```bash
#!/bin/bash
# restore-wordpress.sh

BACKUP_PATH="/backups/20250101-120000"
WP_PATH="/var/www/html"

# Stop web server
sudo systemctl stop nginx

# Backup current state (just in case)
tar -czf /tmp/pre-restore-$(date +%Y%m%d).tar.gz $WP_PATH

# Clear current installation
rm -rf $WP_PATH/*

# Restore files
tar -xzf $BACKUP_PATH/files.tar.gz -C $WP_PATH

# Restore database
gunzip < $BACKUP_PATH/database.sql.gz | wp db import -

# Fix permissions
chown -R www-data:www-data $WP_PATH
find $WP_PATH -type d -exec chmod 755 {} \;
find $WP_PATH -type f -exec chmod 644 {} \;

# Clear caches
wp cache flush
wp rewrite flush

# Start web server
sudo systemctl start nginx

echo "Restoration complete"
```

### Database-Only Recovery

```bash
# Drop and recreate database
wp db reset --yes

# Import backup
gunzip < backup.sql.gz | wp db import -

# Clear caches
wp cache flush
wp transient delete --all
```

### Partial File Recovery

```bash
# Restore single theme
tar -xzf files.tar.gz -C / wp-content/themes/my-theme/

# Restore uploads for specific month
tar -xzf files.tar.gz -C / wp-content/uploads/2025/01/
```

### Migration/URL Change Recovery

```bash
# After restoring to new domain
wp search-replace 'https://old-domain.com' 'https://new-domain.com' --skip-columns=guid

# Update site URLs
wp option update home 'https://new-domain.com'
wp option update siteurl 'https://new-domain.com'

# Clear caches
wp cache flush
wp rewrite flush
```

## Disaster Recovery Plan

### Document Recovery Procedures

```markdown
# WordPress Disaster Recovery Plan

## Contact Information
- Hosting Provider: [contact]
- Domain Registrar: [contact]
- Backup Storage: [credentials location]

## Recovery Steps

### Total Site Loss
1. Provision new server
2. Install WordPress
3. Restore from latest backup
4. Update DNS if needed
5. Verify functionality
6. Notify stakeholders

### Database Corruption
1. Put site in maintenance mode
2. Identify last good backup
3. Restore database only
4. Verify content
5. Remove maintenance mode

### Hacked Site
1. Take site offline
2. Create forensic copy
3. Identify attack vector
4. Restore from pre-hack backup
5. Apply security patches
6. Monitor for re-infection
```

### Recovery Time Objectives

| Scenario | Target RTO |
|----------|------------|
| Database corruption | 1 hour |
| Full site restore | 4 hours |
| Hardware failure | 8 hours |
| Complete disaster | 24 hours |

## Backup Security

### Encrypt Backups

```bash
# Encrypt with GPG
gpg --symmetric --cipher-algo AES256 backup.tar.gz

# Decrypt when needed
gpg --decrypt backup.tar.gz.gpg > backup.tar.gz
```

### Secure Storage

```bash
# Restrict backup file permissions
chmod 600 /backups/*.sql.gz
chmod 600 /backups/*.tar.gz

# Separate backup user
chown backup:backup /backups/
```

### Exclude Sensitive Data

```bash
# Exclude development files
tar --exclude='node_modules' \
    --exclude='*.log' \
    --exclude='.git' \
    --exclude='cache/*' \
    -czf backup.tar.gz wp-content/
```

## Monitoring

### Backup Status Alerts

```bash
#!/bin/bash
# check-backup.sh

BACKUP_DIR="/backups"
MAX_AGE=86400  # 24 hours in seconds
ALERT_EMAIL="admin@example.com"

LATEST=$(ls -t $BACKUP_DIR/*/database.sql.gz 2>/dev/null | head -1)

if [ -z "$LATEST" ]; then
    echo "No backups found!" | mail -s "BACKUP ALERT" $ALERT_EMAIL
    exit 1
fi

AGE=$(( $(date +%s) - $(stat -c %Y "$LATEST") ))

if [ $AGE -gt $MAX_AGE ]; then
    echo "Latest backup is $((AGE/3600)) hours old!" | mail -s "BACKUP ALERT" $ALERT_EMAIL
    exit 1
fi

echo "Backup OK: $(basename $(dirname $LATEST))"
```

### Add to Monitoring

```bash
# Add to cron for hourly checks
0 * * * * /path/to/check-backup.sh
```

## Recommended Plugins

| Plugin | Features | Best For |
|--------|----------|----------|
| UpdraftPlus | Free, cloud storage | Most sites |
| Duplicator Pro | Migration, multisite | Developers |
| BlogVault | Real-time, staging | High-traffic |
| BackWPup | Free, extensive options | Budget sites |
| VaultPress | Jetpack integration | Jetpack users |

## Troubleshooting

### Backup Too Large

1. Exclude unnecessary files:
```bash
tar --exclude='wp-content/cache/*' \
    --exclude='wp-content/uploads/backupbuddy_backups/*' \
    -czf backup.tar.gz wp-content/
```

2. Split into smaller archives:
```bash
# Split database and files
wp db export database.sql
tar -czf uploads.tar.gz wp-content/uploads/
tar -czf themes-plugins.tar.gz wp-content/themes/ wp-content/plugins/
```

### Restoration Fails

1. Check PHP memory limit
2. Increase max_execution_time
3. Try restoring components separately
4. Use WP-CLI instead of web interface

### Database Import Errors

```bash
# Check for encoding issues
file database.sql

# Convert if needed
iconv -f ISO-8859-1 -t UTF-8 database.sql > database-utf8.sql

# Import with correct charset
wp db import database.sql --dbcharset=utf8mb4
```

## Related Documents

- ref-security-checklist.md - Security audit checklist
- howto-security-hardening.md - General hardening
- concept-owasp-wordpress.md - Understanding security
