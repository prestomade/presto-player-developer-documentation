# Architecture Overview

Presto Player is a WordPress video player plugin built with a PHP backend (DI container pattern) and a React/StencilJS frontend. The free plugin (`presto-player`) handles core playback and administration; the Pro plugin (`presto-player-pro`) extends it via a WordPress filter without modifying core code.

---

## Boot Flow

```
presto-player.php  (entry point)
  └─ require vendor/autoload.php
  └─ new Factory($plugin_instance)
       └─ getRules()  →  DICE DI container rules
  └─ new Controller($container)
       └─ run()
            └─ loop through app.php components[]
                 └─ $container->create($class)
                      └─ $component->register()  →  add_action / add_filter
```

**Pro plugin hooks in via:**
```
presto-player-pro.php
  └─ new Plugin()
       └─ bootstrap()
            └─ add_filter('presto_player_pro_components', [$this, 'components'])
                 └─ appends pro component list to core components array
```

The core `Controller` applies the filter before instantiating components, so Pro classes are created and registered the same way as core ones.

---

## Dependency Injection (DICE)

All services are resolved through the [DICE](https://github.com/Level-2/Dice) DI container.

| Pattern | Usage |
|---------|-------|
| `'shared' => true` | Singleton — one instance reused (e.g., `Settings`, `Scripts`, `AdminNotices`) |
| `constructParams` | Inject primitive values (e.g., `$isPro` bool into `Settings` and `Block`) |
| Interface autowiring | Not used — all bindings explicit in `Factory::getRules()` |

Key singletons: `BunnyCDN`, `Visits`, `ReusableVideos`, `AdminNotices`, `Settings`.

---

## Component Lifecycle

Every registered component implements the `Service` contract (`inc/Contracts/Service.php`):

```php
interface Service {
    public function register(): void;
}
```

`register()` is where the component attaches its WordPress hooks. No constructor logic — the DI container builds the object, then `Controller` calls `register()`.

---

## Layer Map

```
┌─────────────────────────────────────────────────┐
│  WordPress (hooks, REST API, Block Editor)       │
├──────────────┬──────────────────────────────────┤
│  Blocks      │  10 block types (free)            │
│  (inc/Blocks)│  + BunnyCDN, Playlist (pro)       │
├──────────────┼──────────────────────────────────┤
│  Services    │  Scripts, Settings, Menu,          │
│  (inc/Serv.) │  Shortcodes, Migrations, Player   │
│              │  + Pro: BunnyService, Email,       │
│              │    Analytics, License              │
├──────────────┼──────────────────────────────────┤
│  REST API    │  /presto-player/v1/*               │
│  Controllers │  Videos, Presets, AudioPresets,   │
│              │  Settings (free)                   │
│              │  + Bunny, Email providers (pro)    │
├──────────────┼──────────────────────────────────┤
│  Models      │  Video, Preset, AudioPreset,       │
│  (inc/Models)│  ReusableVideo, Setting,           │
│              │  EmailCollection, Webhook          │
├──────────────┼──────────────────────────────────┤
│  Database    │  6 custom tables (see             │
│  (inc/DB)    │  Database-Schema)                 │
├──────────────┼──────────────────────────────────┤
│  Integrations│  LearnDash, TutorLMS, LifterLMS,  │
│              │  Elementor, Divi, BeaverBuilder,   │
│              │  Kadence                           │
└──────────────┴──────────────────────────────────┘
```

---

## Free ↔ Pro Relationship

- Pro plugin is a **separate WordPress plugin** — it must be activated alongside the free plugin.
- Pro does **not** fork or copy core code — it hooks in via `presto_player_pro_components` filter.
- Core exposes extension points via `apply_filters('presto_player_*', ...)` throughout.
- The `Plugin::isPro()` static method is checked in core (e.g., in `Settings` constructor) to conditionally expose pro-only options.
- Version compatibility is validated in `Plugin::bootstrap()` — it checks that core version is `>= 1.1.0`.

---

## Namespace Structure

| Namespace | Root Path | Description |
|-----------|-----------|-------------|
| `PrestoPlayer\` | `presto-player/inc/` | All free plugin PHP classes |
| `PrestoPlayer\Pro\` | `presto-player-pro/inc/` | All pro plugin PHP classes |

Both use PSR-4 autoloading via Composer. The Imposter plugin further wraps vendor dependencies under the `PrestoPlayer\` namespace to avoid conflicts with other plugins using the same libraries.

---

## Related Pages

- [Plugin-Structure](Plugin-Structure) — full directory map
- [Block-Development](Block-Development) — how blocks are registered
- [Services-and-Integrations](Services-and-Integrations) — service list
- [Pro-Plugin-Architecture](Pro-Plugin-Architecture) — pro bootstrap detail
- [Database-Schema](Database-Schema) — table definitions
