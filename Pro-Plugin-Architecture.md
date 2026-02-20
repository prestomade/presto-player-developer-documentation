# Pro Plugin Architecture

Presto Player Pro (`presto-player-pro`) is a standalone WordPress plugin that extends the free plugin without modifying its code.

---

## Bootstrap Flow

```
presto-player-pro.php
  ├─ define PRESTO_PLAYER_PRO_PLUGIN_FILE
  ├─ define PRESTO_PLAYER_PRO_PLUGIN_URL
  ├─ define PRESTO_PLAYER_PRO_PLUGIN_DIR
  ├─ require vendor/autoload.php
  └─ (new Plugin())->bootstrap()
       ├─ Validate core plugin version >= 1.1.0
       ├─ Check license (if licensing build)
       └─ add_filter('presto_player_pro_components', [$this, 'components'])
            └─ Returns array from inc/config/app.php
```

The core plugin applies the `presto_player_pro_components` filter inside `Controller::run()` before instantiating components. This means Pro classes go through the same DI container and `register()` lifecycle as core classes.

---

## Namespace and Autoloading

- Namespace: `PrestoPlayer\Pro\`
- PSR-4 root: `presto-player-pro/inc/`
- Composer autoload configured in `presto-player-pro/composer.json`

---

## Version Compatibility Check

On `bootstrap()`, the Pro plugin validates:

```php
// Core must be >= 1.1.0
if ( ! class_exists('PrestoPlayer\Plugin') || ! version_compare(Plugin::version(), '1.1.0', '>=') ) {
    // show admin notice, bail
}
```

If the core plugin is too old (or not active), Pro shows an admin notice and does not load its components.

---

## License System

License management is contained entirely within `Services/License/`. This directory is **excluded from VIP builds**.

### License Flow

1. On activation, redirect to license page if no key stored
2. Admin enters key → `LicensedProduct::activate($key)` called
3. Communicates with `my.prestomade.com` WooCommerce Software Licensing API
4. Response code `s100` = new activation, `s101` = renewal
5. Key stored via `Setting::update('license', 'key', $key)`

### Build Variants

| Variant | Licensing | Ruleset | Distribution |
|---------|-----------|---------|--------------|
| Standard (Pro) | Included (`Services/License/`) | WordPress + VIP-Go | Sold direct / WooCommerce |
| VIP | Excluded | WordPress VIP | WordPress.com VIP |

Build commands:
```bash
npm run build:license   # Standard pro build
npm run build:vip       # VIP build (COMPOSER=composer-vip.json)
npm run package         # Both + ZIP
```

---

## Directory Structure

```
presto-player-pro/
├─ presto-player-pro.php        # Entry point
├─ inc/
│   ├─ Plugin.php               # Bootstrap class
│   ├─ config/app.php           # Component registry
│   ├─ Blocks/
│   │   ├─ BunnyCDNBlock.php    # Bunny stream block
│   │   └─ PlaylistBlock.php    # Playlist block
│   ├─ Contracts/
│   │   └─ EmailProviderInterface.php
│   ├─ Controllers/             # Bunny stream orchestrators
│   │   ├─ BunnyStreamController.php
│   │   ├─ BunnyVideoLibraryController.php
│   │   ├─ BunnyVideosController.php
│   │   ├─ BunnyCollectionsController.php
│   │   └─ BunnyPullZoneController.php
│   ├─ Libraries/               # API client wrappers
│   │   ├─ BunnyApiRequest.php
│   │   ├─ BunnyVideoApiRequest.php
│   │   └─ [ProviderRequest].php (per email provider)
│   ├─ Models/
│   │   ├─ Bunny/               # Bunny settings models
│   │   │   ├─ Account.php
│   │   │   ├─ PullZone.php
│   │   │   ├─ StorageZone.php
│   │   │   └─ Collection.php
│   │   ├─ LicensedProduct.php
│   │   └─ Visit.php
│   ├─ Services/
│   │   ├─ API/                 # REST controllers
│   │   ├─ Bunny/               # BunnyService + CDN/Storage requests
│   │   ├─ License/             # License management (excluded in VIP)
│   │   ├─ ActiveCampaign/
│   │   ├─ Mailchimp/
│   │   ├─ MailerLite/
│   │   ├─ FluentCRM/
│   │   ├─ Webhooks/
│   │   ├─ EmailCollection/
│   │   ├─ Settings.php
│   │   ├─ Visits.php
│   │   ├─ Translation.php
│   │   ├─ GoogleAnalytics.php
│   │   ├─ EmailExport.php
│   │   └─ CoreInstaller.php
│   └─ Support/                 # Base classes and utilities
├─ composer.json                # Standard build
├─ composer-vip.json            # VIP build
├─ gulpfile.js                  # Build tasks
└─ package.json                 # npm scripts
```

---

## Settings Model Pattern

All Bunny configuration models extend `SettingModel` and implement `ArrayAccess`:

```php
class PullZone extends SettingModel {
    protected $option_name = 'presto_player_bunny_pull_zone';
    protected $fillable    = ['id', 'name', 'url', 'token_auth_key'];
}
```

Methods: `get($key)`, `update($data)`, `delete()`, `getFillableSchema()`

Data is stored in `wp_options`. The `ArrayAccess` interface allows array-style access: `$pullZone['url']`.

---

## Core Integration Filters Used by Pro

| Filter | Pro Usage |
|--------|-----------|
| `presto_player_pro_components` | Registers all pro components |
| `presto_player/block/default_attributes` | BunnyCDN injects signed URL attrs |
| `presto_player/component/attributes` | BunnyCDN injects runtime player config |
| `presto_player_rest_prepared_response_item` | BunnyService signs video URLs in API responses |
| `presto_player_admin_block_script_options` | BunnyService injects Bunny JS vars |
| `presto_player/pro/forms/save` | All email providers listen for email gate submissions |
| `presto_player_videos_created/updated` | BunnyService syncs user video meta |

---

## Related Pages

- [Architecture-Overview](Architecture-Overview)
- [BunnyCDN-Integration](BunnyCDN-Integration)
- [Email-Providers](Email-Providers)
- [Settings-Reference](Settings-Reference)
- [Releasing](Releasing)
