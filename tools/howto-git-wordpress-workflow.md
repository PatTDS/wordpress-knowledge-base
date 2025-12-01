---
title: Git Workflow for WordPress Development
category: tools
type: howto
updated: 2025-12-01
version: 1.0.0
tags:
  - git
  - version-control
  - workflow
  - deployment
sources:
  - https://wpengine.com/support/development-workflow-best-practices/
  - https://pressable.com/blog/wordpress-github-workflows/
  - https://belovdigital.agency/blog/wordpress-development-with-git-version-control-best-practices/
  - https://infinum.com/handbook/wordpress/how-we-do-it-in-wordpress/git-workflow
---

# Git Workflow for WordPress Development

Version control best practices for WordPress theme and plugin development.

## Repository Setup

### Initialize Repository

```bash
cd wordpress-project
git init
```

### Recommended .gitignore

```gitignore
# WordPress core (don't track)
/wordpress/
/wp-admin/
/wp-includes/
/index.php
/license.txt
/readme.html
/wp-*.php
!/wp-config-sample.php
/xmlrpc.php

# Configuration (sensitive)
wp-config.php
.htaccess

# User uploads (large files)
/wp-content/uploads/
/wp-content/upgrade/
/wp-content/backup-db/
/wp-content/backups/
/wp-content/blogs.dir/
/wp-content/cache/
/wp-content/advanced-cache.php
/wp-content/wp-cache-config.php
/wp-content/debug.log

# Plugins (track only custom)
/wp-content/plugins/*
!/wp-content/plugins/my-custom-plugin/

# Themes (track only custom)
/wp-content/themes/*
!/wp-content/themes/my-custom-theme/
!/wp-content/themes/my-custom-theme-child/

# Dependencies
node_modules/
vendor/

# Build artifacts
*.map
*.min.css
*.min.js
/dist/

# Environment
.env
.env.*
!.env.example

# IDE
.vscode/
.idea/
*.sublime-*

# OS
.DS_Store
Thumbs.db

# Logs
*.log
error_log

# Local dev
.local/
*.sql
*.sql.gz
```

## Branch Strategy

### Branch Types

| Branch | Purpose | Lifespan |
|--------|---------|----------|
| `main` | Production-ready | Permanent |
| `develop` | Integration | Permanent |
| `staging` | Pre-production testing | Permanent |
| `feature/*` | New features | Short (1-3 days) |
| `bugfix/*` | Bug fixes | Short |
| `hotfix/*` | Emergency fixes | Very short |

### Branch Flow

```
main ─────────────────────────────────────────►
  │                                    ▲
  │                                    │ (merge)
  ▼                                    │
staging ──────────────────────────────►│
  │                              ▲     │
  │                              │     │
  ▼                              │     │
develop ─────────────────────────►     │
  │           ▲           ▲            │
  │           │           │            │
  ▼           │           │            │
feature/auth ─┘           │            │
                          │            │
feature/seo ──────────────┘            │
                                       │
hotfix/critical ───────────────────────┘
```

### Create Branches

```bash
# Feature branch
git checkout develop
git pull origin develop
git checkout -b feature/user-authentication

# Bugfix branch
git checkout develop
git checkout -b bugfix/login-redirect

# Hotfix (from main)
git checkout main
git checkout -b hotfix/security-patch
```

## Workflow Steps

### 1. Start New Feature

```bash
# Update develop
git checkout develop
git pull origin develop

# Create feature branch
git checkout -b feature/new-feature

# Work on feature
# ... make changes ...

# Commit frequently
git add .
git commit -m "feat(auth): add login form"
```

### 2. Keep Branch Updated

```bash
# Regularly pull changes from develop
git checkout develop
git pull origin develop
git checkout feature/new-feature
git merge develop

# Or rebase (cleaner history)
git rebase develop
```

### 3. Create Pull Request

```bash
# Push branch
git push -u origin feature/new-feature

# Create PR via GitHub/GitLab
# Request review from team
```

### 4. Merge to Develop

```bash
# After approval, merge via PR (recommended)
# Or manually:
git checkout develop
git merge --no-ff feature/new-feature
git push origin develop

# Delete feature branch
git branch -d feature/new-feature
git push origin --delete feature/new-feature
```

### 5. Deploy to Staging

```bash
git checkout staging
git merge develop
git push origin staging

# Automated deployment triggers
```

### 6. Deploy to Production

```bash
git checkout main
git merge staging
git tag -a v1.2.0 -m "Release 1.2.0"
git push origin main --tags
```

## Commit Message Standards

### Format

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

### Types

| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation |
| `style` | Formatting (no code change) |
| `refactor` | Code restructure |
| `test` | Add/fix tests |
| `chore` | Maintenance |

### Examples

```bash
# Feature
git commit -m "feat(theme): add dark mode toggle"

# Bug fix
git commit -m "fix(plugin): correct form validation on checkout"

# Documentation
git commit -m "docs: update installation instructions"

# Breaking change
git commit -m "feat(api): change authentication endpoint

BREAKING CHANGE: /auth/login now requires API key header"
```

## Environment Management

### Config Per Environment

Create `.env.example`:

```bash
# Database
DB_NAME=wordpress
DB_USER=root
DB_PASSWORD=
DB_HOST=localhost

# WordPress
WP_ENV=development
WP_HOME=http://localhost:8080
WP_SITEURL=${WP_HOME}/wp

# Security
AUTH_KEY=
SECURE_AUTH_KEY=
LOGGED_IN_KEY=
NONCE_KEY=
```

### Database Workflow

**Critical Rule**: Code moves UP, database moves DOWN.

```bash
# Export production database
wp @production db export --add-drop-table | gzip > prod-$(date +%Y%m%d).sql.gz

# Import to local
gunzip -c prod-20251201.sql.gz | wp db import -
wp search-replace 'https://production.com' 'http://localhost:8080' --skip-columns=guid
```

### Never Track Databases

```bash
# Add to .gitignore
*.sql
*.sql.gz
```

## Pull Request Workflow

### PR Template

Create `.github/pull_request_template.md`:

```markdown
## Description
<!-- What does this PR do? -->

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation

## Testing
<!-- How was this tested? -->

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-reviewed my code
- [ ] Added necessary documentation
- [ ] No new warnings
- [ ] Tests pass locally
```

### Code Review Guidelines

1. All PRs require at least one approval
2. No self-merging to main/develop
3. Address all comments before merge
4. Squash commits if requested

## Hotfix Procedure

### Emergency Fix

```bash
# 1. Create hotfix from main
git checkout main
git pull origin main
git checkout -b hotfix/security-fix

# 2. Make fix
# ... fix issue ...
git commit -m "fix(security): patch XSS vulnerability"

# 3. Merge to main
git checkout main
git merge --no-ff hotfix/security-fix
git tag -a v1.2.1 -m "Security patch"
git push origin main --tags

# 4. Backport to develop
git checkout develop
git merge hotfix/security-fix
git push origin develop

# 5. Cleanup
git branch -d hotfix/security-fix
```

## Useful Git Commands

### Daily Operations

```bash
# Status
git status

# View changes
git diff

# Stage all
git add .

# Commit
git commit -m "message"

# Push
git push origin branch-name

# Pull with rebase
git pull --rebase
```

### History & Logs

```bash
# Pretty log
git log --oneline --graph --decorate

# Last 10 commits
git log -10 --oneline

# Changes in file
git log -p filename

# Who changed what
git blame filename
```

### Undo Operations

```bash
# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo staged changes
git reset HEAD filename

# Discard local changes
git checkout -- filename

# Revert pushed commit
git revert commit-hash
```

### Stashing

```bash
# Stash changes
git stash

# Stash with name
git stash push -m "work in progress"

# List stashes
git stash list

# Apply stash
git stash pop

# Apply specific stash
git stash apply stash@{2}
```

## GitHub Integration

### Actions for WordPress

```yaml
# .github/workflows/test.yml
name: Test

on:
  pull_request:
    branches: [develop, main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'

      - name: Install dependencies
        run: composer install

      - name: Run PHP CodeSniffer
        run: vendor/bin/phpcs

      - name: Run PHPStan
        run: vendor/bin/phpstan analyse
```

### Deploy Workflow

See @howto-deployment-automation.md for complete CI/CD setup.

## Best Practices

### DO

- Commit frequently with clear messages
- Keep branches short-lived (1-3 days)
- Pull changes regularly
- Use PR for all merges
- Tag releases
- Review code before merging

### DON'T

- Commit directly to main
- Force push to shared branches
- Commit sensitive data
- Track generated files
- Merge without review
- Keep long-running branches

## Related Documents

- @howto-deployment-automation.md
- @ref-wp-cli-commands.md
- @howto-development-environment.md
