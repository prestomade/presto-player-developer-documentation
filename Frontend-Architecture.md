# Frontend Architecture

Presto Player uses a Yarn workspaces monorepo combining a StencilJS web component library (the player itself) and React admin interfaces (Gutenberg blocks and settings).

---

## Monorepo Structure

```
presto-player/
├─ package.json              # Workspace root (Yarn 3.3.0)
├─ lerna.json                # Lerna config for workspace orchestration
├─ src/                      # React admin interfaces
│   ├─ admin/                # WordPress admin pages (React)
│   ├─ player/               # Player components (React wrappers)
│   ├─ router/               # Client-side routing for admin pages
│   ├─ hooks/                # Shared React hooks
│   ├─ shared/               # Shared utilities and constants
│   ├─ elementor/            # Elementor editor integration UI
│   ├─ libraries/            # Custom JS libraries
│   └─ shims/                # Browser compatibility shims
├─ packages/
│   ├─ components/           # StencilJS web component library
│   └─ components-react/     # React adaptor for StencilJS components
└─ dist/                     # Compiled output (auto-generated, do not edit)
```

---

## StencilJS Web Components (`packages/components`)

The core player is built as a [StencilJS](https://stenciljs.com/) web component library. This provides:

- **Lazy loading** — JavaScript modules loaded on demand
- **Style encapsulation** — Shadow DOM prevents CSS bleed
- **Framework-agnostic** — Can be used in any context (React, plain HTML)
- **Plyr.io core** — A Presto-maintained fork of [plyr](https://github.com/prestomade/plyr) powers the actual playback controls

The component library compiles to:
- Standard web components (used on the frontend)
- React adaptor (`@presto-player/components-react`) used in Gutenberg editor

---

## React Adaptor (`packages/components-react`)

Wraps the StencilJS components for use in React/Gutenberg:

```js
import { PrestoPlayer } from '@presto-player/components-react';

<PrestoPlayer src="https://example.com/video.mp4" preset={presetConfig} />
```

Both the web component library and the React adaptor must be loaded on the page for the adaptor to work.

---

## Build System

| Tool | Purpose |
|------|---------|
| Yarn 3 workspaces | Monorepo package management |
| WordPress Scripts (`@wordpress/scripts`) | Webpack + Babel config for Gutenberg blocks |
| Lerna | Cross-workspace build orchestration |
| Emotion CSS-in-JS | Styling for admin React components |
| PostCSS | CSS processing |

### Build Commands

```bash
# Development
yarn start              # Watch mode (wp-scripts)
yarn start:workspace    # Watch all workspace packages

# Production
yarn build              # Build main plugin
yarn build:workspace    # Build all workspace packages

# First-time workspace setup
yarn bootstrap          # Installs and builds workspace dependencies
```

> Always run `yarn bootstrap` after a fresh clone or after pulling changes that add/modify workspace packages.

---

## Admin React Interfaces

Admin interfaces in `src/admin/` are React SPA fragments mounted into `<div id="...">` containers in WordPress admin pages. State management uses `@wordpress/data`.

Key areas:
- **Settings page** — `<div id="presto-settings-page">` in `Settings::template()`
- **Media Hub** — block editor sidebar/panel components
- **Analytics dashboard** — charts via `react-apexcharts`
- **Preset editor** — full preset configuration UI

---

## JavaScript Variables Injection

PHP passes runtime data to JS via `wp_localize_script` / `wp_add_inline_script`. The main filters for injecting variables are:

| Filter | Where Used |
|--------|-----------|
| `presto_player_admin_block_script_options` | Block editor admin page |
| `presto-settings-js-options` | Settings page |
| `presto-settings-block-js-options` | Settings block editor |

BunnyService and LMS integrations use these filters to inject their configuration into the frontend.

---

## Styling

- Admin components use **Emotion** (CSS-in-JS) for scoped styles
- Player styles use **Plyr.io** base CSS + Presto overrides
- Custom player CSS can be injected via `presto_player_branding` → `player_css` setting
- Theme integration is via `add_theme_support('appearance-tools')` (added at priority 99999)

---

## i18n

All user-facing strings use WordPress i18n:

```js
import { __ } from '@wordpress/i18n';
const label = __( 'Play video', 'presto-player' );
```

```bash
yarn makepot    # Regenerate .pot file from src/, inc/, templates/
```

---

## Related Pages

- [Block-Development](Block-Development)
- [Architecture-Overview](Architecture-Overview)
- [Getting-Started](Getting-Started)
- [Testing-Guide](Testing-Guide)
