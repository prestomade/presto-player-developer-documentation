Welcome to the developer-documentation wiki!

---

# Presto Player Developer Documentation

**Presto Player** (v4.0.8) is a WordPress video player plugin supporting YouTube, Vimeo, self-hosted HTML5, HLS, and Bunny.net. Built with a PHP DI container backend and React/StencilJS frontend, designed specifically for the Gutenberg block editor.

**Presto Player Pro** (v3.0.2) extends the free plugin with BunnyCDN, private video, email collection, analytics, and email marketing integrations.

---

## Quick Navigation

### Getting Started
- [Getting-Started](Getting-Started) — Dev environment setup, build commands
- [Contributing-Guide](Contributing-Guide) — Code standards, PR checklist, branch conventions

### Architecture
- [Architecture-Overview](Architecture-Overview) — Boot flow, DI container, layer map
- [Plugin-Structure](Plugin-Structure) — Directory maps for both plugins
- [Frontend-Architecture](Frontend-Architecture) — Yarn workspaces, StencilJS, React admin

### Blocks & API
- [Block-Development](Block-Development) — Registering blocks, filters, shortcodes
- [REST-API-Reference](REST-API-Reference) — All endpoints (free + pro)
- [Settings-Reference](Settings-Reference) — All wp_options settings

### Backend
- [Services-and-Integrations](Services-and-Integrations) — All services, filters, actions
- [Database-Schema](Database-Schema) — 6 custom tables with full column docs
- [Pro-Plugin-Architecture](Pro-Plugin-Architecture) — Pro bootstrap, licensing, VIP builds

### Pro Features
- [BunnyCDN-Integration](BunnyCDN-Integration) — Stream, storage, URL signing
- [Email-Providers](Email-Providers) — ActiveCampaign, Mailchimp, MailerLite, FluentCRM, Webhooks

### Quality & Release
- [Testing-Guide](Testing-Guide) — PHPUnit, Jest, Playwright E2E
- [Releasing](Releasing) — Release process for core, pro, and VIP
- [Troubleshooting-FAQ](Troubleshooting-FAQ) — Common issues and fixes

---

## Plugin Requirements

| | Free | Pro |
|-|------|-----|
| PHP | 7.3+ | 7.3+ |
| WordPress | 6.3+ | requires core >= 1.1.0 |
| Current version | 4.0.8 | 3.0.2 |
