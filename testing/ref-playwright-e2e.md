---
title: Playwright E2E Testing Reference
description: Complete reference for Playwright end-to-end testing features, API, and configuration
category: testing
type: reference
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - playwright
  - e2e-testing
  - automation
  - cross-browser
sources:
  - https://playwright.dev/docs/best-practices
  - https://developer.wordpress.org/block-editor/contributors/code/testing-overview/e2e/
  - https://www.deviqa.com/blog/guide-to-playwright-end-to-end-testing-in-2025/
  - https://dev.to/bugslayer/building-a-comprehensive-e2e-test-suite-with-playwright-lessons-from-100-test-cases-171k
---

# Playwright E2E Testing Reference

## Overview

Playwright is a Microsoft-backed end-to-end testing framework that enables testing across Chromium, Firefox, and WebKit with a single test suite. Built specifically to address issues with brittle tests and slow run times.

## Core Features

### Cross-Browser Support

| Browser | Engine | Support |
|---------|--------|---------|
| Chrome | Chromium | Full |
| Edge | Chromium | Full |
| Firefox | Gecko | Full |
| Safari | WebKit | Full |

### Auto-Waiting

Playwright automatically waits for elements before performing actions:

```javascript
// No manual waits needed - Playwright handles this
await page.click('#submit-button');
await page.fill('#email', 'test@example.com');
```

**Key insight**: You don't "wait with an expect." Playwright's auto-waiting already performs actionability checks before interactions.

### Test Isolation

Each test runs in a fresh browser context by default, ensuring complete isolation.

## Configuration Reference

### playwright.config.ts

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',

  // Run tests in parallel
  fullyParallel: true,

  // Fail the build on CI if you accidentally left test.only
  forbidOnly: !!process.env.CI,

  // Retry on CI only
  retries: process.env.CI ? 2 : 0,

  // Opt out of parallel tests on CI
  workers: process.env.CI ? 1 : undefined,

  // Reporter configuration
  reporter: 'html',

  use: {
    // Base URL for all pages
    baseURL: 'http://localhost:8080',

    // Collect trace when retrying the failed test
    trace: 'on-first-retry',

    // Screenshots only on failure
    screenshot: 'only-on-failure',

    // Video retention on failure
    video: 'retain-on-failure',
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
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
  ],

  // Run local dev server before starting tests
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:8080',
    reuseExistingServer: !process.env.CI,
  },
});
```

### Recommended Timeouts

| Setting | Development | CI |
|---------|-------------|-----|
| Test timeout | 30000ms | 30000ms |
| Expect timeout | 5000ms | 5000ms |
| Navigation timeout | 30000ms | 60000ms |
| Action timeout | 10000ms | 15000ms |

## Locator Strategies

### Priority Order (Best Practices)

1. **Role selectors** (highest priority)
2. **Text selectors**
3. **Test IDs**
4. **CSS selectors** (use sparingly)

### Role Selectors (Recommended)

```javascript
// Best practice - accessible queries
await page.getByRole('button', { name: 'Submit' }).click();
await page.getByRole('textbox', { name: 'Email' }).fill('test@example.com');
await page.getByRole('heading', { name: 'Welcome' });
await page.getByRole('link', { name: 'Learn more' });
```

### Text Selectors

```javascript
await page.getByText('Submit form');
await page.getByLabel('Password');
await page.getByPlaceholder('Enter email');
await page.getByAltText('Company logo');
await page.getByTitle('Close dialog');
```

### Test IDs

```javascript
// When semantic selectors aren't suitable
await page.getByTestId('submit-button').click();
```

### CSS Selectors (Last Resort)

```javascript
// Avoid when possible - brittle
await page.locator('.submit-btn').click();
await page.locator('#contact-form').fill('...');
```

## Assertions Reference

### Element Assertions

```javascript
// Visibility
await expect(page.locator('#dialog')).toBeVisible();
await expect(page.locator('#loading')).toBeHidden();

// Content
await expect(page.locator('h1')).toHaveText('Welcome');
await expect(page.locator('h1')).toContainText('Welcome');

// Attributes
await expect(page.locator('input')).toHaveAttribute('type', 'email');
await expect(page.locator('input')).toBeEnabled();
await expect(page.locator('input')).toBeDisabled();

// Form state
await expect(page.locator('input')).toHaveValue('test@example.com');
await expect(page.locator('input[type=checkbox]')).toBeChecked();

// Count
await expect(page.locator('.item')).toHaveCount(5);
```

### Page Assertions

```javascript
await expect(page).toHaveTitle(/Welcome/);
await expect(page).toHaveURL('http://localhost:8080/dashboard');
await expect(page).toHaveURL(/.*dashboard/);
```

## Page Object Model (POM)

### Implementation

```typescript
// pages/LoginPage.ts
import { Page, Locator, expect } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.getByRole('textbox', { name: 'Email' });
    this.passwordInput = page.getByRole('textbox', { name: 'Password' });
    this.submitButton = page.getByRole('button', { name: 'Sign in' });
    this.errorMessage = page.getByRole('alert');
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async expectError(message: string) {
    await expect(this.errorMessage).toContainText(message);
  }
}
```

### Usage in Tests

```typescript
import { test, expect } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';

test('successful login', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login('user@example.com', 'password123');
  await expect(page).toHaveURL('/dashboard');
});
```

## Fixtures Reference

### Built-in Fixtures

| Fixture | Description |
|---------|-------------|
| `page` | Isolated page for the test |
| `context` | Isolated browser context |
| `browser` | Shared browser instance |
| `browserName` | Current browser name |
| `request` | API testing context |

### Custom Fixtures

```typescript
import { test as base } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';

type MyFixtures = {
  loginPage: LoginPage;
  authenticatedPage: Page;
};

export const test = base.extend<MyFixtures>({
  loginPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await use(loginPage);
  },

  authenticatedPage: async ({ page }, use) => {
    await page.goto('/login');
    await page.fill('[name=email]', 'admin@test.com');
    await page.fill('[name=password]', 'admin123');
    await page.click('[type=submit]');
    await page.waitForURL('/dashboard');
    await use(page);
  },
});
```

## API Testing

### Request Context

```typescript
import { test, expect } from '@playwright/test';

test('API test', async ({ request }) => {
  // GET request
  const response = await request.get('/api/users');
  expect(response.ok()).toBeTruthy();

  // POST request
  const createResponse = await request.post('/api/users', {
    data: {
      name: 'John Doe',
      email: 'john@example.com'
    }
  });
  expect(createResponse.status()).toBe(201);

  // Parse JSON response
  const user = await createResponse.json();
  expect(user.name).toBe('John Doe');
});
```

## Debugging Tools

### Visual Tools

```bash
# Debug mode with UI
npx playwright test --debug

# UI mode (interactive)
npx playwright test --ui

# Show browser (headed mode)
npx playwright test --headed
```

### Trace Viewer

```bash
# Generate trace on failure
npx playwright test --trace on

# View trace
npx playwright show-trace trace.zip
```

### Code Generation

```bash
# Record actions and generate code
npx playwright codegen http://localhost:8080
```

## CLI Commands

| Command | Description |
|---------|-------------|
| `npx playwright test` | Run all tests |
| `npx playwright test tests/login.spec.ts` | Run specific file |
| `npx playwright test --grep "login"` | Run tests matching pattern |
| `npx playwright test --project=chromium` | Run on specific browser |
| `npx playwright test --workers=4` | Parallel execution |
| `npx playwright show-report` | View HTML report |
| `npx playwright install` | Install browsers |

## Performance Considerations

### Speed Optimization

1. **Use API for setup** - Set state via API, not UI
2. **Reuse authentication** - Store auth state, reuse across tests
3. **Parallel execution** - Run tests in parallel
4. **Minimize browser launches** - Use contexts instead

### Authentication Reuse

```typescript
// global-setup.ts
import { chromium } from '@playwright/test';

async function globalSetup() {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  await page.goto('http://localhost:8080/login');
  await page.fill('#email', 'admin@test.com');
  await page.fill('#password', 'admin123');
  await page.click('button[type=submit]');
  await page.context().storageState({ path: 'auth.json' });
  await browser.close();
}

export default globalSetup;
```

## Related Documents

- howto-playwright-wordpress.md - WordPress-specific setup
- ref-visual-regression.md - Visual comparison testing
- ref-accessibility-testing.md - Accessibility integration
