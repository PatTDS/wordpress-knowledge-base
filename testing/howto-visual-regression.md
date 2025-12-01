---
title: How to Implement Visual Regression Testing
description: Step-by-step guide to set up visual regression testing for WordPress projects
category: testing
type: howto
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - visual-regression
  - playwright
  - backstopjs
  - testing
prerequisites:
  - Node.js 18+
  - WordPress development environment
  - Basic E2E testing knowledge
sources:
  - https://testingplus.me/visual-regression-playwright-testing-part-one/
  - https://tuutti.iki.fi/2024/04/20/visual-regression-testing-with-backstopjs-part-1/
  - https://testquality.com/visual-regression-testing-playwright-jest/
---

# How to Implement Visual Regression Testing

## Goal

Set up automated visual regression testing to catch unintended UI changes before they reach production.

## Option A: Playwright Built-in (Recommended for Simplicity)

### Step 1: Configure Playwright for Visual Testing

Update `playwright.config.ts`:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/visual',

  expect: {
    toHaveScreenshot: {
      maxDiffPixelRatio: 0.02,
      threshold: 0.2,
      animations: 'disabled',
      caret: 'hide',
    },
  },

  updateSnapshots: 'missing',

  use: {
    baseURL: 'http://localhost:8080',
  },

  projects: [
    {
      name: 'Desktop Chrome',
      use: {
        ...devices['Desktop Chrome'],
        viewport: { width: 1920, height: 1080 },
      },
    },
    {
      name: 'Mobile',
      use: {
        ...devices['iPhone 12'],
      },
    },
  ],
});
```

### Step 2: Create Visual Test Files

Create `tests/visual/homepage.visual.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Homepage Visual Tests', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
    // Wait for all images to load
    await page.waitForLoadState('networkidle');
    // Wait for fonts
    await page.waitForFunction(() => document.fonts.ready);
  });

  test('full page screenshot', async ({ page }) => {
    await expect(page).toHaveScreenshot('homepage-full.png', {
      fullPage: true,
    });
  });

  test('hero section', async ({ page }) => {
    const hero = page.locator('.hero-section');
    await expect(hero).toHaveScreenshot('homepage-hero.png');
  });

  test('navigation', async ({ page }) => {
    const nav = page.locator('header nav');
    await expect(nav).toHaveScreenshot('navigation.png');
  });

  test('footer', async ({ page }) => {
    const footer = page.locator('footer');
    await expect(footer).toHaveScreenshot('footer.png');
  });
});
```

### Step 3: Handle Dynamic Content

Create `tests/visual/helpers.ts`:

```typescript
import { Page } from '@playwright/test';

export async function preparePage(page: Page) {
  // Wait for network idle
  await page.waitForLoadState('networkidle');

  // Wait for fonts
  await page.waitForFunction(() => document.fonts.ready);

  // Hide dynamic content
  await page.addStyleTag({
    content: `
      .timestamp, .date { visibility: hidden !important; }
      .ad-banner { display: none !important; }
      [data-dynamic] { opacity: 0 !important; }
      .animated { animation: none !important; }
    `,
  });

  // Disable animations
  await page.evaluate(() => {
    const style = document.createElement('style');
    style.textContent = `
      *, *::before, *::after {
        animation-duration: 0s !important;
        transition-duration: 0s !important;
      }
    `;
    document.head.appendChild(style);
  });
}
```

### Step 4: Create Page-Specific Tests

Create `tests/visual/pages.visual.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';
import { preparePage } from './helpers';

const pages = [
  { name: 'About', path: '/about' },
  { name: 'Services', path: '/services' },
  { name: 'Contact', path: '/contact' },
  { name: 'Blog', path: '/blog' },
];

for (const pageInfo of pages) {
  test(`${pageInfo.name} page visual test`, async ({ page }) => {
    await page.goto(pageInfo.path);
    await preparePage(page);

    await expect(page).toHaveScreenshot(`${pageInfo.name.toLowerCase()}-page.png`, {
      fullPage: true,
    });
  });
}
```

### Step 5: Run and Update Baselines

```bash
# First run - creates baseline screenshots
npx playwright test tests/visual/

# Update baselines after intentional changes
npx playwright test tests/visual/ --update-snapshots

# Run specific visual test
npx playwright test tests/visual/homepage.visual.spec.ts
```

## Option B: BackstopJS (Advanced Control)

### Step 1: Install BackstopJS

```bash
npm install --save-dev backstopjs
npx backstop init
```

### Step 2: Configure BackstopJS

Edit `backstop.json`:

```json
{
  "id": "wordpress_visual_tests",
  "viewports": [
    { "label": "mobile", "width": 375, "height": 812 },
    { "label": "tablet", "width": 768, "height": 1024 },
    { "label": "desktop", "width": 1920, "height": 1080 }
  ],
  "scenarios": [
    {
      "label": "Homepage",
      "url": "http://localhost:8080",
      "readySelector": ".site-content",
      "delay": 1000,
      "removeSelectors": [".ad-banner", ".dynamic-widget"],
      "hideSelectors": [".timestamp"],
      "misMatchThreshold": 0.1
    },
    {
      "label": "About Page",
      "url": "http://localhost:8080/about",
      "readySelector": ".page-content",
      "delay": 500,
      "misMatchThreshold": 0.1
    },
    {
      "label": "Services",
      "url": "http://localhost:8080/services",
      "readySelector": ".services-grid",
      "delay": 500,
      "misMatchThreshold": 0.1
    },
    {
      "label": "Contact Form",
      "url": "http://localhost:8080/contact",
      "readySelector": "form",
      "delay": 500,
      "clickSelector": "",
      "misMatchThreshold": 0.1
    },
    {
      "label": "Navigation Hover",
      "url": "http://localhost:8080",
      "readySelector": "nav",
      "hoverSelector": "nav .menu-item:first-child",
      "postInteractionWait": 300,
      "selectors": ["nav"],
      "misMatchThreshold": 0.05
    }
  ],
  "paths": {
    "bitmaps_reference": "backstop_data/bitmaps_reference",
    "bitmaps_test": "backstop_data/bitmaps_test",
    "html_report": "backstop_data/html_report",
    "ci_report": "backstop_data/ci_report"
  },
  "engine": "playwright",
  "engineOptions": {
    "browser": "chromium",
    "args": ["--no-sandbox", "--disable-setuid-sandbox"]
  },
  "asyncCaptureLimit": 5,
  "asyncCompareLimit": 50,
  "report": ["browser", "CI"],
  "debug": false
}
```

### Step 3: Add Custom Scripts

Create `backstop_data/engine_scripts/playwright/onReady.js`:

```javascript
module.exports = async (page, scenario, viewport) => {
  // Wait for fonts
  await page.waitForFunction(() => document.fonts.ready);

  // Wait for images
  await page.waitForFunction(() => {
    const images = document.querySelectorAll('img');
    return Array.from(images).every(img => img.complete);
  });

  // Disable animations
  await page.addStyleTag({
    content: `
      *, *::before, *::after {
        animation-duration: 0s !important;
        animation-delay: 0s !important;
        transition-duration: 0s !important;
        transition-delay: 0s !important;
      }
    `,
  });

  // Handle lazy-loaded content
  await page.evaluate(() => {
    window.scrollTo(0, document.body.scrollHeight);
  });
  await page.waitForTimeout(500);
  await page.evaluate(() => {
    window.scrollTo(0, 0);
  });
};
```

### Step 4: Run BackstopJS

```bash
# Create reference images (baseline)
npx backstop reference

# Run test comparison
npx backstop test

# Approve current test as new reference
npx backstop approve

# Open HTML report
npx backstop openReport
```

### Step 5: Add npm Scripts

Update `package.json`:

```json
{
  "scripts": {
    "visual:reference": "backstop reference",
    "visual:test": "backstop test",
    "visual:approve": "backstop approve",
    "visual:report": "backstop openReport"
  }
}
```

## Step 6: CI/CD Integration

### GitHub Actions Workflow

Create `.github/workflows/visual-regression.yml`:

```yaml
name: Visual Regression Tests

on:
  pull_request:
    branches: [main, develop]

jobs:
  visual-test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install chromium --with-deps

      - name: Start WordPress
        run: |
          docker-compose up -d
          sleep 30

      - name: Run visual tests
        run: npx playwright test tests/visual/

      - name: Upload diff artifacts
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: visual-diff-report
          path: |
            playwright-report/
            test-results/
          retention-days: 7

      - name: Comment on PR
        if: failure()
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '⚠️ Visual regression tests failed. Please check the artifacts for diff images.'
            })
```

## Best Practices

### Reduce Flakiness

1. **Wait for stability**
   - Network idle
   - Font loading
   - Image loading
   - Animations complete

2. **Hide dynamic content**
   - Timestamps
   - Ads
   - Random content

3. **Use appropriate thresholds**
   - Start with 0.1% (0.001)
   - Adjust based on tolerance

### Organize Screenshots

```
tests/visual/
├── homepage.visual.spec.ts
├── pages.visual.spec.ts
├── components.visual.spec.ts
└── homepage.visual.spec.ts-snapshots/
    ├── homepage-full-Desktop-Chrome.png
    ├── homepage-full-Mobile.png
    ├── homepage-hero-Desktop-Chrome.png
    └── homepage-hero-Mobile.png
```

### Review Process

1. Run tests on every PR
2. Review diffs before approving
3. Document intentional changes
4. Update baselines explicitly

## Verification

```bash
# Run visual tests
npx playwright test tests/visual/

# Check for failures
echo $?  # 0 = passed, 1 = failed

# View report
npx playwright show-report
```

## Troubleshooting

### Screenshots differ on CI vs local

- Use Docker for consistent environment
- Match browser versions exactly
- Set fixed viewport sizes

### Fonts render differently

```typescript
// Wait for fonts explicitly
await page.waitForFunction(() => document.fonts.ready);
```

### Anti-aliasing differences

Increase threshold in config:

```typescript
expect: {
  toHaveScreenshot: {
    threshold: 0.3, // Higher tolerance
  },
},
```

## Related Documents

- ref-visual-regression.md - Tools reference
- ref-playwright-e2e.md - Playwright reference
- howto-playwright-wordpress.md - WordPress setup
