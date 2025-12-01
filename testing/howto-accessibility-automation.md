---
title: How to Automate Accessibility Testing
description: Step-by-step guide to implement automated accessibility testing with axe-core
category: testing
type: howto
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - accessibility
  - a11y
  - axe-core
  - automation
  - wcag
prerequisites:
  - Node.js 18+
  - Playwright or Cypress installed
  - Basic understanding of WCAG
sources:
  - https://playwright.dev/docs/accessibility-testing
  - https://lastcallmedia.com/blog/automated-accessibility-testing-axe-core-how-were-baking-a11y-every-build
  - https://hogonext.com/how-to-automate-accessibility-testing-with-axe-core/
  - https://www.testevolve.com/automated-axe-accessibility-checks
---

# How to Automate Accessibility Testing

## Goal

Implement automated accessibility testing that runs on every build to catch accessibility issues early in development.

## Step 1: Install Dependencies

```bash
# For Playwright
npm install --save-dev @axe-core/playwright

# For report generation
npm install --save-dev axe-html-reporter
```

## Step 2: Create Accessibility Test Utilities

Create `tests/a11y/axe-helper.ts`:

```typescript
import { Page } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

export interface A11yResult {
  violations: any[];
  passes: any[];
  incomplete: any[];
}

export async function runAccessibilityAudit(
  page: Page,
  options?: {
    include?: string[];
    exclude?: string[];
    tags?: string[];
    disableRules?: string[];
  }
): Promise<A11yResult> {
  let builder = new AxeBuilder({ page });

  // Apply WCAG 2.1 AA by default
  builder = builder.withTags(['wcag2a', 'wcag2aa', 'wcag21aa']);

  if (options?.tags) {
    builder = builder.withTags(options.tags);
  }

  if (options?.include) {
    options.include.forEach(selector => {
      builder = builder.include(selector);
    });
  }

  if (options?.exclude) {
    options.exclude.forEach(selector => {
      builder = builder.exclude(selector);
    });
  }

  if (options?.disableRules) {
    builder = builder.disableRules(options.disableRules);
  }

  const results = await builder.analyze();

  return {
    violations: results.violations,
    passes: results.passes,
    incomplete: results.incomplete,
  };
}

export function formatViolations(violations: any[]): string {
  if (violations.length === 0) {
    return 'No accessibility violations found.';
  }

  return violations.map(violation => {
    const nodes = violation.nodes.map((node: any) =>
      `  - ${node.target.join(', ')}: ${node.failureSummary}`
    ).join('\n');

    return `
[${violation.impact?.toUpperCase()}] ${violation.id}
${violation.help}
${violation.helpUrl}
Affected elements:
${nodes}
`;
  }).join('\n---\n');
}
```

## Step 3: Create Basic Accessibility Tests

Create `tests/a11y/accessibility.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';
import { runAccessibilityAudit, formatViolations } from './axe-helper';

test.describe('Accessibility Tests', () => {
  test('homepage meets WCAG 2.1 AA', async ({ page }) => {
    await page.goto('/');
    await page.waitForLoadState('networkidle');

    const results = await runAccessibilityAudit(page);

    if (results.violations.length > 0) {
      console.log('Accessibility Violations:');
      console.log(formatViolations(results.violations));
    }

    expect(results.violations).toHaveLength(0);
  });

  test('contact page meets WCAG 2.1 AA', async ({ page }) => {
    await page.goto('/contact');
    await page.waitForLoadState('networkidle');

    const results = await runAccessibilityAudit(page);

    expect(results.violations).toHaveLength(0);
  });

  test('blog listing meets WCAG 2.1 AA', async ({ page }) => {
    await page.goto('/blog');
    await page.waitForLoadState('networkidle');

    const results = await runAccessibilityAudit(page);

    expect(results.violations).toHaveLength(0);
  });
});
```

## Step 4: Test Critical User Journeys

Create `tests/a11y/user-journeys.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';
import { runAccessibilityAudit } from './axe-helper';

test.describe('User Journey Accessibility', () => {
  test('navigation flow', async ({ page }) => {
    // Homepage
    await page.goto('/');
    let results = await runAccessibilityAudit(page);
    expect(results.violations, 'Homepage').toHaveLength(0);

    // Navigate to about
    await page.click('nav a:has-text("About")');
    await page.waitForLoadState('networkidle');
    results = await runAccessibilityAudit(page);
    expect(results.violations, 'About page').toHaveLength(0);

    // Navigate to services
    await page.click('nav a:has-text("Services")');
    await page.waitForLoadState('networkidle');
    results = await runAccessibilityAudit(page);
    expect(results.violations, 'Services page').toHaveLength(0);
  });

  test('contact form interaction', async ({ page }) => {
    await page.goto('/contact');
    await page.waitForLoadState('networkidle');

    // Initial state
    let results = await runAccessibilityAudit(page);
    expect(results.violations, 'Initial form').toHaveLength(0);

    // Focus on form field
    await page.click('#name');
    results = await runAccessibilityAudit(page);
    expect(results.violations, 'Form with focus').toHaveLength(0);

    // Trigger validation
    await page.click('button[type="submit"]');
    await page.waitForTimeout(500); // Wait for validation
    results = await runAccessibilityAudit(page);
    expect(results.violations, 'Form with validation').toHaveLength(0);
  });

  test('modal dialog accessibility', async ({ page }) => {
    await page.goto('/');

    // Open modal (adjust selector as needed)
    const modalTrigger = page.locator('[data-modal-trigger]');
    if (await modalTrigger.isVisible()) {
      await modalTrigger.click();
      await page.waitForSelector('[role="dialog"]');

      const results = await runAccessibilityAudit(page, {
        include: ['[role="dialog"]'],
      });

      expect(results.violations).toHaveLength(0);
    }
  });
});
```

## Step 5: Component-Level Testing

Create `tests/a11y/components.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';
import { runAccessibilityAudit } from './axe-helper';

test.describe('Component Accessibility', () => {
  test('navigation component', async ({ page }) => {
    await page.goto('/');

    const results = await runAccessibilityAudit(page, {
      include: ['header nav'],
    });

    expect(results.violations).toHaveLength(0);
  });

  test('footer component', async ({ page }) => {
    await page.goto('/');

    const results = await runAccessibilityAudit(page, {
      include: ['footer'],
    });

    expect(results.violations).toHaveLength(0);
  });

  test('card components', async ({ page }) => {
    await page.goto('/services');

    const results = await runAccessibilityAudit(page, {
      include: ['.service-card'],
    });

    expect(results.violations).toHaveLength(0);
  });
});
```

## Step 6: Generate HTML Reports

Create `tests/a11y/report-generator.ts`:

```typescript
import { Page } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';
import { createHtmlReport } from 'axe-html-reporter';
import fs from 'fs';
import path from 'path';

export async function generateA11yReport(
  page: Page,
  pageName: string
): Promise<void> {
  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa', 'wcag21aa'])
    .analyze();

  const reportDir = path.join(process.cwd(), 'a11y-reports');
  if (!fs.existsSync(reportDir)) {
    fs.mkdirSync(reportDir, { recursive: true });
  }

  createHtmlReport({
    results,
    options: {
      outputDir: reportDir,
      reportFileName: `${pageName}-a11y-report.html`,
    },
  });

  // Also save JSON for programmatic access
  fs.writeFileSync(
    path.join(reportDir, `${pageName}-a11y-results.json`),
    JSON.stringify(results, null, 2)
  );
}
```

Update test to generate reports:

```typescript
import { test, expect } from '@playwright/test';
import { runAccessibilityAudit } from './axe-helper';
import { generateA11yReport } from './report-generator';

test('full site accessibility audit with reports', async ({ page }) => {
  const pages = [
    { name: 'homepage', url: '/' },
    { name: 'about', url: '/about' },
    { name: 'services', url: '/services' },
    { name: 'contact', url: '/contact' },
  ];

  const allViolations: any[] = [];

  for (const pageInfo of pages) {
    await page.goto(pageInfo.url);
    await page.waitForLoadState('networkidle');

    // Generate report
    await generateA11yReport(page, pageInfo.name);

    // Collect violations
    const results = await runAccessibilityAudit(page);
    if (results.violations.length > 0) {
      allViolations.push({
        page: pageInfo.name,
        violations: results.violations,
      });
    }
  }

  expect(allViolations).toHaveLength(0);
});
```

## Step 7: CI/CD Integration

Create `.github/workflows/accessibility.yml`:

```yaml
name: Accessibility Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  a11y-test:
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

      - name: Run accessibility tests
        run: npx playwright test tests/a11y/

      - name: Upload reports
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: accessibility-reports
          path: |
            a11y-reports/
            playwright-report/
          retention-days: 30

      - name: Comment on PR
        if: failure() && github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '⚠️ Accessibility tests failed. Please review the accessibility report in the workflow artifacts.'
            })
```

## Step 8: Create Threshold-Based Testing

For gradual adoption, allow some violations initially:

```typescript
import { test, expect } from '@playwright/test';
import { runAccessibilityAudit } from './axe-helper';

// Define acceptable thresholds during transition
const VIOLATION_THRESHOLDS = {
  critical: 0,  // Never allow critical
  serious: 0,   // Never allow serious
  moderate: 5,  // Allow up to 5 moderate (temporary)
  minor: 10,    // Allow up to 10 minor (temporary)
};

test('accessibility with thresholds', async ({ page }) => {
  await page.goto('/');
  await page.waitForLoadState('networkidle');

  const results = await runAccessibilityAudit(page);

  // Count by impact
  const counts = {
    critical: 0,
    serious: 0,
    moderate: 0,
    minor: 0,
  };

  for (const violation of results.violations) {
    const impact = violation.impact as keyof typeof counts;
    counts[impact]++;
  }

  // Check against thresholds
  expect(counts.critical, 'Critical violations').toBeLessThanOrEqual(
    VIOLATION_THRESHOLDS.critical
  );
  expect(counts.serious, 'Serious violations').toBeLessThanOrEqual(
    VIOLATION_THRESHOLDS.serious
  );
  expect(counts.moderate, 'Moderate violations').toBeLessThanOrEqual(
    VIOLATION_THRESHOLDS.moderate
  );
  expect(counts.minor, 'Minor violations').toBeLessThanOrEqual(
    VIOLATION_THRESHOLDS.minor
  );
});
```

## Step 9: Add npm Scripts

Update `package.json`:

```json
{
  "scripts": {
    "test:a11y": "playwright test tests/a11y/",
    "test:a11y:report": "playwright test tests/a11y/ && open a11y-reports/",
    "a11y:audit": "npx playwright test tests/a11y/accessibility.spec.ts"
  }
}
```

## Verification

```bash
# Run accessibility tests
npm run test:a11y

# Generate and view reports
npm run test:a11y:report

# Run specific test
npx playwright test tests/a11y/components.spec.ts
```

## Best Practices

### 1. Test Early and Often

- Run on every PR
- Block merges on critical/serious issues
- Review moderate/minor in reports

### 2. Combine with Manual Testing

Automated testing catches ~57% of issues. Supplement with:
- Screen reader testing
- Keyboard navigation testing
- User testing with disabled users

### 3. Incremental Adoption

- Start with blocking critical issues
- Gradually reduce thresholds
- Track improvement over time

### 4. Document Known Issues

```typescript
// Known issues to exclude temporarily
const knownIssues = {
  disableRules: ['color-contrast'],
  exclude: ['.third-party-widget'],
};
```

## Troubleshooting

### Tests timeout

Increase timeout for complex pages:

```typescript
test.setTimeout(60000);
await page.waitForLoadState('networkidle');
```

### False positives

axe-core is designed for zero false positives. If you think something is a false positive:

1. Check the latest axe-core version
2. Review the rule documentation
3. Report to axe-core if confirmed

### Third-party widget violations

Exclude third-party content you cannot control:

```typescript
const results = await runAccessibilityAudit(page, {
  exclude: ['.third-party-widget', 'iframe'],
});
```

## Related Documents

- ref-accessibility-testing.md - Accessibility reference
- ref-playwright-e2e.md - Playwright testing
- howto-lighthouse-ci.md - Lighthouse accessibility audits
