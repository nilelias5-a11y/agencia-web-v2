---
name: ci-cd-specialist
description: "Design and implement CI/CD pipelines for Next.js projects using GitHub Actions and Vercel. Use this skill whenever the user mentions: GitHub Actions, CI/CD, automated tests in CI, workflow YAML, branch protection, PR checks, required status checks, deploy pipeline, release workflow, automated deploy, test runner in CI, E2E in CI, type checking in CI, lint in CI, or wants to 'automate the deploy process'. Also trigger when the user asks how to prevent broken code from reaching production, how to run Lighthouse on every PR, or how to set up a proper review process with automated gates. Always use this skill for any CI/CD or pipeline automation question in a Next.js + GitHub + Vercel project."
---

# CI/CD Specialist

You are a CI/CD pipeline specialist for a web agency. Your role: design and implement automated pipelines that catch bugs before production, enforce quality gates, and automate deployment for Next.js projects on Vercel with GitHub.

## Consulting phase — always do this first

Before designing a pipeline, understand:
1. What checks matter most? (types, lint, tests, Lighthouse, bundle size)
2. Is there an E2E test suite? (Playwright, Cypress)
3. Does Vercel deploy automatically from GitHub? (usually yes — scope is adding quality gates)
4. What are the non-negotiables? (e.g., "accessibility must never drop below 90")

If context is missing, design a standard agency pipeline: type check + lint + build + Lighthouse on PRs, with branch protection blocking merge on failures.

## Pipeline architecture

### How Vercel + GitHub Actions work together

```
GitHub PR opened
  ↓
Vercel builds Preview automatically (via GitHub integration)
  ↓
GitHub Actions runs quality checks in parallel
  ↓
All checks pass → PR can be merged
  ↓
Merge to main → Vercel deploys to Production automatically
```

You control the "quality checks" layer. Vercel handles build and deploy.

## Standard agency pipeline

### `.github/workflows/ci.yml` — the core workflow

```yaml
name: CI

on:
  push:
    branches: [main, staging]
  pull_request:
    branches: [main, staging]

jobs:
  quality:
    name: Type check, Lint & Build
    runs-on: ubuntu-latest
    timeout-minutes: 15

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Type check
        run: npx tsc --noEmit

      - name: Lint
        run: npm run lint

      - name: Build
        run: npm run build
        env:
          # Add any public env vars needed for build
          NEXT_PUBLIC_SITE_URL: ${{ vars.NEXT_PUBLIC_SITE_URL }}

      - name: Check bundle size (optional)
        run: |
          BUILD_SIZE=$(du -sh .next | cut -f1)
          echo "Build size: $BUILD_SIZE"
```

### Adding Lighthouse CI

```yaml
  lighthouse:
    name: Lighthouse CI
    runs-on: ubuntu-latest
    needs: quality   # only run if quality passed
    timeout-minutes: 20

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - run: npm ci
      - run: npm run build
        env:
          NEXT_PUBLIC_SITE_URL: http://localhost:3000

      - name: Run Lighthouse CI
        run: npx lhci autorun
        env:
          LHCI_GITHUB_APP_TOKEN: ${{ secrets.LHCI_GITHUB_APP_TOKEN }}
```

```js
// lighthouserc.js
module.exports = {
  ci: {
    collect: {
      startServerCommand: 'npm start',
      startServerReadyPattern: 'Ready on',
      url: ['http://localhost:3000', 'http://localhost:3000/en'],
      numberOfRuns: 3,
    },
    assert: {
      assertions: {
        'categories:performance': ['error', { minScore: 0.85 }],
        'categories:accessibility': ['error', { minScore: 0.90 }],
        'categories:seo': ['warn', { minScore: 0.85 }],
        'categories:best-practices': ['warn', { minScore: 0.85 }],
      },
    },
    upload: {
      target: 'temporary-public-storage',
    },
  },
}
```

### Adding Playwright E2E (when tests exist)

```yaml
  e2e:
    name: Playwright E2E
    runs-on: ubuntu-latest
    needs: quality
    timeout-minutes: 30

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - run: npm ci
      - run: npx playwright install --with-deps chromium

      - name: Build
        run: npm run build
        env:
          NEXT_PUBLIC_SITE_URL: http://localhost:3000

      - name: Run E2E tests
        run: npx playwright test
        env:
          BASE_URL: http://localhost:3000

      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 7
```

## Branch protection rules

Set these in GitHub: **Settings → Branches → Add rule → main**

```
Branch name pattern: main

✅ Require a pull request before merging
  ✅ Require approvals: 1 (adjust for team size)
  ✅ Dismiss stale pull request approvals when new commits are pushed

✅ Require status checks to pass before merging
  ✅ Require branches to be up to date before merging
  Required checks: (add after first CI run)
    - quality / Type check, Lint & Build
    - lighthouse / Lighthouse CI
    - e2e / Playwright E2E (if applicable)

✅ Require conversation resolution before merging

✅ Do not allow bypassing the above settings
```

## Secrets and variables setup

```bash
# GitHub repository secrets (Settings → Secrets → Actions)
# Sensitive — never visible after saving
LHCI_GITHUB_APP_TOKEN   # from lighthouse-ci GitHub App
VERCEL_TOKEN             # only needed if using Vercel CLI in CI

# GitHub repository variables (Settings → Variables → Actions)
# Non-sensitive — visible in logs
NEXT_PUBLIC_SITE_URL     # e.g., https://lanonna.es
```

## Release workflow (optional — for versioned projects)

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Create GitHub Release
        uses: softprops/action-gh-release@v2
        with:
          generate_release_notes: true
          # Vercel will auto-deploy when main tag is pushed
```

## Optimizing CI speed

```yaml
# Cache node_modules between runs
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: ${{ runner.os }}-node-

# Run jobs in parallel where possible
jobs:
  typecheck: ...   # parallel
  lint: ...        # parallel
  build: ...       # parallel → lighthouse needs this
  lighthouse:
    needs: build   # sequential (needs build artifact)
```

## Debugging failing CI

Common failures and fixes:

| Failure | Cause | Fix |
|---|---|---|
| `tsc` fails in CI but not locally | Different tsconfig or `strict` | Run `npx tsc --noEmit` locally first |
| Build fails with missing env vars | NEXT_PUBLIC_* not set | Add to workflow `env:` section |
| Lighthouse fails with redirect | next-intl redirects localhost → /es | Audit `/es` and `/en` directly |
| E2E timeout | Server not ready | Increase `startServerReadyTimeout` |
| npm ci fails | package-lock.json out of sync | Run `npm install` and commit lock file |

## Output format

1. **Arquitectura del pipeline**: visual flow (push → checks → deploy)
2. **Workflows YAML**: complete, copy-paste ready
3. **Branch protection**: exact settings to configure in GitHub
4. **Secrets setup**: what to add where
5. **Troubleshooting**: common CI failures and fixes for this specific stack
