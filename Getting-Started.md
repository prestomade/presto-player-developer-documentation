To get started, run:

`composer install`

`yarn`

Next run:

`yarn bootstrap`

Then you can start the development build and server using:

`yarn dev`

---

## Full Development Setup

### Prerequisites

- PHP 7.3+
- Composer
- Node.js (check `package.json` `engines` field)
- Yarn 3+ (`corepack enable && corepack prepare yarn@stable --activate`)
- Docker Desktop (required for PHP tests via wp-env)

### Clone & Install

```bash
# Both plugins must be siblings in the plugins directory
cd wp-content/plugins/

git clone <presto-player-repo> presto-player
git clone <presto-player-pro-repo> presto-player-pro

cd presto-player
composer install
yarn
yarn bootstrap      # Builds workspace packages — required on first setup
```

### Start Development

```bash
yarn dev            # composer install + watch (recommended for daily dev)
# or separately:
yarn start          # Watch main plugin src/ only
yarn start:workspace  # Watch all workspace packages (use when editing packages/)
```

### Build for Production

```bash
yarn build              # Build main plugin assets
yarn build:workspace    # Build all workspace packages
yarn plugin:release     # Full release: deps → i18n → build → zip
```

---

## WordPress Test Environment (wp-env)

wp-env runs a Docker-based WordPress instance for testing.

```bash
yarn wp-env start       # Start WordPress at http://localhost:3333
yarn wp-env stop        # Stop environment
```

- Dev site: `http://localhost:3333`
- Test site: `http://localhost:4333`
- Loads both `presto-player` and `presto-player-pro` automatically

---

## Pro Plugin Setup

```bash
cd presto-player-pro
composer install        # Standard build
# or for VIP:
COMPOSER=composer-vip.json composer install
```

Build commands (from `presto-player-pro/`):
```bash
npm run build:license   # Standard pro build
npm run build:vip       # VIP build (no licensing)
npm run package         # Both variants + ZIP files in package/
```

---

## Key File References

| Path | Purpose |
|------|---------|
| `presto-player/presto-player.php` | Free plugin entry point |
| `presto-player/inc/config/app.php` | Component registry |
| `presto-player/inc/Factory.php` | DI container rules |
| `presto-player/inc/Controller.php` | Component loader |
| `presto-player-pro/presto-player-pro.php` | Pro plugin entry point |
| `presto-player-pro/inc/config/app.php` | Pro component registry |
| `presto-player-pro/inc/Plugin.php` | Pro bootstrap class |

See [Plugin-Structure](Plugin-Structure) for a full directory map.
