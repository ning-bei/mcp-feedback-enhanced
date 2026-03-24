# MEMORY

## Per-Project Auto-Submit Feature (2026-03-24)
- Added `projectAutoSubmit` map to settings for per-project auto-submit overrides
- Settings stored in `~/.config/mcp-feedback-enhanced/ui_settings.json` under `projectAutoSubmit` key
- Key is project directory path, value is `{ enabled, timeout, promptId }`
- Resolution order: project override > global settings
- UI scope selector added in auto-submit settings card (Global / Project dropdown)
- Files changed: `settings-manager.js`, `app.js`, `feedback.html`, `styles.css`, all 3 locale files, `main_routes.py`
- Backend: fixed Starlette 1.0.0 `TemplateResponse` API (request as first arg)
- i18n: use `{name}` single braces for `i18nManager.t()` interpolation; removed `data-i18n` from dynamically-managed elements to prevent `applyTranslations()` from overwriting JS-set text

## Fix: Project Scope Selector Empty in Other Projects (2026-03-24)
- Bug: Settings Scope selector only showed "Global" in other projects (no "Project: xxx" option)
- Root cause: `currentProjectDirectory` in `SettingsManager` was only set via WebSocket events, which arrive AFTER `applyToUI()` runs during init
- Fix: Added DOM-based initialization in `app.js` that reads `data-full-path` from `#projectPathDisplay` (rendered by Jinja2 template) before calling `applyToUI()`
- This ensures the project directory is available immediately on page load, even before WebSocket connects

## Fix: Browser Cache Preventing Code Updates (2026-03-24)
- Bug: Code changes to JS/CSS not taking effect due to aggressive browser caching
- Root cause 1: All script tags had hardcoded `?v=2025010510` that never changed
- Root cause 2: `Cache-Control: public, max-age=3600` made browsers not check for updates for 1 hour
- Fix 1: Replaced hardcoded version with `{{ cache_bust }}` Jinja2 template variable using `_startup_ts` (server startup timestamp)
- Fix 2: Changed JS/CSS cache strategy from `max-age=3600` to `no-cache` (always revalidate via ETag)
- Files changed: `main_routes.py` (added `_startup_ts` and `cache_bust` context), `feedback.html`, `index.html` (all `?v=` references), `compression_config.py` (JS/CSS cache headers)
