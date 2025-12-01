---
title: WordPress Security Checklist
description: Comprehensive security checklist for WordPress sites covering all hardening areas
category: security
type: reference
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - security
  - checklist
  - hardening
  - audit
audience: intermediate
sources:
  - https://wp-umbrella.com/blog/wordpress-security-best-practices/
  - https://www.wpbeginner.com/wordpress-security/
  - https://wpsecurityninja.com/wordpress-security-best-practices/
  - https://patchstack.com/articles/wordpress-security-headers/
  - https://developer.wordpress.org/advanced-administration/security/https/
---

# WordPress Security Checklist

Quick reference checklist for WordPress security hardening. Use this for security audits and new site setup.

## Pre-Launch Checklist

### Core Updates
- [ ] WordPress core is latest version
- [ ] All plugins updated to latest versions
- [ ] All themes updated to latest versions
- [ ] PHP version 8.0+ installed
- [ ] MySQL 8.0+ / MariaDB 10.5+ running

### Authentication
- [ ] Default "admin" username removed/renamed
- [ ] Strong passwords enforced (min 12 characters)
- [ ] Two-factor authentication (2FA) enabled for admins
- [ ] 2FA enabled for editors and authors
- [ ] Login attempt limiting configured
- [ ] Login page protected (rename or captcha)

### SSL/HTTPS
- [ ] Valid SSL certificate installed
- [ ] TLS 1.2+ enforced (TLS 1.3 preferred)
- [ ] HTTP to HTTPS redirect configured
- [ ] `FORCE_SSL_ADMIN` set to true
- [ ] Mixed content issues resolved
- [ ] HSTS header enabled

### Security Headers
- [ ] X-Content-Type-Options: nosniff
- [ ] X-Frame-Options: SAMEORIGIN or CSP frame-ancestors
- [ ] X-XSS-Protection: 1; mode=block
- [ ] Referrer-Policy configured
- [ ] Content-Security-Policy (CSP) implemented
- [ ] Permissions-Policy set

### File Security
- [ ] wp-config.php permissions: 600
- [ ] .htaccess permissions: 644
- [ ] Directory permissions: 755
- [ ] File permissions: 644
- [ ] uploads/ PHP execution disabled
- [ ] wp-includes/ protected
- [ ] Directory listing disabled

### WordPress Configuration
- [ ] WP_DEBUG disabled in production
- [ ] DISALLOW_FILE_EDIT: true
- [ ] DISALLOW_FILE_MODS: true (if no plugin updates via admin)
- [ ] Security salts configured and unique
- [ ] Database table prefix changed from wp_

### Access Control
- [ ] XML-RPC disabled (if not needed)
- [ ] REST API restricted (if needed)
- [ ] User enumeration blocked
- [ ] Author archives disabled (if not needed)
- [ ] WordPress version hidden

### Plugins & Themes
- [ ] Unused plugins deleted (not just deactivated)
- [ ] Unused themes deleted
- [ ] Only reputable plugins installed
- [ ] Plugin update notifications enabled
- [ ] Auto-updates configured

### Backups
- [ ] Automated backup schedule configured
- [ ] Database backups daily (high-traffic) or weekly
- [ ] Full-site backups weekly
- [ ] Backups stored offsite (cloud storage)
- [ ] 3-2-1 backup rule followed
- [ ] Backup restoration tested

### Monitoring
- [ ] Security plugin installed (Wordfence/Sucuri/Shield)
- [ ] File integrity monitoring enabled
- [ ] Login activity logging configured
- [ ] Failed login notifications enabled
- [ ] Malware scanning scheduled
- [ ] Uptime monitoring configured

## Ongoing Maintenance Checklist

### Weekly
- [ ] Review security scan results
- [ ] Check for plugin/theme updates
- [ ] Verify backup completion
- [ ] Review login activity logs

### Monthly
- [ ] Full security audit
- [ ] User account review
- [ ] Password rotation for service accounts
- [ ] Test backup restoration
- [ ] Review access logs

### Quarterly
- [ ] SSL certificate validity check
- [ ] Security header verification
- [ ] Plugin necessity review
- [ ] User role audit
- [ ] Vulnerability assessment

### Annually
- [ ] Full penetration test (if applicable)
- [ ] Disaster recovery drill
- [ ] Security policy review
- [ ] GDPR compliance audit
- [ ] Third-party security audit

## Emergency Response Checklist

### Suspected Breach
- [ ] Enable maintenance mode
- [ ] Create forensic backup
- [ ] Run malware scan
- [ ] Check file modifications (last 7 days)
- [ ] Review access logs
- [ ] Reset all passwords
- [ ] Regenerate security salts
- [ ] Restore from clean backup
- [ ] Re-verify checksums
- [ ] Document incident

### Post-Incident
- [ ] Identify attack vector
- [ ] Patch vulnerability
- [ ] Review and update security measures
- [ ] Notify affected users (if data breach)
- [ ] Update incident response plan

## Compliance Checklist

### GDPR (if serving EU users)
- [ ] Privacy policy published
- [ ] Cookie consent mechanism
- [ ] Data processing agreements
- [ ] Right to deletion implemented
- [ ] Data export capability
- [ ] Consent logging

### General Compliance
- [ ] Terms of service published
- [ ] Contact information visible
- [ ] Data retention policy defined
- [ ] Third-party data sharing disclosed

## Quick Commands

```bash
# Verify WordPress checksums
wp core verify-checksums
wp plugin verify-checksums --all

# Check security status
wp doctor check

# Update everything
wp core update && wp plugin update --all && wp theme update --all

# Export for backup
wp db export backup-$(date +%Y%m%d).sql

# Clear caches
wp cache flush
wp transient delete --all
```

## Related Documents

- howto-security-hardening.md - Detailed hardening procedures
- howto-authentication-2fa.md - 2FA implementation
- howto-ssl-https.md - SSL configuration
- howto-security-headers.md - Header implementation
- howto-gdpr-compliance.md - GDPR compliance
- howto-backup-recovery.md - Backup strategies
