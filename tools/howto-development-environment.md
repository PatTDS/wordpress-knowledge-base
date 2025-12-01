---
title: WordPress Development Environment Setup
category: tools
type: howto
updated: 2025-12-01
version: 1.0.0
tags:
  - local-development
  - development-environment
  - local-wp
  - wordpress-studio
  - xampp
sources:
  - https://jetpack.com/resources/local-wordpress-development-environment/
  - https://wordpress.com/blog/2024/04/24/studio/
  - https://localwp.com/
  - https://instawp.com/is-local-development-relevant-in-2024/
---

# WordPress Development Environment Setup

Guide to setting up local WordPress development environments using 2024-2025 tools.

## Tool Comparison

| Tool | Best For | OS | Learning Curve |
|------|----------|-----|----------------|
| WordPress Studio | Quick prototyping | Win/Mac/Linux | Low |
| Local (Flywheel) | Professional development | Win/Mac/Linux | Low |
| Docker | Production parity | All | Medium |
| XAMPP | Basic learning | All | Low |
| Lando | Docker abstraction | All | Medium |
| VVV | Contributor development | All | High |

## WordPress Studio (Recommended for 2025)

### Overview

Released April 2024 by Automattic. Uses WordPress Playground (WebAssembly) - no web server, MySQL, or virtualization needed.

### Installation

1. Download from https://developer.wordpress.com/studio/
2. Install and launch
3. Create new site in one click

### Features

- Instant site creation
- No PHP/MySQL installation required
- Built on WordPress Playground
- Blueprint support for configurations
- Export to WordPress.com/hosting

### Create Site

```
1. Click "Add Site"
2. Enter site name
3. Select WordPress version
4. Click "Create"
```

Site is immediately available at local URL.

### Limitations

- No custom PHP extensions
- Limited server configuration
- Not suitable for complex plugins

## Local by Flywheel (Most Popular)

### Installation

Download from https://localwp.com/

### Create Site

```
1. Click "+" button
2. Choose "Create a new site"
3. Enter site name
4. Select environment (Preferred, Custom)
5. Set WordPress credentials
6. Click "Add Site"
```

### Features

- One-click SSL
- Live Links (share local site)
- Multiple PHP versions
- WP-CLI included
- Database management
- SSH access
- Blueprint system

### PHP Version Switching

```
1. Right-click site
2. Go to "PHP Version"
3. Select desired version
4. Local restarts automatically
```

### Enable SSL

```
1. Select site
2. Click "Trust" under SSL
3. Site available at https://
```

### Access WP-CLI

```bash
# Open site shell
Right-click site → "Open Site Shell"

# Run WP-CLI commands
wp plugin list
wp theme list
wp cache flush
```

### Live Links (Client Preview)

```
1. Select site
2. Click "Enable" under Live Link
3. Share generated URL
```

### Blueprints

Create reusable site configurations:

```
1. Set up site with plugins/themes
2. Right-click → "Save as Blueprint"
3. Use blueprint for new sites
```

## XAMPP Setup

### Installation

Download from https://www.apachefriends.org/

### Windows

```
1. Run installer
2. Select Apache, MySQL, PHP, phpMyAdmin
3. Complete installation
4. Start Apache and MySQL from control panel
```

### macOS

```bash
# Download and install
# Or use Homebrew
brew install --cask xampp
```

### Create WordPress Site

```bash
# Navigate to htdocs
cd /Applications/XAMPP/htdocs  # macOS
cd C:\xampp\htdocs             # Windows

# Download WordPress
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
mv wordpress my-site
```

### Create Database

```
1. Open http://localhost/phpmyadmin
2. Click "New"
3. Enter database name
4. Click "Create"
```

### Configure WordPress

```
1. Open http://localhost/my-site
2. Follow installation wizard
3. Enter database credentials
   - Database: your_db_name
   - User: root
   - Password: (empty by default)
   - Host: localhost
```

### Common XAMPP Issues

**Port 80 in use:**
```
Edit httpd.conf:
Listen 8080
ServerName localhost:8080
```

**MySQL won't start:**
```
Check for existing MySQL service
Stop other MySQL instances
```

## Docker Setup

See @howto-docker-wordpress.md for comprehensive Docker guide.

### Quick Start

```bash
# Create docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'
services:
  wordpress:
    image: wordpress:6.7
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - ./wp-content:/var/www/html/wp-content
  db:
    image: mysql:8.0
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
      MYSQL_ROOT_PASSWORD: rootpassword
    volumes:
      - db_data:/var/lib/mysql
volumes:
  db_data:
EOF

# Start
docker-compose up -d
```

Access at http://localhost:8080

## Lando Setup

### Installation

```bash
# macOS
brew install lando

# Windows
# Download from https://lando.dev/download/
```

### Create WordPress Project

```bash
mkdir my-wp-site
cd my-wp-site
lando init --recipe wordpress
lando start
```

### .lando.yml Configuration

```yaml
name: my-wp-site
recipe: wordpress

config:
  webroot: .
  php: '8.2'
  database: mysql:8.0
  xdebug: true

services:
  appserver:
    build:
      - composer install

tooling:
  wp:
    service: appserver
    cmd: wp
```

### Lando Commands

```bash
# Start environment
lando start

# Stop environment
lando stop

# Run WP-CLI
lando wp plugin list

# Import database
lando wp db import backup.sql

# SSH into container
lando ssh
```

## VVV (Vagrant)

### Prerequisites

- VirtualBox
- Vagrant

### Installation

```bash
# Install Vagrant
brew install vagrant  # macOS
# Or download from vagrantup.com

# Clone VVV
git clone https://github.com/Varying-Vagrant-Vagrants/VVV.git
cd VVV

# Create custom config
cp config/default-config.yml config/config.yml

# Start VVV
vagrant up
```

### config.yml

```yaml
sites:
  my-site:
    repo: https://github.com/Varying-Vagrant-Vagrants/custom-site-template.git
    hosts:
      - my-site.test
    custom:
      wpconfig_constants:
        WP_DEBUG: true
        WP_DEBUG_LOG: true
```

### Access Sites

```
Dashboard: http://vvv.test
Site: http://my-site.test
```

## Development Tools Integration

### VS Code Extensions

```json
// .vscode/extensions.json
{
  "recommendations": [
    "wordpresstoolbox.wordpress-toolbox",
    "bmewburn.vscode-intelephense-client",
    "xdebug.php-debug",
    "bradlc.vscode-tailwindcss",
    "dbaeumer.vscode-eslint"
  ]
}
```

### PHP Debugging

#### Local by Flywheel + Xdebug

```
1. Select site
2. Go to "Settings" tab
3. Enable Xdebug
4. Configure VS Code
```

#### VS Code launch.json

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Listen for Xdebug",
      "type": "php",
      "request": "launch",
      "port": 9003,
      "pathMappings": {
        "/app/public": "${workspaceFolder}"
      }
    }
  ]
}
```

### Database Management

**TablePlus (Recommended):**
- Connect to MySQL
- Visual query builder
- Cross-platform

**Sequel Pro (macOS):**
- Free
- SSH tunnel support

**HeidiSQL (Windows):**
- Free
- Portable version available

## Workflow Recommendations

### Theme Development

```bash
# Use Local by Flywheel
1. Create site with Local
2. Clone theme to wp-content/themes/
3. Enable file watching: npm run watch
4. Use Live Links for client review
```

### Plugin Development

```bash
# Use Docker for isolation
1. Create docker-compose.yml
2. Mount plugin directory
3. Easy PHP version switching
4. Test with multiple WP versions
```

### Multisite Testing

```bash
# Use VVV
1. Configure multisite in config.yml
2. vagrant provision
3. Test subdomain/subdirectory setups
```

## Environment Parity

### Production-Like Development

Match production environment:

| Setting | Development | Production |
|---------|-------------|------------|
| PHP Version | 8.2 | 8.2 |
| MySQL | 8.0 | 8.0 |
| WP Version | 6.7 | 6.7 |
| Debug | Enabled | Disabled |

### wp-config.php Development Settings

```php
// Development
define('WP_DEBUG', true);
define('WP_DEBUG_LOG', true);
define('WP_DEBUG_DISPLAY', true);
define('SCRIPT_DEBUG', true);
define('SAVEQUERIES', true);

// Disable auto-updates for testing
define('AUTOMATIC_UPDATER_DISABLED', true);
define('WP_AUTO_UPDATE_CORE', false);
```

## Syncing Between Local and Remote

### Pull Database from Production

```bash
# Export from production
wp @production db export - | gzip > prod.sql.gz

# Import locally
gunzip -c prod.sql.gz | wp db import -
wp search-replace 'https://production.com' 'http://localhost:8080' --skip-columns=guid
```

### Sync Uploads

```bash
# Rsync uploads from production
rsync -avz user@production:/var/www/html/wp-content/uploads/ ./wp-content/uploads/
```

## Cloud-Based Alternatives (2025 Trend)

### InstaWP

- Instant WordPress sandboxes
- No local setup
- Share via URL
- Git integration

### WordPress Playground

- Browser-based WordPress
- Blueprint configurations
- No installation
- https://playground.wordpress.net

### Jurassic Ninja

- Temporary WP sites
- For testing plugins
- Auto-expires

## Best Practices

### DO

1. **Match production environment** - Same PHP/MySQL versions
2. **Use version control** - Git for themes/plugins
3. **Enable debugging** - WP_DEBUG, error logging
4. **Regular backups** - Before major changes
5. **Test with Query Monitor** - Performance debugging

### DON'T

1. Develop on production
2. Use mismatched PHP versions
3. Skip local testing
4. Ignore error logs
5. Commit debug settings

## Troubleshooting

### Port Conflicts

```bash
# Find process using port 80
lsof -i :80

# Use different port
# Local: Change in site settings
# Docker: Modify ports in docker-compose.yml
```

### Permission Issues

```bash
# Fix ownership (Linux/macOS)
sudo chown -R $(whoami):staff wp-content

# Fix permissions
find wp-content -type d -exec chmod 755 {} \;
find wp-content -type f -exec chmod 644 {} \;
```

### SSL Certificate Errors

```bash
# Local by Flywheel
Right-click site → "Trust"

# Docker
Use mkcert for local certificates
```

## Related Documents

- @howto-docker-wordpress.md
- @ref-wp-cli-commands.md
- @howto-git-wordpress-workflow.md
