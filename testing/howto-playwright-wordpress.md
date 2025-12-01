---
title: How to Set Up Playwright E2E Testing for WordPress
description: Step-by-step guide to configure Playwright for WordPress end-to-end testing
category: testing
type: howto
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - playwright
  - wordpress
  - e2e-testing
  - testing-setup
prerequisites:
  - Node.js 18+
  - WordPress development environment
  - Basic understanding of E2E testing
sources:
  - https://vapvarun.com/complete-guide-setting-up-playwright-e2e-testing-with-local-wordpress/
  - https://developer.wordpress.org/block-editor/reference-guides/packages/packages-e2e-test-utils-playwright/
  - https://medium.com/@tetsuaki.hamano/introducing-e2e-testing-to-wordpress-block-development-43efce96a806
  - https://mathieulamiot.com/talk/ai-driven-qa-how-playwright-mcp-facilitates-wordpress-e2e-testing/
---

# How to Set Up Playwright E2E Testing for WordPress

## Goal

Configure Playwright to run end-to-end tests against a local WordPress development environment.

## Prerequisites

- Node.js 18+ installed
- WordPress running locally (Docker or Local by Flywheel)
- npm or yarn package manager

## Step 1: Install Playwright

```bash
# Navigate to your project root
cd /path/to/wordpress-project

# Initialize Playwright
npm init playwright@latest

# Answer the prompts:
# - TypeScript or JavaScript? (TypeScript recommended)
# - Where to put tests? tests/e2e
# - Add GitHub Actions workflow? Yes
# - Install Playwright browsers? Yes
```

## Step 2: Install WordPress Test Utilities

For WordPress-specific testing, install the official package:

```bash
npm install --save-dev @wordpress/e2e-test-utils-playwright @wordpress/scripts
```

## Step 3: Configure Playwright for WordPress

Create or update `playwright.config.ts`:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',

  // Increase timeout for WordPress operations
  timeout: 30000,
  expect: {
    timeout: 10000,
  },

  // Run tests in files in parallel
  fullyParallel: true,

  // Retry on CI only
  retries: process.env.CI ? 2 : 0,

  // Workers - reduce on CI for stability
  workers: process.env.CI ? 1 : undefined,

  reporter: [
    ['html'],
    ['list'],
  ],

  use: {
    // WordPress local URL
    baseURL: process.env.WP_BASE_URL || 'http://localhost:8080',

    // Capture trace on first retry
    trace: 'on-first-retry',

    // Screenshot on failure only
    screenshot: 'only-on-failure',

    // Retain video on failure
    video: 'retain-on-failure',

    // Viewport for WordPress admin
    viewport: { width: 1280, height: 720 },
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'mobile',
      use: { ...devices['iPhone 12'] },
    },
  ],

  // Start WordPress if needed
  webServer: {
    command: 'docker-compose up -d',
    url: 'http://localhost:8080',
    reuseExistingServer: true,
    timeout: 120000,
  },
});
```

## Step 4: Create WordPress Authentication Setup

Create `tests/e2e/auth.setup.ts`:

```typescript
import { test as setup, expect } from '@playwright/test';
import path from 'path';

const authFile = path.join(__dirname, '.auth/user.json');

setup('authenticate as admin', async ({ page }) => {
  // Navigate to WordPress login
  await page.goto('/wp-login.php');

  // Fill login credentials
  await page.fill('#user_login', process.env.WP_ADMIN_USER || 'admin');
  await page.fill('#user_pass', process.env.WP_ADMIN_PASS || 'password');

  // Submit login form
  await page.click('#wp-submit');

  // Wait for dashboard
  await expect(page).toHaveURL(/wp-admin/);
  await expect(page.locator('#wpadminbar')).toBeVisible();

  // Save authentication state
  await page.context().storageState({ path: authFile });
});
```

Update `playwright.config.ts` to use authentication:

```typescript
export default defineConfig({
  // ... other config

  projects: [
    // Setup project for authentication
    { name: 'setup', testMatch: /.*\.setup\.ts/ },

    {
      name: 'chromium',
      use: {
        ...devices['Desktop Chrome'],
        storageState: 'tests/e2e/.auth/user.json',
      },
      dependencies: ['setup'],
    },
    // ... other browsers
  ],
});
```

## Step 5: Create Page Objects for WordPress

Create `tests/e2e/pages/WordPressAdmin.ts`:

```typescript
import { Page, Locator, expect } from '@playwright/test';

export class WordPressAdmin {
  readonly page: Page;
  readonly adminBar: Locator;
  readonly dashboardMenu: Locator;
  readonly postsMenu: Locator;
  readonly pagesMenu: Locator;
  readonly pluginsMenu: Locator;

  constructor(page: Page) {
    this.page = page;
    this.adminBar = page.locator('#wpadminbar');
    this.dashboardMenu = page.locator('#menu-dashboard');
    this.postsMenu = page.locator('#menu-posts');
    this.pagesMenu = page.locator('#menu-pages');
    this.pluginsMenu = page.locator('#menu-plugins');
  }

  async goto() {
    await this.page.goto('/wp-admin/');
    await expect(this.adminBar).toBeVisible();
  }

  async navigateToPosts() {
    await this.postsMenu.click();
    await expect(this.page).toHaveURL(/edit\.php$/);
  }

  async navigateToPages() {
    await this.pagesMenu.click();
    await expect(this.page).toHaveURL(/edit\.php\?post_type=page/);
  }

  async navigateToPlugins() {
    await this.pluginsMenu.click();
    await expect(this.page).toHaveURL(/plugins\.php/);
  }

  async createNewPost(title: string, content: string) {
    await this.navigateToPosts();
    await this.page.click('.page-title-action');

    // Block editor
    await this.page.waitForSelector('.editor-post-title__input');
    await this.page.fill('.editor-post-title__input', title);

    // Add content block
    await this.page.keyboard.press('Enter');
    await this.page.keyboard.type(content);

    // Publish
    await this.page.click('.editor-post-publish-button__button');
    await this.page.click('.editor-post-publish-button__button');

    // Confirm published
    await expect(this.page.locator('.post-publish-panel__postpublish-header')).toBeVisible();
  }
}
```

## Step 6: Write WordPress E2E Tests

Create `tests/e2e/wordpress-admin.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';
import { WordPressAdmin } from './pages/WordPressAdmin';

test.describe('WordPress Admin', () => {
  let admin: WordPressAdmin;

  test.beforeEach(async ({ page }) => {
    admin = new WordPressAdmin(page);
    await admin.goto();
  });

  test('should display admin dashboard', async ({ page }) => {
    await expect(page.locator('#dashboard-widgets')).toBeVisible();
    await expect(page.locator('.wrap h1')).toContainText('Dashboard');
  });

  test('should navigate to posts', async ({ page }) => {
    await admin.navigateToPosts();
    await expect(page.locator('.wrap h1')).toContainText('Posts');
  });

  test('should navigate to pages', async ({ page }) => {
    await admin.navigateToPages();
    await expect(page.locator('.wrap h1')).toContainText('Pages');
  });

  test('should access plugin settings', async ({ page }) => {
    await admin.navigateToPlugins();
    await expect(page.locator('.wrap h1')).toContainText('Plugins');
  });
});
```

Create frontend tests `tests/e2e/frontend.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Frontend', () => {
  test('homepage loads correctly', async ({ page }) => {
    await page.goto('/');

    // Check header
    await expect(page.locator('header')).toBeVisible();

    // Check navigation
    await expect(page.locator('nav')).toBeVisible();

    // Check footer
    await expect(page.locator('footer')).toBeVisible();
  });

  test('navigation works', async ({ page }) => {
    await page.goto('/');

    // Click on a menu item
    await page.click('nav a:has-text("About")');

    // Verify navigation
    await expect(page).toHaveURL(/about/);
  });

  test('contact form displays', async ({ page }) => {
    await page.goto('/contact');

    // Check form elements
    await expect(page.getByRole('textbox', { name: /name/i })).toBeVisible();
    await expect(page.getByRole('textbox', { name: /email/i })).toBeVisible();
    await expect(page.getByRole('button', { name: /submit/i })).toBeVisible();
  });

  test('search functionality works', async ({ page }) => {
    await page.goto('/');

    // Open search
    await page.click('[aria-label="Search"]');

    // Enter search query
    await page.fill('input[type="search"]', 'test post');
    await page.keyboard.press('Enter');

    // Verify search results page
    await expect(page).toHaveURL(/s=test\+post/);
  });
});
```

## Step 7: Add npm Scripts

Update `package.json`:

```json
{
  "scripts": {
    "test": "playwright test",
    "test:ui": "playwright test --ui",
    "test:debug": "playwright test --debug",
    "test:headed": "playwright test --headed",
    "test:chromium": "playwright test --project=chromium",
    "test:report": "playwright show-report",
    "test:codegen": "playwright codegen http://localhost:8080"
  }
}
```

## Step 8: Create Environment Configuration

Create `.env.example`:

```bash
# WordPress Test Environment
WP_BASE_URL=http://localhost:8080
WP_ADMIN_USER=admin
WP_ADMIN_PASS=password

# CI Configuration
CI=false
```

Create `.env` (add to .gitignore):

```bash
WP_BASE_URL=http://localhost:8080
WP_ADMIN_USER=admin
WP_ADMIN_PASS=your-secure-password
```

## Step 9: Add GitHub Actions Workflow

Create `.github/workflows/playwright.yml`:

```yaml
name: Playwright Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest

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

      - name: Run Playwright tests
        run: npx playwright test
        env:
          WP_BASE_URL: http://localhost:8080
          WP_ADMIN_USER: admin
          WP_ADMIN_PASS: password

      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

## Verification

Run tests to verify setup:

```bash
# Run all tests
npm test

# Run with UI
npm run test:ui

# Generate code (interactive)
npm run test:codegen
```

## Troubleshooting

### Tests timeout waiting for WordPress

Increase timeout in `playwright.config.ts`:

```typescript
export default defineConfig({
  timeout: 60000, // 60 seconds
  expect: {
    timeout: 15000,
  },
});
```

### Authentication fails

1. Verify WordPress is running: `curl http://localhost:8080`
2. Check credentials in `.env`
3. Clear auth state: `rm -rf tests/e2e/.auth/`

### Element not found

Use Playwright's codegen to find correct selectors:

```bash
npx playwright codegen http://localhost:8080/wp-admin
```

### Docker networking issues

Use host network mode in GitHub Actions or configure proper service networking.

## Related Documents

- ref-playwright-e2e.md - Complete Playwright reference
- howto-lighthouse-ci.md - Performance testing integration
- ref-github-actions-wordpress.md - CI/CD configuration
