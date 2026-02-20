# Email Providers

Presto Player Pro can gate video access behind an email collection form. Captured emails are forwarded to integrated marketing platforms.

---

## Supported Providers

| Provider | Service Class | REST Controller |
|----------|--------------|-----------------|
| ActiveCampaign | `Services\ActiveCampaign\ActiveCampaign` | `RestActiveCampaignController` |
| Mailchimp | `Services\Mailchimp\Mailchimp` | `RestMailchimpController` |
| MailerLite | `Services\MailerLite\MailerLite` | `RestMailerLiteController` |
| FluentCRM | `Services\FluentCRM\FluentCRM` | `RestFluentCRMController` |
| Webhooks | `Services\Webhooks\Webhooks` | `RestWebhooksController` |

FluentCRM is automatically marked as `connected` if the FluentCRM plugin is active (no API key needed).

---

## Integration Pattern

All email providers extend the `EmailProvider` abstract class and implement `EmailProviderInterface`:

```php
interface EmailProviderInterface {
    public function handle( array $data, object $preset ): bool;
}

abstract class EmailProvider implements EmailProviderInterface {
    // Registered on presto_player/pro/forms/save action
    public function register(): void {
        add_action( 'presto_player/pro/forms/save', [ $this, 'maybeHandle' ], 10, 2 );
    }

    public function maybeHandle( array $data, object $preset ): void {
        // Check provider is connected before delegating to handle()
        if ( ! Setting::get( $this->key, 'connected' ) ) {
            return;
        }
        $this->handle( $data, $preset );
    }
}
```

Each provider's `handle()` method calls the provider's API via its `*Request` library class.

---

## Email Gate Flow

1. Video plays; gate overlay appears based on `behavior` setting (`before`, `after`, `percentage`)
2. Viewer enters email and submits
3. WordPress fires `do_action('presto_player/pro/forms/save', $data, $preset)`
4. Each active provider's `maybeHandle()` runs in sequence
5. Connected providers forward the email to their API
6. Email is stored locally in `presto_player_email_collection` table
7. Webhook triggers fire if webhooks are configured

---

## Email Collection DB Table

Emails collected by the gate are stored in `presto_player_email_collection`. See [Database-Schema](Database-Schema) for full column list.

Key fields: `behavior`, `percentage`, `allow_skip`, `email_provider`, `email_provider_list`, `email_provider_tag`.

---

## API Request Libraries

Each provider has a request library that extends `ApiRequest`:

```
Libraries/
├─ ActiveCampaignRequest.php
├─ MailchimpRequest.php
├─ MailerLiteRequest.php
└─ FluentCRMRequest.php
```

`ApiRequest` base class provides: `get()`, `post()`, `put()`, `patch()`, `delete()` with JSON encoding and error handling.

---

## Webhook Provider

Webhooks send a POST (or custom method) to a configured URL on each email submission.

Configuration stored in `presto_player_webhooks` table:

| Field | Description |
|-------|-------------|
| `url` | Target endpoint |
| `method` | HTTP method |
| `email_name` | Key name for the email field in the payload |
| `headers` | Custom request headers (JSON) |

Multiple webhooks can be configured and all fire on each email collection event.

---

## Email Export

Collected emails can be exported as CSV via `Services\EmailExport`. The export endpoint is available in the Presto Player admin under Settings → Email Collection.

---

## Adding a Custom Provider

1. Create a class extending `EmailProvider` in your own plugin
2. Implement `handle(array $data, object $preset): bool`
3. Register it as a component via the `presto_player_pro_components` filter:

```php
add_filter( 'presto_player_pro_components', function( $components ) {
    $components[] = MyCustomProvider::class;
    return $components;
} );
```

---

## Related Pages

- [Pro-Plugin-Architecture](Pro-Plugin-Architecture)
- [REST-API-Reference](REST-API-Reference)
- [Database-Schema](Database-Schema)
- [Settings-Reference](Settings-Reference)
