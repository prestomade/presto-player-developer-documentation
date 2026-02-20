Presto Player makes use of lerna.js to use one build system to compile the multiple packages inside divi modules, beaver builder, elementor, and event its player components.

## Player Components
Player components are created, tested and compiled with [StencilJS](https://stenciljs.com/). This is located in the packages/components directory. This includes almost all the player functionality.

This lets us use web components to build our player, which makes things easier to test and build. It also has some additional benefits:

1. Lazy loading through javascript modules.
2. Style encapsulation through the shadow dom.
3. React integration - we compile react adaptors (@presto-player/react) that let you use our player components within gutenberg/react environments)
4. Lightweight. No frameworks are needed as a lot of heavy lifting is done through web standards.
5. Easier e2e and unit testing.

### Plyr.io
The core player functionality consists of our web components wrapped around a fork of the [plyr library](https://github.com/prestomade/plyr)

## React Adapter
You can use our player in Gutenberg/react environments by importing the @presto-player/react library. This is just a wrapper for the web components, so the web components library also needs to be included on the page.

## Admin scripts
The admin blocks and other scripts are located in the `src` folder of the plugin and utilize lerna.js to import the sub packages (@presto-player/react) for example.

## BSF Analytics Integration
Overview - 

BSF Analytics is used to collect plugin-level usage statistics and configuration data. The integration is handled entirely on the backend.

Implementation Steps - 

1 - Analytics Initialization

2 - Added analytics-related code in inc/utils/analytics.php.

3 - Initialized BSF Analytics via composer.json using the brainstormforce/bsf-analytics package.

Composer Setup - 

1 - The package is installed via a VCS repository.

2 - Analytics library is placed under:

3 - inc/lib/bsf-analytics

Data Collection - 

Plugin statistics (such as total spaces, integration type counts, settings, etc.) are prepared in PHP.

Data is filtered using: apply_filters( 'suredash_bsf_analytics_data', $stats_data );

Analytics Dashboard - 

Once connected, analytics data becomes visible in the BSF Analytics dashboard, where plugin-specific tables and stats can be reviewed.

⚠️ Note: No frontend JavaScript tracking is required. All analytics data is collected server-side.

---

## Free Plugin Directory Map (`presto-player/`)

```
presto-player/
├─ presto-player.php          # Entry point — defines constants, boots Factory + Controller
├─ inc/
│   ├─ Activator.php          # Runs on plugin activate (migrations, defaults)
│   ├─ Deactivator.php        # Runs on plugin deactivate
│   ├─ Controller.php         # Instantiates and calls register() on all components
│   ├─ Factory.php            # DICE DI container rules (singletons, constructor params)
│   ├─ Core.php               # Singleton holder for plugin instance
│   ├─ Plugin.php             # Static helpers: version(), isPro(), url(), etc.
│   ├─ Files.php              # File handling (attachment validation)
│   ├─ Attachment.php         # WordPress media attachment integration
│   ├─ Playlist.php           # Playlist data model / helper
│   ├─ Requirements.php       # PHP/WP version gate shown before plugin loads
│   ├─ support.php            # Global helper functions
│   ├─ Blocks/                # Gutenberg block PHP classes (10 blocks)
│   │   ├─ SelfHostedBlock.php
│   │   ├─ YouTubeBlock.php
│   │   ├─ VimeoBlock.php
│   │   ├─ ReusableVideoBlock.php
│   │   ├─ ReusableEditBlock.php
│   │   ├─ AudioBlock.php
│   │   ├─ MediaHubBlock.php
│   │   ├─ PopupBlock.php
│   │   ├─ PopupTriggerBlock.php
│   │   └─ PopupMediaBlock.php
│   ├─ config/
│   │   └─ app.php            # Autoloaded component list
│   ├─ Contracts/
│   │   └─ Service.php        # Interface: requires register() method
│   ├─ Database/
│   │   ├─ Table.php          # dbDelta() wrapper
│   │   ├─ Videos.php         # presto_player_videos table (v4)
│   │   ├─ Presets.php        # presto_player_presets table (v22)
│   │   ├─ AudioPresets.php   # presto_player_audio_presets table (v20)
│   │   ├─ EmailCollection.php# presto_player_email_collection table (v1)
│   │   ├─ Visits.php         # presto_player_visits table (v1)
│   │   ├─ Webhooks.php       # presto_player_webhooks table (v1)
│   │   ├─ Migrations.php     # Runs installs/upgrades
│   │   └─ Upgrades/          # Versioned upgrade scripts
│   ├─ Integrations/
│   │   ├─ LearnDash/         # Video progression for LearnDash
│   │   ├─ Tutor/             # TutorLMS integration
│   │   ├─ Lifter/            # LifterLMS integration
│   │   ├─ Elementor/         # Elementor widgets
│   │   ├─ Divi/              # Divi modules
│   │   ├─ BeaverBuilder/     # Beaver Builder modules
│   │   └─ Kadence.php        # Kadence theme compat
│   ├─ lib/
│   │   └─ bsf-analytics/     # BSF Analytics library (composer)
│   ├─ Libraries/             # Custom PHP libraries
│   ├─ Models/
│   │   ├─ Model.php          # Base model (WPDB wrapper)
│   │   ├─ Video.php
│   │   ├─ Preset.php
│   │   ├─ AudioPreset.php
│   │   ├─ ReusableVideo.php
│   │   ├─ Setting.php        # wp_options accessor
│   │   ├─ Block.php          # Block data model
│   │   ├─ Post.php
│   │   ├─ CurrentUser.php
│   │   ├─ EmailCollection.php
│   │   ├─ Player.php
│   │   └─ Webhook.php
│   ├─ Seeds/
│   │   └─ Seeder.php         # Seeds default presets on first activation
│   ├─ Services/
│   │   ├─ API/               # REST controllers (Videos, Presets, AudioPresets, Settings)
│   │   ├─ Blocks/            # Block-specific services (YouTube, Vimeo, PopupTrigger)
│   │   ├─ Scripts.php        # Asset enqueuing
│   │   ├─ Settings.php       # wp_options registration
│   │   ├─ Menu.php           # Admin menu
│   │   ├─ Player.php         # Frontend render
│   │   ├─ Shortcodes.php     # [presto_player], [presto_playlist], etc.
│   │   ├─ Migrations.php     # DB migration runner
│   │   ├─ Translation.php
│   │   ├─ Blocks.php         # Block category
│   │   ├─ VideoPostType.php  # Media Hub CPT
│   │   ├─ ReusableVideos.php # Reusable video CRUD
│   │   ├─ AdminNotices.php   # Notice queue (singleton)
│   │   ├─ ProCompatibility.php
│   │   ├─ Compatibility.php
│   │   ├─ AjaxActions.php
│   │   ├─ Streamer.php       # HLS/private video streaming
│   │   ├─ RewriteRulesManager.php
│   │   ├─ PreloadService.php
│   │   └─ NpsSurvey.php
│   ├─ Support/               # Base classes (Block base, Utility)
│   └─ css/                   # Compiled CSS (do not edit)
├─ src/                       # React/JS source (see Frontend-Architecture)
├─ dist/                      # Compiled assets (auto-generated)
├─ packages/
│   ├─ components/            # StencilJS web component library
│   └─ components-react/      # React adaptor
├─ templates/                 # PHP templates
├─ languages/                 # .pot / .po / .mo translation files
├─ tests/
│   ├─ unit/                  # PHPUnit unit tests
│   └─ feature/               # PHPUnit feature/integration tests
└─ tests-e2e/                 # Playwright / Jest E2E tests
```

---

## Pro Plugin Directory Map (`presto-player-pro/`)

```
presto-player-pro/
├─ presto-player-pro.php      # Entry point
├─ inc/
│   ├─ Plugin.php             # Bootstrap: filter registration, version check
│   ├─ config/app.php         # Pro component registry
│   ├─ Blocks/
│   │   ├─ BunnyCDNBlock.php  # Bunny.net stream block
│   │   └─ PlaylistBlock.php  # Playlist block
│   ├─ Contracts/
│   │   └─ EmailProviderInterface.php
│   ├─ Controllers/           # Bunny stream orchestrators
│   │   ├─ BunnyStreamController.php
│   │   ├─ BunnyVideoLibraryController.php
│   │   ├─ BunnyVideosController.php
│   │   ├─ BunnyCollectionsController.php
│   │   └─ BunnyPullZoneController.php
│   ├─ Libraries/             # API client wrappers
│   │   ├─ BunnyApiRequest.php
│   │   ├─ BunnyVideoApiRequest.php
│   │   └─ [Provider]Request.php (per email provider)
│   ├─ Models/
│   │   ├─ Bunny/             # Bunny settings models (ArrayAccess, SettingModel)
│   │   │   ├─ Account.php
│   │   │   ├─ PullZone.php
│   │   │   ├─ StorageZone.php
│   │   │   ├─ Collection.php
│   │   │   └─ Support/       # SettingModel base, SettingInterface
│   │   ├─ LicensedProduct.php
│   │   └─ Visit.php          # Analytics model + query methods
│   ├─ Services/
│   │   ├─ API/               # REST controllers (Bunny, email providers, analytics)
│   │   ├─ Bunny/             # BunnyService + CDN/Storage request classes
│   │   ├─ License/           # License system (excluded in VIP builds)
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
│   └─ Support/               # Base classes and utilities
├─ languages/
├─ composer.json              # Standard build
├─ composer-vip.json          # VIP build (no licensing)
├─ gulpfile.js                # Gulp build tasks
└─ package.json               # npm scripts
```

---

## Related Pages

- [Architecture-Overview](Architecture-Overview)
- [Block-Development](Block-Development)
- [Frontend-Architecture](Frontend-Architecture)
- [Database-Schema](Database-Schema)