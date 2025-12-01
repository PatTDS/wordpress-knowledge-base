---
title: WordPress Deployment Automation
category: tools
type: howto
updated: 2025-12-01
version: 1.0.0
tags:
  - deployment
  - ci-cd
  - github-actions
  - automation
sources:
  - https://css-tricks.com/continuous-deployments-for-wordpress-using-github-actions/
  - https://kinsta.com/blog/continuous-deployment-wordpress-github-actions/
  - https://github.com/rtCamp/action-deploy-wordpress
  - https://docs.wpvip.com/code-deployment/default-deployment/build-and-deploy/github-actions/
---

# WordPress Deployment Automation

Automate WordPress deployments using CI/CD pipelines with GitHub Actions.

## Prerequisites

- GitHub repository with WordPress theme/plugin
- SSH access to deployment server
- Basic understanding of GitHub Actions

## GitHub Actions Setup

### 1. Create Workflow Directory

```bash
mkdir -p .github/workflows
```

### 2. Basic Deployment Workflow

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy WordPress

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build assets
        run: npm run build

      - name: Deploy to server
        uses: easingthemes/ssh-deploy@v4
        with:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          REMOTE_HOST: ${{ secrets.REMOTE_HOST }}
          REMOTE_USER: ${{ secrets.REMOTE_USER }}
          SOURCE: "./"
          TARGET: "/var/www/html/wp-content/themes/my-theme"
          EXCLUDE: "node_modules/, .git/, .github/, src/, *.md"
```

### 3. Configure Secrets

In GitHub repository: Settings → Secrets → Actions

| Secret | Description |
|--------|-------------|
| `SSH_PRIVATE_KEY` | Private key for server access |
| `REMOTE_HOST` | Server hostname/IP |
| `REMOTE_USER` | SSH username |

## Complete CI/CD Pipeline

### Theme Deployment with Testing

```yaml
name: WordPress Theme CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

env:
  THEME_DIR: wp-content/themes/my-theme

jobs:
  # Job 1: Code Quality
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'
          tools: composer, cs2pr

      - name: Install Composer dependencies
        run: composer install --no-dev

      - name: Run PHP CodeSniffer
        run: vendor/bin/phpcs --standard=WordPress ${{ env.THEME_DIR }}

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install npm dependencies
        working-directory: ${{ env.THEME_DIR }}
        run: npm ci

      - name: Run ESLint
        working-directory: ${{ env.THEME_DIR }}
        run: npm run lint

  # Job 2: Build
  build:
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        working-directory: ${{ env.THEME_DIR }}
        run: npm ci

      - name: Build production assets
        working-directory: ${{ env.THEME_DIR }}
        run: npm run build

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: theme-build
          path: ${{ env.THEME_DIR }}/dist

  # Job 3: Deploy to Staging
  deploy-staging:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/develop'
    environment: staging
    steps:
      - uses: actions/checkout@v4

      - name: Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: theme-build
          path: ${{ env.THEME_DIR }}/dist

      - name: Deploy to staging
        uses: easingthemes/ssh-deploy@v4
        with:
          SSH_PRIVATE_KEY: ${{ secrets.STAGING_SSH_KEY }}
          REMOTE_HOST: ${{ secrets.STAGING_HOST }}
          REMOTE_USER: ${{ secrets.STAGING_USER }}
          SOURCE: "${{ env.THEME_DIR }}/"
          TARGET: "/var/www/staging/wp-content/themes/my-theme"
          EXCLUDE: "node_modules/, src/, *.md, package*.json"

      - name: Clear staging cache
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: ${{ secrets.STAGING_USER }}
          key: ${{ secrets.STAGING_SSH_KEY }}
          script: |
            cd /var/www/staging
            wp cache flush

  # Job 4: Deploy to Production
  deploy-production:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    environment: production
    steps:
      - uses: actions/checkout@v4

      - name: Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: theme-build
          path: ${{ env.THEME_DIR }}/dist

      - name: Create backup
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.PROD_HOST }}
          username: ${{ secrets.PROD_USER }}
          key: ${{ secrets.PROD_SSH_KEY }}
          script: |
            cd /var/www/production
            wp db export /backups/pre-deploy-$(date +%Y%m%d-%H%M%S).sql
            tar -czf /backups/theme-$(date +%Y%m%d-%H%M%S).tar.gz wp-content/themes/my-theme

      - name: Deploy to production
        uses: easingthemes/ssh-deploy@v4
        with:
          SSH_PRIVATE_KEY: ${{ secrets.PROD_SSH_KEY }}
          REMOTE_HOST: ${{ secrets.PROD_HOST }}
          REMOTE_USER: ${{ secrets.PROD_USER }}
          SOURCE: "${{ env.THEME_DIR }}/"
          TARGET: "/var/www/production/wp-content/themes/my-theme"
          EXCLUDE: "node_modules/, src/, *.md, package*.json"

      - name: Post-deployment tasks
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.PROD_HOST }}
          username: ${{ secrets.PROD_USER }}
          key: ${{ secrets.PROD_SSH_KEY }}
          script: |
            cd /var/www/production
            wp cache flush
            wp rewrite flush
```

## rtCamp WordPress Deploy Action

### Using rtCamp's Action

```yaml
name: Deploy WordPress with rtCamp

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy
        uses: rtCamp/action-deploy-wordpress@v3
        env:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          REMOTE_HOST: ${{ secrets.REMOTE_HOST }}
          REMOTE_USER: ${{ secrets.REMOTE_USER }}
          WP_VERSION: "6.7"
```

### Repository Structure for rtCamp

```
repo/
├── .github/
│   └── workflows/
├── themes/
│   └── my-theme/
├── plugins/
│   └── my-plugin/
├── mu-plugins/
├── languages/
└── hosts.yml
```

### hosts.yml

```yaml
default:
  hostname: example.com
  user: deploy
  deploy_path: /var/www/html
  stage: production

staging:
  hostname: staging.example.com
  stage: staging
```

## Plugin Deployment

```yaml
name: Deploy Plugin

on:
  push:
    branches: [main]
    paths:
      - 'wp-content/plugins/my-plugin/**'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'

      - name: Install Composer dependencies
        working-directory: wp-content/plugins/my-plugin
        run: composer install --no-dev --optimize-autoloader

      - name: Deploy plugin
        uses: easingthemes/ssh-deploy@v4
        with:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          REMOTE_HOST: ${{ secrets.REMOTE_HOST }}
          REMOTE_USER: ${{ secrets.REMOTE_USER }}
          SOURCE: "wp-content/plugins/my-plugin/"
          TARGET: "/var/www/html/wp-content/plugins/my-plugin"
          EXCLUDE: "tests/, .git/, composer.lock"
```

## Database Migrations

### Safe Search-Replace

```yaml
- name: Database URL replacement
  uses: appleboy/ssh-action@v1
  with:
    host: ${{ secrets.REMOTE_HOST }}
    username: ${{ secrets.REMOTE_USER }}
    key: ${{ secrets.SSH_PRIVATE_KEY }}
    script: |
      cd /var/www/html
      wp search-replace '${{ secrets.OLD_URL }}' '${{ secrets.NEW_URL }}' \
        --skip-columns=guid \
        --dry-run

      # If dry-run looks good, run for real
      wp search-replace '${{ secrets.OLD_URL }}' '${{ secrets.NEW_URL }}' \
        --skip-columns=guid
```

## Rollback Strategy

### Automatic Rollback on Failure

```yaml
deploy:
  runs-on: ubuntu-latest
  steps:
    - name: Create backup
      id: backup
      uses: appleboy/ssh-action@v1
      with:
        host: ${{ secrets.REMOTE_HOST }}
        username: ${{ secrets.REMOTE_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          BACKUP_NAME="backup-$(date +%Y%m%d-%H%M%S)"
          cp -r /var/www/html/wp-content/themes/my-theme "/backups/$BACKUP_NAME"
          echo "::set-output name=backup_name::$BACKUP_NAME"

    - name: Deploy
      id: deploy
      continue-on-error: true
      # ... deployment steps ...

    - name: Health check
      id: health
      run: |
        RESPONSE=$(curl -s -o /dev/null -w "%{http_code}" https://example.com)
        if [ "$RESPONSE" != "200" ]; then
          echo "Health check failed!"
          exit 1
        fi

    - name: Rollback on failure
      if: failure()
      uses: appleboy/ssh-action@v1
      with:
        host: ${{ secrets.REMOTE_HOST }}
        username: ${{ secrets.REMOTE_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          rm -rf /var/www/html/wp-content/themes/my-theme
          cp -r "/backups/${{ steps.backup.outputs.backup_name }}" /var/www/html/wp-content/themes/my-theme
          cd /var/www/html && wp cache flush
```

## Environment Protection

### GitHub Environments

Configure in Settings → Environments:

**Staging Environment:**
- Required reviewers: None
- Wait timer: 0 minutes

**Production Environment:**
- Required reviewers: 1
- Wait timer: 5 minutes
- Deployment branches: main only

## Notifications

### Slack Notifications

```yaml
- name: Notify Slack
  uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    fields: repo,message,commit,author
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
  if: always()
```

### Discord Notifications

```yaml
- name: Discord notification
  if: failure()
  uses: sarisia/actions-status-discord@v1
  with:
    webhook: ${{ secrets.DISCORD_WEBHOOK }}
    title: "Deployment Failed"
    description: "Check the workflow for details"
    color: 0xff0000
```

## Performance Testing in Pipeline

```yaml
- name: Lighthouse CI
  uses: treosh/lighthouse-ci-action@v11
  with:
    urls: |
      https://staging.example.com
      https://staging.example.com/about
    uploadArtifacts: true
    temporaryPublicStorage: true
    budgetPath: ./budget.json
```

### budget.json

```json
[
  {
    "path": "/*",
    "resourceSizes": [
      { "resourceType": "script", "budget": 300 },
      { "resourceType": "image", "budget": 500 },
      { "resourceType": "total", "budget": 1000 }
    ],
    "resourceCounts": [
      { "resourceType": "script", "budget": 20 }
    ]
  }
]
```

## Security Scanning

```yaml
- name: Security Audit
  uses: shivammathur/setup-php@v2
  with:
    php-version: '8.2'
    tools: phpseclib

- name: Check for vulnerabilities
  run: |
    composer audit
    npm audit --audit-level=moderate
```

## Best Practices

### DO

1. **Use environment secrets** - Never hardcode credentials
2. **Create backups** before deployment
3. **Add health checks** after deployment
4. **Use dry-run** for database operations
5. **Require reviews** for production
6. **Test on staging first**

### DON'T

1. Deploy directly to production without staging
2. Skip backup creation
3. Deploy on Fridays
4. Ignore failed health checks
5. Store secrets in repository

## Troubleshooting

### SSH Connection Issues

```yaml
- name: Debug SSH
  run: |
    ssh -vvv -i ~/.ssh/deploy_key user@host 'echo connected'
```

### Permission Errors

```yaml
- name: Fix permissions
  uses: appleboy/ssh-action@v1
  with:
    script: |
      chown -R www-data:www-data /var/www/html/wp-content
      chmod -R 755 /var/www/html/wp-content
```

## Related Documents

- @howto-git-wordpress-workflow.md
- @ref-wp-cli-commands.md
- @howto-docker-wordpress.md
