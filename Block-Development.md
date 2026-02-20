# Block Development

Presto Player is built block-first. Each video source type is its own Gutenberg block, registered via a PHP class and a React editor component.

---

## Registered Block Types

### Free Plugin (`presto-player`)

| Block Class | Block Name | Description |
|-------------|------------|-------------|
| `Blocks\SelfHostedBlock` | `presto-player/self-hosted` | HTML5 / local media |
| `Blocks\YouTubeBlock` | `presto-player/youtube` | YouTube embed |
| `Blocks\VimeoBlock` | `presto-player/vimeo` | Vimeo embed |
| `Blocks\ReusableVideoBlock` | `presto-player/reusable-video` | Saved/reusable video |
| `Blocks\ReusableEditBlock` | `presto-player/reusable-edit` | Reusable video editor view |
| `Blocks\AudioBlock` | `presto-player/audio` | Audio player |
| `Blocks\MediaHubBlock` | `presto-player/media-hub` | Media hub browser |
| `Blocks\PopupBlock` | `presto-player/popup` | Video popup container |
| `Blocks\PopupTriggerBlock` | `presto-player/popup-trigger` | Button that opens popup |
| `Blocks\PopupMediaBlock` | `presto-player/popup-media` | Media inside popup |

### Pro Plugin (`presto-player-pro`)

| Block Class | Description |
|-------------|-------------|
| `Pro\Blocks\BunnyCDNBlock` | Bunny.net stream block (public + private) |
| `Pro\Blocks\PlaylistBlock` | Video playlist block |

---

## How to Register a New Block

### 1. Create the PHP Block Class

Place it in `inc/Blocks/MyNewBlock.php` (free) or `presto-player-pro/inc/Blocks/MyNewBlock.php` (pro):

```php
<?php

namespace PrestoPlayer\Blocks;

use PrestoPlayer\Support\Block;

class MyNewBlock extends Block {

    public function register(): void {
        register_block_type(
            __DIR__ . '/../../dist/blocks/my-new-block',
            [
                'render_callback' => [ $this, 'render' ],
                'attributes'      => $this->getAttributes(),
            ]
        );
    }

    public function render( array $attributes, string $content ): string {
        // Server-side render logic.
        return '';
    }
}
```

### 2. Register in Component List

Add the class to the `components` array in `inc/config/app.php`:

```php
\PrestoPlayer\Blocks\MyNewBlock::class,
```

For Pro: add to `presto-player-pro/inc/config/app.php` instead.

### 3. Create the React Editor Component

Block editor UI lives in `src/`. Create your block's JS at `src/blocks/my-new-block/index.js` and wire it up with `registerBlockType()` following WP block API conventions.

### 4. Build

```bash
yarn build         # or yarn start for watch mode
```

---

## Block Attribute Filters

Two filters let Pro (and custom code) inject attributes into any block before it renders:

```php
// Modify default attribute values
add_filter( 'presto_player/block/default_attributes', function( $attributes ) {
    $attributes['myCustomAttr'] = 'value';
    return $attributes;
} );

// Modify the data passed to the player component
add_filter( 'presto_player/component/attributes', function( $attributes ) {
    $attributes['signed_url'] = my_sign_url( $attributes['src'] );
    return $attributes;
} );
```

`BunnyCDNBlock` uses both filters to inject signed URLs and HLS library options for private videos.

---

## Block Services

Some blocks have a companion **block service** for additional hook registration:

| Service | Purpose |
|---------|---------|
| `Services\Blocks\YoutubeBlockService` | YouTube nocookie / channel ID injection |
| `Services\Blocks\VimeoBlockService` | Vimeo-specific player options |
| `Services\Blocks\PopupTriggerService` | Popup trigger JS integration |

---

## Shortcode Equivalent

All block types are accessible via shortcode for classic/page-builder contexts:

| Shortcode | Purpose |
|-----------|---------|
| `[presto_player id=""]` | Render a saved reusable video by ID |
| `[presto_playlist id=""]` | Render a playlist |
| `[presto_popup id=""]` | Render a video popup |
| `[presto_timestamp time=""]` / `[pptime time=""]` | Inline timestamp link |

---

## Related Pages

- [Architecture-Overview](Architecture-Overview)
- [Frontend-Architecture](Frontend-Architecture)
- [Plugin-Structure](Plugin-Structure)
