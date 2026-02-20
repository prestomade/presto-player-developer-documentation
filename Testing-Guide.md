# Testing Guide

See [Testing](Testing) for the full testing documentation including PHPUnit, JavaScript unit, E2E, and Playwright test setup.

This page is a quick reference.

---

## Quick Start

```bash
# Start wp-env (required for PHP + E2E tests)
yarn wp-env start

# Run all PHP tests
yarn test:php

# Run JS unit tests
yarn test:unit

# Run Playwright E2E tests
yarn test:e2e:playwright

# Run Playwright in UI mode
yarn test:e2e:playwright:ui
```

---

## Command Reference

| Command | Description |
|---------|-------------|
| `yarn test:php` | PHPUnit in wp-env container |
| `yarn test:php:failing` | Only tests marked `@group failing` |
| `yarn test:unit` | Jest unit tests |
| `yarn test:e2e` | WordPress E2E scripts |
| `yarn test:e2e:playwright` | Playwright E2E suite |
| `yarn test:e2e:playwright:ui` | Playwright visual runner |
| `yarn test:e2e:playwright:debug` | Step-through debugger |
| `yarn test:e2e:playwright:trace` | With request tracing |
| `composer test` | Direct PHPUnit (alternative) |

---

See [Testing](Testing) for full setup instructions, test file locations, and StencilJS component testing.
