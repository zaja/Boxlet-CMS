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

`PLAN.md` §1 states the same thesis and the same non-goals without technical detail, for
whoever is not reading code. It is derived from this file: when the two disagree, this
file is right.

---

## 2. Target environment

| Constraint | Value |
|---|---|
| PHP | 8.1 minimum. Do not use 8.2+ syntax. |
| Database | MySQL/MariaDB (recommended; preselected by the installer) or SQLite (small single-site installs). All SQL runs on both (§5.0). MySQL databases must default to `utf8mb4`. |
| Server | Shared Apache or Nginx, no shell, no Composer |
| URL rewriting | **Required.** Apache `mod_rewrite` (rules ship in `public/.htaccess`) or nginx `try_files`. No fallback URL mode. |
| Extensions required | pdo, mbstring, fileinfo, json, session, dom, and pdo_mysql or pdo_sqlite |
| PHP settings | `max_input_vars` of 1000 or more; 3000 for pages with many blocks. The installer reports it; the page editor refuses saves that hit it. |
| Extensions optional | gd or imagick, intl; AVIF output is best-effort |
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

Admin front-end: vanilla JS, and hand-written CSS using the admin's own fixed token set
(§5.4), never the site's. No Tailwind CDN in production, no Alpine, no HTMX. The admin
is small enough that a framework is a liability.

### Vendored front-end assets

The closed list above governs Composer packages. Front-end code has its own rule, so
that "it is not a Composer dependency" never becomes a way around it.

A vendored asset is allowed when it earns its place, and asking comes first. It must be:

- MIT or similarly permissive,
- dependency-free and distributed as a single file,
- committed to the repository under `public/assets/vendor/`,
- loaded with a plain `<script>` or `<link>` tag.

No npm, no build step, no CDN. The version and source URL are recorded in the file's own
header and in the table below, so updating it later is not archaeology. `README.md` points
here rather than keeping a second copy of the list.

```
sortablejs      1.15.6    MIT    reordering blocks inside the editor canvas
tiptap.bundle   3.31.3    MIT    the rich text editor (§5.3)
```

SortableJS earns its place because reordering happens inside an iframe, where native
HTML5 drag and drop does not handle touch usably, and a tablet is a real case for the
page editor.

TipTap earns its place because a rich text editor lives on edge cases — pasting from a
word processor, nested lists, undo across compound operations, mobile keyboards, IME
input — and those are surfaced only by a large user base, not by good intentions. It is
built on ProseMirror, whose schema allows only the nodes we declare, so the editor cannot
offer markup the server would throw away. It stores HTML, which is what we already store,
so replacing it later costs nothing in content.

**The one exception to "no npm, no build step"** (PLAN.md D-017). TipTap is not published
as a single file, so `public/assets/vendor/tiptap.bundle.min.js` is built from the recipe
in `tools/tiptap/` — pinned versions, a lockfile and one entry file — and committed. The
build runs outside the project, so no `node_modules` ever sits in it, and nobody
installing Boxlet builds anything: the file ships as it is, like every other vendored
asset. Its header names every bundled package, version and licence. A second exception
needs its own decision.

### require-dev

Development tools are not covered by the closed list. They live in `require-dev`,
never ship in the release ZIP (built with `--no-dev`) and never exist on a user's
server. They are allowed when they earn their place, but ask first.

```
phpstan/phpstan           static analysis, level 8, phpVersion 8.1, no baseline
```

---

## 4. Directory layout

```
/public/                      ← document root (or public_html)
  index.php                   front controller
  install.php                 installer, self-disables
  .htaccess
  /assets/                    admin + front-end CSS/JS, shipped built
  /m/                         generated image variants (§5.5)
  /cache/                     tokens.css, page cache
/app/
  /Core/                      Container, Router, Request, Response, Db, Config,
                              View, Migrator        (Cache arrives with Slice 8)
  /Modules/
    /Pages/  /Media/  /Design/  /Forms/  /I18n/  /Settings/  /Mailer/  /Ai/
    /Install/  /Auth/  /Admin/
    /{Module}/views/          templates owned by that module (front-end or admin)
  /Blocks/                    one directory per block: block.php + template.php
  /Support/
/config/
/storage/                     never web-accessible: logs, cache, sessions, backups
  /uploads/                   original media, never modified, never served (§5.5)
/lang/{locale}/               admin UI strings, one directory per locale, split by
                              concern (shell, pages, install, media, update, design).
                              t() merges every file in the admin locale's directory.
                              Nothing sits directly under /lang/.
/migrations/                  NNNN_name.sql, applied in filename order
/vendor/
.env.example
```

`/app`, `/storage`, `/config` must sit outside the document root when the host allows
it. When it does not, the installer writes a `Require all denied` .htaccess into each
and warns the user.

URL rewriting is required on every host; there is no query-string fallback. The
installer checks it with `app/Core/RewriteCheck.php` (one HTTP request to a probe
route, with the user waiting) and refuses to proceed without it, showing the Apache
and nginx configuration to add. The front controller never probes itself. At runtime
only one case is caught: on Apache without `mod_rewrite`, `public/.htaccess` routes
404s to `index.php`, which sees `REDIRECT_STATUS=404` and renders a plain instruction
page before anything else boots. Other misconfigurations (nginx without `try_files`,
Apache ignoring `.htaccess`) show the server's own 404; the installer catches those.

---

## 5. Frozen contracts

Changing any of these after v0.1 ships is a breaking change. Get them right now.

### 5.0 Database portability

Both MySQL/MariaDB and SQLite are supported, and every migration and query must run
unchanged on both. There is no schema-definition layer and no query builder.

Migrations are `migrations/NNNN_name.sql`, applied in filename order by
`app/Core/Migrator.php` and recorded in `migrations`. They are plain SQL both engines
accept, with **exactly one substitution token, `{{pk}}`**, for auto-increment primary
keys:

```
id {{pk}}      MySQL:  INTEGER NOT NULL AUTO_INCREMENT PRIMARY KEY
               SQLite: INTEGER PRIMARY KEY AUTOINCREMENT
```

No single plain-SQL form works on both, and `INTEGER AUTO_INCREMENT PRIMARY KEY` is
actively dangerous: SQLite accepts it and silently stores NULL ids. The Migrator
rejects any direct AUTO_INCREMENT/AUTOINCREMENT, and **any other `{{...}}` is a fatal
error that aborts the run before any file is applied**. `{{pk}}` is the only token that
will ever exist; if a second one seems necessary, that is a design conversation.

Rules that keep SQL portable:

- `VARCHAR(n)` always has a length (MySQL requires it); `TEXT` for long values.
- Timestamps are UTC `VARCHAR(19)` strings, `Y-m-d H:i:s`, which sort and compare the
  same on both engines.
- Booleans are `SMALLINT` 0/1.
- Identifiers that are reserved in MySQL (such as `key`) are quoted with backticks,
  which SQLite also accepts.
- Constraints are named: `CONSTRAINT admin_email_unique UNIQUE (email)`.
- Statements end with `;` at the end of a line; whole-line `--` comments only.
- MySQL commits DDL immediately, so only SQLite wraps each file in a transaction. Keep
  one table per migration file so a failure on MySQL leaves at most one table behind.

**The site is closed by one gate, with two triggers** (PLAN.md D-019, D-021). Pending
migrations close it automatically; the owner closes it deliberately from the dashboard.
Both answer **503 with `Retry-After`**, so a crawler returns later instead of dropping the
page. The difference is who still gets through: while a migration is pending nobody sees
the site, admin included, because it may not render until the migration has run, and the
admin is sent to the update screen instead. In maintenance mode a logged-in admin sees the
real site with a bar saying it is hidden. The maintenance state is a file in `storage/`,
never a settings row: it has to be readable when the database is unavailable, mid-update,
or about to be replaced. The file records why it is on, so an automatic update can clear
its own flag without clearing one the owner set.

**A committed migration is never edited; a change is a new migration.** Once a file has
been released, some installs have already applied it and recorded it by name, and those
installs will never run it again — so an edit reaches only the databases that had not yet
seen it, and the schema silently diverges between two sites running the same version.
This is what makes updating an existing install safe (PLAN.md D-019): the record of
applied filenames is trusted absolutely, and a filename means exactly one thing for ever.

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

**Media variants are generated on upload, not on demand.** There are five presets;
generating them at upload costs about a second per image and means a request for a
variant is always a request for a file that exists on disk. Serving never touches PHP,
works identically on Apache and nginx, and needs no server rule the user may be unable
to add. The `/m/{preset}/{id}-{slug}.{ext}` shape stays, since the files are real.

Accepted cost: slower uploads, and adding a preset later requires regenerating existing
media (Slice 8). This replaces on-demand generation, which does not work on a managed
nginx: a request for a missing `.webp` is answered from disk with nginx's own 404 and
PHP never runs — the same cause as the unstyled Design preview in Slice 4.

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
  layout,                    -- layer-3 layout: one of the block's declared layouts,
                             -- validated on save; '' or a removed one renders the default
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
login_attempts (ip_hash, email_hash, successful, attempted_at)
  -- Rate limiting only. HMAC hashes keyed by APP_KEY, never raw IPs or emails;
  -- pruned on write to the rate-limit window. The audit log gets its own table (Slice 8).
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

**Multi-column arrangements are layout variants of one block, never containers.** A
"two columns of text" block is a single block with two text fields and a `two-column`
layout; it is not a shell holding two other blocks. A page stays a flat, ordered list of
blocks: no zones, no columns, no nesting, and `page_blocks` gains nothing to support one.

A generic column grid is out of scope deliberately, not for want of time. It would hand
the user enough freedom to build something ugly, which is the opposite of what this
project is for — blocks that already know how to look good is the premise. It would also
multiply every later feature (translation, revisions, caching) by the nesting depth.

**Editing a richtext field.** The field is edited with TipTap (§3), and stored as HTML
conforming to the whitelist above. The editor is a convenience; the server-side whitelist
is the security boundary and the storage contract, it sanitises on save whatever arrives,
and it keeps one stored shape however the markup was produced.

**The editor's schema is the whitelist.** TipTap is configured with exactly these nodes
and marks — paragraph, `h2`–`h4`, bold, italic, link, bullet and ordered lists,
blockquote, hard break, and undo/redo — so it cannot produce anything the sanitiser would
discard. The toolbar therefore offers only what can be stored: no strike, no code. A
button whose output is discarded on save is worse than no button.

- Editor output is normalised on save, in the sanitiser where every other rule lives:
  `div → p` (the block wrapper some editors use), `h1 → h2` (the page's own title is the
  h1, so a heading in body copy sits below it), and `h5`–`h6` → `h4`.
- `div → p` is conditional: a div containing a block element is unwrapped instead, since
  a paragraph may not contain a list and renaming regardless would generate invalid
  markup of our own making.
- **A paragraph wrapping the text of an `li` or a `blockquote` is unwrapped.** An editor
  whose schema puts a paragraph inside every list item and quote would otherwise change how
  existing content looks: measured on the front end, `<li><p>one</p></li>` renders 16px
  taller per item than `<li>one</li>`, because the paragraph takes the normal paragraph
  margin. Storage keeps one shape whichever editor produced it.

  What the author made survives. Two paragraphs in one item or quote are kept, both of
  them. A list item may hold a sublist after its text, so a paragraph followed only by
  `ul` or `ol` is still a wrapper and goes, giving `<li>text<ul>…</ul></li>`; a paragraph
  *after* a sublist is not a wrapper and stays. A quote may hold nothing beside its
  paragraph, so anything else there leaves the paragraph in place. The rule removes a
  wrapper and allows nothing new.
- **A `br` at the start or end of a block is dropped on save** — in `p`, `h2`, `h3`, `h4`,
  `li` and `blockquote` (PLAN.md D-014). An editor that marks block boundaries with breaks
  would otherwise add one at each end of every richtext field on a page that was merely
  opened and saved, including fields nobody edited, compounding with every cycle. A break
  between two lines is the author's and stays.
- **The three heading levels are in the toolbar as H2, H3 and H4** (PLAN.md D-016). Each
  button applies its level, replaces another, and returns the block to a paragraph when
  pressed again; the button for the level the cursor is in shows as active.
- **Attachments are not offered, and attachment markup is stripped on save.** No editor in
  this project may store its own container format: a `figure` carrying JSON in a data
  attribute is the one thing that would put content beyond the reach of the whitelist. The
  sanitiser removes it with its content as a backstop, so no later version can reintroduce
  it silently. Images come from the media library (§5.5).
- The textarea is the real field and carries the input name; JavaScript moves the name to
  a hidden input and places the editor above it. Without JavaScript the field is still
  editable, and the plain-HTML toggle on every richtext field is simply what was
  underneath all along.
- **The link panel is in the flow, never over the text**, and is hidden at rest. It opens
  from the toolbar or Ctrl+K carrying the address of the link the cursor is in, and closes
  on Link, Unlink, Escape or a click back into the text, returning the caret to the
  editor. Hiding is by the `hidden` attribute, which `admin.css` enforces against any rule
  that would otherwise lay the element out.

`template.php` receives `$content`, `$style`, `$layout` and outputs HTML that uses
**only** CSS custom properties for colour, spacing, radius, shadow and typography. A
block template containing a hard-coded colour or pixel value is a bug.

`app/Core/Blocks.php` enforces the contract when the application boots; a malformed
definition stops it with a message naming the block and key:

- Exactly the keys above; a missing or unknown key is an error. `type` equals the
  directory name. Field names match `[a-z][a-z0-9_]*`; `id` and `type` are reserved
  for the page editor.
- A field has `type`, optional boolean `required` and `translatable`, and for `select`
  a non-empty list of option values: `'options' => ['cover', 'contain']`.
- Field types from the closed set that are not implemented yet are rejected. Implemented:
  `text`, `textarea`, `richtext`, `media`, `link`, `select`.
- `defaults.layout` is one of `layouts`. The layout chosen for a block instance is stored
  in `page_blocks.layout` and validated against `layouts` on save; a stored layout the
  definition no longer declares renders as `defaults.layout` instead of failing.

There is no `label` key: every admin label derives from the type through the files in
`lang/`, which `t()` merges:
`block.{type}`, `block.{type}.{field}`, `block.{type}.{field}.{option}` and
`block.{type}.layout.{layout}`. A test fails when any of these is missing.

Stored field values (`page_blocks.content_json`):

```
text, textarea   string, trimmed, no control characters
richtext         HTML reduced on save to: p, br, strong, b, em, i, h2, h3, ul, ol, li,
                 blockquote, and a with href only. Other elements are unwrapped, keeping
                 their text; script, style, iframe, svg and similar are removed with
                 their content. Output unescaped by templates.
media            media id (integer) or null; a placeholder renders until Slice 5
link             {"label": string, "url": string}; url must start with /, #, ? or
                 http:, https:, mailto:, tel:, with no whitespace or backslash
select           one of the option values; the first is the default
```

Link URLs in richtext follow the same rule; an `href` that fails it is dropped.

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
                        surface contrast (low / medium / high)
Layer 2  Section      per block instance, stored in page_blocks.style_json:
                        surface:  plain | tinted | contrast | image | gradient
                        rhythm:   tight | normal | airy
                        width:    narrow | normal | wide | full
                        align:    left | center
                        divider:  none | line | slant | curve
Layer 3  Layout       per block instance, one of block.php 'layouts', stored in
                      page_blocks.layout
```

Layers 0 and 1 compile, on save, to `public/cache/tokens.{hash}.css`. The content hash
is in the file name, so a saved change is a new URL that no browser or proxy has cached.
The current name is recorded in `settings.tokens_css`. An ordinary request only reads
that setting. It compiles in exactly one other case, deliberately kept: the recorded
file is missing, after a fresh deploy or a cleared cache. That guard is what stops a
site rendering unstyled, so "a request never compiles CSS" is not true and must not be
written anywhere. The file is a real file on disk, so Apache's file-exists rewrite
condition and nginx's `try_files` serve it without PHP. Layers 2 and 3 render as class
names on the section wrapper. Nothing is inlined as a style attribute.

`site.css` and `sections.css` are shipped files rather than generated ones, so they
cannot carry a hash in the name without a build step the install cannot run. They are
linked with a hash of their content in the query string instead, which busts the same
caches and is still served from disk (`Url::versioned()`).

**Layer 1 storage.** `design_tokens` holds the decisions, one row per key: `seed`,
`secondary` ('' when unused), `typography`, `scale`, `spacing`, `radius`, `shadow`,
`container`, `surface_contrast`. Derived values are never stored.

```
typography        editorial | classic | modern | grotesk | rounded | mono
                  heading and body family, weights, tracking, case, line heights
scale             1.125 | 1.2 | 1.25 | 1.333 | 1.414 | 1.5    --text-sm … --text-4xl
spacing           compact | normal | roomy | generous         --space-xs … --space-3xl
radius            none | subtle | round | pill                --radius-s/m/l/button
shadow            none | soft | hard | layered                --shadow-s/m/l, --border-width/card
container         narrow | normal | wide | full               --container-width/narrow/wide
surface_contrast  low | medium | high                         lightness of --color-surface
```

**Palette.** Derived in OKLCH from the seed: background, tinted surface, border, text and
muted text carry a trace of its hue; the seed itself is the accent and link colour; the
contrast surface is the second colour, or a deep shade of the seed, with its own text
colours; the gradient runs from the seed to a neighbouring hue. Seeds are never adjusted
to pass. Every text/background pair is checked at WCAG AA 4.5:1: text, muted text and
links on the background and the tinted surface; button text on the accent; text and
muted text on the contrast surface; text on both ends of the gradient. A failure refuses
the save and names the pair, its ratio and the decision responsible. Every surface can
hold body text, so no pair qualifies for the 3:1 large-text threshold.

**Fonts.** Self-hosted from `public/assets/fonts`, never a third-party service: Inter,
Playfair Display, Source Serif 4, Space Grotesk and Nunito as variable fonts, IBM Plex
Mono at 400 and 700, each in latin and latin-ext subsets with its SIL OFL licence. The
compiled stylesheet declares only the families the current pairing uses.

**Layer 0.** A character is a complete set of layer-1 values **and the composition it
gives a page** (`app/Modules/Design/Presets.php`): the layer-2 section style every block
starts from, a surface per block type, and the layer-3 layout of block types that offer
several. The five differ in measure, vertical rhythm, alignment, section edge and hero
arrangement, so changing character changes how a page is composed and not only how it is
painted.

Using a character fills the Design form; Save applies it. Because new blocks have to be
composed somehow, the name of the character last applied is remembered in
`settings.design_character`; a site that never chose one composes like the default
character, whose tokens it is already rendering with. That setting is the only thing
that refers back to a character, and it is never read to render an existing block.

Applying a character to a site that already has blocks offers two explicit actions,
never one silent one: **save the design only**, which changes layer 1 and leaves every
section as its author left it, or **save the design and reset section styles**, which
also rewrites layers 2 and 3 of every block on the site. The second is destructive and
is only ever reached by choosing it.

**Alignment is not placement.** `align` sets how text sits inside its column, never
where the column sits on the page. A section's column is always centred, whatever its
width; `width: narrow` therefore gives a narrow column centred on the page with its text
ranged left, which is how a reading column is set. The two ideas are deliberately not
one control, and there is no stored key for placement.

**A divider is an accent.** Drawn on every boundary it stops reading as a boundary at
all, so a character names the block types whose sections carry one rather than setting a
default for all of them. The first section on a page never draws one: it has nothing to
transition from, and a shaped edge there is clipped by the top of the viewport.

**No free colour per section.** Surface is a closed set — plain, tinted, contrast, image,
gradient — and it stays closed. A per-block background colour field would let anyone set
red text on orange, and the entire contrast-checked palette becomes decoration rather
than a guarantee. If five surfaces prove too few, a sixth is added *drawn from the
palette*, not an open colour input.

**The admin's own chrome is not this system.** Its `--ui-*` tokens are a fixed set
declared in `public/assets/admin*.css`, never derived from `design_tokens`, so the tool
stays readable whatever the site is set to. It has its own contrast rule, measured by
`tests/contrast_test.php` rather than at save time: **4.5:1 for text**, and **3:1 for a
control's boundary**, including a disabled one. Opacity is not allowed to take a control
at rest below either figure. `canvas.css` is the one exception and cannot be checked this
way, because it loads into a document full of the site's tokens and what sits behind its
controls is the user's design; its insertion control carries two tones so that one edge
contrasts whatever is behind it (PLAN.md D-012).

**Layer 2.** `page_blocks.style_json` holds all five keys. Values outside the closed sets
fall back to the defaults (plain, normal, normal, left, none) on save and on render. The
only CSS for these classes is `public/assets/sections.css`: each surface sets
`--section-*` colour properties that block CSS uses, so every block works on every
surface. The `image` surface renders like `contrast` until media exist (Slice 5).

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
sha1 for dedup → store the original in `storage/uploads/`, outside the web root →
generate every variant during the upload request → write to `public/m/`, the same
path as the `/m/` URL scheme → serve via `<picture>` with AVIF → WebP.

**Originals are never public** (D-020). Only generated variants are, and `full` — up
to 2400 px wide — is the largest public version. An `.htaccess` disabling script
execution cannot protect a public upload directory on nginx, which ignores it
entirely; a file that is never served cannot be executed on any server. The cost is
that an original can no longer be linked as the exact uploaded file.

Variants are generated on upload, never on demand (§5.1). Every media URL therefore
points at a file that already exists, so the web server serves it without PHP and
without a rewrite rule. The page cache still uses the file-exists rewrite.

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
- Sessions: `httponly`, `secure` whenever the request is HTTPS, `samesite=strict`,
  regenerate on login.
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

They are in `CLAUDE.md`, which Claude Code loads into every session automatically:
vertical slices, the closed dependency list, the frozen contracts, `php tests/run.php`
after a schema change, no abstraction without a second caller, files under 300 lines,
migration before model before controller before view, `t('key')` for every admin string,
a commit at the end of each slice, and the rule that a slice is done only once the
architect has verified it in a browser.

One copy, and it is that one. A rule added here instead would be a rule an agent never
loads.

---

## 8. Slices and acceptance criteria

Each slice has an acceptance test you can perform by hand in a browser.

The order the slices are worked in, and how far each has actually got, are in `PLAN.md`
§2 and §3. Nothing here counts as done until the architect has verified it in a browser
(PLAN.md D-006), so this section claims neither.

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

### Slice 4 — The design layer
Token schema, palette generation with contrast checks, five character presets,
`tokens.css` compilation, layer-2 section styles in the block editor, layer-3 layout
picker.
**Accept:** switch the character preset and the same page looks like a different site.
Change one section's surface and rhythm and only that section changes.

### Slice 4.5 — The admin's own design system, and composition
A fixed admin design system (`--ui-*` tokens defined in `public/assets/admin*.css`,
never derived from `design_tokens`), the Design screen rebuilt as controls beside a wide
sticky preview, and characters extended to carry layer-2 and layer-3 composition with an
explicit choice when applying them to a site that has pages.
**Accept:** switching character changes how the page is composed — measure, rhythm, hero
arrangement, section edges — and the admin looks identical whatever the site is set to.

### Slice 4.6 — The visual page editor
A canvas showing the real page in an iframe, a library of blocks each rendered as a
picture of itself, insertion by aiming at a position and choosing a block, reordering by
dragging inside the canvas (SortableJS), and an inspector holding the selected block's
fields, section style, layout and controls. The plain form editor stays at
`/admin/pages/{id}/form` as the fallback. Characters also stopped over-using their
section edges, and the admin gained its own picture of what a block is.
**Accept:** add a block to a page by picking it out of the library, drag it into place,
edit its text and watch the page change, and save — with the plain editor still able to
fix the same page without JavaScript.

### Slice 5 — Media
Upload, presets, variants generated on upload (§5.1), `<picture>` output, focal point
picker, per-locale alt text, variants served from disk without PHP. Also per-page meta
title and description: two fields defaulting to the page title, emitted in `<head>`,
nothing more.
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
Page cache checked inside PHP, backup to ZIP, in-admin update via ZIP upload with
automatic backup and migration, revisions with rollback, sitemap.xml with hreflang.

The cache is read in the first lines of `public/index.php`, before the autoloader,
the container or the database (D-020): a hit is answered there and nothing else
boots. That works on every host with no configuration, which a server-level rewrite
does not — the rules differ between Apache and nginx, and a managed host may allow
neither. A server rule that skips PHP entirely is documented as an optional extra for
those who can add one, never as what makes the cache work.
**Accept:** upload a version bump ZIP through the admin, confirm migrations run and
the site still works.

### Slice 9 — Release
Six more blocks (gallery, features, CTA, accordion, testimonials, logo strip), three
templates, README with screenshots, demo site, release ZIP build script.
**Accept:** a stranger downloads the ZIP, uploads it to shared hosting, and has a
styled multilingual site in under fifteen minutes.

---

## 9. Open questions

They are in `PLAN.md` §5, each carrying the step of the order of work it belongs to.
This section is a pointer and not a list, so that an open question cannot be answered in
one document while staying open in another.

---

## 10. Testing

`php tests/run.php` is the whole test runner: plain PHP, no dependencies, no
PHPUnit. It loads every `tests/*_test.php`, prints one PASS/FAIL line per test and
exits non-zero on any failure. Any PHP notice, warning or deprecation fails the test
that raised it. CI runs it on every supported PHP version.

`tests/support.php` provides `assertEquals`, `assertTrue`, `assertContains`,
`assertThrows`, `dispatch()` (a request through `app/bootstrap.php` and the Router,
no web server) and `testBothDrivers()` (the same test on SQLite and MySQL).
`tests/fixtures.php` builds state: `freshDatabase()`, `migratedDatabase()`,
`installedSite()` and `createAdmin()`.

Databases: SQLite tests use a file in `tests/tmp/`. MySQL tests use a database that
exists **only** for tests, never a site's database, configured in `.env.test` (see
`.env.test.example`) or environment variables. Every table in it is dropped before
each test and after the run. Without configuration, MySQL tests are skipped with a
message; CI sets `TEST_REQUIRE_MYSQL=1`, which turns a skip into a failure.

Rules:

- **Every slice adds tests for what it builds.** The slice's acceptance criteria in
  §8 are the starting point for what to assert.
- Every test builds its own state. A test that depends on enabled locales, an admin
  account or an install lock creates them itself, never relying on config or
  installer defaults.
- Tests need no web server. Tests must not write outside `tests/`, `public/cache/` and
  the MySQL test database, and must restore anything they delete.

---

## History (before PLAN.md, frozen)

No entry is added here again. From `1782cc4` onwards the history of the project is the
decisions in `PLAN.md` §4.

The entries below are left exactly as they were written, including their cross-references.
Where one says an open question was added to §9, that is what §9 held at the time; those
questions are now `PLAN.md` §5.

```
2026-09-16  Documentation aligned with the code (PLAN.md O-5).

            PLAN.md, in the repository root, is now the architectural decision
            log. §4 stops naming Core parts that have no caller: Hooks is gone,
            since building it would break "no abstraction without a second
            caller", and Cache is marked as arriving with Slice 8 (D-005).
            §5.5 writes variants to public/m/, matching the /m/ URL scheme,
            rather than the /public/cache/media/ left over from the abandoned
            on-demand model (D-003). Slice 5 drops "lazy variant generation"
            and ".htaccess direct serving" for what §5.1 actually says, and
            gains per-page meta title and description — two fields defaulting
            to the page title, emitted in <head>, nothing more (D-004). How a
            variant is served without PHP is deliberately not named: nginx is
            the primary target and Apache must stay fully compatible, which is
            open as PLAN.md O-2.

2026-09-16  Rich text is edited with Trix (MIT, vendored).

            Selection turned on a correction: an editor's internal document
            model is not lock-in, only its storage format is. Quill was
            rejected because its ecosystem stores Delta, putting content in a
            Quill-only format beyond the reach of our server-side whitelist.
            Trix stores HTML and was rejected earlier on the wrong ground.
            Wysi (MIT, 12 KB, plain HTML) satisfied replaceability but has a
            very small user base, and a rich-text editor lives on edge cases —
            paste from Word, nested lists, undo, IME, mobile — that only a
            large user base surfaces. Replaceability protects content, not
            experience.

            Accepted costs: div-to-p normalisation on save, and attachments
            disabled entirely because Trix's attachment attribute is its one
            proprietary format. Images come from the media picker.

            Wysi remains the fallback if Trix proves unworkable in this admin.

            Ruled out and not to be revisited: Quill and Editor.js
            (proprietary storage format), TinyMCE and CKEditor (licence model
            and weight), TipTap (requires a build step), Summernote (requires
            jQuery).

            What Trix emits was established by driving it in a browser through
            keyboard and toolbar, not by reading its documentation, and
            measured against a control — the same word-processor HTML pasted
            into a plain contenteditable. The control keeps MsoNormal classes,
            inline styles, font tags and a whole table; Trix reduces it to
            strong, em, a, real nested lists and div blocks. §5.3 records the
            normalisation; §5.4 records that section surface stays a closed set
            and gains no free colour field.

2026-09-16  Slice 4.6: the visual page editor.

            TRANSPORT REVERSAL. Slice 3 chose a plain form so that editing
            content needed no JavaScript. A visual editor cannot meet that bar,
            and the reversal is deliberate. What that decision was really
            protecting is kept intact: block definitions live in PHP only, and
            no copy of one exists in JavaScript. Adding or re-drawing a block
            POSTs to the server, which answers with HTML — the section for the
            canvas and the field group for the form, as two inert templates.
            No JSON API, no client-side schema, no client-side validation. The
            page is still one form, submitted by an explicit Save, validated by
            the same server-side code, with the same _end sentinel and
            max_input_vars guard. The form editor stays at
            /admin/pages/{id}/form as a fallback, linked both ways, so a broken
            canvas or a browser without JavaScript can still fix a site's text.

            The canvas is an iframe rendering exactly what a visitor gets, so
            site CSS and admin CSS cannot collide. Blocks are not wrapped in
            editor markup: sections.css styles a section by its position among
            its siblings, so anything inserted between them would change the
            page being judged. Selection is a class; the insertion controls are
            an overlay.

            Dragging from the library into the canvas is NOT built. Drag events
            do not cross a document boundary, and tracking pointer coordinates
            across one is a lot of fragile code for an interaction that is
            better served by aiming at a position and then choosing a block —
            which is also reachable from a keyboard. Reordering drags entirely
            inside the canvas, where there is no boundary, using SortableJS for
            touch support.

            §3 gains a rule for vendored front-end assets, closing the same gap
            require-dev exposed earlier: permissive licence, dependency-free,
            single file, committed, plain script tag, no npm or build step or
            CDN, version and source recorded. Currently: SortableJS 1.15.6.
            §5.3 records that multi-column arrangements are layout variants of
            one block and that a generic column grid is out of scope.
            §9 defers block library categories past roughly fifteen blocks.

            Characters: a divider is an accent on chosen transitions, never a
            default for every boundary, and the first section on a page never
            draws one. Minimal is quiet on purpose rather than unstyled.
            Editorial sits on one spine — a left-ranged hero fills its column
            instead of taking a second, narrower measure. §5.4 records that
            alignment is not placement. The media placeholder derives its tint
            from --section-text, so it cannot vanish on an untested surface.

2026-09-15  Slice 4.5: the admin's own design system, and characters that carry
            composition.
            The admin no longer links the site's tokens.css. It has a fixed
            token set of its own (--ui-*) defined in public/assets/admin.css,
            admin-ui.css and admin-forms.css, with its own type stack, spacing,
            neutral palette and widths. Inverted the stylesheet rule: the
            front-end CSS may hold no literal colour or size, the admin CSS may
            read no site token. Both are tested.
            §5.4 Layer 0 now carries composition: per block type, the layer-2
            section style and layer-3 layout new blocks start from. Applying a
            character to a site with pages offers two explicit actions, design
            only or design with section styles reset; the second is the only
            path that overwrites per-section choices. The character last
            applied is remembered in settings.design_character so new blocks
            can be composed; it is never read to render an existing block.
            Page::create takes a character instead of a default style.
            §5.1 records the media decision: variants are generated on upload,
            not on demand, because a managed nginx answers a missing .webp from
            disk and PHP never runs. §5.5 updated to match.
            §5.4 corrected: a request DOES compile tokens.css when the recorded
            file is missing. That guard is deliberate and the old absolute was
            wrong. site.css and sections.css now carry a content hash in the
            query string (Url::versioned), since shipped files cannot be
            renamed without a build step.
            §9: header, logo, navigation and footer recorded as an open
            question, deferred deliberately.

2026-09-15  Slice 4: the design layer. §5.4 documented in full.
            Layer 1: design_tokens stores nine keys (seed, secondary and the
            seven closed decisions; the two colours are one decision). The
            eighth decision, surface contrast, is added to the §5.4 list.
            The palette is derived in OKLCH; every text/background pair is
            checked at 4.5:1 and a failure names the pair, ratio and decision.
            Seeds are never nudged to pass. Six curated typography pairings,
            six OFL font families self-hosted in public/assets/fonts.
            Layer 0: five presets differing in every structural decision
            between Editorial and Brutalist, and in at least four between any
            two. Using a preset fills the form; Save is the confirmation.
            Compilation: on save, to tokens.{hash}.css, name recorded in
            settings.tokens_css; config/tokens.php removed. A request compiles
            only if the recorded file is missing. CACHE_PATH moves the output
            for tests.
            Layer 2: SectionStyle validates style_json against the closed sets
            with fallback; sections.css is the only CSS for section classes.
            Demo site (app/Modules/Demo) covers every block, layout and section
            style; migrations/seed.php adds it to an empty site and the
            installer offers it.
            §9 Open questions added: blog / news content, nested page addresses.

2026-09-15  §5.2 page_blocks gains a layout column for the layer-3 value, not a
            key inside style_json: layout is the only layer whose valid values
            come from the block definition, so it is validated on save and can
            be constrained and queried; style_json holds an open set. A stored
            layout the definition no longer declares renders as the default.
            §5.3 label removed from the block contract: a required but unused
            key in a frozen public contract. Labels derive from the type.
            §5.2 login_attempts is rate limiting only; the audit log gets its
            own table in Slice 8.
            Every front-end page emits a canonical link built through Url
            (absolute, no query string, origin from SERVER_NAME). Error pages
            emit none.

2026-09-15  Slice 3: pages, blocks, front-end render. Migrations add templates
            (three built-ins seeded: landing, article, feature), pages and
            page_blocks, with foreign keys: blocks cascade with their page.
            A new page or block starts its own content_group_id /
            block_group_id, set to its own id.
            §5.3 The block registry enforces the contract at boot; select
            options are a list of values; admin labels come from lang/;
            stored value shapes and the richtext whitelist are documented.
            Slugs are one path segment. Reserved as slugs: every ISO 639-1
            code, plus the system paths admin, assets, cache, uploads, m,
            install and _boxlet.
            Editor: one form, one POST, explicit Save, beforeunload warning.
            Without JavaScript, Add and Move re-render the form without
            saving and removal is a "remove when saving" checkbox. A save
            whose final _end field is missing, or whose field count reached
            max_input_vars, is refused. §2 dom is a required extension
            (DOMDocument sanitises richtext); max_input_vars is reported by
            the installer.
            §9 Open question added: where a block's layer-3 layout is stored.

2026-09-15  §5.0 Plain portable SQL proved insufficient for auto-increment
            primary keys: no single form works on both SQLite and MySQL, and
            one form (INTEGER AUTO_INCREMENT PRIMARY KEY) silently stores
            NULL ids on SQLite. Migrations therefore use exactly one
            substitution token, {{pk}}. Any other {{...}} is a fatal error.
            Engine-specific migration files were rejected (duplicated
            definitions drift); app-generated ids were rejected (index
            locality, collisions, unreadable admin URLs).
            §5.2 adds login_attempts (ip_hash, email_hash, successful,
            attempted_at), pruned on write.

2026-09-15  Slice 2. §2 MySQL is the recommended database and preselected by
            the installer; SQLite is for small single-site installs. MySQL
            databases must default to utf8mb4; the installer blocks anything
            else. pdo_mysql or pdo_sqlite is required, whichever is used.
            §4 Install, Auth and Admin are modules under app/Modules; the
            separate app/Admin directory is gone. §5.0 added: database
            portability rules. The settings and locales tables are migrated
            now: site name, timezone and the primary locale live in the
            database, never in .env. The installer offers every ISO 639-1
            language and enables only the primary one; config/locales.php is
            removed. §6 the session cookie is secure whenever the request is
            HTTPS. §7 migrations/seed.php does not exist until the demo site;
            the test suite on both drivers is the regression check until then.
            §10 tests run against SQLite and a dedicated MySQL test database.

2026-09-15  Rewrite detection lives in the installer, not the front
            controller. A self-probe during page render was considered and
            rejected: it is a network call in the render path that some hosts
            block, and it assumes success when blocked. Apache without
            mod_rewrite is caught by an .htaccess marker; other
            misconfigurations are caught at install time.

2026-09-15  ?route= fallback removed as a supported mode. Rewriting is now
            a hard requirement checked by the installer. Rationale: two URL
            modes taxed every slice and produced the project's only bug so
            far, in the path nobody would use. Users without vhost access
            get an explicit error with the exact configuration to add,
            rather than a half-working site.

2026-09-15  §3 PHPStan raised from level 6 to level 8. The two level 7
            findings were fixed: Db::all() is annotated with int keys because
            PDO's fetchAll() is typed as plain array and cannot prove a list;
            config/locales.php lists enabled locales as {code, label} rows
            instead of a map keyed by code, because PHP turns numeric-string
            keys into ints and only a value guarantees a string code.
            §9 Open question added: Config::get() and Container::get() return
            mixed, which blocks level 9 (~30 findings). Revisit after Slice 3.

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
