# Contributing Guide

---

## Prerequisites

- PHP 7.3+
- Node.js (check `.nvmrc` or `package.json` `engines` field)
- Yarn 3+ (`corepack enable`)
- Composer
- Docker (for PHP tests via wp-env)

---

## Setup

```bash
# In presto-player/
composer install
yarn
yarn bootstrap      # builds workspace packages (run once)
yarn dev            # composer install + watch
```

For pro development, both plugin folders must be siblings:

```
plugins/
├─ presto-player/
└─ presto-player-pro/
```

---

## Branch Conventions

| Branch | Purpose |
|--------|---------|
| `master` | Stable, released code |
| `release/vX.Y.Z` | Release preparation branch |
| `feature/...` | New features |
| `fix/...` | Bug fixes |
| `chore/...` | Non-functional changes (deps, docs, refactor) |

---

## PHP Code Standards

PHPCS is configured in `phpcs.xml` with the `WordPress-Core`, `WordPress-Extra`, and `WordPress-VIP-Go` rulesets.

```bash
composer lint       # Check for violations (PHPCS)
composer format     # Auto-fix violations (PHPCBF)
```

Key rules:
- PSR-4 class naming — class name must match filename
- Escape all output: `esc_html()`, `esc_attr()`, `esc_url()`
- Sanitize all input: `sanitize_text_field()`, `absint()`, etc.
- Use `$wpdb->prepare()` for all raw SQL queries
- Namespace all classes under `PrestoPlayer\` (free) or `PrestoPlayer\Pro\` (pro)

---

## JavaScript Code Standards

```bash
yarn lint:js        # ESLint
yarn lint:css       # Stylelint
yarn format         # Prettier
```

- Functional React components with hooks
- `@wordpress/data` for state management
- `@wordpress/i18n` for all user-facing strings
- Emotion CSS-in-JS for admin component styles

---

## Writing Tests

See [Testing-Guide](Testing-Guide) for full test setup and run instructions.

- PHP unit tests: `tests/unit/`
- PHP feature tests: `tests/feature/`
- JS unit tests: `src/**/test/*.spec.js`
- E2E tests: `tests-e2e/`

All new features should ship with at least unit tests. Bug fixes should include a regression test.

---

## Commit Messages

Use conventional commit format:

```
type(scope): short description

Longer explanation if needed.
```

Types: `feat`, `fix`, `chore`, `docs`, `test`, `refactor`, `perf`

Examples:
```
feat(bunny): add private video URL signing
fix(presets): resolve default preset not applying on first load
chore(deps): update @wordpress/scripts to v27
```

---

## Pull Request Checklist

- [ ] PHPCS passes (`composer lint`)
- [ ] ESLint passes (`yarn lint:js`)
- [ ] Tests pass (`yarn test:php`, `yarn test:unit`)
- [ ] No debug code left (`console.log`, `var_dump`, `error_log`)
- [ ] No hardcoded strings — use i18n functions
- [ ] No secrets or API keys in code
- [ ] Version numbers NOT bumped (done in release process)
- [ ] Changelog entry added if user-facing change

---

## Translations

```bash
yarn makepot        # Regenerates languages/presto-player.pot
```

Source directories scanned: `src/`, `inc/`, `templates/`

Text domain: `presto-player` (free), `presto-player-pro` (pro)

---

## Release Process

See [Releasing](Releasing).

---

## Related Pages

- [Getting-Started](Getting-Started)
- [Testing-Guide](Testing-Guide)
- [Releasing](Releasing)
- [Architecture-Overview](Architecture-Overview)
