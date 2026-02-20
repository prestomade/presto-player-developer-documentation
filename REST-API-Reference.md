# REST API Reference

All endpoints use the WordPress REST API and are namespaced under `/wp-json/presto-player/v1/`.

---

## Authentication

- All write endpoints require the caller to be a logged-in WordPress user with appropriate capabilities.
- Read endpoints vary — some are public, others require `edit_posts` or `manage_options`.
- All permission callbacks accept `apply_filters('presto_player_*_permissions', ...)` for customisation.

---

## Free Plugin Endpoints

### Videos — `RestVideosController`

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| `POST` | `/videos` | `upload_files` + `edit_posts` | Create a video record |
| `GET` | `/videos` | `edit_posts` | List all videos |
| `GET` | `/videos/{id}` | `edit_posts` | Get a single video |
| `PUT/PATCH` | `/videos/{id}` | `upload_files` + `edit_posts` | Update a video |
| `DELETE` | `/videos/{id}` | `upload_files` + `edit_posts` | Delete a video |

### Presets — `RestPresetsController`

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| `POST` | `/presets` | `manage_options` | Create a preset |
| `GET` | `/presets` | `edit_posts` | List presets |
| `GET` | `/presets/{id}` | `edit_posts` | Get preset |
| `PUT/PATCH` | `/presets/{id}` | `manage_options` | Update preset |
| `DELETE` | `/presets/{id}` | `manage_options` | Delete preset |

### Audio Presets — `RestAudioPresetsController`

Same CRUD pattern as Presets, at `/audio-presets`.

### Settings — `RestSettingsController`

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| `GET` | `/settings` | `manage_options` | Get all plugin settings |
| `POST` | `/settings` | `manage_options` | Update settings |

All `wp_options` settings registered with `show_in_rest` are also exposed through the standard WordPress Settings REST API (`/wp/v2/settings`).

---

## Pro Plugin Endpoints

### Bunny CDN — `RestBunnyCDNController`

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| `GET` | `/bunny-cdn` | `manage_options` | Get CDN settings |
| `POST` | `/bunny-cdn` | `manage_options` | Save CDN settings |
| `GET` | `/bunny-cdn/pull-zones` | `manage_options` | List Bunny pull zones |
| `GET` | `/bunny-cdn/storage-zones` | `manage_options` | List storage zones |

### Bunny Stream — `RestBunnyStreamController`

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| `GET` | `/bunny-stream` | `manage_options` | Get stream settings |
| `POST` | `/bunny-stream` | `manage_options` | Save stream settings |
| `GET` | `/bunny-stream/libraries` | `manage_options` | List video libraries |
| `POST` | `/bunny-stream/libraries` | `manage_options` | Create a library |
| `GET` | `/bunny-stream/videos` | `upload_files` | List stream videos |
| `POST` | `/bunny-stream/videos` | `upload_files` | Upload to stream |
| `DELETE` | `/bunny-stream/videos/{id}` | `upload_files` | Delete stream video |
| `GET` | `/bunny-stream/collections` | `upload_files` | List collections |

### Email Collection — `RestEmailCollectionController`

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| `POST` | `/email-collection` | Public | Submit email from video gate |
| `GET` | `/email-collection` | `manage_options` | List collected emails |

### Analytics — `RestAnalyticsController`

| Method | Endpoint | Permission | Description |
|--------|----------|------------|-------------|
| `GET` | `/analytics/videos` | `manage_options` | Top videos by watch time |
| `GET` | `/analytics/users` | `manage_options` | Top users by watch time |
| `GET` | `/analytics/timeline` | `manage_options` | Watch time timeline |
| `GET` | `/analytics/average` | `manage_options` | Average watch time |

### Email Providers

Each provider has its own REST controller following the same pattern:

| Controller | Base | Description |
|-----------|------|-------------|
| `RestActiveCampaignController` | `/activecampaign` | AC connection + list management |
| `RestMailchimpController` | `/mailchimp` | Mailchimp audiences + tags |
| `RestMailerLiteController` | `/mailerlite` | MailerLite groups |
| `RestFluentCRMController` | `/fluentcrm` | FluentCRM lists + tags |
| `RestWebhooksController` | `/webhooks` | Webhook CRUD |

---

## Controller Pattern

All REST controllers follow this structure:

```php
class RestMyController extends \WP_REST_Controller {

    protected $namespace = 'presto-player';
    protected $version   = 'v1';
    protected $base      = 'my-resource';

    public function register(): void {
        add_action( 'rest_api_init', [ $this, 'register_routes' ] );
    }

    public function register_routes(): void {
        register_rest_route(
            "{$this->namespace}/{$this->version}",
            '/' . $this->base,
            [
                [
                    'methods'             => \WP_REST_Server::READABLE,
                    'callback'            => [ $this, 'get_items' ],
                    'permission_callback' => [ $this, 'get_items_permissions_check' ],
                ],
            ]
        );
    }
}
```

---

## Related Pages

- [Architecture-Overview](Architecture-Overview)
- [Database-Schema](Database-Schema)
- [BunnyCDN-Integration](BunnyCDN-Integration)
- [Email-Providers](Email-Providers)
