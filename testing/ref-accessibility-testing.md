---
title: Accessibility Testing Reference
description: Reference for accessibility testing tools, standards, and axe-core integration
category: testing
type: reference
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - accessibility
  - a11y
  - axe-core
  - wcag
  - aria
sources:
  - https://github.com/dequelabs/axe-core
  - https://www.deque.com/axe/axe-core/
  - https://playwright.dev/docs/accessibility-testing
  - https://www.npmjs.com/package/axe-core
  - https://medium.com/@SkorekM/from-theory-to-automation-wcag-compliance-using-axe-core-next-js-and-github-actions-b9f63af8e155
---

# Accessibility Testing Reference

## Overview

Accessibility testing ensures web content is usable by people with disabilities. axe-core is an open-source JavaScript library that powers most accessibility testing tools, including Google Lighthouse.

## axe-core Coverage

With axe-core, you can find on average **57% of WCAG issues automatically**. The remaining issues require manual testing or semi-automated (guided) testing.

## Testing Types

| Type | Description | Coverage |
|------|-------------|----------|
| Automated | Rules engine scans against WCAG | ~57% |
| Semi-automated | Guided testing with Q&A prompts | ~20% |
| Manual | Expert testing with assistive technology | ~23% |

## WCAG Standards Supported

axe-core supports different rule types:

| Standard | Level | Description |
|----------|-------|-------------|
| WCAG 2.0 | A | Minimum accessibility |
| WCAG 2.0 | AA | Target for most sites |
| WCAG 2.0 | AAA | Highest accessibility |
| WCAG 2.1 | A/AA/AAA | Mobile and cognitive additions |
| WCAG 2.2 | A/AA/AAA | Latest (2023) |

## axe-core Rule Categories

### Best Practices

Not WCAG violations but recommended:
- Every page should have an h1
- Avoid "gotchas" in ARIA usage
- Link text should be descriptive

### Critical Issues

Automatic failures:
- Missing alt text on images
- Form labels not associated
- Color contrast failures
- Missing ARIA attributes
- Invalid ARIA values

### Incomplete (Manual Review)

axe-core flags elements as "incomplete" when it cannot determine compliance and manual review is needed.

## Integration Options

### Playwright Integration

```bash
npm install --save-dev @axe-core/playwright
```

```typescript
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('accessibility audit', async ({ page }) => {
  await page.goto('/');

  const results = await new AxeBuilder({ page }).analyze();

  expect(results.violations).toEqual([]);
});
```

### Cypress Integration

```bash
npm install --save-dev cypress-axe
```

```javascript
// cypress/support/e2e.js
import 'cypress-axe';

// In test
cy.visit('/');
cy.injectAxe();
cy.checkA11y();
```

### Jest Integration

```bash
npm install --save-dev jest-axe
```

```javascript
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

test('renders with no accessibility violations', async () => {
  const { container } = render(<App />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

## axe-core Configuration

### Rule Tags

| Tag | Description |
|-----|-------------|
| `wcag2a` | WCAG 2.0 Level A |
| `wcag2aa` | WCAG 2.0 Level AA |
| `wcag2aaa` | WCAG 2.0 Level AAA |
| `wcag21a` | WCAG 2.1 Level A |
| `wcag21aa` | WCAG 2.1 Level AA |
| `wcag22aa` | WCAG 2.2 Level AA |
| `best-practice` | Best practices |
| `experimental` | Experimental rules |

### Configuration Options

```typescript
const results = await new AxeBuilder({ page })
  // Include specific tags
  .withTags(['wcag2a', 'wcag2aa', 'wcag21aa'])

  // Exclude rules
  .disableRules(['color-contrast'])

  // Include specific rules only
  .withRules(['image-alt', 'label', 'button-name'])

  // Limit to specific elements
  .include('.main-content')

  // Exclude elements
  .exclude('.third-party-widget')

  .analyze();
```

### Results Structure

```typescript
interface AxeResults {
  violations: Result[];    // Failed rules
  passes: Result[];        // Passed rules
  incomplete: Result[];    // Needs manual review
  inapplicable: Result[];  // Rules that didn't apply
}

interface Result {
  id: string;              // Rule ID
  impact: 'critical' | 'serious' | 'moderate' | 'minor';
  description: string;
  help: string;
  helpUrl: string;
  nodes: NodeResult[];     // Affected elements
}
```

## Common Violations

### Image Alt Text

```html
<!-- Violation -->
<img src="logo.png">

<!-- Fixed -->
<img src="logo.png" alt="Company Logo">

<!-- Decorative image -->
<img src="decoration.png" alt="" role="presentation">
```

### Form Labels

```html
<!-- Violation -->
<input type="text" name="email">

<!-- Fixed: explicit label -->
<label for="email">Email</label>
<input type="text" id="email" name="email">

<!-- Fixed: implicit label -->
<label>
  Email
  <input type="text" name="email">
</label>

<!-- Fixed: aria-label -->
<input type="text" name="email" aria-label="Email address">
```

### Color Contrast

WCAG AA requirements:
- Normal text: 4.5:1 ratio
- Large text (18pt+ or 14pt+ bold): 3:1 ratio

```css
/* Violation: low contrast */
.text { color: #777; background: #fff; } /* 4.48:1 - fails */

/* Fixed */
.text { color: #666; background: #fff; } /* 5.74:1 - passes */
```

### Button Names

```html
<!-- Violation -->
<button><i class="icon-search"></i></button>

<!-- Fixed: visible text -->
<button><i class="icon-search"></i> Search</button>

<!-- Fixed: aria-label -->
<button aria-label="Search">
  <i class="icon-search"></i>
</button>
```

### Heading Order

```html
<!-- Violation: skipped heading level -->
<h1>Page Title</h1>
<h3>Section</h3>

<!-- Fixed -->
<h1>Page Title</h1>
<h2>Section</h2>
```

### ARIA Violations

```html
<!-- Violation: invalid ARIA attribute -->
<div role="button" aria-pressed="yes">Click</div>

<!-- Fixed -->
<div role="button" aria-pressed="true">Click</div>

<!-- Violation: missing required ARIA -->
<div role="checkbox">Option</div>

<!-- Fixed -->
<div role="checkbox" aria-checked="false">Option</div>
```

## Impact Levels

| Level | Description | Action |
|-------|-------------|--------|
| Critical | Blocks users entirely | Fix immediately |
| Serious | Significant barrier | Fix before release |
| Moderate | Degraded experience | Fix soon |
| Minor | Inconvenience | Fix when possible |

## CI/CD Integration

### Fail Threshold Configuration

```typescript
// Only fail on critical/serious issues
const results = await new AxeBuilder({ page }).analyze();

const criticalViolations = results.violations.filter(
  v => v.impact === 'critical' || v.impact === 'serious'
);

expect(criticalViolations).toEqual([]);
```

### Generate Reports

```typescript
import { createHtmlReport } from 'axe-html-reporter';

const results = await new AxeBuilder({ page }).analyze();

createHtmlReport({
  results,
  options: {
    outputDir: 'reports',
    reportFileName: 'accessibility-report.html',
  },
});
```

## Release Cadence

axe-core has a new minor release every 3 to 5 months, which usually introduces new rules and features. Schedule time to upgrade to these versions.

## Zero False Positives

axe-core is designed for zero false positives. If axe-core reports a violation, it is a real issue. This approach prevents testing abandonment from chasing incorrect results.

## Related Documents

- howto-accessibility-automation.md - Implementation guide
- ref-playwright-e2e.md - Playwright integration
- howto-lighthouse-ci.md - Lighthouse accessibility audits
