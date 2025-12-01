---
title: GitHub Actions for WordPress Reference
description: Complete reference for GitHub Actions workflows for WordPress testing and deployment
category: testing
type: reference
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - github-actions
  - ci-cd
  - wordpress
  - automation
  - deployment
sources:
  - https://docs.wpvip.com/code-deployment/default-deployment/build-and-deploy/github-actions/
  - https://github.com/10up/actions-wordpress
  - https://css-tricks.com/continuous-deployments-for-wordpress-using-github-actions/
  - https://www.hostinger.com/tutorials/wordpress-continuous-integration-and-deployment
  - https://pressidium.com/blog/building-a-cicd-workflow-automatically-deploying-a-wordpress-theme-with-github-actions/
---

# GitHub Actions for WordPress Reference

## Overview

GitHub Actions automates builds, tests, and deployments for WordPress projects. With GitHub Actions, Continuous Deployment is free for everyone.

## Available WordPress Actions

### 10up/actions-wordpress

Collection of GitHub Actions for WordPress development:

| Action | Description |
|--------|-------------|
| `10up/action-wordpress-plugin-deploy` | Deploy plugin to WordPress.org |
| `10up/action-wordpress-plugin-asset-update` | Update assets on WordPress.org |
| `10up/wpcs-action` | Run PHPCS against WordPress Coding Standards |

### WordPress VIP Actions

Only Standard-class, Linux runners are supported for WordPress VIP applications.

## Workflow Templates

### Basic WordPress CI

```yaml
name: WordPress CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: root
          MYSQL_DATABASE: wordpress_test
        ports:
          - 3306:3306
        options: --health-cmd="mysqladmin ping" --health-interval=10s --health-timeout=5s --health-retries=3

    steps:
      - uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'
          extensions: mbstring, intl, mysqli
          coverage: xdebug

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Get Composer cache directory
        id: composer-cache
        run: echo "dir=$(composer config cache-files-dir)" >> $GITHUB_OUTPUT

      - name: Cache Composer
        uses: actions/cache@v4
        with:
          path: ${{ steps.composer-cache.outputs.dir }}
          key: ${{ runner.os }}-composer-${{ hashFiles('**/composer.lock') }}
          restore-keys: ${{ runner.os }}-composer-

      - name: Install Composer dependencies
        run: composer install --prefer-dist --no-progress

      - name: Install npm dependencies
        run: npm ci

      - name: Build assets
        run: npm run build

      - name: Run PHP linting
        run: composer lint

      - name: Run PHPCS
        run: composer phpcs

      - name: Run PHP tests
        run: composer test
        env:
          WP_DB_HOST: 127.0.0.1
          WP_DB_NAME: wordpress_test
          WP_DB_USER: root
          WP_DB_PASSWORD: root
```

### WordPress Coding Standards Check

```yaml
name: WordPress Coding Standards

on: [push, pull_request]

jobs:
  phpcs:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'
          tools: composer, cs2pr

      - name: Install dependencies
        run: composer install --prefer-dist --no-progress

      - name: Run PHPCS
        run: |
          ./vendor/bin/phpcs --report=checkstyle \
            --standard=WordPress \
            --extensions=php \
            --ignore=vendor,node_modules \
            . | cs2pr
```

### Using 10up PHPCS Action

```yaml
name: PHPCS

on: [push, pull_request]

jobs:
  phpcs:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: PHPCS check
        uses: 10up/action-wordpress-plugin-asset-update@stable
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}
```

### Security Scan Workflow

```yaml
name: Security Scan

on:
  push:
    branches: [main]
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight

jobs:
  security:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'

      - name: Install WP-CLI
        run: |
          curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
          chmod +x wp-cli.phar
          sudo mv wp-cli.phar /usr/local/bin/wp

      - name: Vulnerability scan
        run: |
          wp package install developer
          wp plugin security-check --all || true
          wp theme security-check --all || true

      - name: Composer audit
        run: composer audit --no-dev

      - name: npm audit
        run: npm audit --production
```

### Full E2E Testing Workflow

```yaml
name: E2E Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  e2e:
    runs-on: ubuntu-latest
    timeout-minutes: 30

    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: root
          MYSQL_DATABASE: wordpress
        ports:
          - 3306:3306
        options: --health-cmd="mysqladmin ping" --health-interval=10s --health-timeout=5s --health-retries=3

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps

      - name: Start WordPress
        run: |
          docker-compose up -d
          sleep 30
          curl -I http://localhost:8080

      - name: Run E2E tests
        run: npx playwright test
        env:
          WP_BASE_URL: http://localhost:8080
          WP_ADMIN_USER: admin
          WP_ADMIN_PASS: password

      - name: Upload test artifacts
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

### Theme Deployment Workflow

```yaml
name: Deploy Theme

on:
  push:
    branches: [main]
    paths:
      - 'wp-content/themes/my-theme/**'

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Build theme
        working-directory: wp-content/themes/my-theme
        run: |
          npm ci
          npm run build

      - name: Deploy to server
        uses: easingthemes/ssh-deploy@v5
        with:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          REMOTE_HOST: ${{ secrets.REMOTE_HOST }}
          REMOTE_USER: ${{ secrets.REMOTE_USER }}
          SOURCE: "wp-content/themes/my-theme/"
          TARGET: "/var/www/html/wp-content/themes/my-theme/"
          EXCLUDE: "node_modules/, .git/, .github/"

      - name: Clear cache
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.REMOTE_HOST }}
          username: ${{ secrets.REMOTE_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /var/www/html
            wp cache flush
            wp rewrite flush
```

### Plugin Deployment to WordPress.org

```yaml
name: Deploy Plugin

on:
  release:
    types: [published]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: WordPress Plugin Deploy
        uses: 10up/action-wordpress-plugin-deploy@stable
        env:
          SVN_PASSWORD: ${{ secrets.SVN_PASSWORD }}
          SVN_USERNAME: ${{ secrets.SVN_USERNAME }}
          SLUG: my-plugin
```

## Composite Workflows

### Reusable Build Workflow

Create `.github/workflows/build.yml`:

```yaml
name: Build

on:
  workflow_call:
    inputs:
      node-version:
        required: false
        type: string
        default: '20'
      php-version:
        required: false
        type: string
        default: '8.2'

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ inputs.php-version }}

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: 'npm'

      - name: Install dependencies
        run: |
          composer install --prefer-dist --no-progress
          npm ci

      - name: Build
        run: npm run build

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build
          path: |
            wp-content/themes/*/dist
            wp-content/plugins/*/dist
```

### Use Reusable Workflow

```yaml
name: CI

on: [push, pull_request]

jobs:
  build:
    uses: ./.github/workflows/build.yml
    with:
      node-version: '20'
      php-version: '8.2'

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/download-artifact@v4
        with:
          name: build
      - run: npm test
```

## Action Configuration Reference

### Environment Variables

| Variable | Description |
|----------|-------------|
| `GITHUB_TOKEN` | Auto-provided authentication token |
| `GITHUB_REPOSITORY` | Repository owner/name |
| `GITHUB_SHA` | Commit SHA |
| `GITHUB_REF` | Branch or tag ref |
| `GITHUB_WORKFLOW` | Workflow name |

### Secrets Configuration

Required secrets for deployment:

| Secret | Description |
|--------|-------------|
| `SSH_PRIVATE_KEY` | SSH key for server access |
| `REMOTE_HOST` | Server hostname |
| `REMOTE_USER` | SSH username |
| `SVN_USERNAME` | WordPress.org username |
| `SVN_PASSWORD` | WordPress.org password |

### Caching Strategies

```yaml
# Composer cache
- name: Get Composer cache directory
  id: composer-cache
  run: echo "dir=$(composer config cache-files-dir)" >> $GITHUB_OUTPUT

- uses: actions/cache@v4
  with:
    path: ${{ steps.composer-cache.outputs.dir }}
    key: ${{ runner.os }}-composer-${{ hashFiles('**/composer.lock') }}
    restore-keys: ${{ runner.os }}-composer-

# npm cache (automatic with setup-node)
- uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: 'npm'

# Docker layer cache
- uses: docker/setup-buildx-action@v3

- uses: docker/build-push-action@v5
  with:
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

## Matrix Testing

### Multiple PHP Versions

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        php: ['8.0', '8.1', '8.2', '8.3']
        wp: ['6.4', '6.5', '6.6']

    steps:
      - uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ matrix.php }}

      - name: Run tests
        run: composer test
        env:
          WP_VERSION: ${{ matrix.wp }}
```

## Scheduled Workflows

### Daily Security Check

```yaml
name: Daily Security

on:
  schedule:
    - cron: '0 6 * * *'  # 6 AM UTC daily

jobs:
  security-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: composer audit
      - run: npm audit --production
```

### Weekly Dependency Updates

```yaml
name: Update Dependencies

on:
  schedule:
    - cron: '0 0 * * 0'  # Sunday midnight

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Update npm dependencies
        run: |
          npm update
          npm audit fix

      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v6
        with:
          commit-message: 'chore: update dependencies'
          title: 'Weekly dependency updates'
          branch: dependency-updates
```

## Best Practices

### 1. Use Latest Action Versions

```yaml
# Good
uses: actions/checkout@v4
uses: actions/setup-node@v4

# Avoid
uses: actions/checkout@v2  # Outdated
uses: actions/checkout@main  # Unstable
```

### 2. Minimize Redundant Steps

Use caching and composite workflows.

### 3. Fail Fast

```yaml
strategy:
  fail-fast: true
  matrix:
    php: ['8.0', '8.1', '8.2']
```

### 4. Secure Secrets

- Never hardcode credentials
- Use environment-specific secrets
- Rotate secrets regularly

### 5. Timeout Protection

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 30
```

## Related Documents

- howto-lighthouse-ci.md - Lighthouse CI setup
- howto-playwright-wordpress.md - E2E testing
- howto-accessibility-automation.md - Accessibility CI
