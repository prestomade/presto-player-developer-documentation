# Settings Reference

All Presto Player settings are stored in `wp_options` and registered with REST API access via `register_setting()`. They are accessible at `/wp-json/wp/v2/settings` (with `manage_options` cap) and via the dedicated `/wp-json/presto-player/v1/settings` endpoint.

---

## Free Plugin Settings

### `presto_player_analytics`
Internal video analytics (watch tracking).

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enable` | boolean | `false` | Enable watch time tracking |
| `purge_data` | boolean | `true` | Purge old analytics data automatically |

---

### `presto_player_branding`
Player visual customisation.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `logo` | string (URL) | `''` | Custom logo URL shown in player |
| `logo_width` | number | `150` | Logo width in px |
| `color` | string (hex) | Default accent | Player accent/highlight colour |
| `player_css` | string | `''` | Raw CSS injected into player styles |

---

### `presto_player_performance`
Performance and module loading.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `module_enabled` | boolean | `false` | Enable JS module loading |
| `automations` | boolean | `true` | Enable automation features |

---

### `presto_player_uninstall`
Data cleanup on deactivation.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `uninstall_data` | boolean | `false` | Delete all plugin data on uninstall |

---

### `presto_player_google_analytics`
GA4 event tracking for video events.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enable` | boolean | `false` | Enable GA tracking |
| `use_existing_tag` | boolean | `false` | Use existing GA tag on the site |
| `measurement_id` | string | `''` | GA4 Measurement ID (e.g. `G-XXXXXX`) |

---

### `presto_player_presets`
Default preset configuration.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `default_player_preset` | integer | `1` | ID of the default video preset |

---

### `presto_player_audio_presets`
Default audio preset configuration.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `default_player_preset` | integer | `1` | ID of the default audio preset |

---

### `presto_player_youtube`
YouTube-specific settings.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `nocookie` | boolean | `false` | Use `youtube-nocookie.com` embed domain |
| `channel_id` | string | `''` | YouTube channel ID |

---

### `presto_player_instant_video_width`
| Type | Default | Description |
|------|---------|-------------|
| string | `'800px'` | Default width for instant/popup video |

---

### `presto_player_media_hub_sync_default`
| Type | Default | Description |
|------|---------|-------------|
| boolean | `true` | Whether Media Hub sync is on by default for new videos |

---

## Pro Plugin Settings

### `presto_player_bunny_stream_public`
Public Bunny Stream video library.

| Key | Type | Description |
|-----|------|-------------|
| `video_library_id` | integer | Bunny video library ID |
| `video_library_api_key` | string | Library API key |
| `pull_zone_id` | integer | Associated pull zone ID |
| `pull_zone_url` | string | Pull zone CDN URL |
| `token_auth_key` | string | Token authentication key |

---

### `presto_player_bunny_stream_private`
Private Bunny Stream video library. Same keys as `presto_player_bunny_stream_public`.

---

### `presto_player_bunny_stream`
General Bunny Stream settings.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `hls_start_level` | integer | `480` | Default HLS quality level (height in px) |
| `disable_legacy_storage` | boolean | `false` | Disable legacy storage zone fallback |

---

### `presto_player_activecampaign`
| Key | Description |
|-----|-------------|
| `api_url` | AC instance URL |
| `api_key` | AC API key |
| `connected` | Connection status |

---

### `presto_player_mailchimp`
| Key | Description |
|-----|-------------|
| `api_key` | Mailchimp API key |
| `connected` | Connection status |

---

### `presto_player_mailerlite`
| Key | Description |
|-----|-------------|
| `api_key` | MailerLite API key |
| `connected` | Connection status |

---

### `presto_player_fluentcrm`
| Key | Description |
|-----|-------------|
| `connected` | Auto-set based on FluentCRM plugin active status |

---

## Accessing Settings in PHP

```php
use PrestoPlayer\Models\Setting;

// Get a single setting value
$color = Setting::get( 'branding', 'color' );

// Update
Setting::update( 'branding', 'color', '#ff0000' );
```

---

## Related Pages

- [Services-and-Integrations](Services-and-Integrations)
- [BunnyCDN-Integration](BunnyCDN-Integration)
- [Email-Providers](Email-Providers)
- [REST-API-Reference](REST-API-Reference)
