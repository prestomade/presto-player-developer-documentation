# Services and Integrations

All services and integrations are components — they implement `register()` and are auto-loaded by the DI container. See `inc/config/app.php` (free) and `presto-player-pro/inc/config/app.php` (pro) for the full lists.

---

## Free Plugin Services

| Service | File | Description |
|---------|------|-------------|
| `Scripts` | `Services/Scripts.php` | Enqueues all JS/CSS assets conditionally (block editor, frontend, Elementor, admin) |
| `Settings` | `Services/Settings.php` | Registers all `wp_options` settings with REST API exposure |
| `Menu` | `Services/Menu.php` | Creates the Presto Player admin menu and submenu pages |
| `Player` | `Services/Player.php` | Frontend player rendering and template loading |
| `Shortcodes` | `Services/Shortcodes.php` | Registers `[presto_player]`, `[presto_playlist]`, `[presto_popup]`, `[pptime]` |
| `Migrations` | `Services/Migrations.php` | Runs DB table creation and versioned upgrades on plugin activation |
| `Translation` | `Services/Translation.php` | Loads plugin text domain |
| `Blocks` | `Services/Blocks.php` | Block category registration |
| `VideoPostType` | `Services/VideoPostType.php` | Registers the Video CPT used by Media Hub |
| `ReusableVideos` | `Services/ReusableVideos.php` | CRUD for saved/reusable video configurations |
| `AdminNotices` | `Services/AdminNotices.php` | Manages admin notice queue (singleton) |
| `ProCompatibility` | `Services/ProCompatibility.php` | Backwards-compat shims between core and pro versions |
| `Compatibility` | `Services/Compatibility.php` | Third-party plugin compatibility (caching plugins, etc.) |
| `AjaxActions` | `Services/AjaxActions.php` | WordPress AJAX handlers for admin interactions |
| `NpsSurvey` | `Services/NpsSurvey.php` | Periodic NPS survey prompt |
| `PreloadService` | `Services/PreloadService.php` | Preloads player assets for performance |
| `Streamer` | `Services/Streamer.php` | HLS/private video streaming handler |
| `RewriteRulesManager` | `Services/RewriteRulesManager.php` | Custom URL rewrite rules for private video access |
| `AdminNotice` | `Services/AdminNotice.php` | Single notice model (used by AdminNotices) |

### Block Services

| Service | Description |
|---------|-------------|
| `Services\Blocks\YoutubeBlockService` | YouTube nocookie, channel ID, and extra player options injection |
| `Services\Blocks\VimeoBlockService` | Vimeo-specific player configuration |
| `Services\Blocks\PopupTriggerService` | Popup trigger JS vars and AJAX registration |

---

## Pro Plugin Services

| Service | Description |
|---------|-------------|
| `Pro\Services\Settings` | Registers all pro `wp_options` (Bunny, email providers) |
| `Pro\Services\Translation` | Loads pro text domain |
| `Pro\Services\Visits` | Hooks into player events to record watch data; daily cron cleanup |
| `Pro\Services\Bunny\BunnyService` | URL signing, JS vars injection, user video meta sync |
| `Pro\Services\EmailCollection\EmailCollection` | Registers email gate hooks |
| `Pro\Services\EmailExport` | CSV export of collected emails |
| `Pro\Services\GoogleAnalytics` | GA4 event tracking for video events |
| `Pro\Services\License\*` | License activation, validation, auto-update (excluded in VIP builds) |
| `Pro\Services\ActiveCampaign\ActiveCampaign` | Email provider integration |
| `Pro\Services\Mailchimp\Mailchimp` | Email provider integration |
| `Pro\Services\MailerLite\MailerLite` | Email provider integration |
| `Pro\Services\FluentCRM\FluentCRM` | Email provider integration (auto-connected if plugin active) |
| `Pro\Services\Webhooks\Webhooks` | Outbound webhook triggers on email collection |
| `Pro\Services\CoreInstaller` | Detects/prompts for core plugin installation if only pro is active |

---

## LMS Integrations

| Integration | Trigger | Features |
|-------------|---------|----------|
| `Integrations\LearnDash` | `sfwd-lms` active | Video progression enforcement, `ld_video_provider` filter, LearnDash settings fields |
| `Integrations\Tutor\Tutor` | `tutor` active | Video progression for TutorLMS lessons |
| `Integrations\Lifter\Lifter` | `lifterlms` active | Video completion for LifterLMS |

All LMS integrations hook into `plugins_loaded` and bail early if their parent plugin is not active.

---

## Page Builder Integrations

| Integration | Description |
|-------------|-------------|
| `Integrations\Elementor\Elementor` | Registers Presto Player Elementor widgets |
| `Integrations\Divi\Divi` | Registers Presto Player Divi modules |
| `Integrations\BeaverBuilder\BeaverBuilder` | Registers Presto Player Beaver Builder modules |
| `Integrations\Kadence` | Kadence theme compatibility |

---

## Key WordPress Filters

These are the main extension points exposed to developers:

| Filter | Description |
|--------|-------------|
| `presto_player_pro_components` | Add components to the pro component list |
| `presto_player/block/default_attributes` | Modify block default attribute values |
| `presto_player/component/attributes` | Modify player component data at render time |
| `presto_player_rest_prepared_response_item` | Modify REST API response items |
| `presto_player_allowed_mime_types` | Extend allowed video MIME types |
| `presto_player_*_permissions` | Override REST endpoint permission callbacks |
| `presto_player_admin_block_script_options` | Inject JS variables into admin block editor |
| `presto-settings-js-options` | Inject JS variables into settings page |
| `presto_player/templates/player_tag` | Add HTML attributes to the player element |

## Key WordPress Actions

| Action | Description |
|--------|-------------|
| `presto_player_videos_created` | Fires after a new video record is created |
| `presto_player_videos_updated` | Fires after a video record is updated |
| `presto_player/pro/forms/save` | Fires when an email gate form is submitted |
| `presto_daily_cron` | Daily maintenance (visit data cleanup, etc.) |
| `presto_player_settings_header` | Before settings page header |
| `presto_player_settings_footer` | After settings page footer |

---

## Related Pages

- [Architecture-Overview](Architecture-Overview)
- [Pro-Plugin-Architecture](Pro-Plugin-Architecture)
- [Email-Providers](Email-Providers)
- [BunnyCDN-Integration](BunnyCDN-Integration)
