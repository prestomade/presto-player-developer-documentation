# Hooks & Filters

Presto Player and Presto Player Pro are built to be customized. Almost everything — what gets rendered, who can view a private video, how long a signed CDN link stays valid, what happens when someone submits their email — runs through a WordPress **action** or **filter**, so you can change it from your own theme or a small helper plugin without touching Presto Player's code.

If you're new to this: an **action** lets you *run your own code* at a specific moment (`add_action`). A **filter** lets you *change a value* before Presto Player uses it (`add_filter`, and you must `return` something). Every example below is safe to drop into a site-specific plugin or your theme's `functions.php`.

This page is a complete reference generated from the plugin source, grouped by what you're trying to do. Use `Cmd/Ctrl+F` to jump to a hook name.

---

## Contents

- [Video Rendering & Blocks](#video-rendering--blocks)
- [Playback & View Tracking](#playback--view-tracking)
- [Private Videos & Access Control](#private-videos--access-control)
- [Licensing](#licensing)
- [REST API](#rest-api)
- [File Storage & Media Library](#file-storage--media-library)
- [Data Models (Videos, Presets, Webhooks…)](#data-models-videos-presets-webhooks)
- [Settings, Branding & Dynamic Text](#settings-branding--dynamic-text)
- [Pro: Bunny CDN Storage & Streaming](#pro-bunny-cdn-storage--streaming)
- [Pro: Video Transcription (Captions)](#pro-video-transcription-captions)
- [Pro: Email Collection Forms](#pro-email-collection-forms)
- [Pro: Analytics](#pro-analytics)
- [Pro: Third-Party API Requests (Bunny, Mailchimp, etc.)](#pro-third-party-api-requests-bunny-mailchimp-etc)
- [API Examples](#api-examples)
  - [PHP: Progress Tracking](#php-progress-tracking)
  - [PHP: Email Capture & Validation](#php-email-capture--validation)
  - [JS: Frontend Player Events](#js-frontend-player-events)

---

## Video Rendering & Blocks

These fire while a video block is turned into the HTML that gets sent to the browser — the most useful group if you want to change *what* renders.

### `presto_player/block/default_attributes` (filter)
The big one. Fires just before Presto Player assembles the final config object it hands to the `<presto-player>` element — skin, branding, preset, tracks, overlays, everything.

```php
add_filter( 'presto_player/block/default_attributes', function( $config, $attributes ) {
    $config['class'] .= ' my-custom-wrapper';
    return $config;
}, 10, 2 );
```
*Params:* `$config` (array), `$attributes` (array, original block attributes) · `inc/Support/Block.php:248`

### `presto_player_load_video` (filter)
Return `false` to stop a specific video from rendering — the go-to hook for gating a video behind your own membership/paywall logic.

```php
add_filter( 'presto_player_load_video', function( $load, $attributes, $content, $block_name ) {
    if ( ! empty( $attributes['id'] ) && ! current_user_can( 'read_private_videos' ) ) {
        return false;
    }
    return $load;
}, 10, 4 );
```
*Params:* `$load` (bool), `$attributes` (array), `$content` (string), `$name` (string, block slug) · `inc/Support/Block.php:527`

### `presto_player_load_video_fallback` (filter)
Only fires when the hook above blocked a video. Return your own HTML instead of the default "unauthorized" message.

```php
add_filter( 'presto_player_load_video_fallback', function( $fallback, $attributes, $content, $block ) {
    return '<p>Please log in to view this video.</p>';
}, 10, 4 );
```
*Params:* `$fallback` (bool|string), `$attributes` (array), `$content` (string), `$block` (Block instance) · `inc/Support/Block.php:529`

### `presto_video_block_attributes_override` (filter)
Fires earlier than `default_attributes`, right as the block's own saved attributes are read. Handy for overriding one attribute (e.g. force autoplay off for a specific provider) before the rest of the pipeline runs.

```php
add_filter( 'presto_video_block_attributes_override', function( $attributes, $block ) {
    if ( 'youtube' === $block->name ) {
        $attributes['autoplay'] = false;
    }
    return $attributes;
}, 10, 2 );
```
*Params:* `$attributes` (array), `$this` (Block instance) · `inc/Support/Block.php:198`

### `presto_player_block_data` (filter)
The final data handed to the template file, right before markup is generated — the last stop for tweaking what gets output.
*Params:* `$data` (array), `$block` (Block instance) · `inc/Support/Block.php:537`

### `presto_player/presto_player_presets/data` (filter)
Fires whenever a video or audio preset is resolved for a render. Use it to tweak preset values (skin, colors, watermark) at render time without editing the saved preset.

```php
add_filter( 'presto_player/presto_player_presets/data', function( $preset, $type ) {
    if ( 'video' === $type ) {
        $preset->skin = 'dark';
    }
    return $preset;
}, 10, 2 );
```
*Params:* `$preset` (Preset|AudioPreset object), `$type` (`'video'` or `'audio'`) · `inc/Support/Block.php:351` (video) and `:368` (audio)

### `presto_player_registered_block_types` (filter)
The list of block names Presto Player recognizes as "a video block" (`presto-player/self-hosted`, `presto-player/youtube`, etc.). Add your own if you're building a custom provider block on top of Presto Player.

```php
add_filter( 'presto_player_registered_block_types', function( $types ) {
    $types[] = 'presto-player/my-custom-provider';
    return $types;
} );
```
*Params:* `$block_types` (array) · `inc/Models/Block.php:20`

### `presto_player_get_block_from_content` (filter)
Fires once per block while Presto Player scans a post's content looking for a video. Useful if you wrap Presto Player blocks in your own container blocks and need to unwrap them here.
*Params:* `$block` (array|null, parsed block) · `inc/Models/Post.php:84`

### `presto_player/templates/player_tag` (action)
Fires literally inside the opening `<presto-player ...>` tag. The sanctioned way to add your own `data-*` attribute to the rendered player element.

```php
add_action( 'presto_player/templates/player_tag', function( $data ) {
    echo ' data-my-tracking-id="' . esc_attr( $data['id'] ) . '"';
} );
```
*Params:* `$data` (array, same data driving the whole render) · `templates/video.php:42`

### `presto_player_video_schema_enabled` (filter) & `presto_player_video_schema` (filter)
Control the Google-style VideoObject JSON-LD schema Presto Player prints for SEO. Return `false` from the first to disable it for a video (e.g. private content you don't want indexed); use the second to add/adjust properties like `duration`.
*Params:* `$enabled` (bool) / `$data` (array) · and · `$schema` (array) / `$data` (array) · `inc/Support/Block.php:600` / `:629`

### `presto_player_before_block_output` (action)
Fires once per block instantiation, passing a reference to that block's internal `middleware()` gating method. This is an advanced/low-level hook — most people want `presto_player_load_video` instead, which is simpler and fires with real attributes. · `inc/Support/Block.php:119`

### `presto_player/scripts/load_iframe_fallback` (filter)
Controls the no-JS iframe fallback markup Presto Player prints for YouTube/Vimeo embeds. Presto Player turns this on automatically when needed; override if you need the fallback in a context it doesn't detect. · `inc/Services/Scripts.php:460`

---

## Playback & View Tracking

### `presto_player_progress` (action)
Fires every time the frontend reports playback progress. Great for triggering a "video completed" automation once someone crosses a percentage threshold.

```php
add_action( 'presto_player_progress', function( $id, $percent, $visit_time ) {
    if ( $percent >= 90 ) {
        do_action( 'my_plugin_video_completed', get_current_user_id(), $id );
    }
}, 10, 3 );
```
*Params:* `$id` (int, video ID), `$percent` (int), `$visit_time` (int|false, seconds watched) · `inc/Services/Player.php:86`

### `presto_player_daily_views_enabled` (filter)
Master switch for the lightweight daily view-counter used on the dashboard KPI widget. Return `false` to turn it off entirely.

```php
add_filter( 'presto_player_daily_views_enabled', '__return_false' );
```
*Params:* `$enabled` (bool, default `true`) · `inc/Services/Scripts.php:182`, `inc/Services/Usage.php:725` and `:747`

### `presto_player_load_js` (filter)
Controls whether Presto Player enqueues its main frontend JS bundle on the current page. Force it `true` if a player gets injected in a way Presto Player's auto-detection misses.

```php
add_filter( 'presto_player_load_js', function( $should_load ) {
    return $should_load || is_page( 'custom-video-landing' );
} );
```
*Params:* `$should_load` (bool) · `inc/Services/Scripts.php:447`

---

## Private Videos & Access Control

### `presto-player-show-private-video` (filter)
The main gate for private videos. Default behavior is "must be logged in" — override it to plug in your own membership/rules logic.

```php
add_filter( 'presto-player-show-private-video', function( $can_view, $id ) {
    return user_can( get_current_user_id(), 'view_premium_videos' );
}, 10, 2 );
```
*Params:* `$can_view` (bool, default: `is_user_logged_in()`), `$id` (int, video/attachment ID) · `inc/Models/CurrentUser.php:21`

---

## Licensing

Mostly relevant if you're building on top of Presto Player rather than just using it — white-labeling, VIP/enterprise deployments, or custom renewal flows.

| Hook | Type | Default | What it's for |
|---|---|---|---|
| `presto_player_license_api_url` | filter | `https://my.prestomade.com/index.php` | Point license checks at a different server (staging/mirror/on-prem) |
| `presto_player_license_domain` | filter | site's `wpurl`, scheme stripped | The domain string sent to the licensing server — useful behind reverse proxies |
| `presto_player_license_renew_url` | filter | `https://my.prestomade.com/my-account/` | The "Renew Now" link in the expired-license notice |
| `presto_player_license_pricing_url` | filter | `https://prestoplayer.com/pricing/` | The "Buy Subscription" link in the no-key notice |
| `presto_player_license_is_vip` | filter | computed | Whether this counts as a WP VIP-provisioned (always-licensed) build |
| `presto_player_force_license_check` | filter | `false` | Forces the real license check to run even under `PRESTO_TESTSUITE` |

All in `inc/Services/License/License.php` except the last, in `inc/Plugin.php:42`.

---

## REST API

### `presto_player_rest_prepared_database_item` (filter)
Fires right before a REST-submitted video item is written to the database. Add/validate/strip fields on create or update requests through `/wp-json/presto-player/v1/videos`.
```php
add_filter( 'presto_player_rest_prepared_database_item', function( $prepared ) {
    $prepared['source_ip'] = $_SERVER['REMOTE_ADDR'] ?? '';
    return $prepared;
} );
```
*Params:* `$prepared` (array) · `inc/Services/API/RestVideosController.php:408`

### `presto_player_rest_prepared_response_item` (filter)
Fires right before a video item is sent back in a REST response — add computed/custom fields for your own frontend to consume.
*Params:* `$prepared` (array) · `inc/Services/API/RestVideosController.php:427`

### Localized JS config filters
If you're extending the block editor or frontend JS and need extra data available on `window`, these are where Presto Player builds those payloads:

| Hook | JS global | File |
|---|---|---|
| `presto-settings-block-js-options` | `window.prestoPlayer` (frontend) | `Services/Scripts.php:160` |
| `presto_player_admin_script_options` | `prestoPlayer` (block editor) | `Services/Scripts.php:243` |
| `presto_player_admin_block_script_options` | `prestoPlayerAdmin` (block editor, incl. transcription endpoints) | `Services/Scripts.php:292` |

### `presto_player_after_plugin_activation` (action)
Fires after Presto Player's onboarding REST endpoint activates an allowlisted companion plugin.
*Params:* `$plugin_path` (string), `$plugin_slug` (string) · `inc/Services/PluginInstaller.php:136`

### `presto_player_learn_progress_saved` (action)
Fires after a user's progress through the in-dashboard "Learn" walkthrough is saved.
*Params:* `$progress` (array, `chapter_id => step_id => bool`) · `inc/Services/Learn.php:134`

---

## File Storage & Media Library

### `presto_player_private_foldername` / `presto_player_private_folder_name` (filters)
Two related filters (same purpose, different call sites — one at construction, one at folder-creation time) controlling the name of the private-media upload folder. Default: `presto-player-private`.
*Files:* `inc/Files.php:36` and `:236`

### `presto_player_get_external_attachments` (filter)
Presto Player hides externally-hosted video attachments from the Media Library grid by default. Return `true` to show them anyway.
*Params:* `$show` (bool, default `false`) · `inc/Files.php:109`

### `presto_player_redirect_legacy_media_list` (filter)
Presto Player redirects the old `edit.php?post_type=pp_video_block` admin screen to the new Media Hub dashboard. Return `false` to keep the classic list-table view.
*Params:* `$should_redirect` (bool, default `true`), `$target` (string, dashboard URL) · `inc/Services/VideoPostType.php:115`

---

## Data Models (Videos, Presets, Webhooks…)

Every Presto Player database table (videos, presets, audio presets, webhooks, email collection) shares the same base Model class, so these three hooks each expand into **one concrete hook per table**:

| Table | `_created` action | `_updated` action | `/data` filter |
|---|---|---|---|
| Videos | `presto_player_videos_created` | `presto_player_videos_updated` | `presto_player/presto_player_videos/data` |
| Presets | `presto_player_presets_created` | `presto_player_presets_updated` | `presto_player/presto_player_presets/data` |
| Audio Presets | `presto_player_audio_presets_created` | `presto_player_audio_presets_updated` | `presto_player/presto_player_audio_presets/data` |
| Webhooks | `presto_player_webhooks_created` | `presto_player_webhooks_updated` | `presto_player/presto_player_webhooks/data` |
| Email Collection | `presto_player_email_collection_created` | `presto_player_email_collection_updated` | `presto_player/presto_player_email_collection/data` |

> Note: the `/data` filter here is the low-level, fires-on-every-load-or-save filter (`inc/Models/Model.php:652`). It's a different, more general hook than the render-time `presto_player/presto_player_presets/data` filter documented under [Video Rendering & Blocks](#video-rendering--blocks) — same name, different job (that one only fires for presets at render time; this one fires for every table, every time a row is read or written).

**`_created`** fires right after a new row is inserted:
```php
add_action( 'presto_player_videos_created', function( $video ) {
    error_log( 'New Presto Player video created: ' . $video->id );
} );
```

**`_updated`** fires after any update (including trash/untrash, which use `update()` internally):
```php
add_action( 'presto_player_videos_updated', function( $video ) {
    // e.g. re-generate a cached embed snippet
} );
```

Both actions receive the Model instance · `inc/Models/Model.php:446` and `:696`

---

## Settings, Branding & Dynamic Text

### `presto_player_default_color` (filter)
The default brand accent color used as an editor default and styling fallback (`#00b3ff`). Change it to reskin the default player color site-wide.

```php
add_filter( 'presto_player_default_color', function() {
    return '#ff5722';
} );
```
*Params:* `$color` (string, hex) · `inc/Models/Setting.php:96`

### `presto-player/dynamic-data` (filter)
The list of `{token}` → value replacements available in dynamic watermark/overlay text (e.g. `{user.display_name}`, `{site.url}`). Add your own custom tokens here.

```php
add_filter( 'presto-player/dynamic-data', function( $values ) {
    $values['{company.name}'] = get_bloginfo( 'name' );
    return $values;
} );
```
*Params:* `$values` (array) · `inc/Support/DynamicData.php:23`

---

## Pro: Bunny CDN Storage & Streaming

### `presto_player_bunny_read_permissions` / `_connect_permissions` / `_upload_permissions` / `_sign_url_permissions` (filters)
Capability gates for the Bunny CDN REST endpoints (browse/list files, connect an account, upload, or request a signed private URL). Default requires `upload_files` + `edit_posts` (read), building up from there. Use to open Bunny management up to a custom role.

```php
add_filter( 'presto_player_bunny_upload_permissions', function( $allowed, $request ) {
    return $allowed && current_user_can( 'edit_others_posts' );
}, 10, 2 );
```
*Params:* `$allowed` (bool), `$request` (`WP_REST_Request`) · `inc/Services/API/RestBunnyCDNController.php` and `RestBunnyStreamController.php`

### `presto_player_allowed_mime_types` (filter)
The whitelist of video file extensions/MIME types accepted for Bunny storage and stream uploads/listing/deletion. Add or remove supported formats.

```php
add_filter( 'presto_player_allowed_mime_types', function( $mimes ) {
    $mimes['ts'] = 'video/mp2t'; // allow MPEG transport stream uploads
    unset( $mimes['wmv'] );
    return $mimes;
} );
```
*Params:* `$mimes` (array, extension pattern => MIME type) · `RestBunnyCDNController.php` and `RestBunnyStreamController.php`

### `presto_player_bunny_token_expires` (filter)
How long a signed (token-authenticated) private Bunny CDN URL stays valid. Default: 2 hours.

```php
add_filter( 'presto_player_bunny_token_expires', function( $seconds ) {
    return 6 * HOUR_IN_SECONDS;
} );
```
*Params:* `$seconds` (int, default `2 * HOUR_IN_SECONDS`) · `inc/Services/Bunny/URL.php:22`

### `presto_player_bunny_stream_default_library_config` (filter)
Default security settings applied when creating/updating a Bunny Stream video library (token auth, IP verification, MP4 fallback, etc.).

```php
add_filter( 'presto_player_bunny_stream_default_library_config', function( $config, $type, $id ) {
    if ( 'private' === $type ) {
        $config['EnableTokenIPVerification'] = true;
    }
    return $config;
}, 10, 3 );
```
*Params:* `$config` (array), `$type` (`'public'` or `'private'`), `$id` (int, library ID) · `inc/Controllers/BunnyVideoLibraryController.php:157`

---

## Pro: Video Transcription (Captions)

### `presto_player_transcribe_capability` (filter)
Who's allowed to trigger AI caption/transcription generation. Default requires `upload_files` + `edit_posts`.

```php
add_filter( 'presto_player_transcribe_capability', function( $allowed, $request ) {
    return current_user_can( 'manage_options' );
}, 10, 2 );
```
*Params:* `$allowed` (bool), `$request` (`WP_REST_Request`) · `RestBunnyTranscriptionController.php:233`, `RestBunnyWebhookController.php:204`

### `presto_player_bunny_caption_formatted` (filter)
Fires once per caption/subtitle track as it's prepared for the player. Rewrite caption URLs, rename labels, or add metadata.

```php
add_filter( 'presto_player_bunny_caption_formatted', function( $formatted, $caption ) {
    $formatted['label'] = strtoupper( $formatted['srcLang'] );
    return $formatted;
}, 10, 2 );
```
*Params:* `$formatted` (array: `label`, `src`, `srcLang`, `version`, `source`), `$caption` (raw Bunny caption metadata) · `inc/Services/Bunny/BunnyTranscriptionService.php:251`

---

## Pro: Email Collection Forms

### `presto_player/pro/forms/save` (action)
The main integration point for leads — fires every time someone submits the email-collection form on a video, whether it's a new email or a repeat viewer.

```php
add_action( 'presto_player/pro/forms/save', function( $data, $preset, $post, $created ) {
    if ( $created ) {
        wp_mail( 'sales@example.com', 'New video lead', $data['email'] . ' watched a video and left their email.' );
    }
}, 10, 4 );
```
*Params:* `$data` (array: `email`, `firstname`, `lastname`, `video_id`, `preset_id`), `$preset` (Preset/AudioPreset object), `$post` (`WP_Post`, the submission record), `$created` (bool — `false` if this email already existed) · `inc/Services/EmailCollection/EmailCollectionAjaxHandler.php:71`

### `presto_player/pro/forms/validation` (filter)
Add your own validation rules to the email form (default only checks the email isn't empty).

```php
add_filter( 'presto_player/pro/forms/validation', function( $errors ) {
    if ( empty( $_POST['firstname'] ) ) {
        $errors[] = 'Please enter your first name.';
    }
    return $errors;
} );
```
*Params:* `$errors` (array of message strings) · `inc/Services/EmailCollection/EmailCollectionAjaxHandler.php:152`

### `presto_player_export_permissions_check` (filter)
Who can export collected email submissions via REST. Default: `current_user_can('export')`.
*Params:* `$allowed` (bool) · `inc/Services/API/RestEmailCollectionController.php:42`

---

## Pro: Analytics

### `presto_player_analytics_block` (filter)
Return `true` to silently skip recording a video view — useful for excluding internal/staff IPs or test accounts from analytics.

```php
add_filter( 'presto_player_analytics_block', function( $blocked, $ip, $user_id ) {
    $internal_ips = array( '203.0.113.10' );
    return in_array( $ip, $internal_ips, true ) ? true : $blocked;
}, 10, 3 );
```
*Params:* `$blocked` (bool, default `false`), `$ip` (string), `$id` (int, user ID, `0` if logged out) · `inc/Services/API/RestAnalyticsController.php:63`

### `presto_player_analytics_time_period` (filter)
How far back analytics/visit data is kept before the daily cron purges it. Default: `'90 days'`.

```php
add_filter( 'presto_player_analytics_time_period', function( $older_than ) {
    return '30 days';
} );
```
*Params:* `$older_than` (string, `strtotime()`-compatible) · `inc/Services/Visits.php:81`

### `presto_top_user_stats` (filter)
Add a custom column to the "Top Users" analytics report.

```php
add_filter( 'presto_top_user_stats', function( $row ) {
    $row['stats'][] = array(
        'id'    => 'account_type',
        'title' => 'Plan',
        'data'  => get_user_meta( $row['user']['id'], 'plan_type', true ),
    );
    return $row;
} );
```
*Params:* `$row` (array with `user` and `stats` keys) · `inc/Services/API/RestAnalyticsController.php:567`

---

## Pro: Third-Party API Requests (Bunny, Mailchimp, etc.)

Every outgoing request to Bunny.net, MailerLite, ActiveCampaign, or MailChimp goes through the same three filters — each one expands into **four concrete hook names**, one per integration (`bunny`, `mailerlite`, `active-campaign`, `mailchimp`):

| Filter pattern | Fires | Params |
|---|---|---|
| `presto_player_{key}_request_args` | right before the HTTP request goes out | `$args` (array, `wp_remote_request`-style), `$endpoint` (string) |
| `presto_player_{key}_request_endpoint` | same moment, for the URL path | `$endpoint` (string), `$args` (array) |
| `presto_player_{key}_request_response` | after a successful response comes back | `$response` (decoded JSON object), `$args`, `$endpoint` |

So in practice you'd hook e.g. `presto_player_bunny_request_args` or `presto_player_mailchimp_request_response`.

```php
add_filter( 'presto_player_bunny_request_args', function( $args, $endpoint ) {
    $args['timeout'] = 30; // Bunny uploads can be large
    return $args;
}, 10, 2 );

add_filter( 'presto_player_bunny_request_response', function( $response, $args, $endpoint ) {
    error_log( 'Bunny API response for ' . $endpoint . ': ' . wp_json_encode( $response ) );
    return $response;
}, 10, 3 );
```
*File:* `inc/Support/ApiRequest.php:37-71`

### `presto_player_license_domain` (Pro)
Pro has its own copy of the same license-domain filter documented under [Licensing](#licensing) above, for the `LicensedProduct` model used by add-ons. Same purpose, same signature. · `inc/Models/LicensedProduct.php:52`

---

## API Examples

Prefer copy-pasting a working file over reading a reference table? We keep a small collection of ready-to-use snippets in [prestomade/api-examples](https://github.com/prestomade/api-examples) — the full source for everything below lives there, feel free to fork it and open a PR if you've built something worth sharing.

### PHP: Progress Tracking

Hooks `presto_player_progress` and loads the video's own data using the `Video` model.

```php
/**
 * Do something on video progress
 *
 * @param integer $id      Video ID. Not a video hub id, but the id created
 *                         when someone adds a video anywhere.
 * @param integer $percent Progress percentage, 0-100, in multiples of 10.
 */
function custom_do_something_with_progress( $id, $percent ) {
    // Get video data with the ID using our video model.
    $video = new \PrestoPlayer\Models\Video( $id );

    $title = $video->title;
    $src   = $video->src;

    // Maybe do your own custom action.
    do_action( 'my_action', $video, $percent );
}
add_action( 'presto_player_progress', 'custom_do_something_with_progress', 10, 2 );
```
Source: [`php-actions.php`](https://github.com/prestomade/api-examples/blob/main/php-actions.php)

### PHP: Email Capture & Validation

Sends Pro email-collection submissions to your own list, plus adds custom validation to reject a submission before it saves.

```php
/**
 * Do something with email capture.
 *
 * @param array                       $data       Submission data (already validated).
 * @param \PrestoPlayer\Models\Preset $preset     Preset used for the email collection.
 * @param \WP_Post                    $email_post The pp_email_submission post storing the capture.
 * @param boolean                     $created    True if created, false if an existing entry was updated.
 */
function custom_do_something_with_email( $data, $preset, $email_post, $created ) {
    $email = $data['email'];

    if ( $created ) {
        custom_add_person_to_list( $email ); // add a person to a list, or perform some other action
    } else {
        custom_update_person_on_list( $email );
    }
}
add_action( 'presto_player/pro/forms/save', 'custom_do_something_with_email', 10, 4 );

/**
 * Validate email submission — reject with your own error message if needed.
 * Read the raw input from the global $_POST array.
 *
 * @param array $errors
 * @return array
 */
function custom_validate_email_input( $errors ) {
    if ( ! my_custom_validation_function( $_POST['email'] ) ) {
        $errors[] = __( 'This email address is blacklisted.', 'textdomain' );
    }
    return $errors;
}
add_filter( 'presto_player/pro/forms/validation', 'custom_validate_email_input' );
```
Source: [`email-capture.php`](https://github.com/prestomade/api-examples/blob/main/email-capture.php) — see also [Pro: Email Collection Forms](#pro-email-collection-forms) above for the full hook reference.

### JS: Frontend Player Events

Plain JavaScript, not PHP — these fire in the browser as someone actually watches a video, using WordPress's own `wp.hooks` JS library (the same one block editor extensions use). Every player on the page fires them, so this is the easiest way to react to playback without touching PHP at all.

They all follow the pattern `presto.player{Event}`, and every callback receives the underlying [plyr.io](https://github.com/sampotts/plyr) player instance.

```js
wp.hooks.addAction( 'presto.playerReady', 'my-plugin-namespace', ( player ) => {
  console.log( { player } );
} );

wp.hooks.addAction( 'presto.playerTimeUpdate', 'my-plugin-namespace', ( player ) => {
  console.log( 'current time is ' + player.currentTime );
} );
```

| Event | Fires when |
|---|---|
| `presto.playerReady` | The player instance is ready for API calls |
| `presto.playerPlay` | Playback resumes after being paused |
| `presto.playerPlaying` | Playback begins (first time, after pausing, or after replaying) |
| `presto.playerPause` | Playback is paused |
| `presto.playerTimeUpdate` | The current playback time changes |
| `presto.playerEnded` | Playback finishes (doesn't fire if autoplay looping) |
| `presto.playerSeeked` | A seek operation completes |
| `presto.playerEnterFullScreen` | The player enters fullscreen |
| `presto.playerExitFullScreen` | The player exits fullscreen |
| `presto.playerHidden` | The tab is closed or loses focus while a video is playing |
| `presto.playerVisible` | The tab regains focus after having been hidden |

*Second argument to `addAction` is just your own namespace string (like a slug) — use something unique to your plugin/theme so WordPress doesn't collide it with someone else's.*

Source: [`js-actions.js`](https://github.com/prestomade/api-examples/blob/main/js-actions.js) · verified against `packages/components/src/components/core/player/functions/actions.js`

---

*Didn't find the hook you were looking for, or think one's missing from this page? Open an issue on this repo, or ping us — we keep this page in sync with the plugin source.*
