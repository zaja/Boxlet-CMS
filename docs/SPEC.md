# Boxlet — Build Specification

> This file is the contract. Read it at the start of every session.
> If a change conflicts with anything in "Frozen Contracts", stop and ask.

---

## 1. Product thesis

Boxlet is a small, self-hosted PHP CMS for people who build **many small sites**:
freelancers, small agencies, anyone who currently reaches for WordPress and then
spends a day deleting things.

The differentiator is **not** the feature list. It is the design layer: you pick a
character and a handful of decisions, and the whole site takes on a distinct look
without anyone writing CSS. Everything else (pages, media, forms) is table stakes and
must stay boring and small.

Guiding philosophy: **constrained freedom**. Page builders give unlimited control and
users produce ugly sites. Boxlet offers a bounded option space where every combination
looks acceptable. When in doubt, remove a knob.

### Non-goals (do not build these)

- User accounts, roles, permissions. One admin, full stop.
- A visitor-facing account area, registration, comments.
- Blog/post types, taxonomies, e-commerce, plugins-as-marketplace.
- Anything requiring Node, npm, or a build step at install time.
- Anything requiring shell access, cron, or Composer on the target server.
- A REST/GraphQL API in v1.

---

## 2. Target environment

| Constraint | Value |
|---|---|
| PHP | 8.1 minimum. Do not use 8.2+ syntax. |
| Database | SQLite by default, MySQL/MariaDB optional |
| Server | Shared Apache or Nginx, no shell, no Composer |
| Extensions required | pdo, pdo_sqlite, mbstring, fileinfo, json |
| Extensions optional | pdo_mysql, gd or imagick, intl |
| Install method | Upload ZIP (vendor/ included), open `/install.php` |
| Assets | Shipped pre-built. No npm in the release artifact. |

Deploy target for the release ZIP: under 8 MB, install to first page in under
two minutes.

---

## 3. Closed dependency list

This list governs **runtime** dependencies (`composer require`). Adding anything to it
requires explicit approval. Do not introduce a library because it is convenient.

```
nikic/fast-route          routing
vlucas/phpdotenv          .env parsing
symfony/mailer            SMTP transport
monolog/monolog           logging
spomky-labs/otphp         TOTP for 2FA
bacon/bacon-qr-code       QR rendering for 2FA enrolment
```

**Deliberately excluded:** any full framework, any ORM, Twig (plain PHP templates with
an `e()` helper are sufficient because all block templates are first-party), and
intervention/image (wrap GD/Imagick directly — we need perhaps six operations, not a
whole imaging library).

Admin front-end: vanilla JS plus one small drag-and-drop helper, hand-written CSS using
the same token variables as the front-end. No Tailwind CDN in production, no Alpine, no
HTMX. The admin is small enough that a framework is a liability.

### require-dev

Development tools are not covered by the closed list. They live in `require-dev`,
never ship in the release ZIP (built with `--no-dev`) and never exist on a user's
server. They are allowed when they earn their place, but ask first.

```
phpstan/phpstan           static analysis, level 6, phpVersion 8.1, no baseline
```

---

## 4. Directory layout

```
/public/                      ← document root (or public_html)
  index.php                   front controller
  install.php                 installer, self-disables
  .htaccess
  /assets/                    admin + front-end CSS/JS, shipped built
  /uploads/                   original media, never modified
  /cache/                     generated images, tokens.css, page cache
/app/
  /Core/                      Container, Router, Request, Response, Db, Config,
                              View, Cache, Hooks, Migrator
  /Modules/
    /Pages/  /Media/  /Design/  /Forms/  /I18n/  /Settings/  /Mailer/  /Ai/
    /{Module}/views/          front-end templates owned by that module
  /Admin/                     controllers, views, middleware
  /Blocks/                    one directory per block: block.php + template.php
  /Support/
/config/
/storage/                     logs, cache, sessions, backups (not web-accessible)
/lang/                        admin UI strings, one PHP file per locale
/migrations/                  NNNN_name.sql, applied in filename order
/vendor/
.env.example
```

`/app`, `/storage`, `/config` must sit outside the document root when the host allows
it. When it does not, the installer writes a `Require all denied` .htaccess into each
and warns the user.

---

## 5. Frozen contracts

Changing any of these after v0.1 ships is a breaking change. Get them right now.

### 5.1 URL scheme

The **primary locale** is chosen at install time and is immutable. It renders with no
URL prefix. Every additional locale always carries its prefix.

```
/{slug}                       page in the primary locale
/                             home page in the primary locale
/{locale}/{slug}              page in an additional locale
/{locale}/                    home page in an additional locale
/admin/...                    admin
/m/{preset}/{id}-{slug}.{ext} media
```

A first path segment is a locale **only if it is an enabled locale**. It is never
detected by its shape. Anything else is part of a primary-locale slug.

Canonical redirects are 301. That is safe because the primary locale cannot change.

```
/{primary}/{slug}             301 → /{slug}
/{primary}/  and  /{primary}  301 → /
/{locale}                     301 → /{locale}/
```

Example with primary `en` and `hr` enabled:

```
/hello        200, English         /en/hello     301 → /hello
/             home, English        /en/          301 → /
/hr/hello     200, Croatian        /hr           301 → /hr/
/hr/          home, Croatian       /de/hello     404 (slug "de/hello"; de not enabled)
/nope         404                  /hr/nope      404
```

A code that is not enabled is never redirected to another language; that would be a
soft 404. All ISO 639-1 codes are reserved as top-level page slugs and rejected on
save, so a page can never collide with a locale enabled later.

Every URL is generated by one helper (`app/Support/Url.php`). No template or
controller concatenates a path.

Locale is present in the router from the first commit. There is no code path that
renders a page without knowing its locale.

### 5.2 Database schema (v1)

```sql
pages (
  id, content_group_id, locale, slug, title, status,
  template_id, parent_id, sort, seo_json,
  translation_status,        -- 'source' | 'ai_draft' | 'reviewed'
  source_hash,               -- hash of the source-locale page when translated
  published_at, created_at, updated_at
)
-- unique (locale, slug), index (content_group_id)

page_blocks (
  id, page_id, block_group_id, block_type, sort,
  content_json,              -- the editable content
  style_json,                -- layer-2 section style, see §5.4
  translation_status, source_hash,
  created_at, updated_at
)
-- block_group_id links the same block across locales

page_revisions (id, page_id, data_json, created_at)

templates (id, name, layout_json, preview_image, is_builtin)

media (
  id, filename, original_name, path, mime, size,
  width, height, hash, focal_x, focal_y, created_at
)
media_meta (id, media_id, locale, alt, caption)

forms (id, name, slug, fields_json, settings_json)
form_submissions (id, form_id, data_json, ip_hash, created_at)

design_tokens (id, group_key, value_json)
settings (key, value_json)
locales (code, label, is_primary, fallback, sort, enabled)
ui_translations (id, key, locale, value)   -- overrides for /lang files
admin (id, email, password_hash, totp_secret, recovery_codes_json, created_at)
migrations (filename, applied_at)
```

Multilingual model: **row-based**. A page exists once per locale, tied together by
`content_group_id`. Blocks are tied together by `block_group_id`. This allows a locale
to diverge structurally, and makes per-block staleness detection possible.

`source_hash` is the hash of the source content at translation time. When the source
changes, the hash no longer matches and the admin sees a "stale" badge on **that block
only**. Re-translation is never all-or-nothing.

### 5.3 Block contract

Each block lives in `/app/Blocks/{type}/` with `block.php` and `template.php`.

```php
// /app/Blocks/hero/block.php
return [
    'type'     => 'hero',
    'label'    => 'Hero',
    'icon'     => 'hero',
    'version'  => 1,
    'fields'   => [
        'heading'   => ['type' => 'text',     'required' => true, 'translatable' => true],
        'subheading'=> ['type' => 'textarea', 'translatable' => true],
        'image'     => ['type' => 'media'],
        'cta'       => ['type' => 'link',     'translatable' => true],
    ],
    'layouts'  => ['left', 'center', 'split'],   // layer 3
    'defaults' => ['layout' => 'center'],
];
```

Field types (closed set for v1): `text`, `textarea`, `richtext`, `media`,
`media_multi`, `link`, `select`, `toggle`, `number`, `repeater`.

`translatable: true` marks a field the AI translator touches. Everything else is
copied verbatim across locales.

`template.php` receives `$content`, `$style`, `$layout` and outputs HTML that uses
**only** CSS custom properties for colour, spacing, radius, shadow and typography. A
block template containing a hard-coded colour or pixel value is a bug.

### 5.4 Design layers

```
Layer 0  Character    one preset: Editorial / Minimal / Bold / Soft / Brutalist
                      sets everything below to a coherent starting point
Layer 1  Tokens       eight decisions, not forty values:
                        1-2 seed colours → full palette generated with WCAG checks
                        typography pairing (curated list) + scale ratio
                        spacing base unit
                        radius character (none / subtle / round / pill)
                        shadow character (none / soft / hard / layered)
                        container width
Layer 2  Section      per block instance, stored in page_blocks.style_json:
                        surface:  plain | tinted | contrast | image | gradient
                        rhythm:   tight | normal | airy
                        width:    narrow | normal | wide | full
                        align:    left | center
                        divider:  none | line | slant | curve
Layer 3  Layout       per block type, from block.php 'layouts'
```

Layers 0 and 1 compile to `/public/cache/tokens.css` on save. Layers 2 and 3 render as
class names on the section wrapper. Nothing is inlined as a style attribute.

Palette generation must run a contrast check on every text/background pair it produces
and refuse (with a clear message) combinations below WCAG AA.

### 5.5 Media presets

Media URLs use **named presets only**, never free-form dimensions. Free parameters
turn the resize endpoint into a disk-filling vector.

```
thumb   200×200  crop
card    600×400  crop
wide    1200×630 crop
hero    1920×1080 crop
full    max 2400 wide, no crop
```

Pipeline: upload original untouched → validate with finfo against a MIME whitelist →
sha1 for dedup → generate variants lazily on first request → cache to
`/public/cache/media/` → serve via `<picture>` with AVIF → WebP → original.

The .htaccess rewrite must check for the cached file on disk and serve it directly,
bypassing PHP entirely on a cache hit. Same pattern for the page cache.

AVIF is best-effort. If the server cannot produce it, ship WebP and move on. Never
block on it.

### 5.6 Replacement tags

Inline only, for use inside rich text. Anything structural is a block.
Closed list for v1:

```
{{form:slug}}  {{page:slug}}  {{snippet:key}}  {{lang:switcher}}  {{year}}
```

Strict regex whitelist. Never `eval`. Never interpolate user content into a callable.

---

## 6. Security rules

- CSRF token on every state-changing request.
- Sessions: `httponly`, `secure`, `samesite=strict`, regenerate on login.
- Login rate limit: 5 attempts per 15 minutes, per IP and per account.
- 2FA (TOTP) is strongly encouraged but **not forced**, because a single-admin system
  with forced 2FA and no recovery path means a lost phone is a lost site. Provide ten
  single-use recovery codes at enrolment, plus a documented reset: drop a file named
  `storage/disable-2fa` on the server via FTP.
- Uploads: MIME sniffed with finfo, extension whitelist, explicit rejection of anything
  PHP-adjacent, and an `.htaccess` in `/uploads` disabling script execution.
- CSP header on the admin. No inline scripts in admin views.
- `install.php` writes `storage/install.lock` and refuses to run again. It also
  attempts to delete itself and warns loudly if it cannot.
- Form submissions store a hashed IP, not the raw address. GDPR matters for the
  audience.
- Anti-spam: honeypot field plus a minimum 3-second submit delay. Turnstile optional.

---

## 7. Working rules for the agent

1. **Vertical slices.** Every slice ends with something visible in a browser. Do not
   build all of Core before anything renders.
2. **Never add a dependency** that is not in §3.
3. **Never change a frozen contract** in §5 without asking first.
4. **Run the seed** (`/migrations/seed.php`) after schema changes and confirm the demo
   site still renders. This is the primary regression check.
5. **No abstraction without a second caller.** No interface with one implementation, no
   hooks system before a second module needs it, no repository layer over PDO.
6. **Keep code files under 300 lines** (PHP, templates, CSS, JS; documentation is
   exempt). If a controller grows past that, the feature is probably too big.
7. **Write the migration first**, then the model, then the controller, then the view.
8. Every user-facing string in the admin goes through `t('key')` and lands in
   `/lang/en.php`. No bare English in a template.
9. Commit at the end of each slice with a message naming the slice.

---

## 8. Build order

Each slice has an acceptance test you can perform by hand in a browser.

### Slice 1 — Skeleton renders
Container, Router (locale-aware), Request/Response, Config, Db (SQLite), View with
`e()` helper, error handler.
**Accept:** `/en/hello` renders a hard-coded page. `/hr/hello` renders the same page
in a different locale context. A 404 renders a 404.

### Slice 2 — Install and log in
Migration runner, installer (requirements check → admin account → site info → migrate
→ seed → lock), login, session hardening, rate limit, admin shell layout.
**Accept:** delete the database, run the installer on a clean copy, log in.

### Slice 3 — Pages and blocks
Pages CRUD, block registry, block editor with drag-and-drop ordering, three blocks
(hero, text, image+text), front-end render.
**Accept:** create a page in the admin with three blocks, view it on the front end.

### Slice 4 — The design layer ← this is the demo moment
Token schema, palette generation with contrast checks, five character presets,
`tokens.css` compilation, layer-2 section styles in the block editor, layer-3 layout
picker.
**Accept:** switch the character preset and the same page looks like a different site.
Change one section's surface and rhythm and only that section changes.

### Slice 5 — Media
Upload, presets, lazy variant generation, `<picture>` output, focal point picker,
per-locale alt text, .htaccess direct serving.
**Accept:** upload a 4 MB photo, place it in a hero, confirm the served file is WebP
and under 200 KB, confirm a second request does not hit PHP.

### Slice 6 — Multilingual
Locale management, locale switcher block, fallback chain, hreflang, per-block
translation status, AI provider interface with one implementation, glossary,
tag-count validation, stale detection.
**Accept:** create a page in Croatian, translate to English and German, edit one
Croatian block, confirm only that block shows as stale in both translations.

### Slice 7 — Forms and mail
Form builder, submissions, SMTP and Resend drivers, admin notification, autoreply,
honeypot, test-mail button.
**Accept:** build a contact form, embed it, submit it, receive both emails, see the
submission in the admin.

### Slice 8 — Operations
Page cache with .htaccess bypass, backup to ZIP, in-admin update via ZIP upload with
automatic backup and migration, revisions with rollback, sitemap.xml with hreflang.
**Accept:** upload a version bump ZIP through the admin, confirm migrations run and
the site still works.

### Slice 9 — Release
Six more blocks (gallery, features, CTA, accordion, testimonials, logo strip), three
templates, README with screenshots, demo site, release ZIP build script.
**Accept:** a stranger downloads the ZIP, uploads it to shared hosting, and has a
styled multilingual site in under fifteen minutes.

---

## 9. Open questions to resolve before Slice 6

- Which AI provider ships as the default, and does Boxlet ship without an API key
  (translation disabled until the user adds one)? Almost certainly yes.
- What happens to a page whose translation does not exist yet — 404, fallback render,
  or hide from navigation? Recommend: configurable per site, default to hiding from
  navigation and 404 on direct hit.

---

## 10. Testing

`php tests/run.php` is the whole test runner: plain PHP, no dependencies, no
PHPUnit. It loads every `tests/*_test.php`, prints one PASS/FAIL line per test and
exits non-zero on any failure. Any PHP notice, warning or deprecation fails the test
that raised it. CI runs it on every supported PHP version.

`tests/support.php` provides `assertEquals`, `assertTrue`, `assertContains`,
`dispatch()` (a request through `app/bootstrap.php` and the Router, no web server)
and `testBothModes()` (the same test with pretty URLs and the `index.php?route=`
fallback).

Rules:

- **Every slice adds tests for what it builds.** The slice's acceptance criteria in
  §8 are the starting point for what to assert.
- Tests need no web server. Tests must not write outside `tests/` and
  `public/cache/`, and must restore anything they delete.

---

## Changelog

```
2026-09-15  §3 The closed dependency list governs runtime dependencies only.
            Dev tools go in require-dev, never ship in the release ZIP and
            never reach a user's server; allowed when they earn their place,
            with approval. phpstan/phpstan added at level 6 with phpVersion
            8.1 and no baseline: findings are fixed, or ignored inline with a
            reason. CI runs it once, on PHP 8.4.

2026-09-15  §10 Minimal test runner added (tests/run.php, no dependencies).
            Every slice adds tests for what it builds, starting from its §8
            acceptance criteria. Rationale: from the installer onwards the
            surface grows faster than manual checking scales, and routing
            breaks silently.

2026-09-15  §5.1 A first path segment is treated as a locale only if it
            is an enabled locale, not by matching a two-letter shape.
            /{primary}/slug 301-redirects to /slug. Rationale: shape
            matching made every two-letter slug unreachable. 301 rather
            than 302 is safe because the primary locale is immutable.
            Consequence (Slice 8): the CLI escape hatch that changes the
            primary locale is documented as safe only before a site
            receives public traffic, because 301s cached by visitors
            would otherwise produce a redirect loop.

2026-09-15  §5.1 The primary locale is chosen during installation and is
            immutable. It always renders without a URL prefix; every additional
            locale always carries one. locale_prefix as a configurable setting
            is removed.
            Rationale: a mutable primary locale would require URL migration,
            301 generation and admin warnings for an action performed at most
            once in a site's lifetime. Locking it eliminates the problem rather
            than managing it. Same model as Shopware.
            Consequence (Slice 3): all ISO language codes are reserved and
            rejected as top-level page slugs, checked on save.
            Consequence (Slice 8): a documented CLI escape hatch can change the
            primary locale and generate redirects. Deliberately not exposed in
            the admin UI.

2026-09-15  §5.1 Routing split into two cases. A missing locale segment
            redirects (302) to the primary locale keeping the slug; an unknown
            or disabled locale returns 404, not a redirect. Rationale: silently
            redirecting /de/ to English is a soft 404.
            tokens.css is always compiled from token values, never shipped as a
            static file. Source is config/tokens.php in Slice 1, the
            design_tokens table from Slice 4. One mechanism, two data sources.
            All URL generation goes through a single helper.
            NOTE: the routing half of this entry is superseded by the primary-locale
            entry above. The tokens.css and URL-helper decisions still stand.
```
