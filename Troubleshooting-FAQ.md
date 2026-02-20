# Troubleshooting & FAQ

---

## Development Environment

### `yarn bootstrap` fails or packages are missing

Run in order:
```bash
yarn
yarn bootstrap
```

`bootstrap` must be run after `yarn` on a fresh clone. It builds workspace packages (`packages/components`, `packages/components-react`) that the main plugin depends on.

### Changes to `src/` not appearing

Make sure the watch process is running:
```bash
yarn start          # watches main plugin src/
yarn start:workspace  # watches all workspace packages
```

If you changed something in `packages/components`, you need `start:workspace`.

### `composer install` fails with namespace conflicts

The Imposter plugin is responsible for re-namespacing vendor dependencies under `PrestoPlayer\`. If conflicts occur:
```bash
composer remove typisttech/imposter-plugin
composer install
composer require typisttech/imposter-plugin
```

---

## wp-env Issues

### Docker not found

wp-env requires Docker Desktop. Install it from [docs.docker.com/desktop](https://docs.docker.com/desktop/) and ensure it is running before:

```bash
yarn wp-env start
```

### wp-env won't start / port conflict

Check if something is already on port 3333:
```bash
lsof -i :3333
```

Change the port in `.wp-env.json` or stop the conflicting process.

### PHP tests fail with "WordPress test suite not found"

```bash
yarn wp-env start   # ensure environment is running first
yarn test:php
```

The test suite is installed inside the Docker container — it must be running.

### Both plugins not loading in wp-env

Check `.wp-env.json` in `presto-player/` — it should reference both `presto-player` and `presto-player-pro`:

```json
{
  "plugins": [".", "../presto-player-pro"]
}
```

Both plugin folders must be siblings in the `plugins/` directory.

---

## BunnyCDN

### Videos not playing / signed URL errors

1. Verify `token_auth_key` is set correctly in Presto Player → Settings → BunnyCDN
2. Confirm token authentication is enabled in Bunny.net dashboard for the library
3. Check that the pull zone URL does not have a trailing slash
4. Confirm server time is accurate — signed URLs include an expiry timestamp

### Bunny CDN setup wizard fails

1. Check API key has sufficient permissions (Stream API key, not account key)
2. Verify the library ID is correct
3. Check server error logs: `WP_DEBUG_LOG` in `wp-config.php`

### File deletion error after v3.0.2 patch

The arbitrary file deletion vulnerability was patched in v3.0.2. If you are seeing permission errors on delete endpoints, ensure you are running v3.0.2+ and that the user has `upload_files` capability.

---

## License (Pro)

### License activation fails

- Ensure the site URL matches what was registered at `my.prestomade.com`
- Try activation with `https://` if the site supports SSL
- Check for response code: `s100` = new activation, `s101` = renewal — both are success codes

### License page not appearing

The license page only appears if `Services/License/` is included (standard Pro build). VIP builds exclude the license system by design.

---

## Email Providers

### Emails not forwarding to provider

1. Check provider is marked as connected: Settings → Email Providers → [Provider]
2. Confirm the list/audience ID is set in the email collection preset config
3. Enable `WP_DEBUG_LOG` and look for API errors from the provider request library
4. For FluentCRM: it auto-detects connection status based on plugin activity — no API key needed

---

## Block Editor

### Block not appearing in inserter

1. Ensure `yarn build` has been run and `dist/` is populated
2. Check browser console for JS errors
3. Verify the block class is in `inc/config/app.php` components array
4. Run `wp-env start` and check PHP error logs for block registration errors

### Preset changes not applying

Presets are loaded at block insertion time. To apply a changed preset to an existing block, re-select the preset in the block's sidebar panel.

---

## Database

### Tables not created after activation

Run the migration manually:
```php
// In wp-cli or a throwaway snippet:
do_action('presto_player_activate');
```

Or deactivate and reactivate the plugin from WP admin.

### `presto_player_visits` table growing too large

Enable the purge option: Settings → Analytics → Purge old data. The `presto_daily_cron` action handles cleanup.

---

## Related Pages

- [Getting-Started](Getting-Started)
- [BunnyCDN-Integration](BunnyCDN-Integration)
- [Testing-Guide](Testing-Guide)
- [Pro-Plugin-Architecture](Pro-Plugin-Architecture)
