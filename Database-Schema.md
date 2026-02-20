# Database Schema

Presto Player creates 6 custom database tables. All are prefixed with `{$wpdb->prefix}` (default: `wp_`). Table creation and versioned migrations are managed by `inc/Services/Migrations.php`.

---

## Table Summary

| Table | Class | Purpose |
|-------|-------|---------|
| `presto_player_videos` | `Database\Videos` v4 | Central video registry |
| `presto_player_presets` | `Database\Presets` v22 | Video player preset configs |
| `presto_player_audio_presets` | `Database\AudioPresets` v20 | Audio player preset configs |
| `presto_player_email_collection` | `Database\EmailCollection` v1 | Email gate settings per preset |
| `presto_player_visits` | `Database\Visits` v1 | Video watch analytics |
| `presto_player_webhooks` | `Database\Webhooks` v1 | Webhook endpoint configs |

---

## presto_player_videos

Tracks every video added through Presto Player, regardless of source type.

| Column | Type | Notes |
|--------|------|-------|
| `id` | bigint(20) unsigned PK | Auto-increment |
| `title` | varchar(255) NOT NULL | Video title |
| `type` | varchar(155) NOT NULL | Source type: `self_hosted`, `youtube`, `vimeo`, `bunny`, `audio` |
| `external_id` | varchar(155) NULL | YouTube/Vimeo/Bunny external ID |
| `attachment_id` | bigint(20) unsigned NULL | WordPress media library attachment ID |
| `post_id` | bigint(20) NULL | The post this video is embedded on |
| `src` | varchar(255) NULL | Video URL (self-hosted) |
| `created_by` | bigint(20) unsigned NULL | WordPress user ID |
| `created_at` | TIMESTAMP | Auto-set on insert |
| `updated_at` | TIMESTAMP | Auto-updated |
| `deleted_at` | TIMESTAMP NULL | Soft-delete timestamp |

**Indexes:** `external_id`, `attachment_id`, `created_at`, `updated_at`

---

## presto_player_presets

Stores named player presets. Each preset defines which controls appear and how the player behaves. JSON columns store complex sub-settings.

| Column | Type | Notes |
|--------|------|-------|
| `id` | bigint(20) unsigned PK | |
| `name` | varchar(155) NULL | Human-readable preset name |
| `slug` | varchar(155) NULL | Machine-readable identifier |
| `icon` | varchar(155) NULL | Icon identifier |
| `skin` | varchar(155) NULL | Visual skin name |
| `play-large` | boolean | Show large centre play button |
| `rewind` | boolean | Show rewind control |
| `play` | boolean | Show play/pause button |
| `fast-forward` | boolean | Show fast-forward control |
| `progress` | boolean | Show progress bar |
| `current-time` | boolean | Show current time display |
| `mute` | boolean | Show mute button |
| `volume` | boolean | Show volume slider |
| `speed` | boolean | Show speed control |
| `pip` | boolean | Show picture-in-picture button |
| `fullscreen` | boolean | Show fullscreen button |
| `captions` | boolean | Show captions toggle |
| `reset_on_end` | boolean | Reset to start when video ends |
| `auto_hide` | boolean | Auto-hide controls |
| `show_time_elapsed` | boolean | Show elapsed vs remaining time |
| `captions_enabled` | boolean | Enable captions by default |
| `save_player_position` | boolean | Resume from last position |
| `sticky_scroll` | boolean | Sticky mini-player on scroll |
| `sticky_scroll_position` | varchar(16) NULL | e.g. `bottom-right` |
| `on_video_end` | varchar(16) NULL | Action on end: `none`, `loop`, etc. |
| `play_video_viewport` | boolean | Auto-play when scrolled into view |
| `hide_youtube` | boolean | Hide YouTube branding |
| `lazy_load_youtube` | boolean | Lazy-load YouTube iframe |
| `hide_logo` | boolean | Hide Presto logo |
| `border_radius` | bigint(20) unsigned NULL | Player corner radius (px) |
| `caption_style` | varchar(155) NULL | Caption text style |
| `caption_background` | varchar(155) NULL | Caption background colour |
| `is_locked` | boolean | Prevent editing in UI |
| `cta` | LONGTEXT | JSON — call-to-action overlay config |
| `watermark` | LONGTEXT | JSON — watermark overlay config |
| `search` | LONGTEXT | JSON — in-video search config |
| `email_collection` | LONGTEXT | JSON — email gate config |
| `action_bar` | LONGTEXT | JSON — action bar config |
| `created_by` | bigint(20) unsigned NULL | |
| `created_at` | TIMESTAMP | |
| `updated_at` | TIMESTAMP | |
| `deleted_at` | TIMESTAMP NULL | Soft delete |

**Indexes:** `name`

---

## presto_player_audio_presets

Same control-flag pattern as video presets, tuned for audio players.

| Column | Type | Notes |
|--------|------|-------|
| `id` | bigint(20) unsigned PK | |
| `name` | varchar(155) NULL | |
| `slug` | varchar(155) NULL | |
| `icon` | varchar(155) NULL | |
| `skin` | varchar(155) NULL | |
| `rewind` | boolean | |
| `play` | boolean | |
| `play-large` | boolean | |
| `fast-forward` | boolean | |
| `progress` | boolean | |
| `current-time` | boolean | |
| `mute` | boolean | |
| `volume` | boolean | |
| `speed` | boolean | |
| `pip` | boolean | |
| `reset_on_end` | boolean | |
| `save_player_position` | boolean | |
| `sticky_scroll` | boolean | |
| `sticky_scroll_position` | varchar(16) NULL | |
| `on_video_end` | varchar(16) NULL | |
| `play_video_viewport` | boolean | |
| `show_time_elapsed` | boolean | |
| `hide_logo` | boolean | |
| `border_radius` | bigint(20) unsigned NULL | |
| `background_color` | varchar(155) NULL | Player background colour |
| `control_color` | varchar(155) NULL | Control icon colour |
| `is_locked` | boolean | |
| `cta` | LONGTEXT | JSON |
| `email_collection` | LONGTEXT | JSON |
| `action_bar` | LONGTEXT | JSON |
| `created_by` | bigint(20) unsigned NULL | |
| `created_at` | TIMESTAMP | |
| `updated_at` | TIMESTAMP | |
| `deleted_at` | TIMESTAMP NULL | |

**Indexes:** `name`

---

## presto_player_email_collection

Stores the email gate configuration attached to a preset. One row per email collection config.

| Column | Type | Notes |
|--------|------|-------|
| `id` | bigint(20) unsigned PK | |
| `enabled` | boolean | Gate active or not |
| `behavior` | varchar(155) NOT NULL | When to show: `before`, `after`, `percentage` |
| `percentage` | bigint(20) NULL | If behavior=`percentage`, the % point to show gate |
| `allow_skip` | boolean | Let user skip the email gate |
| `headline` | varchar(155) NOT NULL | Overlay headline text |
| `bottom_text` | varchar(155) NOT NULL | Below-form text |
| `button_text` | varchar(155) NOT NULL | Submit button label |
| `preset_id` | bigint(20) NULL | FK → `presto_player_presets.id` |
| `border_radius` | bigint(20) NOT NULL | Overlay corner radius |
| `email_provider` | varchar(155) NULL | Provider slug: `mailchimp`, `activecampaign`, etc. |
| `email_provider_list` | varchar(155) NULL | Provider list/audience ID |
| `email_provider_tag` | varchar(155) NULL | Provider tag to apply on submission |
| `created_by` | bigint(20) unsigned NULL | |
| `created_at` | TIMESTAMP | |
| `updated_at` | TIMESTAMP | |
| `deleted_at` | TIMESTAMP NULL | |

**Indexes:** `preset_id`

---

## presto_player_visits

Tracks individual video watch events for analytics (Pro feature).

| Column | Type | Notes |
|--------|------|-------|
| `id` | bigint(20) unsigned PK | |
| `user_id` | bigint(20) unsigned NULL | WordPress user ID (null = guest) |
| `duration` | bigint(20) unsigned NOT NULL | Seconds watched |
| `video_id` | bigint(20) unsigned NOT NULL | FK → `presto_player_videos.id` |
| `ip_address` | varchar(39) NULL | Hashed/stored IP for deduplication |
| `created_at` | TIMESTAMP | |
| `updated_at` | TIMESTAMP | |
| `deleted_at` | TIMESTAMP NULL | |

Analytics queries: `topVideos()`, `topUsers()`, `averageWatchTime()`, `timeline()` — all in `Pro\Models\Visit`.

---

## presto_player_webhooks

Stores outbound webhook endpoint definitions.

| Column | Type | Notes |
|--------|------|-------|
| `id` | bigint(20) unsigned PK | |
| `name` | varchar(155) NULL | Friendly name |
| `url` | varchar(255) NULL | Target URL |
| `method` | varchar(155) NULL | HTTP method: `POST`, `GET`, etc. |
| `email_name` | varchar(155) NULL | Email field name to pass |
| `headers` | varchar(255) NULL | Custom request headers (JSON string) |
| `created_by` | bigint(20) unsigned NULL | |
| `created_at` | TIMESTAMP | |
| `updated_at` | TIMESTAMP | |
| `deleted_at` | TIMESTAMP NULL | |

**Indexes:** `name`

---

## Migrations

Table creation and schema upgrades are handled by `inc/Services/Migrations.php`. Each table class stores a `$version` integer — if the stored DB version differs, `dbDelta()` is called to apply changes. Upgrades (column additions, index changes) live in `inc/Database/Upgrades/`.

---

## Related Pages

- [Architecture-Overview](Architecture-Overview)
- [REST-API-Reference](REST-API-Reference)
- [Pro-Plugin-Architecture](Pro-Plugin-Architecture)
