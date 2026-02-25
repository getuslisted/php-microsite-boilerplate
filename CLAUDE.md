# CLAUDE.md — PHP Microsite Boilerplate

This file provides essential context for AI assistants working on this codebase.

## Project Overview

**PHP Microsite Boilerplate** (v2.0.16) is a lightweight PHP framework for building small, fast, and secure websites deployable on virtually any hosting environment — including shared hosting. It prioritizes simplicity, performance (Lighthouse 100), security, and SEO out of the box.

- **License:** GPL 3.0
- **PHP version:** 8.0 (via Docker; compatible with most shared hosts)
- **Primary language:** PHP (procedural, minimal OOP)
- **Demo:** https://phpmicrosite.jekuer.com/

---

## Repository Structure

```
php-microsite-boilerplate/
├── assets/
│   ├── css/            # style.css (source), style.min.css (built output)
│   ├── fonts/          # Open Sans web fonts
│   ├── images/         # Site images and icons
│   ├── favicons/       # PWA and browser favicons
│   └── js/             # base.js (source), all.min.js (built output)
├── cache/              # Runtime cache for Directus API responses
├── class/
│   └── class.page.php  # Page object class (slug resolution, meta, Directus)
├── controller/         # Pre-view PHP files (business logic before HTML)
│   ├── index.php       # Default empty controller
│   └── sample_directus.php
├── lib/                # Core library functions
│   ├── helper_functions.php    # Language switching, array/SVG utilities
│   ├── url_parsing.php         # URL + query string parsing (UTF-8 safe)
│   ├── sitemap_generator.php   # Auto XML sitemap generation
│   ├── directus_connect.php    # Directus v8/v9 API client
│   ├── directus_dyn_pages.php  # Dynamic page generation from CMS
│   ├── cache_purge_rebuild.php # Cache management and purge hooks
│   └── gettext/                # Gettext fallback library
├── nginx_conf/         # Nginx config (alternative to Apache .htaccess)
│   ├── nginx.conf
│   ├── 7g-firewall.conf
│   └── nginx_deployment.sh
├── pages/              # Page view files (one PHP file per page/language)
│   ├── main_en.php, main_de.php, main_es.php
│   ├── legal-notice_en.php, legal-notice_de.php, legal-notice_es.php
│   ├── privacy-policy_en.php, privacy-policy_de.php, privacy-policy_es.php
│   ├── error.php, offline.php
│   └── sample_directus.php
├── templates/          # Shared HTML scaffolding
│   ├── header.php          # <head>, GTM, service worker, CSS includes
│   ├── footer.php          # JS includes, language switcher, </body>
│   ├── general_meta.php    # SEO meta, Open Graph, Twitter cards
│   └── php_security_headers.php  # Alternative server-side security headers
├── translations/       # Gettext .po/.mo files
│   ├── de_DE/LC_MESSAGES/main.po, main.mo
│   └── es_ES/LC_MESSAGES/main.po, main.mo
├── config.php          # All non-code configuration (languages, Directus, PWA, meta)
├── index.php           # Single application entry point (~7100 lines)
├── routing.php         # Page definitions and slug/meta configuration
├── redirects.php       # URL redirect rules
├── tailwind.config.js  # TailwindCSS configuration
├── postcss.config.js   # PostCSS pipeline (Tailwind, autoprefixer, cssnano, purgecss)
├── Gruntfile.js        # JS minification tasks
├── package.json        # NPM build dependencies
├── docker-compose.yml  # Local dev container (PHP-Apache 8.0)
├── .htaccess           # Apache: security headers, 7G firewall, caching, SSL rewrites
├── manifest.json       # PWA manifest
├── serviceworker-cache.js / .min.js  # PWA service worker
└── robots.txt          # SEO robots configuration
```

---

## Architecture: How the Application Works

### Single Entry Point

All requests route through `index.php`. It:
1. Loads `config.php` and `routing.php`
2. Detects language from URL prefix or browser headers
3. Parses the URL to find the matching page in `$pages`
4. Optionally fetches data from Directus CMS
5. Instantiates the `Page` object (`class/class.page.php`)
6. Includes the controller (from `controller/`) if defined
7. Includes `templates/header.php`
8. Includes the view (from `pages/`)
9. Includes `templates/footer.php`

### Routing

Pages are defined in `routing.php` as entries in the `$pages` array, keyed by language code then page ID:

```php
$id = 'about';
$pages['en'][$id]['view'] = 'about_en';        // file in pages/ (without .php)
$pages['en'][$id]['controller'] = 'about';     // file in controller/ (optional)
$pages['en'][$id]['name'] = 'About Us';
$pages['en'][$id]['title'] = 'About | Site';
$pages['en'][$id]['description'] = 'Meta description.';
$pages['en'][$id]['slug'] = 'about-us';        // optional custom slug
$pages['en'][$id]['robots'] = 'noindex,nofollow'; // optional
$pages['en'][$id]['sitemap'] = false;          // optional, default true
$pages['en'][$id]['directus_collection'] = 'mypages'; // optional Directus
$pages['en'][$id]['directus_id'] = '1';        // optional Directus
```

Reserved IDs that **must** exist for each language:
- `main` — home page (URL: `/` or `/lang/`)
- `error` — 404 page
- `offline` — PWA offline page

### Language System

- Default language is configured in `config.php` via `$language['default']`
- Available languages defined in `$language['available']` (associative array)
- Language is detected from URL prefix (e.g., `/de/`, `/es/`) or browser Accept-Language
- Gettext is used for string translation with a fallback library if the PHP extension is unavailable
- Translation files are `.po`/`.mo` files named `main` in `translations/{locale}/LC_MESSAGES/`
- Each language needs its own page view files (e.g., `main_en.php`, `main_de.php`)

### Directus CMS Integration (Optional)

Configured in `config.php`. When `$directus_url` is set:
- Per-page Directus data is fetched via `lib/directus_connect.php`
- Data is accessible inside page views via `$the_page->directus['FIELD_NAME']`
- API responses are cached locally in `cache/` when `$directus_cache = true`
- Cache can be purged via `YOURDOMAIN.com/purge/directus_cache`
- Dynamic pages from Directus collections are defined via the `$directus_pages` array in `routing.php`

---

## Development Workflow

### Local Development (Docker)

```bash
# 1. Set base URL for local dev in config.php:
#    $the_page_url = '/';

# 2. Start the container
docker-compose up -d

# 3. Open http://localhost:80
```

The Docker container uses `webdevops/php-apache:8.0` with PHP error reporting enabled.

### Building CSS and JS Assets

Node.js and npm are required for the build pipeline.

```bash
npm install        # Install build dependencies
npm run build      # Runs: grunt && postcss ./assets/css/style.css -o ./assets/css/style.min.css
```

**What the build does:**
- **Grunt:** Minifies `assets/js/base.js` → `assets/js/all.min.js` + minifies `serviceworker-cache.js`
- **PostCSS:** Processes `assets/css/style.css` through TailwindCSS, autoprefixer, cssnano, and PurgeCSS → `assets/css/style.min.css`

The built files (`style.min.css`, `all.min.js`) are what `templates/header.php` and `templates/footer.php` load via versioned query strings (e.g., `style.min.css?v=2.0.16`).

**Important:** After any CSS or JS change, run `npm run build` before testing. When working on TailwindCSS classes, PurgeCSS scans PHP files to determine which utility classes to keep.

### Adding a New Page

1. Define the page in `routing.php` for each language in `$language['available']`
2. Create the view file in `pages/` (e.g., `pages/newpage_en.php`)
3. Optionally create a controller in `controller/` for data fetching
4. The page is automatically included in the sitemap unless `sitemap = false`

### Adding a New Language

1. Add entry to `$language['available']` in `config.php` (e.g., `'fr' => 'Français'`)
2. Add entry to `$language['locale']` (e.g., `'fr' => 'fr_FR'`)
3. Add entry to `$language['directus']` if using Directus
4. Create `translations/fr_FR/LC_MESSAGES/main.po` and compile to `main.mo`
5. Add all required page entries for the new language in `routing.php`
6. Create the view files in `pages/` for each page in the new language
7. Update `serviceworker-cache.js` to include the new language prefix

---

## Code Conventions

### PHP

- **Style:** Procedural code; snake_case for variables and functions; camelCase for class methods
- **Globals:** Configuration values (`$language`, `$pages`, `$directus_*`) are global; access them with `global $varname` inside functions
- **Input sanitization:** Use `filter_var()` and `make_safe()` (from `lib/helper_functions.php`) on all user input
- **Page object:** Access current page data via `$the_page` (instance of `Page` class)
  - `$the_page->id` — page ID
  - `$the_page->slug` — URL slug
  - `$the_page->meta['title']` — page title
  - `$the_page->directus['FIELD']` — Directus CMS data
- **Views:** Plain PHP files in `pages/`; use `<?= ?>` for output; keep business logic in controllers
- **Translations:** Use `_('String to translate')` or `gettext('String')` for translated strings

### JavaScript

- camelCase for variables and functions
- `'use strict'` in service worker
- Cookie helpers (`getCookie`, `setCookie`) available in `base.js`
- YouTube embeds use a deferred/privacy-safe loading pattern

### CSS / TailwindCSS

- Source file: `assets/css/style.css`
- Use TailwindCSS utility classes in PHP/HTML templates
- Custom CSS goes in `assets/css/style.css` (PostCSS nesting is supported)
- PurgeCSS scans all `.php`, `.html`, and `.js` files to remove unused Tailwind classes
- After editing Tailwind config (`tailwind.config.js`), re-run `npm run build`

### Templates

- `templates/header.php` — outputs everything from `<!DOCTYPE html>` through the opening `<main>` tag
- `templates/footer.php` — outputs closing tags, JS scripts, and language switcher
- Meta tags are managed in `templates/general_meta.php`; they pull from `$the_page->meta`
- Security headers are set in `.htaccess` (Apache) or via `templates/php_security_headers.php`

---

## Security Considerations

- **Never** expose `$directus_password` or other credentials in committed code — use server environment variables
- The deployment hook (`$the_deployment_slug`) and cache purge URL (`$random_cache_code`) should use hard-to-guess values
- The `.htaccess` includes a 7G Firewall, CSP headers, HSTS, X-Frame-Options, and MIME sniffing protection
- All user-supplied input must be sanitized before use
- The `deploy.php` file (not included in the repo) must perform checksum validation before executing git pull

---

## Configuration Reference (`config.php`)

| Variable | Purpose |
|---|---|
| `$version_nr` | Site version; used as cache-busting query string on assets |
| `$language['default']` | Default language code |
| `$language['available']` | Array of `code => name` for all languages |
| `$language['locale']` | Gettext locale per language |
| `$the_page_url` | Absolute base URL (use `'/'` for local Docker dev) |
| `$directus_url` | Directus API URL; leave empty to disable |
| `$directus_version` | `'8'` or `'9'` |
| `$directus_cache` | Cache API responses locally |
| `$random_cache_code` | Secret code for cache purge endpoint |
| `$the_deployment_slug` | URL slug that triggers deployment script |
| `$the_gtm_id` | Google Tag Manager ID; leave empty to disable |
| `$the_webapp_name` | PWA app name (also update `manifest.json`) |
| `$the_webapp_status` | Enable/disable PWA manifest link |
| `$the_theme_color` | PWA theme color (also update `manifest.json`) |
| `$the_page_meta_defaults` | Default title, description, keywords, author, robots |

---

## No Test Framework

This project has no automated tests (no PHPUnit, Jest, etc.). Validation is done manually:
- **PHP:** Test in Docker locally (`docker-compose up -d`)
- **Performance:** Google Lighthouse (target: 100)
- **PWA:** Browser DevTools Application tab
- **CSS:** W3C CSS Validator
- **HTML:** W3C HTML Validator

---

## Deployment

### Apache (default)
- Upload files; `.htaccess` handles routing and security automatically

### Nginx
- Use `nginx_conf/nginx_deployment.sh` to configure
- All non-file requests must be redirected to `index.php`

### Git-based Deployment Hook
- Set `$the_deployment_slug` in `config.php` to a private slug
- Create a `deploy.php` at the project root (not in the repo) that runs `git pull`
- Trigger via POST to `YOURDOMAIN.com/{deployment_slug}`

### Cache Purging
- Directus cache: `GET YOURDOMAIN.com/purge/directus_cache?purge_rebuild_code={code}`
- Add this URL as a webhook in Directus to auto-purge on content updates
