# Boxlet — the plan

The one document to read to understand the project: what Boxlet is, where the work
stands, what comes next, what has been decided and what is still open.

**Who writes what.** Only the architect session edits this file, and the owner approves
every decision before anyone acts on it. The executor reports progress by message and
never edits this file. Section 2 is updated only after the architect has checked the
work (D-006). How the system works technically is in `docs/SPEC.md`, the contract. How an
agent must work is in `CLAUDE.md`. Installing is covered in `README.md`. Each fact lives
in one place; everywhere else only points to it.

A decision's status is one of **proposed**, **approved** (by the owner, with a date) or
**superseded** (naming the entry that replaced it).

1. [What Boxlet is](#1-what-boxlet-is)
2. [Where the work stands](#2-where-the-work-stands)
3. [Order of work](#3-order-of-work)
4. [Decisions](#4-decisions)
5. [Open items](#5-open-items)

---

## 1. What Boxlet is

### In one paragraph

Boxlet is a small self-hosted CMS for people who build **many small sites**: freelancers
and small studios who currently reach for WordPress and then spend a day removing things.
It installs by uploading a ZIP and opening a page in a browser: no shell, no Composer, no
build step, no Node. One person runs it, with no user accounts to manage. What it sells is
not a feature list but a **design layer**. You choose a character and a handful of
decisions, and the whole site takes on a coherent, distinct look without anyone writing a
line of CSS.

It serves two users equally: the owner's own client sites, and a public release.

### Principles

- **A rock-solid small CMS.** It does not need hundreds of features, but what it does must
  be built properly, never a quick fix just to make something work (D-006).
- **Constrained freedom.** Page builders hand the user unlimited control, and users produce
  ugly sites. Boxlet offers a bounded space of options in which every combination looks
  acceptable. When a choice is in doubt, the knob is removed rather than added. The
  consequences:
  1. *No free colour for a section.* A section uses one of five surfaces (plain, tinted,
     contrast, image, gradient), all derived from the site's palette. An open colour field
     would let anyone put red text on orange, and the palette's guarantee of readable
     contrast would become decoration.
  2. *No arbitrary grid.* Columns are one block with a bounded set of content (D-008), not
     an empty container that any block can be dropped into.
  3. *Every block already knows how to look good* in every design, on every surface, at
     every width. The block author does that once, so the site owner cannot get it wrong.
- **The admin must look good and be original** (D-007).
- **Hosting:** nginx is the primary target, and full Apache compatibility is required.

### What it will never be

- No user accounts, roles or permissions. One administrator.
- No visitor accounts, registration or comments.
- No plugin marketplace, and no plugin API in version one.
- No e-commerce.
- No taxonomies: no tags and no content categories.
- No page builder with free-form positioning. A page is an ordered list of blocks.
- No build step, ever, on the server or for the person installing it.
- No public API in version one.
- No third-party fonts, analytics or scripts added casually. Anything that sends a
  visitor's data elsewhere is a deliberate decision, not a settings field.

### What "finished" looks like

A freelancer takes a ZIP, uploads it to a cheap shared host, and within fifteen minutes
has a multilingual site that looks designed, not like a template with the logo swapped.
They choose a character, adjust two or three decisions, write their pages on a canvas that
shows exactly what visitors will see, put in their photographs without thinking about image
sizes, and add a contact form that reaches their inbox. They never open a CSS file, never
see a database and never install a plugin. When the site needs changing a year later, they
still recognise it.

---

## 2. Where the work stands

*Checked by the architect on 2026-09-16 against the code, the docs and git.*

| | |
| --- | --- |
| Last commit | `2054ec9`: push permission and PLAN.md additions; CI green on GitHub |
| Tests | 259 passing on both drivers, PHPStan clean at level 8, as reported by the executor; not re-run by the architect |
| CI | checked by the architect on GitHub after every push; see the latest run for the current tip |
| Live site | https://boxlet.svejedobro.hr, MySQL `boxletcms`, demo site, Brutalist character (D-002) |
| Demo admin | `acceptance@example.com`; the password is never in the repository |

Slices 1–4.6 are committed. **Under D-006 none of them counts as done** until it has been
verified in a browser against an architect's checklist. That checklist is written in
step 2 of the order of work.

### What exists today

**Installing and running**
- Upload a ZIP, open `/install.php`, answer four screens: requirements, database,
  administrator, site details. The installer refuses to continue if the server cannot
  support the site, and says exactly what is missing.
- It checks what fails silently on cheap hosting (how many form fields PHP accepts, how
  large an upload may be) and reports it in plain language.
- It offers to install a demo site, so the design layer can be judged on real pages.
- MySQL or SQLite. One administrator, with a rate limit that locks out repeated wrong
  passwords by both account and address.

**The design layer**
- Five characters (Editorial, Minimal, Bold, Soft, Brutalist). Each changes the page's
  composition (reading measure, vertical rhythm, heading placement, section edges, hero
  arrangement), not only its colours.
- Eight decisions: two free seed colours, typeface pairing, type scale, spacing unit,
  corner character, shadow character, container width (four values), surface contrast.
  The full palette is derived, and a colour pair that fails readable contrast is refused
  with the pair named.
- Per block: surface, rhythm, width, alignment, top edge, and the block's layout.
- Applying a character to a site that has pages asks first: design only, or also reset
  per-section choices.
- Self-hosted fonts.
- Missing: header width, boxed layout, page background (step 5).

**Editing a page**
- A visual editor: the real page on a canvas, a library of blocks shown as pictures of
  themselves, a "+" between sections to insert, dragging to reorder, an inspector for the
  selected block.
- Page settings: title, address, parent page, published. The address follows the title
  while the page is a draft, can be edited, and never changes by itself once published.
- A plain editor at `/admin/pages/{id}/form` that works without JavaScript.
- Nothing is saved until Save, and leaving with unsaved changes warns.
- Three block types: hero, text, image and text.
- Missing: page SEO fields (D-004), reordering the page list (step 2), more blocks and
  columns (step 6).

**Writing (rich text with Trix)**
- Bold, italic, links, headings, quotations, nested lists. Stored as plain HTML, cleaned
  on the server. The toolbar offers only what can be stored.
- Pasting from Word is cleaned. Measured against a control: the same Word HTML pasted into
  a plain editable element keeps MsoNormal classes, inline styles, font tags and a whole
  table, while Trix reduces it to strong, em, a, real lists and div blocks.
- Every rich text field can be switched to plain HTML and back.
- **Content corruption bug (found 2026-09-16, mechanism confirmed by the executor):** after
  Move or drag-reorder, in both the visual editor and the plain editor, a rich text editor
  can write into another block's field, and a renumbered toolbar can drive another block's
  editor. Trix binds to its hidden input and toolbar by id, and renumbering rewrites those
  ids by block position. Duplicate in the visual editor also clones an editor that is never
  set up again. Fix (2a): the ids Trix binds to are stable per editor and never renumbered;
  a duplicate copies the text currently on screen and gets a freshly set-up editor.
- **Quality defect reported by the owner:** in the block inspector the link dialog covers
  the text, the toolbar wraps onto two rows, the disabled undo and indent buttons are barely
  visible, and field labels are cramped. Step 2.

| Trix construct | Trix emits | Stored as |
| --- | --- | --- |
| paragraph | `<div>` | `<p>` |
| bold / italic | `<strong>` / `<em>` | unchanged |
| link | `<a href>` | unchanged |
| heading | `<h1>` | `<h2>` |
| quote | `<blockquote>` | unchanged |
| lists, nested 3 deep | real nested `<ul><li><ul>` | unchanged |
| strike, code | `<del>`, `<pre>` | buttons removed |
| attachments | `<figure data-trix-attachment>` | disabled and stripped |

Verified in the running admin (plain editor only): Trix loads, the toolbar has exactly
the eleven permitted controls, the field name moves to a hidden input, typing reaches the
field, and the plain toggle works both ways.

**Not yet verified:**
1. *Round trip, the priority.* Load existing demo content into Trix, save without editing,
   and confirm the stored HTML is byte-identical. The server-side cleaner is proven
   idempotent, but that says nothing about whether Trix hands back what it was given. If
   they differ, report what changed; do not adjust the test.
2. *Visual editor interaction.* The earlier probe selected a rich text field in a hidden
   block group. Scope it to the visible group.
3. *Editors built inside hidden block groups*, a known cause of broken sizing and dead
   selection.
4. *Rescan of newly inserted blocks* (`boxletRichText.scan`): written, never exercised.
5. *Ctrl+Shift+V paste as plain text*: implemented, never verified.

**The admin**
- One persistent navigation, a consistent layout, a comfortable reading width.
- A fixed design of its own that never takes on the site's design, so an unreadable site
  colour scheme cannot lock the owner out of the screen that fixes it.
- Validation errors next to the field; saving confirms visibly.
- Rule: no control is invisible until hovered. The promised automated contrast check over
  every admin control is not written yet (step 2).

### Media (Slice 5): model decided, almost nothing built

Model (SPEC §5.1, §5.5, D-003): variants are generated **on upload, never on demand**,
stored in `public/m/`, and served from disk without PHP. Five presets: `thumb` 200×200,
`card` 600×400, `wide` 1200×630, `hero` 1920×1080 (all cropped), `full` max 2400 wide.

Measured on this server: GD 2.3.3 (JPEG, PNG, WebP, GIF, **no AVIF**); Imagick 6.9.12
(**AVIF, WebP, HEIC**); exif and fileinfo present; no command-line encoders. On a synthetic
2400×1600 source: GD WebP 1200×630 in 129 ms / 29 KB; Imagick WebP 261 ms / 21 KB; Imagick
AVIF 385 ms / 9 KB; Imagick WebP 1920×1080 455 ms. Imagick is the primary encoder, GD the
fallback, AVIF best-effort. **Not yet measured:** a real large photograph, and the GD-only
path forced, so no figure is claimed for what ships.

Built: the `media` and `media_meta` migrations (committed in `517cfbe`, **not applied on the
live install**, and not to be amended or added to before O-1 is decided); the five presets
with focal-point crop geometry (never enlarges, returns true output dimensions); the
uploads guard; installer reporting of upload limits.

Not built:
1. Encoder: apply EXIF orientation before cropping; Imagick first, GD fallback.
2. Upload: finfo MIME sniff plus a separate extension check, explicit rejection of
   anything PHP-adjacent, sha1 dedup, original stored untouched, a readable failure when PHP
   truncates an oversized post.
3. Resumable variant generation: `thumb` and `card` first, then `wide`, `hero`, `full`; a
   record of which variants exist per item (a new column on `media`); a check of remaining
   execution time before each encode; an incomplete item shown as such, with an offer to
   finish. Without this, a slow host kills PHP mid-pipeline and the missing variants 404
   for ever.
4. Library: newest first, search by filename, replace, delete, focal point, alt text and
   caption per language. Deletion is blocked while an image is in use, naming the pages.
5. Picker in the inspector, replacing the numeric id field, and for `surface: image`.
6. Front end: `<picture>` with AVIF, then WebP, then the original; width and height always
   set; focal point honoured; alt text for the locale, empty for decorative images.
7. Demo images: generated with GD at seed time, or CC0 photographs if approved.
8. Per-page meta title and description (D-004).

---

## 3. Order of work

Approved as D-009. Each step gets its own architect's checklist before it starts.

1. **Documentation consolidation** (D-010). Done, verified in `f2a3520`.
2. **Quality pass on what exists:** ← *current*, in parts: 2a content corruption bug,
   2b editor appearance, 2c rich text verification, 2d contrast test (D-012), 2e page
   order (D-011), 2f README upload size, 2g browser checklist for slices 1–4.6. Covers: the rich text editor in blocks; the rich text
   verification listed in section 2; an automated contrast check over every admin
   control; reordering the page list; the browser checklist for slices 1–4.6; README
   states the maximum upload size (the installer already reports it).
3. **Foundations:** O-1 (upgrading an existing install) and O-2 (serving that works on
   nginx and Apache).
4. **Slice 5, media**, and per-page SEO (D-004).
5. **Site settings, header, footer and a menu builder.** The Design screen gains boxed
   layout, page background and header width. See O-7, O-8 and O-9.
6. **Repeater field, the Columns block (D-008), more blocks.** See O-11.
7. **Slice 6, languages**, including adding a language from the admin. See O-12.
8. **Slice 7, forms and mail:** form builder, `{{form:slug}}`, submissions, SMTP and
   Resend, admin notification, autoreply, honeypot, test-mail button. See O-6.
9. **Slice 8, operations:** page cache, backup, update by ZIP upload, revisions,
   sitemap, regenerating media variants (O-13), and 2FA (O-4).
10. **Slice 9, release:** six more blocks (gallery, features, CTA, accordion,
    testimonials, logo strip), three templates, demo site, release ZIP, and the original
    admin look (D-007).

Acceptance criteria for each slice are in `docs/SPEC.md` §8.

---

## 4. Decisions

### D-001: Where decisions are recorded

**Status:** superseded by D-010

This file held architectural decisions only, alongside `docs/plan.md` (progress) and
`docs/product.md` (functional description).

### D-002: The role of each database

**Status:** approved 2026-09-16

- `boxletcms-test` is for the automated test suite only. The suite drops every table
  before each test, so it can never hold anything a person looks at.
- `boxletcms` (https://boxlet.svejedobro.hr) is a demo and acceptance install with no real
  content. It may be reinstalled. No ad-hoc INSERT, UPDATE or DELETE against it.
- No third database for now. Revisit when the first real client site is built on Boxlet.

**Trade-offs.** Reinstalling on a schema change is cheap today, but wipes the demo and the
chosen design each time; that is O-1.

### D-003: Media variants live in `public/m/`

**Status:** approved 2026-09-16

Generated image sizes are stored under `public/m/`, the same path as their address
`/m/{preset}/{id}-{slug}.{ext}`, so the web server sends them from disk without PHP.

**Trade-offs.** They sit outside `public/cache/`, so clearing the cache does not remove
them. That is correct: they are regenerated only deliberately.

### D-004: Per-page meta title and description belong to Slice 5

**Status:** approved 2026-09-16

Two fields per page, defaulting to the page title, emitted in `<head>`. Nothing more: no
sharing image, no robots, no sitemap.

**Trade-offs.** It widens Slice 5 slightly. Moving it to the site settings step would
leave pages without their own search title for several more steps.

### D-005: The spec does not list code that has no caller yet

**Status:** approved 2026-09-16

SPEC §4 does not name `Hooks`, and marks `Cache` as arriving with Slice 8. A document that
names parts that do not exist misleads whoever reads it next.

### D-006: Quality before features

**Status:** approved 2026-09-16

- A slice is done when it has been verified working in a browser against a checklist
  the architect writes, not when it is committed.
- A weak existing feature is fixed before new features are built on top of it.
- No workaround ships as a solution. If the right fix is too big for now, the gap is
  recorded here as an open item.

**Trade-offs.** Slower visible progress. The project already shows the cost of the
opposite: slices marked done with open verification, and a plan quietly bypassed.

### D-007: The admin must look good and be original

**Status:** approved 2026-09-16

A release requirement (step 10). Until then nothing may make it harder: the admin keeps
its own fixed design system, separate from the site's design tokens.

### D-008: A "Columns" block, not a nested grid

**Status:** approved 2026-09-16

A basic grid is one block with two, three or four columns. Every column holds the same
bounded content: image, heading, text, button. It is built on the repeater field. No other
block can be placed inside a column, and a page stays a flat list of blocks.

**Trade-offs.** Less freedom than a real grid. In return the frozen schema is unchanged,
translation, revisions and caching stay simple, and every combination still looks
designed.

### D-009: Order of work

**Status:** approved 2026-09-16

The order in section 3. Features the owner wants (chrome, menus, more blocks) wait behind
foundations: media needs O-1 and O-2 first, and chrome needs media for the logo.

### D-010: Two working documents

**Status:** approved 2026-09-16

This file (what, where, next, decided, open) and `docs/SPEC.md` (the technical contract),
plus `CLAUDE.md` (rules for agents, loaded automatically by Claude Code) and `README.md`
(installation). `docs/plan.md` and `docs/product.md` are folded into this file and removed.
SPEC §9's open questions moved to section 5. SPEC §8 keeps only acceptance criteria, and
the order lives in section 3. SPEC's changelog stays frozen as history; from now on
history is the decisions in this file. A fact is written once, and every other mention is
a pointer.

Verified by the architect 2026-09-16 in `f2a3520`. The owner chose plain pointers in
`CLAUDE.md`, without restating the rules they point to.

**Trade-offs.** This file is long. In return the owner follows the whole project from one
place, and each file has a single author, so documents cannot silently contradict each
other.

### D-011: Page order

**Status:** approved 2026-09-16

Pages are ordered by `sort` within the same parent and locale. The page list shows the
tree, with children indented under their parent. Reordering is by dragging among siblings,
with Up/Down buttons that work without JavaScript, the same fallback principle as the page
editor. A page's parent is changed in page settings, never by dragging.

**Trade-offs.** A page cannot be dragged to another level. In return nothing gets
re-parented by accident, which matters more once addresses follow the hierarchy (O-10).

### D-012: "Visible at rest" is measurable

**Status:** approved 2026-09-16

Every admin control, including disabled ones, has at least 3:1 contrast against its
surface, and enabled text controls 4.5:1. Disabled looks different but is never
invisible. A test enforces it over every admin stylesheet by computing the colour pairs
controls use and rejecting opacity that breaks them. A browser check stays as the second
line, because a stylesheet test cannot see one rule overriding another, which is exactly
how the rich text toolbar got to 5% opacity.

**Trade-offs.** Some disabled states look more present than a designer might choose. A
control nobody can see is worse.

### D-013: A browser for checks, installed on the server

**Status:** approved 2026-09-16

Playwright and Chromium are installed on this server, so the work can be checked in a real
browser (D-006). Conditions:

- Installed outside the project and outside the web root (under the user's home). Never
  in the repository, `composer.json`, `vendor`, or a `package.json`/`node_modules` in the
  project, so nothing of it can reach the release ZIP.
- Run only on demand for a check, against the site on this machine. No service, no open
  port, nothing started at boot.
- Driver scripts stay outside the project. Keeping them is decided when there is a second
  use.
- The rule "fix the instrument before judging the subject" (CLAUDE.md) still applies:
  slowed driver, real input, a control.

**Trade-offs.** Several hundred MB of developer tooling on a live web server, and a
browser binary that has to be kept updated. Accepted over driving the owner's own Chrome,
because checks can run whenever work finishes, without the owner present. The owner's
hands-on pass stays for anything visual.

---

## 5. Open items

**O-1. Upgrading an existing install.** The Migrator runs only from the installer, so a
new migration never reaches an existing install. The live install has not applied
`0011_media.sql` or `0012_media_meta.sql`. Needed before any new table arrives, and
required by the public release. *Step 3.*

**O-2. Serving on nginx and Apache.** The page cache (Slice 8) and CLAUDE.md's security
reminders assume `.htaccess` file checks, which nginx ignores. Needs a design that works on
both. *Step 3.*

**O-4. 2FA.** SPEC §6 describes it: optional, ten single-use recovery codes, and a reset by
placing `storage/disable-2fa` on the server over FTP. To confirm: it stays optional rather
than forced (a single admin with forced 2FA and a lost phone means a lost site).
*Step 9.*

**O-6. Resend transport.** Slice 7 promises Resend, but no Resend package is on the closed
dependency list. Options: Resend's SMTP endpoint through symfony/mailer, or its HTTP API
called directly. *Step 8.*

**O-7. Menus.** A menu as its own entity (what a "menu builder" implies) or derived from
the page tree via `parent_id` and `sort`. Leaning: its own entity. Whether a menu gets a
replacement tag is part of this. *Step 5.*

**O-8. Header and footer.** Blocks rendered by the same registry, which inherit every
style layer for free but must appear on every page without the user adding them, or chrome
outside the block system, which needs its own styling mechanism and duplicates what
exists. Header variants (centred, left, transparent over a hero, sticky) are a design
decision, so characters should carry them like any other composition default. The logo
depends on media. Language switcher placement once Slice 6 enables a second locale.
*Step 5.*

**O-9. Site settings and SEO.** The settings table is written only by the installer.
There is no screen for site name, favicon, time zone, default sharing image, analytics,
robots.txt or sitemap.xml. Open: does a favicon go through the media pipeline (probably
yes), and which sizes? Does the default sharing image belong to the site or to the
character? Is sitemap.xml generated on demand or written on publish? It must carry hreflang
after Slice 6. Analytics means third-party script, which collides with the admin CSP and
the privacy posture behind self-hosted fonts, so it is decided deliberately rather than
added as a snippet field. *Step 5.*

**O-10. Nested page addresses.** Addresses are one path segment, unique per locale, while
`parent_id` expresses hierarchy only in the admin, so the data model and the address
disagree. Planned: nested paths (`/about/team`) together with a redirects table. The two
are inseparable: renaming a parent otherwise regenerates every descendant's address into a
404. A redirects table is needed regardless, since renaming any slug breaks an existing
address today. *Not scheduled; must be placed in the order before the release.*

**O-11. Repeater blocks.** Three block types are too few for a real site. The missing
shape is repeating content: a team grid, a features row, a logo strip. `repeater` is in
the closed field-type list and has never been built. It is the basis of the Columns block
(D-008). It comes after media, because a team grid without photographs cannot be judged.
*Step 6.*

**O-12. Before Slice 6, languages.** Which AI translation provider ships as the default,
and does Boxlet ship with translation switched off until the owner adds their own key
(almost certainly yes, since the alternative is shipping someone else's bill)? What does a
visitor get for a page with no translation yet? Recommended: configurable per site,
defaulting to hidden from navigation and a 404 on a direct hit. *Step 7.*

**O-13. Regenerating media variants.** Adding, removing or resizing a preset leaves
existing media with the wrong set. A regeneration pass must be runnable from the admin,
resumable and safe on a live site. It should be a thin wrapper around the resumable upload
machinery from Slice 5, not a new subsystem. *Step 9.*

**O-14. News / blog.** Rejected: a separate post type, which brings chronological indexes,
pagination, archives, RSS and prev/next, each multiplied by language, and tags, which the
non-goals exclude. Also rejected: a page-level "blog container" flag, which moves behaviour
into the render path and creates invisible state (a page showing twenty entries that appear
nowhere in its editor). Answer: a `page_list` block listing child pages by `published_at`,
with `list` and `grid` layouts, filled automatically by parent only, plus a seeded "News
index" template. Manual selection ("Featured work") is a different need and is deferred
until someone asks. Open: what a listed page contributes to its card (title and date are
obvious; an excerpt and a thumbnail are not fields on a page). *Not scheduled.*

**O-15. Block library categories.** Revisit when the block count passes roughly fifteen. A
category filter over fewer items is furniture, and a filter quicker to ignore than to use
is worse than none. *Not scheduled.*

**O-16. Typed configuration getters.** `Config::get()` and `Container::get()` return
`mixed`, which keeps PHPStan below level 9 (about 30 findings). It was due to be revisited
after Slice 3 and was not. *After Slice 5.*

*Closed:* O-3 (order after Slice 5) was resolved by D-009. O-5 (documentation that
disagreed with the code) was done in `1782cc4` and checked against the diff.
