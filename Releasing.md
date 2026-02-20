# Core Version
https://share.zight.com/Z4upOb5j

Absolutely. Here is the consolidated step-by-step list for the Presto Player Pro release process, formatted entirely in Markdown for clarity and easy reading.

## 📝 Presto Player Core Release Process

This checklist details the steps from preparation through to post-release verification.

### 1. 🛠️ Preparation and Branching

* **Create Release Branch:** Create a dedicated branch from `master` (e.g., `release/vX.Y.Z`). All subsequent changes for this version are committed here.
* **Update Version Numbers:** Update the stable version string in the main plugin file (`presto-player.php`) and in `readme.txt` to the new version (e.g., **v1.2.3**).
* **Draft Changelog:** Finalize the release notes/changelog entry in the appropriate file (e.g., `changelog.txt`).
* **Final Code Review:** Perform a final, focused code review on the dedicated release branch.
* **Run Final QA/Tests:** Execute full quality assurance and testing on a staging environment using the code from the release branch.
* **Get Final Approval:** Share the release branch for final review and sign-off from a second developer/stakeholder.

### 2. 🚀 Release and Deployment

* **Merge to Master:** **Merge the `release/vX.Y.Z` branch into `master`** (using a **non-squash** merge to preserve individual commits).
* **Create & Publish Release Tag:** Create and publish the version tag on GitHub (e.g., `v1.2.3`) pointing to the newly merged `master` commit. **This action triggers the deployment.**
* **Monitor Deployment:** Check the progress and confirm the successful completion of the triggered GitHub Action (or equivalent CI/CD pipeline).

### 3. ✅ Verification and Communication

* **Check Successful Release:** Verify that the update notification is successfully received on a test WordPress site running an older version of the plugin.
* **Quick Sanity Check (Live):** Update a test site with the **latest released version** and quickly verify critical functionality (e.g., video playback, licensing).
* **Internal Communication:** Inform the **Support Team** and internal stakeholders via the dedicated channel (e.g., `#pp-dev`).
* **Public Communication:** Publish the official changelog and release notes on the product website or blog.

### 4. 🧹 Cleanup

* **Delete Release Branch:** Delete the temporary `release/vX.Y.Z` branch.
* **Reset for Development:** Ensure your main development branch (e.g., `develop`) is synced with `master` to begin work on the next version.

---


# Pro Version
https://share.zight.com/X6unmqdO

# Vip Version
https://share.zight.com/NQu58L4O

```
Hash code : Woo: 18734000847108:b51fbd0ce349b853bb7da3b849f488b5

/**
 * Plugin Name: Presto Player Pro
 * Plugin URI: http://prestoplayer.com/
 * Description: Pro extension for Presto Player to enable chapters, private video and more.
 * Version: 1.2.2
 * Author: Presto Player
 * Text Domain: presto-player-pro
 * Tags: private, video, lms, hls
 * Domain Path: languages
 * Woo: 18734000847108:b51fbd0ce349b853bb7da3b849f488b5
 */
```
* Update plugin main file with header ^
* Integrate VIPCS rulesets & solve all cases
* Make sure to remove all prestomade licensing stuff (fields, SDK)
* VIPCS ruleset: https://github.com/brainstormforce/astra-addon/blob/next-release/phpcs.xml.dist
* Provide a zip file, named presto-player-pro.zip, with the source code. Top root directory in the zip must be presto-player-pro. The file size cannot exceed 64 MB.
* Review final & share zip to upload :tada:

---

## Build Commands Reference

### Free Plugin (`presto-player/`)

```bash
# Full release build (recommended)
yarn plugin:release
```

This command runs:
1. `composer install` — install PHP dependencies
2. `yarn makepot` — regenerate translation POT file
3. `composer install --no-dev` — strip dev dependencies
4. `yarn build:workspace` — build all workspace packages
5. `yarn build` — build main plugin assets
6. Creates distributable ZIP

Individual steps:
```bash
yarn build              # Production JS/CSS build
yarn build:workspace    # Build workspace packages (components, components-react)
yarn makepot            # Regenerate languages/presto-player.pot
```

### Pro Plugin (`presto-player-pro/`)

```bash
# Standard (licensed) build
npm run build:license
# Runs: composer install → makepot → composer install --no-dev → gulp buildPro

# VIP build (no licensing)
npm run build:vip
# Runs: COMPOSER=composer-vip.json composer update → makepot → gulp buildVip

# Create both variants + ZIP files
npm run package
# Output in package/pro/ and package/vip/
```

### VIP Zip Requirements

- Top-level directory in ZIP must be `presto-player-pro`
- File size must not exceed 64 MB
- Must exclude `inc/Services/License/`
- Must pass VIPCS ruleset: `composer run lint`

---

## Version Bump Checklist

When bumping version numbers for a release:

**Free plugin:**
- `presto-player.php` — `Version:` header + `PRESTO_PLAYER_VERSION` constant
- `readme.txt` — `Stable tag:` + `== Changelog ==` entry

**Pro plugin:**
- `presto-player-pro.php` — `Version:` header
- `changelog.txt` — new version entry

---

## Related Pages

- [Contributing-Guide](Contributing-Guide)
- [Testing-Guide](Testing-Guide)
- [Pro-Plugin-Architecture](Pro-Plugin-Architecture)