# BunnyCDN Integration

Presto Player Pro integrates with [Bunny.net](https://bunny.net) for video hosting and delivery. The integration covers two distinct Bunny products: **Bunny Stream** (video encoding and delivery library) and **Bunny Storage** (file storage with CDN pull zones).

---

## Architecture Overview

```
BunnyService (Services/Bunny/BunnyService.php)
  │  Main facade — URL signing, JS vars, user video sync
  │
  ├─ BunnyCDNBlock (Blocks/BunnyCDNBlock.php)
  │    Block that renders Bunny stream videos (public + private)
  │
  ├─ Controllers/
  │   ├─ BunnyStreamController       — library + pull zone setup
  │   ├─ BunnyVideoLibraryController — create/update video libraries
  │   ├─ BunnyVideosController       — upload/list/delete stream videos
  │   ├─ BunnyCollectionsController  — video collection management
  │   └─ BunnyPullZoneController     — fetch/configure pull zones
  │
  ├─ Libraries/
  │   ├─ BunnyApiRequest             — general Bunny API (CDN/storage API)
  │   ├─ BunnyVideoApiRequest        — Bunny Stream API
  │   └─ Services/Bunny/CDNRequest   — CDN/storage HTTP requests
  │
  └─ Models/Bunny/
      ├─ Account                     — account-level settings
      ├─ PullZone                    — pull zone config (ArrayAccess)
      ├─ StorageZone                 — storage zone config (ArrayAccess)
      └─ Collection                  — video collection config (ArrayAccess)
```

---

## Configuration Settings (wp_options)

| Option Key | Description |
|-----------|-------------|
| `presto_player_bunny_stream_public` | Public library: `video_library_id`, `video_library_api_key`, `pull_zone_id`, `pull_zone_url`, `token_auth_key` |
| `presto_player_bunny_stream_private` | Private library: same fields as public |
| `presto_player_bunny_stream` | General: `hls_start_level` (default 480p), `disable_legacy_storage` |
| `presto_player_bunny_pull_zone` | Pull zone details (stored via `PullZone` model) |
| `presto_player_bunny_storage_zone` | Storage zone details (stored via `StorageZone` model) |

---

## Public vs Private Videos

Bunny Stream supports two video library types:

| Type | Access | URL Signing | HLS Auth |
|------|--------|------------|----------|
| Public | Open — no auth | Not required | No token needed |
| Private | Restricted | Signed URLs with expiry | Token authentication |

The `BunnyCDNBlock` checks the video's library type and injects the appropriate signed URL and HLS configuration via:

```php
add_filter( 'presto_player/block/default_attributes', [$this, 'addVideoAttributes'] );
add_filter( 'presto_player/component/attributes',     [$this, 'injectSignedUrl'] );
```

---

## URL Signing

`BunnyService` signs video URLs before they are served to the browser:

1. Checks `BunnyService::isSetup()` — validates that pull zone, library, and token key are configured
2. On `presto_player_rest_prepared_response_item` filter: calls `maybeSignURL()` to replace the raw URL with a time-limited signed URL
3. Signed URL format: `https://{pull-zone-url}/{video-id}/playlist.m3u8?token={hash}&expires={timestamp}`

---

## Video Upload Flow (Bunny Stream)

1. User uploads via Media Hub block or WP admin
2. `RestBunnyStreamController` receives `POST /bunny-stream/videos`
3. `BunnyVideosController` calls `BunnyVideoApiRequest` to create a video in the library
4. File is uploaded to Bunny Stream via tus or direct upload
5. Bunny transcodes to HLS
6. `presto_player_videos_created` fires → `BunnyService::updateUserVideos()` stores `external_id` in user meta

---

## User Video Meta

`BunnyService` stores a list of each user's Bunny video IDs in user meta to scope API results:

```php
update_user_meta( $user_id, 'presto_bunny_video_ids', $video_ids );
```

This prevents users from seeing other users' videos in the Media Hub browser.

---

## Storage API (Legacy)

For direct file storage (not Stream), `Services/Bunny/Storage/StorageAPI.php` handles:
- File upload to Bunny storage zone
- File deletion
- Directory listing

> **Security Note (v3.0.2):** An arbitrary file deletion vulnerability in Bunny CDN API endpoints was patched. Always validate file paths and user permissions before processing delete requests.

---

## Setup Checklist

1. Obtain API key from Bunny.net dashboard
2. Create a video library (public or private)
3. In Presto Player settings → BunnyCDN → enter library ID + API key
4. Run the pull zone setup wizard (creates CDN pull zone automatically)
5. For private videos: enable token authentication in Bunny dashboard and copy the token auth key
6. Add `BunnyCDNBlock` to a post and select the video

---

## Related Pages

- [Pro-Plugin-Architecture](Pro-Plugin-Architecture)
- [REST-API-Reference](REST-API-Reference)
- [Settings-Reference](Settings-Reference)
- [Database-Schema](Database-Schema)
