# Overview
Testing makes use of a few different types of tests:

1. [PHPUnit](#phpunit-tests)
2. [WP Admin JavaScript Unit Testing](#wordpress-admin-javascript-unit-tests)
3. [WP Admin E2E Testing](#wordpress-admin-end-to-end-tests)
3. [Front-End JavaScript Unit Testing](#component-tests)
4. [Front-End JavaScript E2E Testing](#component-tests)

# PHPUnit Tests
PHPUnit tests are the basis for the backend of the plugin. Testing replies on the @wordpress/env package to run tests in a controlled Docker environment. This simplifies setting up PHPUnit on various machines since it's run in an isolated environment. Additionally, it also allows us to test with different php versions and more easily in a CI environment like GitHub Actions.

Tests should be placed in the `tests/feature` or `tests/unit` directories for respective tests.

## Prerequisites
- Unit tests require Docker to be installed and running. There are instructions available for installing [Docker Desktop](https://docs.docker.com/desktop/)
- You must have *BOTH* the Presto Player and Presto Player Pro plugin folders as siblings in your file browser (i.e. both in the plugins directory)

```
plugins
└───presto-player  
└───presto-player-pro
```

## Installation
Once you have Docker running, you'll want to make sure you're in the `presto-player` plugin directory to install node_modules and composer dependencies. 

```bash
yarn
composer install
```

## Usage
To run php unit tests, run the following commands:

```bash
yarn wp-env start
yarn test:php
```

&nbsp;


# WordPress Admin JavaScript Unit Tests
For Presto Player, we only write javascript tests for the WordPress admin interactions. Unit tests for the front-end should be handled with the *Component Tests* (see below).

Tests should be placed in the `tests` folder within the individual component directories and should be named `*.spec.js`.

## Installation
```bash
yarn
composer install
```

## Usage
To run javascript unit tests, run the following commands:

```bash
yarn test:js
```

or 

```bash
yarn test:js --watch
```

&nbsp;


# WordPress Admin End-To-End Tests
For this package, we only write e2e tests for the WordPress admin interactions. e2e tests for the front-end should be handled with the *Component Tests* (see below).

Admin e2e Testing replies on the @wordpress/env package to run tests in a controlled Docker environment.

Tests should be placed in the `tests-e2e` directory.

## Installation
```bash
yarn
composer install
```

## Usage
To run javascript unit tests, first, be sure the wp-env environment is running:

```bash
yarn wp-env start
```

Then run

```bash
yarn test:admin-e2e
```

or 

```bash
yarn test:admin-e2e --watch
```

&nbsp;

# Component Tests
Presto Player uses [StencilJS](https://stenciljs.com/docs/testing-overview) for it's various components, which provides it's own isolated testing environment for both "unit" and "e2e" tests. Component tests are run directly in the `packages/components` directory. 

The benefit of component tests is we can easily seed component data without needing to spin up WordPress or seed post content directly.

Tests should be placed in the `tests` folder within the individual component directories.

## Documentation
- [Stencil Unit Testing Documentation](https://stenciljs.com/docs/unit-testing)
- [Stencil End-To-End Testing Documentation](https://stenciljs.com/docs/unit-testing)

## Installation
```bash
cd packages/components
yarn
```

## Usage

```bash
yarn test
```

## Watch Mode

```bash
yarn test.watch
```

---

## Playwright E2E Tests

In addition to the Jest-based admin E2E tests above, Presto Player also uses [Playwright](https://playwright.dev/) for end-to-end browser testing.

Tests are located in `tests-e2e/` and use `.spec.{js,ts}` naming.

### Prerequisites

- wp-env must be running (`yarn wp-env start`)
- Playwright browsers installed

### Installation

```bash
yarn
composer install
# Install Playwright browsers (first time):
npx playwright install
```

### Usage

```bash
yarn test:e2e:playwright         # Run all Playwright tests
yarn test:e2e:playwright:ui      # Playwright UI mode (visual test runner)
yarn test:e2e:playwright:debug   # Debug mode (step through tests)
yarn test:e2e:playwright:trace   # Run with tracing enabled
```

---

## Updated Command Reference

> Note: Some commands in the sections above reference older script names. The current canonical commands from `package.json` are:

| Purpose | Command |
|---------|---------|
| PHP unit tests | `yarn test:php` |
| PHP failing tests only | `yarn test:php:failing` |
| JS unit tests | `yarn test:unit` |
| Admin E2E (Jest) | `yarn test:e2e` |
| Playwright E2E | `yarn test:e2e:playwright` |
| Direct PHPUnit | `composer test` |

---

## Test File Locations

| Test Type | Location |
|-----------|----------|
| PHP unit | `tests/unit/` |
| PHP feature | `tests/feature/` |
| JS unit | `src/**/test/*.spec.js` |
| Admin E2E (Jest) | `tests-e2e/` |
| Playwright E2E | `tests-e2e/` (`.spec.ts` files) |

### Example PHP unit test classes

- `ActivatorTest.php`
- `DatabaseTablesTest.php`
- `PlayerTest.php`
- `PresetTest.php`
- `VisitsTest.php`
- `ShortcodeTest.php`
- `RewriteRulesManagerTest.php`
- `Pro/` — Pro-specific tests

### Example PHP feature test classes

- `analytics/`
- `blocks/`
- `email-collection/`
- `email-provider/`
- `models/`
- `ReusableMediaTest.php`
- `SearchExclusionTest.php`

---

## Related Pages

- [Getting-Started](Getting-Started)
- [Contributing-Guide](Contributing-Guide)