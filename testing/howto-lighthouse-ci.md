---
title: How to Set Up Lighthouse CI with GitHub Actions
description: Step-by-step guide to automate performance testing with Lighthouse CI
category: testing
type: howto
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - lighthouse
  - performance
  - ci-cd
  - github-actions
  - core-web-vitals
prerequisites:
  - Node.js 18+
  - GitHub repository
  - WordPress development environment
sources:
  - https://github.com/GoogleChrome/lighthouse-ci
  - https://github.com/marketplace/actions/lighthouse-ci-action
  - https://github.com/treosh/lighthouse-ci-action
  - https://dev.to/jacobandrewsky/performance-audits-with-lighthouse-ci-github-actions-3g0g
  - https://www.somethingsblog.com/2024/10/21/lighthouse-meets-github-actions-ci-cd-perf-optimization/
---

# How to Set Up Lighthouse CI with GitHub Actions

## Goal

Automate Lighthouse performance audits on every PR to catch regressions in accessibility, SEO, offline support, and performance best practices.

## Step 1: Install Lighthouse CI

```bash
npm install --save-dev @lhci/cli
```

## Step 2: Create Lighthouse Configuration

Create `lighthouserc.json` in project root:

```json
{
  "ci": {
    "collect": {
      "url": [
        "http://localhost:8080/",
        "http://localhost:8080/about/",
        "http://localhost:8080/contact/",
        "http://localhost:8080/services/"
      ],
      "numberOfRuns": 3,
      "settings": {
        "preset": "desktop",
        "throttling": {
          "cpuSlowdownMultiplier": 1
        }
      }
    },
    "assert": {
      "preset": "lighthouse:recommended",
      "assertions": {
        "categories:performance": ["error", { "minScore": 0.7 }],
        "categories:accessibility": ["error", { "minScore": 0.9 }],
        "categories:best-practices": ["error", { "minScore": 0.8 }],
        "categories:seo": ["error", { "minScore": 0.9 }],
        "first-contentful-paint": ["warn", { "maxNumericValue": 2000 }],
        "largest-contentful-paint": ["error", { "maxNumericValue": 2500 }],
        "cumulative-layout-shift": ["error", { "maxNumericValue": 0.1 }],
        "total-blocking-time": ["warn", { "maxNumericValue": 300 }]
      }
    },
    "upload": {
      "target": "temporary-public-storage"
    }
  }
}
```

## Step 3: Create Performance Budget

Create `budget.json`:

```json
[
  {
    "path": "/*",
    "timings": [
      { "metric": "first-contentful-paint", "budget": 2000 },
      { "metric": "largest-contentful-paint", "budget": 2500 },
      { "metric": "cumulative-layout-shift", "budget": 0.1 },
      { "metric": "total-blocking-time", "budget": 300 },
      { "metric": "time-to-interactive", "budget": 3800 }
    ],
    "resourceSizes": [
      { "resourceType": "script", "budget": 300 },
      { "resourceType": "stylesheet", "budget": 100 },
      { "resourceType": "image", "budget": 500 },
      { "resourceType": "font", "budget": 100 },
      { "resourceType": "total", "budget": 1000 }
    ],
    "resourceCounts": [
      { "resourceType": "script", "budget": 10 },
      { "resourceType": "stylesheet", "budget": 5 },
      { "resourceType": "image", "budget": 25 },
      { "resourceType": "font", "budget": 5 },
      { "resourceType": "third-party", "budget": 10 }
    ]
  }
]
```

## Step 4: Create GitHub Actions Workflow

Create `.github/workflows/lighthouse.yml`:

```yaml
name: Lighthouse CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  lighthouse:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build production assets
        run: npm run build

      - name: Start WordPress
        run: |
          docker-compose up -d
          sleep 30
          curl -I http://localhost:8080 || exit 1

      - name: Run Lighthouse CI
        uses: treosh/lighthouse-ci-action@v12
        with:
          configPath: './lighthouserc.json'
          budgetPath: './budget.json'
          uploadArtifacts: true
          temporaryPublicStorage: true

      - name: Upload Lighthouse Report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: lighthouse-report
          path: .lighthouseci/
          retention-days: 30
```

## Step 5: Create Mobile Testing Configuration

Create `lighthouserc.mobile.json`:

```json
{
  "ci": {
    "collect": {
      "url": [
        "http://localhost:8080/",
        "http://localhost:8080/contact/"
      ],
      "numberOfRuns": 3,
      "settings": {
        "preset": "mobile",
        "throttling": {
          "cpuSlowdownMultiplier": 4,
          "requestLatencyMs": 150,
          "downloadThroughputKbps": 1600,
          "uploadThroughputKbps": 750
        },
        "screenEmulation": {
          "mobile": true,
          "width": 375,
          "height": 667,
          "deviceScaleFactor": 2
        }
      }
    },
    "assert": {
      "assertions": {
        "categories:performance": ["error", { "minScore": 0.6 }],
        "categories:accessibility": ["error", { "minScore": 0.9 }],
        "largest-contentful-paint": ["error", { "maxNumericValue": 4000 }],
        "cumulative-layout-shift": ["error", { "maxNumericValue": 0.1 }]
      }
    },
    "upload": {
      "target": "temporary-public-storage"
    }
  }
}
```

## Step 6: Add Multiple Configuration Support

Update workflow to run both configurations:

```yaml
name: Lighthouse CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  lighthouse-desktop:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup and build
        run: |
          npm ci
          npm run build
          docker-compose up -d
          sleep 30

      - name: Run Lighthouse CI (Desktop)
        uses: treosh/lighthouse-ci-action@v12
        with:
          configPath: './lighthouserc.json'
          uploadArtifacts: true
          temporaryPublicStorage: true

  lighthouse-mobile:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup and build
        run: |
          npm ci
          npm run build
          docker-compose up -d
          sleep 30

      - name: Run Lighthouse CI (Mobile)
        uses: treosh/lighthouse-ci-action@v12
        with:
          configPath: './lighthouserc.mobile.json'
          uploadArtifacts: true
          temporaryPublicStorage: true
```

## Step 7: Local Testing Script

Create `scripts/lighthouse-local.sh`:

```bash
#!/bin/bash

# Start WordPress if not running
if ! curl -s http://localhost:8080 > /dev/null; then
  echo "Starting WordPress..."
  docker-compose up -d
  sleep 30
fi

# Run Lighthouse CI locally
npx lhci autorun --config=lighthouserc.json

# Open report
if [ -d ".lighthouseci" ]; then
  echo "Opening Lighthouse report..."
  npx lhci open
fi
```

## Step 8: Custom Assertions

Create advanced assertions in `lighthouserc.json`:

```json
{
  "ci": {
    "collect": {
      "url": ["http://localhost:8080/"],
      "numberOfRuns": 3
    },
    "assert": {
      "assertions": {
        "categories:performance": ["error", { "minScore": 0.7 }],
        "categories:accessibility": ["error", { "minScore": 0.9 }],
        "categories:best-practices": ["error", { "minScore": 0.8 }],
        "categories:seo": ["error", { "minScore": 0.9 }],

        "first-contentful-paint": ["warn", { "maxNumericValue": 2000 }],
        "largest-contentful-paint": ["error", { "maxNumericValue": 2500 }],
        "cumulative-layout-shift": ["error", { "maxNumericValue": 0.1 }],
        "total-blocking-time": ["warn", { "maxNumericValue": 300 }],
        "speed-index": ["warn", { "maxNumericValue": 3500 }],

        "uses-responsive-images": "warn",
        "offscreen-images": "warn",
        "uses-optimized-images": "warn",
        "uses-webp-images": "warn",
        "uses-text-compression": "error",
        "uses-rel-preconnect": "warn",

        "render-blocking-resources": "warn",
        "unminified-css": "error",
        "unminified-javascript": "error",
        "unused-css-rules": "warn",
        "unused-javascript": "warn",

        "meta-description": "error",
        "document-title": "error",
        "html-has-lang": "error",
        "canonical": "warn",

        "image-alt": "error",
        "button-name": "error",
        "link-name": "error",
        "color-contrast": "warn"
      }
    },
    "upload": {
      "target": "temporary-public-storage"
    }
  }
}
```

## Step 9: Add npm Scripts

Update `package.json`:

```json
{
  "scripts": {
    "lighthouse": "lhci autorun",
    "lighthouse:desktop": "lhci autorun --config=lighthouserc.json",
    "lighthouse:mobile": "lhci autorun --config=lighthouserc.mobile.json",
    "lighthouse:open": "lhci open",
    "lighthouse:server": "lhci server --storage.storageMethod=sql --storage.sqlDialect=sqlite --storage.sqlDatabasePath=./lhci.db"
  }
}
```

## Step 10: Self-Hosted Lighthouse Server (Optional)

For historical tracking, set up a self-hosted server:

```yaml
# docker-compose.lighthouse.yml
version: '3.8'

services:
  lhci-server:
    image: patrickhulce/lhci-server:latest
    ports:
      - "9001:9001"
    volumes:
      - lhci-data:/data
    environment:
      - LHCI_STORAGE__SQL_DIALECT=sqlite
      - LHCI_STORAGE__SQL_DATABASE_PATH=/data/lhci.db

volumes:
  lhci-data:
```

Update `lighthouserc.json` to use server:

```json
{
  "ci": {
    "upload": {
      "target": "lhci",
      "serverBaseUrl": "http://localhost:9001"
    }
  }
}
```

## Verification

```bash
# Run local test
npm run lighthouse

# Open report
npm run lighthouse:open

# Check output
cat .lighthouseci/manifest.json
```

## Expected Results

Successful run output:

```
Running Lighthouse CI...

✓ Collected 3 runs for http://localhost:8080/
✓ Collected 3 runs for http://localhost:8080/about/

Lighthouse scores:
  Performance: 78
  Accessibility: 94
  Best Practices: 92
  SEO: 100

All assertions passed!

Reports uploaded to:
https://storage.googleapis.com/lighthouse-infrastructure.appspot.com/reports/xxxxx
```

## Troubleshooting

### Tests timeout

Increase timeout in workflow:

```yaml
- name: Start WordPress
  run: |
    docker-compose up -d
    sleep 60  # Increase wait time
    curl --retry 5 --retry-delay 5 http://localhost:8080
```

### Flaky performance scores

Use more runs for stability:

```json
{
  "ci": {
    "collect": {
      "numberOfRuns": 5
    }
  }
}
```

### Different results locally vs CI

Ensure consistent environment:
- Use same Node.js version
- Use same Chrome version
- Match throttling settings

### Budget exceeded

Review and adjust budgets based on actual performance:

```bash
# Get current metrics
npx lhci autorun --collect.numberOfRuns=1 --output=json
```

## Best Practices

### 1. Run Multiple Times

Use at least 3 runs to account for variance. Single runs can be flaky.

### 2. Set Realistic Thresholds

Start with current baseline, then gradually improve targets.

### 3. Fail on Regressions

Use assertions to catch performance regressions:

```json
"assertions": {
  "categories:performance": ["error", { "minScore": 0.7 }]
}
```

### 4. Track Trends

Use temporary public storage or self-hosted server to track improvements over time.

### 5. Separate Mobile and Desktop

Mobile has different performance characteristics. Test both.

## Related Documents

- ref-github-actions-wordpress.md - GitHub Actions reference
- howto-playwright-wordpress.md - E2E testing
- ref-accessibility-testing.md - Accessibility audits
