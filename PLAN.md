# Boxlet — the plan

The one document to read to understand the project: what Boxlet is, where the work
stands, what comes next, what has been decided and what is still open.

**Who writes what.** Since D-033 one working session keeps this file current as it goes.
The owner decides what they will see and judge and anything that changes the product's
shape; small technical decisions are made by the session and recorded here. Older entries
mention an architect and an executor, the arrangement before D-033, and are left as
written. How the system works technically is in `docs/SPEC.md`, the contract. How an
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

*The table is current as of 2026-09-18. The lists below it were last reviewed on
2026-09-16; later progress is recorded under section 3.*

| | |
| --- | --- |
| Last commit | see `git log`; a commit is pushed once its tests pass on both drivers and PHPStan is clean |
| Tests | 1,069 on both drivers, PHPStan clean at level 8 (2026-09-22) |
| CI | read after every push from GitHub's public API (CLAUDE.md) |
| Development site | https://boxlet.svejedobro.hr, MySQL `boxletcms`, demo site (D-002). It is this checkout: no separate clone, no deploy step (D-033) |
| Demo admin | `acceptance@example.com`; the password is never in the repository |

Slices 1–4.6 are verified in a browser against the architect's checklist (2g and 2h,
2026-09-17), and the owner has tried the editor by hand.

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
- Pages are listed as a tree and reordered among siblings, by dragging or with Up/Down
  buttons that work without JavaScript (D-011).
- Missing: page SEO fields (D-004), more blocks and columns (step 6).

**Writing (rich text with TipTap, D-017)**
- Bold, italic, links, headings H2/H3/H4, quotations, bullet and numbered lists with
  nesting. The toolbar offers exactly what can be stored, and the editor's schema cannot
  produce anything else.
- Changing a heading's level replaces it; pressing the active level returns to a
  paragraph. The owner tried this on the live demo on 2026-09-16 and approved it.
- The link panel stays hidden until the link button (or Ctrl+K) is pressed. It opens
  prefilled on an existing link, and closes on Link, Unlink, Escape or a click back into
  the text.
- Stored as HTML and cleaned on the server, which keeps one stored shape whatever the
  editor sends (D-014, D-016, SPEC §5.3). Opening and saving without edits leaves content
  byte-identical.
- Pasting from Word is cleaned. Measured against a plain editable element, which kept
  MsoNormal classes, inline styles, font tags and a whole table; TipTap keeps none of them.
- Every rich text field can be switched to plain HTML and back.
- Moving, dragging and duplicating blocks keeps every editor bound to its own field.
  Found as a content corruption bug and fixed in `a6487e8`.
- The editor makes zero Content Security Policy violations.

**Checked by the architect:** the code in `e770358` (CI green on GitHub), and the
executor's browser checks. **Confirmed by the owner** on the live demo, 2026-09-16: heading
levels, formatting, lists, saving, and the link panel hidden until clicked.
**Not yet verified:** the editor inside blocks newly inserted from the library, and paste
as plain text (2c-3).

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
fallback, AVIF best-effort. Measured on a real photograph (2400×1600, 691 KB), all five
presets: Imagick 15 files (AVIF, WebP, original format) 2227 KB in 6.1 s; forced GD 10 files
(WebP, original format) 1989 KB in 2.4 s. The slowest single encode is a full-size AVIF at
1.27 s. Thumbnails come out at 4–10 KB, hero at 117–361 KB, AVIF smallest every time.

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
2. **Quality pass on what exists:** done, closed 2026-09-17
   - Done: 2a content corruption bug (`a6487e8`); 2b editor appearance, confirmed by the
     owner; 2c rich text survives open and save, and its edge cases (`0029c27`, `7629925`);
     the editor replaced by TipTap (D-017, `e770358`); files split under the 300-line rule
     (`6f967e1`); 2d contrast test, D-012 (`1235ace`); 2e page order, D-011 (`11f045d`),
     which also fixed the plain editor dropping a page's parent on save; 2f README upload
     size (`26c842c`); Title and Address inputs aligned (`2a1ca38`).
   - Owner's hands-on pass, 2026-09-16: writing, blocks, dragging, page ordering,
     selection outlines and the insertion control all confirmed.
   - 2g, browser checklist for slices 1–4.6, run 2026-09-17 with the reusable suite
     (`~/boxlet-browser/suite`, D-018): 46 pass, 0 fail, 2 not checkable. Slices 2, 3, 4 and
     4.5 are verified. Adding a second language from the admin does not exist yet (Slice 6);
     its routing was checked with seeded data.
   - 2h, text on the canvas updates while you type (`159bec8`), confirmed by the owner
     2026-09-17. It also fixed rich text edits being missed by the unsaved-changes warning.
   - Open question: `h4` is set at body size in every character, because the type scale
     has no step between body and the next size up.
   - Noted for Slice 5: nothing handles uploads yet, and nginx refuses request bodies over
     1 MB by default (`client_max_body_size`) before PHP runs, so the uploader and O-2
     must account for it.
3. **Foundations:** done 2026-09-17. D-019 updating an existing install and D-021
   maintenance mode (`edcb6cb`); D-020 serving without PHP on nginx and Apache (`3a65aef`).
   The owner ran the first real update on the live site (media tables applied).
4. **Slice 5, media**, and per-page SEO (D-004). ← *current*
   - Done: 4a encoder, upload and resumable generation (`da428df`, migration 0013 applied
     on live by the owner); admin strings split by concern and nested by locale
     (`34b01d8`, `30214ff`).
   - Since `30214ff` the executor works in `~/boxlet-dev` and deploys to the demo by pull
     (D-023). Live is at `30214ff`.
   - Also done: 4b library (`c6195e1`), 4c picker (`9695e1d` and follow-ups), 4d pictures on
     the front end (`79113c5`), library renamed Media, CI repaired after twelve red runs
     (`14253eb`), D-025 suggested alt text (`d2bd06e`, migration 0014 applied by the owner).
   - Also done: D-026 crop (`47558d0`), a contrast guard for button variants (`19717d6`),
     4e per-page SEO (`613bacf`), 4f CC0 photographs on the demo (`765236b`), confirmed by
     the owner 2026-09-17.
   - The Slice 5 acceptance check from SPEC §8 passed 2026-09-19 on the development site
     (scenario 16, rewritten to save nothing and delete its photograph): the 4.16 MB
     photograph is served in the hero as AVIF at 144 KB, and the second request is answered
     from disk — its ETag is the file's own mtime and size, which only nginx reading the
     file sends. It first failed at 279 KB, which is how O-18's cause was found.
5. **Site settings, header, footer and a menu builder.** Built 2026-09-17/18 and green on
   CI: menus (`bc7a370`), the browser suite moved into the repository (`b47a031`), the
   render wrapper (`ca92d1c`), header and footer (`a782347`), the chrome screen
   (`632a474`), the stale-code guard (`5a3a2fb`), and the page as a sheet (`521231d`).
   Deployed to the demo on 2026-09-18 (`521231d`), with migration 0015 applied by the owner
   through the update screen — D-019's gate behaved on the live site exactly as described.
6. **Before more blocks, in the owner's order of 2026-09-18:** ← *current*
   - Housekeeping: one session on the development site (D-033), pending migrations applied
     from the command line.
   - a. Links point at pages, not typed paths: a page reference wherever a link is entered,
     rich text included (D-034). Built 2026-09-18 with tests on both drivers and browser
     scenario 20-page-links.
   - b. The admin's own design system reworked (D-007 brought forward): spacing, type,
     panels, buttons, tables, forms, empty states. The owner judges before/after screenshots.
   - c. The design layer: D-032's chrome choices, a real mobile menu, the current page
     marked (built 2026-09-18, D-036); then proposals for what makes two Boxlet sites look
     genuinely different (sent to the owner 2026-09-18, waiting for a yes).
   - d. Then the Columns block (D-008) and more blocks. The repeater field it stands on is
     done (6a, `e141816`). Columns built 2026-09-18 (D-041), taken before 6c's answer with
     the owner's go-ahead; scenarios 24-columns and 25-columns-look.
7. **Slice 6, languages**, including adding a language from the admin. Decisions in D-043.
   ← *next*, started 2026-09-19. Design elements are paused while the owner analyses
   them (6c and the header and footer proposals).
8. **Slice 7, forms and mail:** form builder, submissions, SMTP and Resend, admin
   notification, autoreply, honeypot, test-mail button. Built 2026-09-19 (D-045, D-046);
   left: receiving both emails for real, with the owner's mail account.
8a. **Statistics** (D-051), round 1: counting, the Statistics screen, the dashboard card,
   Settings. Round 1 done 2026-09-19 (counting, Settings panel, screen, dashboard card,
   countries, privacy text). Round 2 is O-20.
8b. **The admin redesign, "Workbench"** (D-052). Done 2026-09-19; the whole browser suite
   green at the end of it. Steps:
   1. Dark palette: every `--ui-*` token retuned, contrast matrix re-measured.
   2. The shell: a grouped left rail and a top strip.
   3. The activity log: migration, writes from the controllers, a Full log screen.
   4. Overview: a metric strip, Needs attention, Most read, Recent activity.
   5. Pages: the table restyled, a language filter, the Home badge, an overflow menu.
   6. Media: a table with "Used on" and its filters.
   7. Settings as a ledger with its own sub-navigation.
   8. The ⌘K palette, with a `/admin/search` page behind it.
8c. **Statistics, round 2** (O-20). Done 2026-09-20, at the owner's choice over Slice 8:
   narrowing by clicking with the state in the address and a range of your own, counting the
   addresses that are not there, the world map, the visitor's address behind a proxy,
   gathering small rows, export, and the footer credit (O-20).
8d. **Appearance: one screen for the design, the header and the footer** (D-057). Begun
   2026-09-20 at the owner's request, against `docs/design_handoff_appearance/`. Three rounds
   were approved, then a reassessment: (1) the preview draws the real header and footer;
   (2) the feedback loop closes (D-058); (3) the two screens become one `/admin/appearance`
   with five tabs (D-059), the old addresses go (D-077), and the toolbar over the
   picture — viewport, zoom, Compare — follows.
9. **Slice 8, operations:** ← *next*. The page cache (D-053, decided and not yet built), backup,
   update by ZIP upload, revisions. Done already: the sitemap (D-049), regenerating media
   variants (O-13, D-048) and two-step login (O-4, D-050).
10. **Slice 9, release:** replace the development photographs (D-022); six more blocks (gallery, features, CTA, accordion,
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
editor. A page's parent is changed in page settings, never by dragging. Dragging uses SortableJS, which is already vendored for the canvas (SPEC §3), so no new asset is added.

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

Clarified 2026-09-17: the 3:1 boundary is for controls whose edge is the only thing that
marks them (empty inputs, icon-only buttons, an empty picker). A button with a readable text
label is visible through its text, and its text meets 4.5:1. Every button variant declares
its own background, so its text is measured against the ground it actually sits on.

**Trade-offs.** Some disabled states look more present than a designer might choose. A
control nobody can see is worse.

### D-013: A browser for checks, installed on the server

**Status:** approved 2026-09-16, corrected the same day after installation

A headless browser is installed on this server, so work can be checked in a real browser
(D-006). It is **Puppeteer 25**, not Playwright: Playwright needs Node 20 and the host runs
Node 18.19.1. Chrome lives in `~/.cache/puppeteer` (658 MB); the driver lives under the
user's home. Chrome needs five system libraries (libasound2, libatk-1.0, libatspi,
libXdamage, libxkbcommon). The owner installed them as root on 2026-09-16 (plus ten dependencies, about 5 MB);
chrome-headless-shell (arm64, 153.0.8010.36) then starts. Only headless-shell is used; full
Chrome would need three more libraries and is not installed for.

Conditions:

- Installed outside the project and outside the web root. Never in the repository,
  `composer.json`, `vendor`, or a `package.json`/`node_modules` in the project, so nothing
  of it can reach the release ZIP.
- Run only on demand for a check, against the site on this machine. No service, no open
  port, nothing started at boot.
- Driver scripts stay outside the project. Whether to keep them in the repository is
  decided when there is a second use.
- The rule "fix the instrument before judging the subject" (CLAUDE.md) still applies:
  slowed driver, real input, a control.

**Trade-offs.** About 700 MB of developer tooling, five system libraries on a live web
server, and a browser that has to be kept updated. Accepted over driving the owner's own
Chrome, because checks can run whenever work finishes, without the owner present. The
owner's hands-on pass stays for anything visual.

### D-014: Rich text survives being opened and saved

**Status:** approved 2026-09-16; the load mapping below was superseded by D-017 (TipTap needs none), the save rules stand

Fixed at the source and guarded on the server:

- **On load**, the editor hands Trix the shapes it owns: `p` becomes `div` and `h2` becomes
  `h1`. This is the exact inverse of the sanitiser's `div → p` and `h1 → h2` on save, so
  what is stored does not change. Measured on the load path alone: without it, a break
  trapped inside `strong`, `em` or `a` keeps growing, beyond the reach of any server rule.
- **On save**, the server strips `<br>` at the start and end of every block element (`p`,
  `h2`, `h3`, `li`, `blockquote`). It repairs content that has already grown, and it holds
  whatever an editor sends.
Acceptance: sanitising is idempotent, and in the browser three open-and-save cycles leave
every rich text field byte-identical.

**Trade-offs.** A deliberate line break at the very edge of a paragraph is lost. In Trix
a blank line is a new paragraph, so nothing a user can type is lost.

### D-015: A subheading button in the rich text editor

**Status:** superseded by D-016 (the separate button); its round-trip requirement stands

Trix gets a second heading level that emits `h3`, with a "Subheading" button in the
toolbar, so a subheading survives being opened and saved. `h3` is already on the
whitelist, so the toolbar still offers only what can be stored. If Trix cannot round-trip
it cleanly in the browser, the fallback is to load `h3` as the single heading level,
stored as `h2`.

**Trade-offs.** One more toolbar button in a narrow inspector. The alternative was a
subheading silently turning into bold body text on its first save.

### D-016: One heading button with levels H2, H3 and H4

**Status:** approved 2026-09-16; the menu was superseded by D-017 (TipTap has H2, H3 and H4 as buttons), the stored `h4` level and the smaller toolbar stand

The rich text toolbar has a single "Heading" button that opens a small menu: H2, H3, H4.
It replaces the separate heading and subheading buttons.

- The stored set gains `h4`: rich text may hold `h2`, `h3` and `h4`. `h1` still becomes
  `h2`, and `h5`–`h6` now become `h4` (previously `h4`–`h6` became `h3`).
- Every level must survive being opened and saved (D-014), and choosing a level from the
  menu must store exactly that tag.
- The menu works from the keyboard, shows the active level, and closes on a choice or on
  Escape. Its styles live in `admin-richtext.css`, because the admin's security policy
  refuses anything Trix injects.
- `h4` gets a front-end style in `site.css`, through design tokens only, checked under all
  five characters.
- The toolbar also gets smaller at the owner's request. Buttons are about 1.75rem and never
  under a 24px target, icons are about the size of field text, and the icon weight is
  close to Trix's own. It stays on one row.

**Trade-offs.** A third heading level to design in every character, and a rule in the
technical contract that changes before v0.1 (SPEC §5.3). A heading level is one click
further away. In return the toolbar is smaller, and authors get the depth they asked for.

### D-017: Try TipTap as the rich text editor (spike first)

**Status:** approved 2026-09-16, and adopted the same day: the owner tried the spike on the live demo and said yes

Trix fights what we need from it. It has one heading level, and it manages focus and the
selection itself, so every addition around it (a second level, a heading menu) produced a
new bug. The owner tried the heading menu and found a stray highlight, no direct H2→H3
switch, and a heading applied to the wrong paragraph. The owner also tried TipTap's demo,
which worked cleanly. TipTap is built on ProseMirror (MIT, very widely used).

It was ruled out before because it is not distributed as one file. The exception approved
here:

- **One maintainer-built bundle.** Pinned TipTap/ProseMirror packages (MIT core only, no
  paid extensions) are bundled once into `public/assets/vendor/tiptap.bundle.min.js`,
  committed with a header naming every version. The recipe (package.json, lockfile, one
  entry file) lives in `tools/tiptap/`, is excluded from the release ZIP, and is built
  outside the project so no `node_modules` ever sits in it. Users never build anything,
  so the product promise "no build step" holds. A second exception would need its own
  decision.
- **Why it should fit.** ProseMirror's schema allows only the nodes we define, which is
  exactly our whitelist. A heading is one node with a level, so changing H2 to H3 replaces
  it. HTML goes in and out, so the storage contract and the server sanitiser are unchanged.
  TipTap's CSS injection can be turned off, so the admin's security policy stays as it is.
- **Spike, on a branch.** The same textarea contract, including the no-JavaScript
  fallback, and a toolbar of bold, italic, link, H2/H3/H4, quote, lists, undo/redo.
  Acceptance:
  - the owner's own scenarios: select text and change its level, change an existing
    heading's level, remove a heading, no stray highlight, nothing applied to the wrong
    paragraph;
  - three open-and-save cycles byte-identical on every field;
  - Word paste against a control;
  - moving and duplicating blocks still bind each editor to its own field.
  The owner tries it on the live demo with the branch checked out. It merges only on their
  yes; otherwise main stays on Trix.

**Spike result (2026-09-16, `spike/tiptap`).** TipTap 3.31.3 bundles on Node 18. The
bundle is 372 KB minified, 120 KB compressed (Trix: 208 KB and 52 KB), almost all of it
ProseMirror itself, and it loads only in the admin. All 33 bundled packages are MIT,
checked in each package's own metadata. It causes zero CSP violations (Trix: four per page
load). The owner's scenarios, moves and duplicates, round trips and Word paste all pass.
The server now also unwraps the paragraph TipTap puts inside list items, quotes, and list
items holding a nested list, so stored content keeps one shape whichever editor wrote it.

**Trade-offs.** Updating the editor needs Node and npm on a maintainer's machine, and the
bundle is about 1.8 times Trix's size (2.3 times compressed). Part of the editor work already done for Trix is redone. In
return we get an editor we build on rather than around.

**If adopted:** Trix and its CSS are removed; the Trix-specific parts of D-014, D-015 and
D-016 are superseded (their requirements stand); SPEC §3 and §5.3 are updated.

### D-018: Working speed

**Status:** approved 2026-09-17; its architect parts (restarting a stalled executor,
reviewing each commit) are superseded by D-033, and the suite moved to
`tools/browser-suite` (D-029)

The work was stalling on process, not on the product: every command and every message
waited for the owner, the executor ended turns without doing the steps it announced, and
each browser check was written from scratch. Four changes:

- **Standing permissions for the executor.** It may run the tests, PHPStan, git (including
  commit and push to main), the browser scripts in `~/boxlet-browser`, and the local dev
  server on the site copy, and it may edit files inside the project, without asking.
  Changes to `CLAUDE.md`, permissions and configuration still go to the owner. The owner
  grants this directly in the executor's window; a peer cannot.
- **The architect restarts a stalled executor.** When the executor goes idle without a
  report, the architect sends it a message to continue, so the owner does not have to.
- **A reusable browser suite.** Browser checks live in `~/boxlet-browser` as one scenario
  file per area, run with one command. A new feature adds its scenario instead of a one-off
  probe (D-013 conditions stand).
- **Verification in proportion to the change.** The browser is for what a person clicks
  and sees. Tests cover logic, storage and documentation. A task is worked through without
  pausing between its parts, and the architect reviews each commit as it lands.

**Trade-offs.** Less of the work passes before the owner's eyes as it happens. D-006 still
holds: nothing counts as done without the architect's review, and anything visual still
goes to the owner.

### D-019: Updating an existing install from the admin

**Status:** approved 2026-09-17 (resolves O-1)

- When the code carries migrations the database has not applied, the admin shows a
  "Database update needed" screen with one button. Migrations run only when the owner
  presses it: never on their own, never on a visitor's request.
- While an update is pending, the public site shows a short "being updated" page with
  status 503 instead of failing on a table that does not exist yet.
- Before running, a SQLite database is copied into `storage/backups/`. On MySQL, where a
  real backup arrives with Slice 8, the screen tells the owner to take one through the
  host first.
- Only one run at a time (a lock file). A failure stops the run and names the file.
- A migration that has been committed is never edited again; a change is a new migration.

**Trade-offs.** Between uploading a new version and pressing the button, visitors see the
"being updated" page. That beats a broken site, and the window is in the owner's hands.

### D-020: Serving without PHP, identically on nginx and Apache

**Status:** approved 2026-09-17 (resolves O-2)

- Anything served without PHP is a real file at its own URL under `public/`: the compiled
  design stylesheet and the media variants in `/m/`. Both servers already serve existing
  files first, so no server-specific rule is needed.
- The page cache (Slice 8) runs inside PHP, checked in the first lines of the front
  controller before anything else boots, so it works on every host without
  configuration. A server rule that skips PHP entirely is documented as an optional extra.
- Original uploads are stored outside the web root, in `storage/`. Only the generated
  variants are public. `.htaccess` cannot stop a script in a public upload folder from
  running on nginx, and a file that is never public cannot run anywhere. The `full`
  variant, up to 2400 px wide, is the largest public version.

**Trade-offs.** SPEC §4 and §5.5 change where originals live (allowed before v0.1). An
original can no longer be linked for download as the exact uploaded file.

### D-021: Maintenance mode

**Status:** approved 2026-09-17

One mechanism with two triggers. The "being updated" page from D-019 is also a maintenance
mode the owner switches on and off in the admin.

- While it is on, visitors get the maintenance page with status 503 and `Retry-After`, so
  search engines come back later instead of dropping pages. A logged-in admin still sees
  the real site, with a visible bar saying maintenance is on.
- It switches on automatically while a database update is pending (D-019), and later
  during an update by ZIP upload (Slice 8), switching off when that finishes.
- The state is a file in `storage/`, not a database row, so it works when the database is
  unavailable or mid-update.
- A custom message for visitors comes with site settings (step 5). Until then it is a
  short standard text.

**Trade-offs.** A small addition to step 3. It reuses the page and gate being built, so
nothing is duplicated.

### D-022: Photographs for the demo

**Status:** approved 2026-09-17, revised the same day

Real photographs are needed to judge the design. They are **public domain (CC0)**, from
sources such as Wikimedia Commons, and they may ship with the release.

- Unsplash was chosen first, for development only. It turned out to refuse downloads from
  this server (401, an anti-bot filter), and its licence would have required replacing the
  pictures before release anyway. Going straight to CC0 skips a step rather than adding one.
- They are fetched by a script outside the repository and uploaded into the demo through
  the media library, as a person would, which exercises the uploader too.
- The source URL, author and licence of each one are recorded next to the script.
- The demo site installs and renders without them, with placeholders.
- The owner judges the selection from screenshots of the demo.

**Trade-offs.** The choice on CC0 sources is narrower than on Unsplash, so picking good
photographs takes more care. In return there is no licence question anywhere, no API key,
and nothing to swap out before release.

### D-023: Develop in a separate checkout, deploy to the demo on purpose

**Status:** superseded by D-033 (decided by the architect on the owner's delegation,
2026-09-17)

Until now the executor worked directly in the live checkout, so half-written code was live
the moment it was saved, and every new migration took the demo offline unannounced.

- The executor works in its own clone, `~/boxlet-dev`, with its own `vendor/` and no
  connection to `boxletcms`. Commits and pushes happen there. Browser checks keep using
  the copies in `~/boxlet-browser` (D-013).
- The live checkout (`htdocs/boxlet.svejedobro.hr`) is never edited. It changes only by
  `git pull --ff-only` from `origin/main`, after a part is committed, pushed and green.
- A deploy without migrations happens as soon as a part is done. A deploy that carries a
  migration is announced first through the architect, so the owner is ready to press the
  update button (D-019) and the demo is dark for seconds rather than for however long it
  takes someone to notice.
- The architect edits `PLAN.md` in `~/boxlet-dev`; it reaches the live checkout with the
  next deploy.
- The switch happens at a clean boundary: after 4a is committed. The executor keeps its
  session; the owner runs `/add-dir ~/boxlet-dev` in it so file edits apply there.

**Trade-offs.** One more step between finished work and the demo, and a second copy of the
project on the server (a few MB plus `vendor/`). In return the demo only ever runs code
that was finished, tested and pushed, which is how a client site will have to be treated.

### D-024: A section's background picture lives in its section style

**Status:** approved 2026-09-17

Layer 2 (`page_blocks.style_json`) gains a sixth key, `image`: the id of a picture from
the media library, or null. It is used only when `surface` is `image`.

- It is not a free value. On save and on render it must name an existing media item;
  anything else falls back to null, and `surface: image` without a picture renders as it
  does today (like `contrast`).
- It works identically for every block type, because section style belongs to every block.
  No block definition gains a background field.
- Checking whether a picture is in use therefore looks in `content_json` (media fields) and
  in `style_json` (`image`).
- Text over the picture stays legible through tokens only (D-012 thinking applied to the
  site). The owner judges it under all five characters.

**Trade-offs.** Layer 2 stops being five enumerated keys; SPEC §5.4 changes before v0.1.
The constrained-freedom rule still holds, because the value is a reference to a picture
already in the library, not an open input.

### D-025: Suggested alt text on upload

**Status:** approved 2026-09-17

When a picture is uploaded, its alt text in the primary language is filled in
automatically, unless it already has one:

1. From the picture's own metadata, if it carries a title or description (IPTC headline or
   object name, EXIF image description or title). Generic camera strings such as
   "OLYMPUS DIGITAL CAMERA" are ignored.
2. Otherwise from the file name, tidied: extension removed; underscores, hyphens and dots
   become spaces; first letter capitalised. `tim-u-uredu_2024.jpg` becomes
   "Tim u uredu 2024".
3. Names a device generated (`IMG_1234`, `DSC_0012`, `PXL_…`, `Screenshot…`, bare
   numbers, hash-like strings) are skipped, and the alt stays empty.

A filled-in alt is marked **suggested** in the library until the owner confirms or edits
it, so a guessed description is never mistaken for a written one. Replacing a picture
never overwrites alt text that was confirmed.

**Trade-offs.** A migration (the "suggested" mark) and a small amount of guessing. This
refines the earlier rule "no alt rather than a file name": a meaningful file name is used,
a device-generated one still is not, because a screen reader reading "IMG 4032" aloud is
worse than silence.

### D-026: Cropping a picture in the library

**Status:** approved 2026-09-17

The library gets a crop tool built on Cropper.js 1.6 (MIT, one JS and one CSS file,
vendored under the SPEC §3 rule, requested by the owner).

- Aspect ratios tied to the presets: 16:9 (hero), 3:2 (card), 1.91:1 (wide, also the
  social sharing ratio), 1:1 (thumb), and Free.
- **Save as new picture** creates a new library item from the crop. The original stays as it
  was; alt text and caption are copied.
- **Replace this picture** keeps the same item, so every page using it shows the crop, and
  regenerates all variants. It cannot be undone, and the dialog says so and points to
  "Save as new" for keeping the original.
- The browser only sends the crop rectangle. The server cuts the full-quality original,
  with EXIF orientation applied first, and validates the rectangle.
- The focal point carries over when it lies inside the crop; otherwise it returns to the
  centre.

**Trade-offs.** SPEC §5.5's "original stored untouched" gains one exception: an explicit
Replace. A replaced original is gone; keeping it was offered and not chosen, to avoid a
second copy of every cropped picture and an extra "restore" control.

### D-027: Faster small changes

**Status:** approved 2026-09-17

Small changes were taking as long as large ones, because every change went through the
same full cycle of browser checks, long reports and questions. Four changes:

- **300 lines is a guideline, not a wall.** Past 300, a file is split only along a real
  seam. The hard limit is 500. Tests, language files and browser-suite scripts are exempt.
- **Verification is proportional to the change.** A small logic or wording change needs
  the test suite. A visual change needs one screenshot. The whole browser suite runs for
  larger parts, and before a deploy that carries a migration or a new screen.
- **Changes are batched.** Several small changes share one round of checks, one CI run and
  one deploy.
- **Short reports, fewer questions.** A report is at most about 15 lines: commit, CI
  conclusion, deviations, questions. Detail only when something went wrong. The architect
  makes small technical decisions and records them; the owner is asked about what they will
  see and about the product.

**Trade-offs.** Less evidence per small change, and a larger batch is harder to pin down
when something breaks. D-006 still holds for every larger part: CI green before deploy, the
architect's review, and the owner's eye on anything visual.

### D-028: Site chrome, menus and site settings

**Status:** approved 2026-09-17 (resolves O-7, O-8 and most of O-9)

**Header and footer are set once for the whole site**, on their own screen, not per page.
They are drawn by the same block machinery as everything else, so they inherit the design
tokens and the section style layers, and each character carries its own header variant
(centred, left, transparent over a hero, sticky). The header holds a logo, a menu and an
optional button; the footer holds text, a menu and small print.

**Menus are built by hand** (O-7). A menu is its own thing, not the page tree. An item
points at a page or at an address of its own, can carry its own label, is ordered by
dragging, and may have one level of submenu. A menu exists per locale, so a translation
has its own labels. The page tree still orders the admin list (D-011) and will feed a
future page_list block.

**Site settings** (O-9), this round: site name, logo, favicon, time zone, the default
sharing image, and the maintenance message (D-021). Favicon and logo come from the media
library.

**Analytics is not built.** Embedding someone else's script touches a visitor's privacy and
the admin's security policy, and it gets its own decision.

**The design layer gains** header width, a boxed page layout, and a page background.

**Trade-offs.** One header for the whole site means no per-page chrome; that is the
constrained-freedom choice and it keeps a site coherent. A hand-built menu is a little more
work than one derived from the page tree, in exchange for deciding what is in it and what it
is called.

### D-029: The browser suite lives in the repository

**Status:** decided by the architect on the owner's delegation, 2026-09-17

The suite (nineteen scenarios and its harness) existed only on one machine's disk, under no
version control. It moves into the repository as `tools/browser-suite/`, beside the TipTap
recipe.

No release ZIP is built yet — there is no build script, so nothing is excluded from anything
today. When the release build is written (Slice 9) it must leave out `tools/`, `tests/`,
`.github/` and the development files, and that is where the mechanism is decided.

- What stays outside: Chrome and Puppeteer, the site copies, screenshots, downloaded
  photographs and anything with credentials in it (D-013).
- Paths that point outside the repository become configuration with a default, so the suite
  runs from a checkout on another machine.
- A scenario reports NOT CHECKABLE only for a limit of the environment (no browser, a
  server feature we cannot prove here). Missing test data is a failure: after the
  photographs changed, six scenarios quietly reported NOT CHECKABLE for weeks' worth of
  runs and read as passes in the totals.

**Trade-offs.** The repository carries development-only code, and the release build must
keep excluding `tools/`. In return the checks have history, can be reviewed, and survive
this machine.

### D-030: Chrome definitions live apart from page blocks

**Status:** decided by the architect on the owner's delegation, 2026-09-17

The header and footer are drawn by the block machinery (D-028) but they are not blocks a
person can put on a page.

- Their definitions live in `app/Chrome/header/` and `app/Chrome/footer/`, discovered by a
  second instance of the same registry. The block contract in SPEC §5.3 is untouched: no
  new key, and nothing in `app/Blocks/` changes, so the page library cannot offer them.
- The renderer takes an optional wrapper element, defaulting to `section`, so the header
  renders inside `<header>` and the footer inside `<footer>` — one of each per page,
  semantically right, with no nested section inside them.
- The language switcher is one partial, included by the footer (never a second `<footer>`).

**Trade-offs.** A second registry instance and one more argument on the renderer. The
alternative — a `scope` key in the block definition — would have changed a frozen contract
to express something the directory already says.

### D-031: The three new design decisions, and what they are not

**Status:** decided by the architect on the owner's delegation, 2026-09-18

- **Header width**: two values, "same as the content" or "full width". Not a second
  container decision with four values: the header either lines up with the page's column or
  it spans the window, and everything else follows the container the site already has.
- **Boxed layout**: yes or no. No separate measure — a boxed page uses the container width
  that is already chosen.
- **Page background**: a shade derived from the palette (a small closed set), never a free
  colour, for the same reason §5.4 refuses a free colour per section: an open colour puts
  unreadable combinations back within reach. It applies only around a boxed page, so no
  text ever sits directly on it, and `Palette::failures()` gains no new pairs. A test
  asserts that: nothing places text on the page background.

**Trade-offs.** Less freedom than a colour picker and a fourth width. In return the palette
keeps its guarantee, the Design form gains two switches and one small choice rather than
three more decisions, and the contrast work stays the size it is.

### D-032: What the chrome lets the owner change, and what it never will

**Status:** approved 2026-09-18

There is no free-form header builder: no dragging elements, no colour picker, no per-page
chrome, no hand-written HTML. The header and footer follow the same rule as blocks — a
small set of tried choices, with colour and spacing coming from the character.

Already there: logo, menu, button, footer text and small print; header layout (left,
centred, transparent, sticky); footer layout (simple, columns); header width; everything
the design tokens give.

Added by this decision, each a closed set:
- **Surface** for the header and for the footer: plain, tinted or contrast — the same
  surfaces sections use, so contrast stays guaranteed.
- **Density**: compact, normal or roomy.
- **Rule under the header**: on or off.
- **Logo size**: small, medium or large.

Also owed, and the real gap for visitors: **a proper mobile menu** (a button that opens and
closes it, correct for keyboard and screen readers), **the current page marked in the
menu**, and a look at the transparent and sticky variants over real content, which nobody
has judged yet.

**Trade-offs.** Four more controls to design for every character. In return the owner can
make the chrome feel like theirs without any combination that can come out unreadable.

### D-033: One session, working on the development site

**Status:** decided by the owner in the handover task of 2026-09-18

There is no architect session and no separate development clone any more. One session
works directly in the checkout that serves https://boxlet.svejedobro.hr, commits and
pushes as it goes, and keeps this file current.

- **D-023 is superseded.** `~/boxlet-dev` is gone and there is no deploy by pull: what is
  saved is what the development site runs. The site is a demo with no real content and may
  be reinstalled; the rule against ad-hoc writes to `boxletcms` stands (D-002).
- **D-018's architect parts are gone**: nobody restarts a stalled session or reviews each
  commit. Where D-006 says "verified against the architect's checklist", the session now
  verifies its own work in the browser, and the owner judges anything visual from
  screenshots.
- **What stays:** the tests on both drivers and PHPStan before every commit, CI read after
  every push and fixed before new work, and the browser suite (D-029).
- **The browser suite runs against the development site.** Only the scenarios that would
  damage it — installing from nothing, locking the account, writing into the database or
  the migrations directory directly — declare `copy: true` and run against the throwaway
  copy under `~/boxlet-browser`. The development site's admin password comes from a file
  outside the repository (`~/boxlet-browser/dev-admin.json`) or the environment. This
  replaces the lesson "checks run on a copy" below for everything else.
- **Pending migrations are applied from the command line** (`php migrations/migrate.php`).
  D-019's update screen stays as the product's way, because a real site has no shell; in
  development it must never wait for the owner to press a button. The script goes through
  the same code as the button, so the lock, the SQLite backup and the gate behave the same.

**Trade-offs.** Half-written code is briefly live on the development site, and a migration
takes it dark until the script runs — seconds, because the session runs it at once. In
return the owner sees progress as it happens, with nobody relaying it.

### D-034: A link points at a page, not at a typed path

**Status:** decided 2026-09-18 on the owner's task A (links must point at pages); the
owner was told what it changes before it was built

A menu item could choose a page (D-028), but a block's link field and a link in rich text
were typed addresses: `/about` copied by hand, broken the moment the address changed.

- **One stored form for a page reference: `page:{n}`**, where `n` is the page's
  `content_group_id`. It is the value of a link field's `url` and the `href` of a rich text
  link alike, so the stored shapes stay what SPEC §5.3 already says — `{label, url}` and
  `<a href>` — and only the set of allowed values grows by one. No new attribute enters the
  rich text whitelist, and no column is added.
- **The group, not the row.** A block is copied into a translation verbatim (only its words
  are translated), so a reference by group means "this page, in whatever language the
  visitor is reading": the Croatian copy of a link points at the Croatian page without
  anyone editing it. On a one-language site the group is simply the page's id.
- **Resolved at render, never stored as an address.** Every reference on a page is looked
  up in one query before anything renders, the rule pictures already follow; a template
  receives a plain URL and knows nothing of references. Renaming a page's address moves
  every link to it at once.
- **A reference that cannot be followed draws no link**: the page was deleted, is not
  published, or has no version in the visitor's language. A link field's button is left out;
  a rich text link becomes its plain text. This is what a menu already does with an item
  whose page is gone (D-028), for the same reason: never a link to nothing. The editor marks
  such a link so the owner sees why it is missing.
- **The label may be left empty** when a page is chosen; the page's own title stands in, in
  the visitor's language.
- **Typed addresses stay** for everything that is not a page: another site, an email, a
  phone number, a fragment. Choosing a page is the first option wherever a link is entered —
  the link field, the rich text link panel, the header's button.
- Menus keep `menu_items.page_id`. A menu belongs to one locale, so the row is the right
  target there, and it already works.

What the owner will see: a page chooser in front of every address field, and a link to an
unpublished or deleted page quietly missing from the site while the editor says why.

**Trade-offs.** A deleted page silently removes the links to it rather than being refused
while something links to it, as a picture in use is. Blocking would be the safer default
but needs a "where is this page linked from" search across every block; it is noted as a
follow-up rather than built now. SPEC §5.6's `{{page:slug}}` tag, never built, is dropped
in favour of this: a reference by slug breaks exactly the way typed paths do.

### D-035: The admin's design system, second version

**Status:** proposed 2026-09-18; built on the development site for the owner to judge from
before/after screenshots of every screen

The owner's verdict on the first version: cards with no vertical rhythm, buttons stuck to
the card edge, no character anywhere. The second version keeps the rule that the admin has
its own fixed `--ui-*` tokens and never reads the site's (SPEC §5.4), and changes:

- **Character.** Warm paper neutrals instead of cool grey; one dark bar across the top with
  a mark drawn in CSS beside the site's name; a deep indigo accent for what can be pressed.
  Titles in Space Grotesk, text in Inter — both already shipped for the site's pairings,
  declared under admin-only names so a site pairing can never change the admin.
- **Rhythm.** One spacing scale. Panels, fieldsets, tables and a form's closing button never
  touch: whatever follows one sits a panel's gap below it. A heading that opens a panel is
  not spaced twice.
- **Components.** One button height (2.5rem, 2rem inside table rows), inputs of the same
  height with a focus ring, fieldset legends as panel titles, tables with row titles in ink
  rather than as underlined links, status pills with a dot, empty states with a drawn box.
- **A dashboard** that shows where the site stands (pages, pictures, character, menus) and
  the four things an owner comes to do, instead of one sentence.
- **Phones.** No screen scrolls sideways: three screens did before this, measured, and the
  screens scenario now asserts it (`21-admin-screens`).

The bar's light inks on the dark bar are measured by their own contrast test; everything
else stays inside the existing matrix.

### D-036: How D-032 is built

**Status:** decided 2026-09-18 while building the approved D-032

- **Seven closed choices**, stored as settings and chosen on the Header and footer screen:
  header arrangement (left, centred, over the first section, sticky), footer arrangement
  (one column, words beside the menu), header and footer surface (plain, tinted,
  contrast), density, a rule under the header, logo size. Each is empty until the owner
  picks one, and empty means *as the character has it*: every character now carries a
  chrome look (`ChromeLook::CHARACTER`), so changing character re-dresses the chrome the
  way it re-composes the page, and the owner's own choices stay theirs.
- **Found while building:** D-032 listed the header arrangements as already there. They
  were declared, but nothing chose them and the stylesheet had no rules for them: every
  site rendered "left". They are real now.
- **Over the first section** takes that section's colours (contrast, picture, gradient or
  tinted) rather than painting its own, which is what keeps it readable over any hero, and
  gives the first section the room it covers.
- **The current page** is marked in the menu with `aria-current` and a rule under the words;
  its parent is marked when the page is one of its children. The renderer marks the
  resolved menu, so templates compare no addresses.
- **The mobile menu is the site's first script** (`site-nav.js`, first-party, loaded only
  when the header has a menu). Without it the navigation wraps openly and submenus are
  listed under their parents; with it, one Menu button folds the navigation and the call to
  action on a phone, and each submenu opens from its own button, never from hover alone.
  Escape closes and returns focus.
- **Found while checking:** a boxed page's frame took 144 of a 390-pixel phone screen. The
  frame is now capped on narrow screens.

**Trade-offs.** One script on visitors' pages, where there were none; a page without it is
complete, so the cost is only the file. Seven more controls on one screen, each a closed set.

### D-037: Icons in the admin

**Status:** approved by the owner 2026-09-18 ("an icon library in the admin, to use icons in
many places"); the library chosen by the session

Lucide (ISC licence, a large consistent outline set), vendored under the SPEC §3 rule as one
SVG sprite holding only the icons in use, fetched at a pinned version by
`tools/icons/build.php`. Drawn with `icon('name')`, in the text colour, hidden from assistive
technology: a control showing an icon alone carries a visually-hidden label and a title. A
test fails when the sprite is not well-formed or lacks an icon the code asks for.

**Trade-offs.** One more vendored file (8 KB). Adding an icon means running the script, which
needs the network on a maintainer's machine, never on a user's server.

### D-038: The owner's review of the admin, 2026-09-18

**Status:** approved by the owner 2026-09-18 (their list of changes)

- The bar: Header and footer moves under Design, as a second entry in a Design group;
  Settings leaves the navigation and joins View site and Log out on the right, all three as
  icons. On a phone the navigation folds under a menu button.
- **Pages:** the list shows when each page was last edited, in the site's time zone.
- **Links everywhere a page can be chosen** (a block's link field, the header's button, a
  menu item; not rich text): choosing a page fills in its address, read-only, and offers its
  title as the text, which the owner may change. This replaces D-034's hidden address field.
- **Menus:** arrows instead of Move up and Move down; an item can be edited in place; the
  name and Rename on one line. Found while doing it: renaming a menu took it off the site,
  because the header and footer find their menu by name. A rename is now followed.
- **Media:** a drop zone that uploads as soon as pictures are dropped or chosen, beside the
  search. On a picture's page: the focal point is gone (its controls and its route; stored
  points keep their meaning, and the crop dialog still carries one over), Crop, Replace and
  Delete are one word and an icon each, and Replace opens its form only when pressed. The
  "Suggested — check it" badge is gone; a suggested description is still recorded as one, so
  replacing a picture never overwrites a description the owner confirmed (D-025).
- **Settings:** "Pictures" is Branding. The maintenance message sits with the maintenance
  switch, explained, and saves on its own. There was a logo here that nothing drew, and a
  second one on the header screen that the header did draw: now there is one, under
  Branding, and the header's earlier choice is carried over until Branding is saved.
- **The logo keeps its shape**, on the site and in its chooser: drawn from `full`, the one
  size that is never cropped. The cost: a logo uploaded as a large photograph is sent at up
  to 2400 pixels wide. Logos are usually small, so this was chosen over a new preset.
- **Softer field borders.** The top and sides of a text field are light; its bottom edge
  keeps the 3:1 contrast D-012 requires, so a field is still found at rest. A test holds it.
- **Every field says what it does**, in `lang/en/hints.php`; a test fails for a block field
  without a description.

### D-039: The owner's second review of the admin, 2026-09-18

**Status:** approved by the owner 2026-09-18 (their list of changes)

- **Wider lists.** Dashboard, Pages and Menus use the wide column, as Media and Design did: a
  table in the reading column scrolled sideways.
- **The status is the switch.** On the page list "Published" and "Draft" are the buttons
  that change them; Delete is a trash icon. Both are named for a screen reader and on hover.
- **Media:** the drop zone the full width, and under it a search field and button, with no
  words around them. Replace opens a drop zone under the buttons, like the library's, and
  replaces as soon as a picture is dropped or chosen.
- **Editing a menu item happens in a dialog** over the list. Without a script the Edit link
  asks the server for the page with that dialog drawn open, and Cancel and × close it.
- **Emails and phone numbers are links as typed.** `info@example.com` becomes `mailto:`,
  `+385 91 234 5678` becomes `tel:+385912345678`, in every link field, rich text, the
  header's button and menus. A visitor on a phone can tap a number to call it, and an email
  opens their mail app. A path of digits (`/2024/05/01`) is never taken for a number.
- **The page editor's toolbar:** devices and View page as icons, Save for Save page. The
  plain editor is offered only when the visual editor cannot run (without JavaScript); its
  address still works.
- Found on the way: a template error in the menu screen reached the development site for a
  few minutes, because no test drew that screen. A test now draws every admin screen.

### D-040: The page editor gives the block the room

**Status:** approved by the owner 2026-09-18

- **Page settings & SEO** is one folded line at the top of the panel, opened when wanted,
  as a block's section style is. It opens by itself when one of its fields was refused, so
  an error is never folded away.
- **A selected block's controls are on the block**: move up, move down, duplicate and
  remove, as icons on its top right corner in the canvas, drawn in the overlay like the
  insertion control and in its two tones. They send the builder the same actions the
  panel's buttons did; the panel's row of four buttons is gone. At the ends of the page the
  move that would do nothing is shown as unavailable.

**Trade-offs.** The controls now live inside the canvas frame, so they are reached by
clicking the block first — which is also how a block is chosen. The plain editor keeps its
own buttons.

### D-041: The Columns block

**Status:** built 2026-09-18, the owner's go-ahead to take step 6d before the answer on 6c

D-008 built: one block, `columns`, whose layout is how many columns share a row — two,
three (the default) or four. More items than that start a new row on the same grid, so a
team of eight is two rows of four. At most twelve items, and at least one on save.

- **A column** holds a picture, a heading, rich text and a link, all optional. The block
  has its own optional heading and introduction, and one **picture shape** for every
  column — landscape, square, or round — so a row lines up whatever was uploaded. Round is
  a portrait at 60% of the column, cut by `clip-path`, not a radius token.
- **A column without a picture is a column of words**, not a grey box; a picture chosen and
  since deleted keeps its place as the placeholder the other blocks draw.
- **A column's link is a text link**, not a button: three or four buttons in a row shout.
- **Every item is drawn, an empty one too**, so the canvas and the page show the same grid.
  The editor outlines an empty column (`is-empty`, canvas.css) and gives it height.
- **A new block starts with three empty columns** (`Blocks::fresh()`, used wherever a
  block is added): a repeater with no items drew an empty band. The library preview samples
  three items the same way.
- **Four in a row folds to two rows of two below 64rem**, and every layout to one column
  on a phone. The editor's canvas is narrower than the screen, so on a 1400-wide window it
  shows four in a row as two rows — correctly, as a tablet would.
- The demo gains three: What we do (three, words only), The people (two, round) and How a
  project runs (four, square); its seed now turns `demo:` links inside items into page
  references too.

Found and fixed on the way: `site.css` had passed the 500-line limit (my D-036 work); the
header and footer are now `chrome.css` and the blocks `blocks.css`, both linked after
`site.css` in the order the rules had. A hero heading with a word wider than a phone (Bold,
"Northwind") pushed the page sideways; it now breaks.

Open: the repeater's own labels say "Item 1" and "Add item", not "Column 1". A per-block
word needs a label key that cannot collide with an item field's; not decided yet. And `app/Core/Blocks.php` is at 325 lines, past the 300 guideline (301 before `fresh()`);
its seam is content shaping — `normalize`, `value`, `emptyItem`, `fresh` — to split out when
it is next touched.

### D-042: A scenario that asks for the copy goes to the copy

**Status:** decided 2026-09-18, after a mistake of mine

Scenario 25 declared `copy: true` but imported the development site's address, and
`applyCharacter()` read that address from config regardless — so its first run applied all
five characters to the development site. 03-design and 14-front applied characters there
without declaring the copy at all, and 03 also resets section styles.

- `applyCharacter(page, base, …)` takes the site as an argument.
- 03, 14, 22 and 25 import `COPY_BASE as BASE` and declare `copy: true`.
- `run.mjs` refuses, before a browser opens, a copy scenario that does not import
  `COPY_BASE as BASE`, and any scenario that applies a character without being a copy one.

What it changed on the development site: the design (its four hand-set decisions were
overwritten; recovered exactly by matching the old stylesheet's hash, tokens.366b260d4cbb:
spacing normal, container normal, boxed yes, page background surface, on Brutalist's
colours and type), and About and Services were re-saved unchanged through the plain editor.
Section styles, pictures and menus were untouched. Restored 2026-09-19 with the owner's
permission, through the Design screen: the site links tokens.366b260d4cbb.css again, the
exact file it had.

The copy's server needs `PHP_CLI_SERVER_WORKERS=4`: the installer checks URL rewriting
by requesting the server from inside a request, which a single-process `php -S` cannot
answer.

### D-043: Languages, the owner's answers before Slice 6

**Status:** approved by the owner 2026-09-19 (resolves O-12)

- **AI translation ships switched off.** It works once the site owner enters their own
  key; Boxlet never carries someone else's bill. Which provider comes first is decided when
  the owner has a key to try; the provider interface has one implementation until then, and
  the AI part is built last in the slice.
- **A page with no translation yet is not shown in that language**: it is left out of
  that language's navigation, and its address there answers "not found" rather than the
  source language's page. A site can change this in its settings.
- **What a translation is missing falls back to the site's primary language** — alt text
  first among it — rather than being drawn empty.

Built in steps, each visible: (1) the Languages panel on the Settings screen, 2026-09-19 —
add from the installer's ISO list (a blank first choice, so nothing is added by accident),
switch on and off, order, and remove only while no page or menu is written in it; the
main language is fixed, first and always on; (2) translating a page; (3) a stale mark on
the block whose source changed; (4) the switcher leading to the same page, hreflang, and
the D-043 rules on the front end; (5) AI translation, once there is a key.

Step 2, built 2026-09-19: the builder's language menu (a `<details>`, no script needed)
lists every language with this page's version there — "Open" where it exists,
"Translate" where it does not. Translating makes a **draft copy** in the same content group,
from the group's source whichever version it was asked from, each block tied to its source
block by block_group_id and carrying `source_hash`: a hash of the source block's
**translatable fields only** (a picture swapped in the source is not something a translation
falls behind on). The address is the source's if free in that language, else one from the
title; the parent is the parent's translation where there is one. Links to pages that have
no version in the new language draw as no link there until those pages are translated
(D-034, as designed). Scenario 27-translate leaves the site as found.

Step 3, built 2026-09-19: a translation's block is **stale** when its source block's
translatable words today hash differently from what it was translated from — computed when
asked (`TranslationStatus`), never stored, so no save has to remember to set a flag. Shown
at rest as an amber edge on the canvas (kept through a redraw), in the inspector as a notice
with what the original says now and "Mark as up to date", as a count at the top of the
panel, and as a badge on the pages list. Blocks added to the source since are counted as
missing. SPEC §8 Slice 6's acceptance ("edit one Croatian block, only that block shows as
stale in both translations") is a test on both drivers and scenario 28-stale in the
browser, which leaves the site as found. Marking current reloads the editor, so unsaved
work there is guarded by the leave warning rather than kept.

Step 4, built 2026-09-19: the switcher leads to **this page in each language** where a
published translation exists, else to that language's home page, else leaves the language
out (`Alternates`); a translated page names every published version with hreflang, the main
language's as x-default; and a picture with **no alt row** in the page's language takes the
fallback language's (the main one), while an alt left empty on purpose stays empty. Two
tests encoded the old rules and were changed deliberately (switcher to every home; no alt
fallback). Scenario 29-switcher leaves the site as found.

Not built, and said so to the owner: the per-site setting D-043 mentions ("a site can change
this"). Menus are per language and the switcher never links an untranslated page in another
language, so there is nothing left for the setting to switch except serving the main
language's page at another language's address — left until a site asks for it.

### D-044: The site's own words to visitors

**Status:** decided 2026-09-19 (resolves O-19)

The few words Boxlet itself puts on a visitor's page — "page not found" and its sentence,
the names of the menu and of the language switcher — live in `lang/{code}/site.php`, in
each language's own folder beside the admin's (the owner's suggestion: one folder per
language, so a Croatian admin later is `lang/hr/` too), read by `site_t($key, $locale)`: the page's language, else the site's
main language, else English. Shipped complete for en, hr, de, fr, it, es and sl; a test
fails if any file lacks a word English has. Not `t()`, which is the admin's language.

Not covered, deliberately: the maintenance and update pages. They are drawn without the
database (D-019, D-021), so they cannot know the site's languages; the maintenance page
shows the owner's own message where there is one. Overriding these words from the admin is
not built; the owner writes everything else a visitor reads.

### D-045: Mail through Symfony Mailer, Resend through its API

**Status:** approved by the owner 2026-09-19 (resolves O-6)

symfony/mailer, already on the closed list, sends all mail. Transports: SMTP, for any mail
server; Resend through its **HTTP API**, not its SMTP endpoint, because shared hosts often
block outbound SMTP ports while HTTPS always gets out, and the API's errors say what went
wrong; and the server's own sendmail as the last resort. The Resend transport is Boxlet's
own — a small class on Symfony Mailer's transport interface calling the API with PHP's curl
— rather than symfony/resend-mailer, which would add it and symfony/http-client to the
closed list (the owner chose this over adding them).

Built 2026-09-19 as step 7a of Slice 7: the Email panel on the Settings screen — way of
sending, sender, where notifications go (else the admin's login address), the chosen way's
fields only (a script hides the others; without it all are shown), and a test message.
Passwords and the Resend key are sealed with APP_KEY (libsodium secretbox, `Secret`) and
never drawn back; an empty field keeps them. SMTP offers STARTTLS or SSL only: Symfony
Mailer 6.4 cannot switch off its automatic STARTTLS, so a "none" choice would have done
nothing. Tests on both drivers (with a capturing transport); scenario 30-mail saves nothing.

### D-046: Forms

**Status:** decided 2026-09-19 (Slice 7)

- **A form is placed on a page with a Form block**, not the `{{form:slug}}` tag SPEC §5.6
  listed: "anything structural is a block", and as a block it takes the section styles
  every other block has. The tag is not built, as `{{page:slug}}` was not (D-034). Forms
  have no slug.
- **A form belongs to one language**, as a menu does (`forms.locale`, a schema addition
  before v0.1): its labels, button and replies are words. A new one starts as a contact
  form — name, email, message — labelled in its own language.
- **Six field kinds**: short text, email, phone, message, list of choices, tick box. No
  uploads: a public upload is a door. At most 20 fields.
- A field's **key** is fixed when it is made and never follows its label. A message keeps
  each answer **with the label it was asked under**, so editing a form later leaves old
  messages readable as sent.
- The edit screen needs no script: every button (add, move, remove, save) posts the whole
  form and saves it. A new form notifies its owner and sends visitors no reply until the
  owner writes one.

7b built 2026-09-19: migration 0016 (forms, form_submissions), the Forms screen in the admin
bar, the edit screen. The browser check found the default labels stored as undefined admin
keys ("forms.default.email"); they are now visitor words in the form's language, and a test
reads them. A disabled ghost button now looks disabled everywhere, not only on the pages
list, where the rule used to live.

7c–7e built 2026-09-19:
- **The Form block** chooses a form through a new field type, `form` (a reference, like
  `media`; the frozen field list changed deliberately before v0.1, SPEC §5.3). It draws only
  on a page of the form's own language, so a translated page shows nothing until the owner
  picks a form in its language. Layouts: heading above, or beside the form.
- **Sending** posts to `/form/{id}`, the one route exempt from the session CSRF check
  (`Router::visitorPost`, guarded by a test): visitors have no session. It stands guard
  itself — a signed time token (three seconds at least, no upper limit, because the Slice 8
  page cache will serve old tokens), a honeypot, and five messages per sender in ten
  minutes. Honeypot and timing failures are thanked and dropped. A refused send draws the
  page again with the answers kept and each problem beside its field; a good one is stored
  and redirected (303) to the page with the thank-you. The words a visitor reads are
  site_t() in seven languages.
- **Mail after a send**: the owner is notified (reply-to set to the sender), and the sender
  gets the owner's own reply if one is written. A mail failure never costs the message: it
  is recorded and shown on the messages screen until a test message succeeds.
- **Messages** in the admin: the list (who, when, new ones marked), one message (marked read
  on opening, a Reply by email button), delete, and a CSV of all of them — a column per
  field now or once asked, a cell starting like a formula kept as text.

SPEC §8 Slice 7's acceptance: the form is built, embedded, sent and seen in the admin in the
browser (scenario 32-contact, leaving the site as found); both emails are asserted through a
capturing transport. **Receiving them for real waits for the owner's mail account** — the
development site has no way of sending chosen, and choosing one would send real mail.
Scenario 33-form-look photographs the form under all five characters on the copy; "beside"
came out a narrow strip in its column and now fills it. The copy's sync now carries
migrations/, which it never did: the demo seed needed table 0016 and the copy's install
failed without it.

### D-047: A replaced picture is a new address, and replacing asks first

**Status:** decided 2026-09-19, from the owner's report

Replacing a picture kept its id and name, so every variant came back at the address the old
one had, and browsers kept showing the old picture from their cache — measured in the
library, still red after a reload when the new one was blue. Every variant address now
carries `?v=` and the first eight characters of the original's hash, which a replacement
changes (`MediaPresets::version`); the file on disk and its serving without PHP are
untouched, as with site.css. The picture's address in SPEC §5.1 is unchanged; only a query
string is added.

Dropping or choosing a file in Replace now asks first, naming the file and how many pages
show the picture, because it changes all of them and the old picture cannot be brought
back. Declining leaves everything as it was. Scenario 34-replace checks both, reading the
colours off the thumbnails themselves.

### D-048: Making every picture's sizes again

**Status:** built 2026-09-19 (resolves O-13)

A panel under the Media library, "Make every size again": Start owes every finished picture
a remake, and each Continue does what fits in one request; media-remake.js presses Continue
by itself, so with a script the pass runs to the end while the page is open, and leaving
simply pauses it. Migration 0017 adds `media.remake` (the variants already remade in this
pass, NULL when nothing is owed) and `media.revision`.

Safe on a live site: nothing is deleted first; every variant is written beside its file and
moved over it whole, so a visitor gets the old file or the new one, never half. When all of
a picture's variants are new its revision goes up, and `?v=` carries it with the hash
(D-047), so browsers fetch the new files. A picture whose original is missing is dropped
from the pass rather than holding up the rest. Encoding and the AVIF size rule are the
upload's own (MediaWriter, MediaVariants::smallerAvif) — the thin wrapper O-13 asked for.

The first use: the pictures on the development site were made before D-047's AVIF fix and
are heavier than they need be; pressing the button makes them again.

Found on the way, the D-042 trap once more: media-helpers.mjs went to the development site's
address whatever site a scenario ran on, so a copy scenario's uploads reached the development
site's login screen and uploaded nothing. The helpers now act on the site the page is on, and
harness.mjs no longer imports the development site's address at all.

### D-049: The sitemap is a file the site writes

**Status:** built 2026-09-19

`public/sitemap.xml` lists every published page in every language that is switched on, with
its translations as hreflang alternates and its last change. It is a real file because
managed nginx answers any address ending in .xml or .txt from disk and never asks PHP (the
same reason no route may end in an extension). It is written again after anything that
changes what it lists — a page saved, published, unpublished or deleted, a language
switched on, off or removed — and on a dashboard visit when it is missing, which gives a
site installed before this existed its file. Where public/ cannot be written, nothing
fails, and the same document is at `/sitemap` for the owner to hand a search engine.

`robots.txt` is written beside it, pointing at it and keeping crawlers out of /admin — but
only one the site wrote itself, marked on its first line; the owner's own is never touched.
Both are ignored by git. Tests write into their own public directory (a new `PUBLIC_PATH`
setting), never the development site's.

### D-050: Two-step login

**Status:** built 2026-09-19 (resolves O-4)

Optional, never forced, from a panel on the Settings screen. Setting up shows a QR code drawn
on this server (bacon-qr-code, SVG) and the key as text; it is switched on only by a code
the app then makes, so nobody is left with it on and no app holding the secret. Ten
recovery codes are shown once, on the page that answers that step, and stored only as
password hashes; each works once. The secret is sealed with APP_KEY like the mail
passwords. New codes need a code from the app; switching off needs the password.

At login the password is checked as before; with two-step login on, the session holds a
pending login for ten minutes and `/admin/login/code` takes the app's code or a recovery
code in the same box. Wrong codes count against the same LoginThrottle as wrong passwords.
The FTP way back in: an empty `storage/disable-2fa` switches it off for every admin at the
next login attempt and is removed, and the login screen says so (and warns if the file
could not be removed). Codes are accepted one 30-second period either side of now.

The secret is 160 bits, as RFC 4226 recommends: otphp's default is 64 bytes, which the
setup screen showed as a 103-letter key nobody could type. Scenario 37-two-step runs on the
copy, never the development site, whose one account is the owner's.

### D-051: Statistics on the site's own server

**Status:** approved by the owner 2026-09-19, from his specification ("Statistika posjeta —
bazična+") and two sketches; built in rounds, SPEC §5.7

Visits counted by the site itself: no external service, no script on the page, no cookie,
nothing stored that identifies a visitor, so no consent banner. The owner can switch it off
in Settings for a site that uses another tool; it is **on by default**.

**Fitted to Boxlet**, where the specification was written without the code in view:
- **No new dependency.** A MaxMind DB reader of Boxlet's own instead of MaxMind's library;
  charts drawn on the server as SVG instead of Chart.js; countries shown by their code, as in
  the owner's sketch, instead of flag icons.
- **No cron** (a non-goal): the daily salt and the retention clean-up run on the first counted
  request of a day; the country database is refreshed by a button.
- **The page cache** (Slice 8) is read inside `public/index.php`, so cached pages are still
  counted without any script. The optional server rule that skips PHP will say that it skips
  counting too.

**Round 1 — the reduced version:** counting (views and daily unique visitors by page, source,
country, device, browser, system); the Statistics screen (today, 7 days, 30 days, 12 months;
four figures against the previous period; a trend chart; six tables with share bars and
"show all"); a dashboard card; a Settings panel (on or off, DNT/GPC, retention, the country
database, delete everything, a suggested privacy-policy text in English and Croatian).

**Round 2**, open as O-20: the world map (one SVG map, Natural Earth, public domain — the
owner approved adding it when it comes), filtering by clicking with the state in the address
and a chosen date range, export as CSV, counting 404s,
trusted proxies, grouping small numbers, an optional attribution in the footer.

**Counting, as built (2026-09-19):**
- `App\Modules\Stats\Tracker`, `Bots` and `Agent`, called from `public/index.php` after
  `send()`.
  - `Tracker::wanted()` needs no database: method, status, content type, path, prefetch,
    the admin's cookie and the bot list. The connection is released and the database
    touched only after it says yes.
  - Measured on the development site: 10 counted and 10 uncounted requests each averaged
    52–60 ms, with no difference between them.
- **Three migrations, 0018–0020, not the one the plan named.** SPEC §5.0 keeps one table per
  migration file.
- **An empty string means "none" or "unknown"** in every dimension: a direct visit, an
  unknown country, an unrecognised browser. The admin supplies the words, so SPEC §5.7's
  `--` and "Direct" were changed to match.
- **An Android app's Referer** (`android-app://com.slack/`) is kept as the package name. It
  tells where the visitor came from as well as a domain would.
- **Link previews are programs; in-app browsers are people.** WhatsApp, Telegram and the
  other messengers' link previews are listed as bots. A page opened inside Instagram,
  Facebook or Line is a person and is counted.
- **Known gap:** while maintenance mode is on, `UpdateGate::isAdmin()` starts a session for
  every visitor to see whether they are the admin. A visitor who came during maintenance
  keeps that cookie until the browser closes, and is not counted until then. The fix would
  read the cookie before starting the session, but the maintenance tests log in without a
  cookie, so they would have to change with it. This is left for when maintenance is next
  worked on.

**The Statistics screen and dashboard card, as built (2026-09-19):**
- **The screen**, `/admin/statistics?period=today|7d|30d|12m`:
  - The period and a full table (`&all=`) are in the address. Nothing on it needs a script.
  - `12m` is the last 365 days, charted by week (Monday first). Every other period is
    charted by day.
- **Today's chart is the week that today ends.** There is no hour in the counts, so one day
  would be one point, not a line. Adding an hour would multiply every day's rows by up to
  24, which the chart does not justify.
- **Visitors over a period are the sum of each day's visitors.** A key lives one day, so
  someone who comes on two days counts as two. The screen says so under the tables.
- **The four figures:** visitors, views and views per visitor are each shown against the
  period before. The share of visitors on a phone shows the period before's share instead
  of a change: a percentage of a percentage reads badly.
- **The chart**, drawn by `Chart` as SVG:
  - The lines stretch to the box. The axis numbers are HTML, so they keep their size on a
    phone.
  - Exact figures are a `<title>` tooltip per point, plus a "show as a table" under the
    chart.
- **Share bars** are an SVG `<rect>` sized by attribute, since the admin CSP refuses a
  `style` attribute.
- **The bar item and the dashboard card** appear only while statistics are on.
  `AdminView` reads that setting on every admin page, which costs one small query.
- **Dark mode**, which the owner's specification asks for, is a question for the whole
  admin, which has none. It is not settled for this screen alone.


**Countries, as built (2026-09-19):**
- **`Mmdb`**, Boxlet's own MaxMind DB reader. It reads the file where it lies, a few bytes
  at a time.
  - Checked against DB-IP's real September 2026 file (8.3 MB): 8.8.8.8 → US, two Croatian
    addresses → HR, an IPv6 address → DE, private addresses → unknown.
  - About 0.4 ms per lookup, 1 ms to open, 0.8 MB peak memory.
  - The tests write their own small files with record sizes 24, 28 and 32. DB-IP uses 24.
- **`Geo`** keeps the file in `storage/geo/`. Before a file replaces the one in use:
  - it must open and know a country for 8.8.8.8;
  - it replaces the old one in one rename;
  - a download takes this month's file, or last month's when this month's is not out.
  - It downloads with curl over HTTPS where PHP has curl, and with PHP's streams otherwise.
  - The real download through the Settings button worked on the development site.
- **Credit:** "IP geolocation by DB-IP" appears on the Statistics screen only while its
  database is in use (CC BY 4.0).
- **Bug found and fixed outside statistics:** five controllers set a refusal's flash as
  `'error'`, and `AdminView` drew anything but `'warning'` as success. A refused Mail save,
  two-step code, translation or language was shown in green. `AdminView` now passes
  `'error'` through to `notice-error`.


**The privacy text (2026-09-19):**
- Settings offers a suggested privacy-policy paragraph in English and Croatian, written for
  the site's own settings:
  - the retention period it has, with Croatian declension (24 mjeseca, 6 mjeseci);
  - the country paragraph only while the database is in use;
  - the Do Not Track paragraph only while that setting is on.
- It lives in `lang/{code}/stats-privacy.php`, so another language is one new file. It is
  never shown to a visitor.
- It is labelled "a starting point, not legal advice".
- **Round 1 is complete.** Round 2 stays O-20.

**One control row (2026-09-20, the owner, on seeing it).** The period, a range of your own
and the days that adds up to took three lines above the figures. They are one wrapping row
now (`.stats-controls`): side by side wherever there is room, in rows of their own where
there is not, with no breakpoint of its own to keep in step with the rail's. Their own
bottom margins had to go with them — a margin inside a flex line adds to that line's height,
which is why the first attempt still measured two lines on a wide screen. Scenario 38-stats
measures the row against its tallest part, because the parts are centred on the line and
three heights give three different tops.


**The whole browser suite at the end of statistics round 1 (2026-09-19).** Every failure
was traced before anything was believed. None came from statistics.
- **Four stale checks, fixed in the suite:**
  - `17-settings` looked for Settings in `.admin-nav`. Settings has been an icon at the
    bar's end since D-038.
  - `15-crop` judged what it left behind by a name pattern, which caught a picture the owner
    had cropped on 17 September. It now checks the ids it made.
  - `20-page-links` required no `/services` link before its own. The owner's text on the
    development site already has one, so it now counts one more.
  - `35-remake` allowed 25 s for the first batch, which a full run exceeded. It now allows
    180 s.
- **Missing test data, fixed:** the demo seed ships no menu, so a freshly installed copy
  draws no header. `03-design` measured a header of width 0, and `22-chrome-look` found
  nothing to fold. `ensureHeaderMenu()` in the harness now puts a menu in the copy's header
  through the admin, only where none is named.
- **Flaky under a full run, passed alone:** `12-picker`.
- **The copy's sync now includes `public/index.php`** (see sync-copy.sh).
- **Stopping the copy's server:** `pgrep -f "127.0.0.1:8100" | xargs kill`, chained with
  other steps, ended the whole call with exit 144 and nothing after it ran. The cause is
  not known: a later check found that the pattern does not match the calling shell. The
  kill now runs in a call of its own.


### D-052: The admin redesign, "Workbench" (v2)

**Status:** approved by the owner 2026-09-19, from the handoff in
`docs/design_handoff_admin_v2/`. That folder holds the prototype, Nocturne's tokens and
`IMPLEMENTATION-boxlet.md`, the designer's notes after reading this code.

This is D-007's "the admin must look good and be original", brought forward. The owner
chose three things, and everything else follows the handoff.

**The owner's three choices:**
- **Dark, as drawn.** This reverses admin.css's "warm paper with one dark bar" position.
  - Every `--ui-*` token is retuned, starting from the values in the implementation notes
    §3.
  - The contrast matrix in `tests/contrast_test.php` is re-measured, not loosened.
  - The site's own tokens are untouched: the admin stays its own fixed system (SPEC §5.4).
- **Buttons stay filled.** Nocturne's outlined primary gives way to CLAUDE.md's "no control
  is ever invisible at rest". This is the one point where the design gives way.
- **Everything, the activity log included.** The log is a new table written to from the
  controllers.

**Where the handoff meets the rules:**
- **No style attributes** (the admin CSP). Every value goes in a stylesheet. Dynamic sizes
  become bucketed classes or server-drawn SVG, as the statistics bars already are.
- **No CDN and no new dependency.** Icons come from the existing Lucide sprite, extended
  through `tools/icons/build.php`, not Phosphor from unpkg. Fonts stay self-hosted: Inter is
  already "Boxlet UI".
- **Every string goes through `t()`.**
- **Controls stay controls.** Page status stays a switch with a visible edge (D-039).
  Destructive actions go behind a scriptless `<details>` overflow menu, not a script-only
  one.
- **Things Boxlet does not have are left out:**
  - the site switcher: one site per install;
  - "Users": one admin, no accounts (CLAUDE.md);
  - the actor column in the activity log: there is only ever one actor.
  - The top strip shows the site's own host, without a switcher.
- **Works without a script.** ⌘K is a script on top of a real `/admin/search` page that
  works without one.


**The rail folds (owner's request, 2026-09-19):**
- **The page editor opens with the rail folded to its icons.** The server marks the frame,
  so this needs no script, and the canvas gains 164px.
- **A round toggle on the rail's edge folds it or opens it again, anywhere.**
  - The choice is kept in a `boxlet_rail` cookie on `/admin`, set by admin-nav.js and read
    by AdminView. The server draws the next page folded or open before any script runs,
    and scenario 39-rail measures that at DOMContentLoaded.
  - A change made in the editor is not remembered.
  - Under 1000px the rail starts folded and can be opened. On a phone the toggle gives way
    to the drawer.
- The shell's stylesheet was split at the same time: `admin-shell.css` (the rail),
  `admin-strip.css` (the strip) and `admin-rail-compact.css` (the folded rail, its rules
  written twice because a media query cannot sit in a selector list).


**The redesign as built (2026-09-19), beyond the steps in §2:**
- **Ctrl+K belongs to the editor inside rich text.** The palette took it everywhere at
  first, and with it the editor's "make a link" (D-034). Caught by scenario 20-page-links.
- **The page list's address is words, not a link:** opening a page on the site moved into
  the row's menu. Scenario 32-contact read the address from that link and then visited
  `https://…hrnull`; its cleanup also clicked a delete button inside a closed `<details>`
  and left a page behind. Both fixed in the scenario, and the two pages it left were
  deleted through the admin by their ids.
- **A guessed description counts as described.** The first Media table brought back the
  "check it" mark the owner removed in D-038; scenario 10-media caught it.
- **install.php deletes itself when it is opened on an installed site**, which on this
  development checkout removes a file git tracks. It happened at 20:44 UTC on 19 September
  from the owner's browser, and `git checkout -- public/install.php` put it back. The
  behaviour is right (SPEC §6); only the development site feels it.
- **Sizes:** admin-ui.css was split (admin-parts.css), admin-shell.css split three ways,
  and admin-media-table.css, admin-activity.css, admin-settings.css and admin-palette.css
  are new. Every admin stylesheet is under the 300-line rule.


### D-053: The page cache

**Status:** approved 2026-09-20, building on D-020 and SPEC §5.7

A visitor's page is written to disk as it is sent, and the next visitor gets the file.

- **Checked in the first lines of `public/index.php`**, before the autoloader, the container
  or the database: `require`d directly, since there is no autoloader yet. It therefore works
  on every host with no server configuration, which a rewrite rule does not (D-020).
- **Where:** `public/cache/pages/`, one file per address, named by a hash of the host and the
  path. The directory is where index.php can find it without configuration.
- **What is cached:** a GET with no query string, answered 200 with HTML, outside `/admin`,
  `/form` and the installer, from someone with no `boxlet_session` cookie. Never while
  maintenance is on or a migration is pending: the gate answers those before this runs.
- **What a hit costs:** one `is_file()` and one `readfile()`. The visitor is then released
  (`fastcgi_finish_request`) and the application boots only to count the view — SPEC §5.7
  promises a cached page is still counted, and that is where the promise is kept.
- **Emptied by any change.** `Activity::record()` already runs on every change the admin
  makes (D-052), so the cache is emptied there, in one place, rather than from thirty
  controllers. A "Clear now" button in Settings does it by hand.
- **On by default**, with a switch in Settings. A site whose pages are cached and never
  cleared is worse than no cache; the emptying rule above is what makes the default safe.
- **A form on a cached page still works.** Its token carries the time the page was drawn and
  has no upper limit (FormToken), so the three-second delay counts from when the page was
  cached; the honeypot and the rate limit stand beside it, as that file already says.


### D-054: The light theme, and a switch for it

**Status:** asked for by the owner 2026-09-20, from the designer's second handoff
(`docs/design_handoff_admin_v2/IMPLEMENTATION-theme-switch.md`, read after
`IMPLEMENTATION-boxlet.md` and the README, in that order at the owner's instruction).

The admin can be read in two palettes. **Dark stays the default** — D-052 is the owner's
choice and what every install without a preference gets.

- **Three choices, not two:** Light, Dark, and Match the system. A two-state toggle cannot
  say "follow the machine", and its label would mean the state one moment and the action the
  next, which is the control CLAUDE.md calls invisible at rest.
- **A form, not a script.** `POST /admin/appearance` stores the choice and returns to the
  screen it was pressed on, already painted. Server-rendered into `data-ui-theme` on the
  `.admin` element, so there is no flash of the other palette and none of the blocking
  inline script the admin's CSP would refuse anyway.
- **Kept in a cookie** (`boxlet_theme`, path `/admin`, a year), like the rail's folded state
  and for the same reason: it is how the admin looks on this screen, not something about the
  site. No migration; a laptop in the dark and a desktop by a window may disagree.
- **`public/assets/admin-tokens.css` is new** and holds every colour the admin has, in three
  blocks: the dark default, the light set, and the light set again inside
  `@media (prefers-color-scheme: light)` for "match the system". Plain CSS cannot give one
  block two selectors across a media query and Boxlet has no build step, so the duplication
  is honest and a test fails if the two copies ever differ by a character. admin.css keeps
  the measurements and the type and writes no colour at all; the palette test now skips
  admin-tokens.css, because that file *is* the palette.
- **The light palette is D-007's warm paper**, which had years of measurement behind it, put
  into the Workbench's roles: a panel a step up from the page, the rail a step down — which
  on paper means the rail is the darker one.
- **`tests/contrast_test.php` runs its whole matrix twice**, once per palette, and both pass
  unloosened. It also now measures what the rail actually paints — the faint ink its counts
  are set in, and the accent tint behind its current entry — and the brand mark against the
  rail, a panel and the page.

**Where you are, more quietly** (the owner, on seeing the light admin): the current entry in
the rail sat on the accent tint, which on paper is a pale lavender *lighter* than the rail —
a highlight rather than a place. It has its own token now, `--ui-current`, deeper than its
surroundings in both palettes, with the accent spent on the bar down its left edge. The
Settings sub-navigation, drawn as the same control, follows it. The dark value is the one
that was already there, so the dark admin is unchanged. The count in the current rail row
moves to the muted ink: the faint one measured 4.09:1 on that deeper ground.

**Two things the handoff was wrong about, and both were measured:**
- **The orange mark does not survive the move.** `#ff6b3d` on warm paper's rail is 2.27:1,
  under the 3:1 a shape needs. The light palette carries `#d94f1e`: the same hue, a value
  that reads on paper. "The mark stays" is true of its hue, not of its number.
- **A token named for a role lies in one of the two palettes.** The checkerboard behind a
  transparent logo was drawn in `--ui-ink` and `--ui-ink-muted`, which are light on the dark
  ground; on paper it became a black-and-grey block. It has its own `--ui-checker` pair now,
  light in both palettes, because a logo is drawn for a light site.

**Also fixed while looking at it:** the Overview said "unchanged for 1 days" for as long as
that metric has existed. One day has its own wording now, with a test.

**A deprecation the local suite could not see (2026-09-20).** `StatsFilter::where(string
$fromDay = null)` is implicitly nullable, which PHP 8.4 deprecates. The suite fails a test on
any notice, warning or deprecation — but only where one is raised while that test runs, and
this one is raised as the class is first LOADED, where a `catch (Throwable)` somewhere up the
stack swallowed it. Three commits went out with CI red on PHP 8.4 and green on 8.1–8.3 while
every local run passed.

Fixed with `?string`, and `tests/deprecations_test.php` now loads every class file in app/ in
a process of its own, with nothing catching anything, and fails on whatever PHP says. It was
run against the old signature first, where it fails, and against the new one, where it is
silent. `php -l` does not see this class of problem at all.


### D-055: Where visitors are, past the country

**Status:** asked for by the owner 2026-09-20, from the updated specification in
`docs/Statistika posjeta za PHP CMS – specifikacija (bazična+) (1).md` §"Lokacija: regija i
grad". His reason, and it is the right one: **a site that serves one country learns nothing
from a map that says everybody is in that country.** An admin in the United States seeing
"United States: 100%" has been told nothing.

**Measured before deciding anything** (the specification's own figures were out by a factor
of three, so these are this machine's):

| | Country Lite | City Lite |
| --- | --- | --- |
| Download | 3.9 MB | **57.5 MB** (the spec said 19) |
| On disk | 9.6 MB | **121.4 MB** |
| One lookup | — | **0.47 ms** |
| Unpacking it | — | **0.7 s** for the whole file |

What the City Lite record actually carries: the country's ISO code, `subdivisions[0].names.en`
(there is **no** `iso_code` in the Lite file, whatever the specification says), `city.names.en`
and the city's latitude and longitude. Boxlet's own reader already decodes every type in it.

**The decisions:**

- **Three levels, one setting** (`stats_location`): **Country** stays the default, then
  Region, then City. The level is a choice, not a consequence of which file is installed:
  a site may hold the city database and still count only countries.
- **The city database is the owner's to fetch**, from the same Settings panel, and the
  country one stays supported. 121 MB is not a thing to put on a shared host without asking.
- **Downloaded in pieces and resumable.** 57 MB will not come down inside one request on a
  shared host's execution limit. Range requests, a few megabytes a step, the ETag pinned so
  a file that changes underneath is caught, and the part file's own size is the resume point
  — no state to keep in step. It is the media remake's Start/Continue pattern (D-048),
  which already works without a script, and that script is generalised rather than copied.
  Unpacking stays one step: 0.7 s, measured.
- **A table of its own, `stats_places`** — day, country, region, city, latitude, longitude,
  views, visitors. **The city is never combined with the path**, in any view or any export.
  That is the specification's privacy rule and it is also what keeps the rows bounded.
- **Region names are normalised.** DB-IP's own data puts two Zagreb addresses in "City of
  Zagreb" and in "Zagreb" — measured, on 161.53.1.1 and 31.147.200.1. The leading
  administrative words are stripped, so one place is one row. It merges a city-region with
  its surrounding county where a country has both; that is the cost, and the table reads
  better for it.
- **A threshold, on by default at the city level:** a city under five visitors in the period
  is shown with the others as "Other". Round 2's grouping (O-20) already does this work.
- **Drill-down, not a filter.** Clicking a region or a city narrows the location panel —
  world, then country, then region — while the rest of the screen keeps narrowing by country
  alone. The path and the city must never meet, so a city cannot be a filter for the whole
  screen.
- **The map gets bubbles** at the cities' own coordinates. Boxlet's map is equirectangular
  and drawn by Boxlet, so a coordinate is two multiplications; and when one country holds
  more than 70% of the visitors, the map opens on that country, from a bounding box the map
  build computes per country.
- **The detail is kept for less time.** City rows collapse into their region after a few
  months (a setting), and the region rows live out the ordinary retention.

**In three rounds, each ending with something on a screen:**
1. The database: the level setting, the city file, the resumable download, and the place a
   lookup returns. **Done 2026-09-20.** The download works: eight pieces, 2.3 s on this
   server, and the file installed itself.
2. The counting and the tables: the migration, the Tracker, Regions and Cities on the
   Statistics screen, the threshold, the privacy sentence, and export. **Done
   2026-09-20**, and this is the round that answers the owner's complaint. Seen live on the
   development site: one real visit became "Hesse, 1 visitor" in the regions and
   "Other (fewer than 5)" in the cities — the floor doing exactly what it is for.
3. The map: bubbles at the cities' coordinates, the country view when one country holds the
   traffic, and the collapse of old city detail into its region. **Done 2026-09-20.**
   - The map build writes each country's own box into the file (`data-box`) and the
     projection onto the `<svg>`, so the two figures that decide where a dot goes live in
     one place rather than in two that have to agree.
   - The map opens cut to one country when the screen is narrowed to it, or when one
     country holds more than 70% of the visitors — the owner's case exactly. The address
     can always ask for the world back, and the legend carries the way there.
   - A dot's size follows the visitors, and shrinks with the zoom, so a dot on a country is
     the same size on the screen as a dot on the world.
   - **Three things were measured and then changed**, each of them after looking at the
     picture: the map goes no closer in than eight times, because cut to Croatia alone the
     1:110m shapes Boxlet ships are a six-line cartoon; every line is a non-scaling hairline,
     because a border 0.4 wide became a twenty-pixel black band at that magnification; and a
     dot is drawn in the ink rather than the accent, because the busiest country is drawn in
     the accent and an accent dot on it is a dot you cannot see.
   - **The dots are only in the country view.** On the whole world, five Croatian cities are
     five circles inside each other — a spill of milk over the Balkans. Each view answers the
     question it can: the shading says which countries, the cut view says where inside one.
   - The city is forgotten before the rest of a place: after three months by default (a
     setting, and "as long as the counts" is one of its answers) a row keeps its country and
     region and loses its city, in one transaction so the counts cannot be lost or doubled.
   - Tracker passed 300 lines, so what happens once a day — the salt, the pruning, this
     collapse — moved into NewDay, which is a seam the module already had.

**Two things the tables do not do, and why.** A region and a city cannot narrow the screen:
they are counted in a table the path never enters, so there is nothing for the rest of the
screen to be narrowed by. And while the screen IS narrowed to a page, source, device,
browser or system, the two tables are left off with a line saying why, rather than answering
a question nobody asked.


### D-056: The release package

**Status:** built 2026-09-20, at the owner's request, after he tried to install Boxlet from
a GitHub download on a second account and got one line of plain text.

The line was Boxlet's own: *"Dependencies are missing. Upload the release ZIP, which
includes vendor/."* The product told him the right thing; the thing it named did not exist.

**The requirement was never in danger, but it was never honoured either.** SPEC's server row
says *no shell, no Composer*, and that is about the person installing. Composer belongs to
the person PACKAGING, once per release. What GitHub hands out is the source, and the source
has no `vendor/` on purpose: committing 53 MB of libraries makes a version bump a diff
nobody can read. Between the two there has to be a step, and until now there was none — so
nobody without SSH could install Boxlet at all.

`tools/release/build.php` is that step, beside the icon sprite and the map:

- It packs the **last commit** (`git archive HEAD`), not the working tree, and says so
  loudly when the tree is dirty. A release has to be a thing that can be pointed at.
- `composer install --no-dev` runs in the staging directory, never in the checkout, so the
  maintainer's own PHPStan is not swept away by making a release.
- Out: tests, tools, docs, .github, phpstan.neon, CLAUDE.md, PLAN.md — and
  **composer.json and composer.lock**, because with them present a `composer install` on a
  live site would pull the development tools back onto it.
- It **checks the file it wrote** rather than trusting itself: opens the ZIP again, insists
  on the installer, the front controller, both .htaccess files, the autoloader, the icons
  and the map, and refuses anything that looks like tests, tools or PHPStan.

**Measured, not estimated:** 1061 files, **2.1 MB**, against SPEC's 8 MB limit. Then
unpacked into a directory of its own and installed through a browser, with scenario
01-install pointed at it: eighteen requirement checks, SQLite, the demo site rendering at
`/`, `install.php` deleting itself, login and logout — eleven verdicts, all green. That is
the first time anything has proved the claim on SPEC's own acceptance line for Slice 9.

**Left for the real release** (Slice 9, and Slice 8's update by ZIP): a version. The package
is named by date and commit, which is enough to tell two test builds apart and not enough to
be a release. Boxlet has no version number yet, and the ZIP update will need one.


### D-057: One source for the site layout's variables, and a preview that draws the chrome

**Status:** round 1 of the Appearance rebuild, built 2026-09-20.

**The debt it pays.** There have always been two renderers of `Pages/views/layout.php`: a
visitor's page, and the admin's design preview. The second answered *"what does that layout
need?"* from a literal array somebody had to remember to update, and it was caught out three
times — by `description`, then by `icon`, then by the chrome of slice 5c. Each time the
comment above it got louder, and the comment itself said the real fix was one source for
those variables. The suite could not catch any of the three: it fails on any notice, but only
along the branch some test happens to render, and `$icon` sits behind an `if`.

`app/Modules/Pages/PageLayoutData.php` is that source. `forPage()` is what
`PageController::render()` used to assemble, chrome included; `forPreview()` is the same for
the admin. **Two locks, not a louder warning:**

- the declared `@phpstan-type LayoutData` return of both factories: a factory that DROPS a
  key fails `phpstan analyse` before any test runs;
- `tests/page_layout_test.php` reads the template with `token_get_all`, works out which
  variables it actually READS — a foreach's own value and anything it assigns are its own
  business — and compares that with `PageLayoutData::KEYS`. A variable the template GROWS
  fails there, by name. Measured: a `$bodyClass` added to the template and nowhere else fails
  the test and is named in the failure.

`locales` went the other way and is no longer handed to the layout at all: no front-end
template reads it, because the language switcher gets it as an argument to `Blocks::render`.

**The preview now draws the header and the footer.** This is a DELIBERATE REVERSAL, not a
forgotten case. The old comment gave a reason: the header and footer are decisions about the
whole site, while that preview existed to judge the tokens and section styles of one
character, so chrome around the specimen was furniture competing with what was being looked
at. On the screen this is being rebuilt into, the header and footer are among the things
being judged, so the reason goes with the old screen.

**Three things the implementation had to get right, each of which would have been a quiet
wrong answer:**

1. **Precedence is three levels: request → saved → character.** `ChromeLook::fromRequest()`
   returns only the choices the request actually named (`array_key_exists`). Returning all
   seven as `''` would make the Design preview — which sends no `look_*` at all — stop
   honouring what the owner saved, and draw chrome the site does not have.
2. **The preview was hard-coded to `'en'`.** The chrome's words and its menu are per locale,
   so a site whose main language is Croatian would have judged its design under an empty
   English footer. It takes the home page's language.
3. **The chrome follows the character being PREVIEWED**, not the active one, or Bold's
   sections would stand under Minimal's header.

`fromRequest()` reads a request and writes nothing; saving stays `ChromeLook::save()`, behind
a POST with a CSRF token. A value outside its closed set is not drawn.

**The demo site now has a menu.** A fresh install had none, and an empty header is
deliberately not drawn (D-032) — so a new owner's site had no navigation at all, and the
preview would have had no header to show them. `DemoSite::seed()` builds "Main" over the demo
pages and points the chrome at it. The browser suite already knew about this and worked
around it in `harness.mjs`.

**Left for round 3:** `fromRequest()` has one caller today; the second arrives with the
merged screen. `DesignController` is at its size limit and splits when the routes are renamed.


### D-058: The loop closes — a gauge, numbers, and Save beside the picture

**Status:** round 2 of the Appearance rebuild, built 2026-09-20.

Four changes, none of them a new decision about the design model. They are about the
distance between making a choice and seeing what it did.

**A contrast gauge, because a refusal is not a measurement.** The screen could say a pair
FAILED and nothing else, so a palette passing at 4.51:1 looked exactly like one passing at
17:1, and a person cannot aim at a number they are never shown. `Palette::pairs()` now
returns all eleven pairs with their ratios and their verdicts, and `failures()` is a filter
over that one list — two lists would drift. `/admin/design/check` carries `pairs` beside
`errors` and `colors`. **Nothing was removed from the server:** Save refuses exactly what it
refused before.

Six pairs stand open and the rest fold away, **except a pair that fails, which is never
folded** — hiding the one thing the owner has to act on would be the whole feature
backwards. Measured, not assumed: a grey seed fails the tenth pair of eleven, which is
inside the fold, and the test asserts that row is lifted out of it.

**Numbers a person reads, instead of CSS.** The screen printed
`clamp(2.038rem, 1.508rem + 1.759vw, 2.827rem)` and a row of rem values. That is the
compiler's language on the owner's screen. `Tokens::readable()` gives the same decisions as
pixels: body text, the heading sizes, what the largest becomes on a phone, one step of
space, the distance between sections, the corner radius, the content width. There is no live
SPECIMEN in the admin and there cannot be one — the admin's own type is fixed by the `--ui-*`
set and may never follow the site's (SPEC §5.4). The specimen is the preview beside it.

**Save moved to the preview's bar.** It used to sit at the foot of the left column, which is
3604px tall: change a colour, watch the preview, then scroll back past every control to keep
it. **In the BAR rather than under the frame, and that was measured rather than preferred:**
the column is sticky, so anything below a frame 78vh tall sits past the bottom of the window
and stays there however far the page is scrolled. The browser suite caught the second button
as "not clickable" — which is the same failure an owner would have met as "I cannot press
it". The bar is the one part of a sticky column always in view. The buttons submit the form
through `form="design-form"`, so they work while standing outside it, and a test asserts both
halves: that they are not inside the form element, and that saving still works.

**Two speeds, because one was wrong for both.** A select or a checkbox refreshes the preview
AT ONCE: the person has already chosen, and a quarter-second of nothing reads as a screen
that did not hear them. A colour input, which fires continuously while dragged, still waits
250ms. And the "Update preview" button is REMOVED by the script that makes it pointless — a
control that repeats what already happened makes the person doubt whether it did. Without
JavaScript the button is there and everything still works.

**What this cost elsewhere:** three browser selectors said `form.design-form button[…]` and
now say `button[form="design-form"][…]`. That is the price of the move, paid once.


### D-059: Design and Header & footer become one screen

**Status:** round 3 of the Appearance rebuild, built 2026-09-20.

**Two screens were one screen cut in half.** Header & footer had seven look choices and **no
picture at all**; Design had a picture that deliberately drew no header or footer. Each was
incomplete in exactly the way the other would have fixed, and the owner had to hold the
result in their head while walking between them. They are now `/admin/appearance`: the
character strip, five tabs of controls, and one picture of the whole site.

**The five tabs are the questions a person actually asks** — Colour, Type, Shape, The page,
Header and footer — instead of one column 3604px tall. Without JavaScript they are a row of
LINKS and every panel is on the page, exactly as before; `appearance-tabs.js` upgrades that
into a real tablist with arrow keys. **The tab holding a refusal opens by itself**, because a
message inside a closed panel is a message nobody reads.

**The words went in too, and the old route dies with the screen.** The plan had kept the
per-language words — the button, the footer line, the small print — on `/admin/chrome`,
renamed. The owner asked what that route would then be FOR, and the honest answer was
"nothing but those five fields per language", which is the same arrangement this rebuild
exists to end. So they are the fifth tab's second half, and `/admin/design` and
`/admin/chrome` are redirects and nothing more. The rail lost two items and gained one.

**A name collision had to be settled first:** `/admin/appearance` already belonged to the
admin's own light/dark switch (D-054). The site's look is what an owner goes looking for
under that word, so the switch moved to `/admin/theme` and its controller is now
`ThemeController` — which is what it always was.

**What the merge cost, and what it caught.** One screen means one form, so a post that
carries half of it is refused rather than half-applied; the tests now send the whole screen
through `appearanceFields()`, as a browser does. And the character cards were each their own
little form posting nothing but a character's name — harmless while the screen held only the
design, **destructive the moment it also held the header**: loading a character posted a
screen whose every chrome field was empty, and the menu and the words were read back as
cleared. The browser suite caught it on the copy, where the site lost its header. A card is
no longer a form; its button names the one form, so loading a character carries the whole
screen.

**The preview draws the words as they are typed**, for the language it renders (the site's
main one). `PageLayoutData::forPreview()` now takes one `$trying` array — character, look,
menu, words — rather than a parameter per thing, because they all mean the same: draw the
site as it WOULD be.

**New seams, so nothing grew past the size where a file stops being readable:**
`AppearanceForm` (what the screen sends and the preview reads back, agreed in one place),
`AppearancePreview` (the three endpoints, which answer a machine), `ChromeWords` (the words,
which are free text that needs checking, beside `ChromeLook`, which is closed sets).

### D-060: The picture gets a toolbar

**Status:** the rest of round 3, built 2026-09-20.

Three widths (1280 · 834 · 390), a zoom (Fit · 100 · 75 · 50), **Compare**, and one word
saying what the screen is.

**The zoom scales the STAGE, never the frame's width.** A page judged at 1280 has to lay
itself out at 1280; a frame simply made narrower hands the page a smaller window and it
answers with its phone layout, which is a different question. So the frame is laid out at the
chosen width, its height divided by the factor, and the whole thing scaled — at half zoom you
see twice as much page, which is what zooming out means. Measured on the copy: `1280px` wide
in the layout, `scale(0.573)` on screen, filling a stage 734 across.

**Nothing goes below half.** Under that the text stops being text and the picture stops
answering anything.

**Compare is HELD, not toggled.** A comparison you can walk away from is one you can mistake
for the site. Holding the button points the frame at the preview address with no query at
all — which draws exactly what is stored — and releasing it gives back the unsaved work. The
keyboard holds it too, on Space or Enter.

**The screen says what it is:** *Published*, *Not published yet*, or *Fix the contrast to
publish* — the third when the check endpoint reports what Publish would refuse, so a palette
that cannot be published is never dressed as work merely unsaved. Discard changes appears
beside it, and is an ordinary link back to the screen.

**Two things this had to be measured against rather than assumed:**

- **The admin's Content Security Policy.** `setAttribute('style', …)` is REFUSED — measured
  in this very screen, with the console complaint to prove it — while setting one property at
  a time through the CSSOM is allowed. The toolbar sets width, height and transform that way
  and invents no rule the stylesheet does not own.
- **The bar's own layout.** Eight controls across a column this narrow wrapped into a
  different arrangement at every width, with the label stranded below the widths. The rows
  are now declared rather than left to wrap: what this is and what to do with it, then the
  tools for looking at it.

**The tools are hidden until the script runs.** Without JavaScript there is no way to scale a
frame, and a row of controls that did nothing would be worse than none: the frame is then the
column's width at full size, exactly as before.


### D-061: The designs the owner keeps

**Status:** round 4 of the Appearance rebuild, built 2026-09-21. Migration
`0024_design_library.sql`.

**This is the thing the design layer was missing.** Five characters could be LOADED and
nothing could be SAVED. An afternoon spent on colour and type had nowhere to go, and loading
any character threw it away. That is the real source of "too few options" — not the number of
controls, but that nothing done with them could be kept.

**One row is one whole look:** the ten decisions and the seven header-and-footer choices, as
they were on the screen. **Not** the menu and **not** the owner's words — those are the
site's content, and a design carrying them would put an English footer on a Croatian site the
moment it was used there.

**Decisions, never derived values** (SPEC §5.4), in JSON in one column rather than seventeen
columns: the shape of layer 1 belongs to SPEC, not to this table, so a new decision there
never needs a migration here. A row is validated on the way OUT as well as in, so one written
by an older version — or edited by hand — is filled in with the default preset's values
instead of reaching the screen as something the design layer does not accept. Tested by
damaging a row on purpose.

**Three actions and no more: keep, write over, delete.** The name is unique, and saving under
a name that exists WRITES OVER IT rather than making a second — that is what "save" means
everywhere else, and two designs called "Autumn" would make the library ask a question it
should never ask. The screen says which of the two happened.

**None of the three touches the site, and each answers with the SCREEN rather than a
redirect.** A redirect would hand back the published design, so the owner would press "keep
this design" and watch the work they had just kept vanish from the screen. Bringing one back
fills the form with it; Publish is still the confirmation, exactly as it is for a character.

**What the browser check proves, rather than the unit tests:** keep a design, load a
character over it, bring the kept one back, and the seed that comes back is the one that went
in — then delete it and find the library empty again.

**One thing the strip got wrong first:** `auto-fit` stretched two kept designs across the
whole screen. The character strip above happens to hold exactly five, so the difference never
showed there; the library uses `auto-fill`, and a kept design is a card the width of a
character.


### D-062: Two controls that were coarser than the question

**Status:** round 5 of the Appearance rebuild, built 2026-09-21. **This changes the layer-1
decision list, which SPEC §5.4 freezes at v0.1 and not before** — so it is done deliberately,
SPEC is updated, and it is written here.

**Content width is a NUMBER of rem, 36 to 88, in steps of two.** It was four names, and four
names cannot answer "a little narrower than this". The measure — how many characters fit on a
line — is the single decision that most changes whether a page is comfortable to read, and it
was the one the owner could only move in jumps of fourteen rem. The control is a slider with
the number beside it, and the line under the controls gives it in pixels too.

**The four names are still read.** `narrow`, `normal`, `wide` and `full` load as 42, 56, 68
and 80, so no stored design needed migrating or re-choosing, and the five characters kept
exactly the widths they had. Outside the bounds is REFUSED and named rather than quietly
clamped: the owner asked for something the design layer does not do, and saying so is what a
refusal is for.

**Text size is its own decision** — small, normal, large, larger — because size and scale are
different questions that were one control. The scale is how much bigger each heading is than
the one under it; this is how big the text is. A site for people who are not twenty-five
needed the second, and the only way to get it was to pick a scale that made the headings
wrong. It moves the TYPE AND NOTHING ELSE — not the spacing, not the corners, not the
measure — because a person asking for larger text is asking for larger text. Editorial ships
at `large`: a control no character demonstrates is a control nobody finds (the same reasoning
that made Soft the boxed one).

**Three things this turned up, each worth more than the feature:**

- **A loop's order became the stored order.** `container` left the closed-set loop and
  arrived at the end of the decisions array, which nothing intended. `validate()` now puts
  the decisions in one declared order, with `+ $decisions` so a decision the list forgets is
  mis-ordered rather than lost.
- **A test bound to the wrong source.** `install_test` counted "the closed choices plus the
  two colours", which stopped being the whole set the moment a decision was neither. It now
  counts a CHARACTER, which is the complete set by definition.
- **The browser check measured the border box.** It read 784px for a 704px measure, because a
  container carries side padding. It now reads the content box AND the compiled
  `--container-width` inside the frame: what the control says, what the page is built with,
  and what it actually measures, all three.

**`Tokens` was 355 lines and is now two files.** `Tokens` answers "is this a decision Boxlet
accepts, and what does it mean to a person"; `Derived` answers "what CSS does it come to".
They were never one concern, and nothing there validates — everything it is handed has been
through `validate()` already.


### D-063: Colours by hand, and the check that makes them safe

**Status:** round 6 — the last of the Appearance rebuild — built 2026-09-21. Approved by the
owner at the start, and deliberately left until last: it needed the gauge from D-058 and the
tests from the handoff's §7.

**Six roles may be set by hand:** the page background, the tinted surface, borders, text,
muted text and links. `''` means the palette works it out, which is the default and what
every character ships — this adds a way to DISAGREE with the palette, not a new thing to
fill in.

**Nine are not on offer, and that is the whole safety story.** The accent and the contrast
surface are the two seeds already. `on-accent`, `on-contrast`, `muted-on-contrast`,
`contrast-raised` and `on-gradient` are the palette choosing which of two inks can be READ on
a colour; handing those over would hand over the one decision that keeps text legible,
dressed as a choice.

**The guarantee moves from derivation to checking** (SPEC §5.4, updated). Until now the
palette could not produce an unreadable pair. Now it can, so the contrast check is no longer
a formality about the seed — it is the only thing between the owner and a site nobody can
read. It refuses exactly as before, and **a hand-set colour owns its own failure**: "text on
the background is 2.1:1" now names `color_text`, not surface contrast, which is a control
that cannot fix it.

**The bug this round was built around, killed by construction:** the owner's colours go into
the palette BEFORE anything is derived from them. Applied afterwards, `on-accent`,
`on-contrast` and `on-gradient` would still be answers about the colour that had gone —
legible against a background nobody has any more. A test sets a near-black page and asserts
all three moved.

**Two things the work turned up that were wrong before it:**

- **The neutrals assumed a light page.** They were fixed lightnesses — 0.99 for the
  background, 0.2 for text — which was true of every palette a seed could produce and false
  the moment a background could be set. Measured: a dark page gave muted text at 2.6:1. They
  now follow the page's own lightness, so ONE hand-set colour gives a coherent dark palette
  rather than a list of refusals. Every character leaves the background alone, so all five
  are untouched.
- **A proxy standing in for a measurement.** `$lightText` asked whether the chosen ink WAS
  the background colour and took that to mean "light text" — true while every background was
  near-white. On a dark page the background is the dark ink, and the muted text beside it was
  pushed the wrong way, to 1.40:1. It measures the ink's lightness now.

**And one on the screen itself:** the five colours left to the palette kept showing the
values they were rendered with while the preview beside them went dark. The same
stale-dependent bug, in the browser; they follow the palette now, and a role the owner has
taken over is never touched.

**The browser check is waited for, not slept through.** A fixed delay passed once and failed
once on the very check whose subject is whether the dependents were worked out again. It now
waits for the server's answer about the second change and for the frame to have loaded it.


### D-064: The Appearance screen becomes a workshop

**Status:** built 2026-09-21, after the owner compared what was built with the handoff's own
screenshots and found the screen a different shape. He is right, and §2 of the handoff
describes the shape it should be; this is that.

**Three columns under one bar, filling the window.** The owner's designs on the left, the
site in the middle, the controls on the right. Each column scrolls on its own and the bar
never scrolls at all. The admin's rail folds to its icons beside it, exactly as it does in
the page editor — the owner asked for that in the same breath, and `bare` already did it.

**Why it matters, rather than being a rearrangement:** read as a document, this screen put
the picture BELOW the controls and the library above both, so the owner scrolled between the
thing they were changing and the thing they were judging. A workshop puts them side by side.

**The character and the saved designs become a rail of cards**, each with three swatches, a
name, an "in use" badge where it applies, and a line **built from its own decisions** —
`modern · 56rem · normal · full bleed`. Not a sentence somebody wrote about it: a sentence
that cannot go out of date. A saved design's card carries its own two tools, so **writing
over it needs no name typed** and cannot be mistyped into a second design nobody meant.

**One form around all three columns.** The cards, the library and every control post the
same screen — that is what stopped a character load from clearing the header (D-059) — so
the form IS the layout.

**Two things measured rather than assumed:**

- **A 13px scrollbar on a screen that fits the window.** The no-script preview form is
  `.visually-hidden`, which is an absolutely positioned 1em box; sitting after a full-height
  screen it lengthened the page by exactly its own height. It is `display: none` now — a form
  with no content of its own, which exists to be submitted, and submits fine unrendered.
- **The contrast rule caught the hover I wrote.** The card's cover button faded a sheet over
  it at `opacity: 0.35`, and `tests/contrast_test.php` refused it within a minute: opacity is
  the one thing that must not be what makes a control quiet (D-012). The card's border
  lights up instead.

**Still not what the handoff draws, and named here so it is not mistaken for done:** the
controls are selects, where §2.4 asks for segmented buttons with a monospace readout; the
"follow the character" state is the first option of a select rather than a visible
`following` badge; the Type tab has no specimen; and twelve of the fifteen decisions in the
handoff's §3 table are not built. Those are the next rounds.


### D-065: A closed set is a row of buttons

**Status:** built 2026-09-21, the second half of matching the handoff's §2.4.

**Every option visible at rest.** A select hides four of five answers behind the one already
given, which on a screen whose whole point is "change it and look" is the wrong shape. The
ten closed decisions and the seven chrome choices are segmented rows now, with the number the
choice comes to on the label's own line — `20px`, `4px`, `42rem · 672px` — rather than in a
paragraph underneath.

**Radio inputs, not buttons with a hidden field:** they submit with no script, the browser
gives arrow-key movement inside the group for free, and a screen reader already knows what a
radio group is. The input itself is clipped, not faded — opacity is the one property the
admin's contrast rule forbids for making a control quiet (D-012), and it would be a strange
thing to write in a rule that means "this is the mechanism, not the control".

**"Following the character" is a state you can see** (handoff §3.5): the group's readout says
`following` in the accent colour while nothing is chosen, and the first segment — named for
what the character actually gives, "Follow: Name left" — is how it goes back. Same data as
the old first option of a select; no hidden default.

**The typefaces are cards set in the faces themselves.** "Modern" means nothing on its own;
"Modern · Inter", in Inter, is a choice a person can make. This is the ONE place the admin
loads the site's fonts, and it is not the site's design leaking into the tool: those six
faces are the thing being chosen. They are served by `/admin/appearance/typefaces`, built
from `Typography` rather than a hand-written stylesheet, so a card can never show a face the
site would not use.

**The type specimen shows the sizes**, labelled in the pixels the site will use, set in the
admin's own face. I had said the admin could not show a specimen at all; that was too strong,
and wrong in a way worth recording: it cannot take the site's TYPEFACE for its own text, but
the sizes are exactly what the two controls above decide.

**A defect the browser check found, which was the product's and not the test's:** the six
hand-set colour inputs show what the palette works out until the owner takes a role over, and
the script rewrites the untaken ones whenever the palette moves. That put the two in a race
the owner loses — choose a colour, and a refresh landing before the switch is flipped writes
the choice back over. **Choosing a colour now takes the role over**, and the switch remains
the one press that gives it back, which is the direction that deserves a deliberate gesture.


### D-066: Type in detail — five of the twelve remaining decisions

**Status:** built 2026-09-21. SPEC §5.4 updated: this adds six keys to layer 1 and changes
`scale` from a closed set to a number.

- **The step between sizes is a number**, 1.1 to 1.6. Six named ratios were six answers to a
  question with a continuum behind it, and the gap between two of them was a decision nobody
  could make. A number is CLAMPED, never rounded to the control's step, so the five
  characters keep the exact ratios they were written with — 1.333 is still 1.333.
- **Three nudges**, in pixels, each bounded and asymmetric: the largest heading −30…+40, the
  subhead −12…+20, small print −3…+5. A ratio cannot say "that headline, two pixels smaller",
  and the nudge lands AFTER the scale so the scale stays the relationship it is and the nudge
  stays the exception it is.
- **Heading weight, letter spacing and capitals** may be taken over from the pairing, with
  `''` meaning "as the typeface has it" — the same convention the chrome's choices use. A
  weight chosen by hand survives changing the typeface; one never chosen follows it.

**Two duplications this turned up, both of which had already caused a wrong answer:**

- **A size was worked out in two places.** The compiler needed it as CSS and the screen as a
  number a person reads, so each did the arithmetic — and the screen's copy did not know
  about the nudges, which made every readout in the Type tab wrong the moment one was used.
  Caught by a test within a minute of the nudges existing. `Derived::sizeOf()` is the one
  formula now, and `Tokens::readable()` asks it.
- **Readouts were rendered once and went stale.** A number beside a control that stopped
  being true when the control moved is worse than no number, because it is the thing being
  read. `AppearanceForm::readouts()` builds them all in one place, the screen renders them,
  and `/check` returns them so they follow a control that is being dragged.

**And a type PHPStan was right about:** the heading weights are their own labels, and PHP
turns the key `'600'` into `600`. The helper's parameter is `array<array-key, string>`
because that is what it really takes; narrowing it to `string` would have been a type that
reads better and is false.


### D-067: The sheet, its edges, and what breaks out of it

**Status:** built 2026-09-21. Seven more of the twelve; SPEC §5.4 updated again.

**The frame wraps the SHEET, not the page.** It used to wrap everything, so the header and
footer were inset with the content and could not reach the edge of the window. The page is
now three boxes — `.page` (what surrounds), `.page-frame` (the room between), `.page-sheet`
(the page itself) — and `header_bleed` / `footer_bleed` say which side of the frame the
chrome renders on. **They are read by the LAYOUT, not compiled into a token**, which is what
keeps every rule in the stylesheet free of "is this boxed"; that property is what the frame's
own docblock has always been proud of.

**Four more decisions about the sheet:** how much room is around it (thin/narrow/normal/wide,
1/2/3/5 spacing units — it was hard-coded at three), its corners, whether it lifts off the
page, and the two bleeds. Every one is ZERO when the page is not boxed, for the same reason
the frame is: a rounded corner on something with no visible edge is a rule that does nothing
and still has to be read by everyone who comes after.

**Cards and panels are a colour of their own** — the seventh role the owner may set. They
used to take the tinted surface's colour on a plain section, so a card and a tinted band were
the same tone and a card INSIDE a tinted section had nothing to be distinct from. It sits
half a step from the page, between the two, and its pair — text on a card — is measured like
every other surface text can land on. All five characters still pass at twelve pairs.

**The footer menu's columns**, which is the one place more columns actually help. The handoff
asks for the footer's own grid to take the number; measured against what a footer holds — the
owner's words, the menu, the switcher, the small print — three and four columns would leave
two of them empty. A long menu is the thing that needs the room, and the control says so.

**An interaction worth naming rather than hiding:** a header laid over the first section can
only do that INSIDE the sheet. Outside it there is nothing to overlay, so it draws as an
ordinary header. The CSS says this by matching `.page-sheet`, and needs no code to enforce it.

**Two tests bound to a literal, both caught in the same run:** a count of seven chrome choices
(there are eight now) and a label convention I had sidestepped by sharing one key between two
decisions. Both now read the source they are about.

**Left of the twelve: the hero arrangement**, and it is not an oversight — it was refused,
see D-069.


### D-068: Publish asks once, instead of three buttons in the bar

**Status:** built 2026-09-21, from the handoff's §2.1 — and from looking at the screen, where
a loaded character turned the bar into two rows of long buttons.

Applying a character to a site that already has blocks can rewrite every section's style and
layout, so it has always needed two explicit answers (D-018). They sat in the bar
PERMANENTLY, which is the wrong place for a choice that matters on one publish in twenty and
is destructive on that one.

The bar is **one Publish** now. Press it with a character loaded on a site that has blocks
and the screen comes back asking which of the two it is, with what each answer does written
beside it. Nothing is written while it asks — a test measures the stored design across the
question rather than against zero, because the publish before it had written one.

It works without JavaScript, because it is a form post answered by a form post. The browser
suite's `applyCharacter()` presses Publish and then answers, which is what a person does.


### D-069: The hero arrangement stays the block's — the handoff's last row is refused

**Status:** decided by the owner 2026-09-21, and it closes what was O-23.

The handoff's §3 table asks for `hero_layout` — left / centred / split — as a layer-1
decision, set "directly" rather than through the character's composition. **It will not be
built.** The owner's question is the whole argument: *why would this be global at all when it
is already a setting on the block?*

**It already is where it belongs.** Every hero carries its own layout in
`page_blocks.layout`, chosen in the editor for that page, and SPEC freezes that: what a block
looks like belongs to the block. A design decision that re-arranged every hero on the site
would overrule choices the owner made page by page — and the one tool that legitimately does
that, "save design and reset section styles", asks first and says what it will do.

**Where the idea came from, so nobody re-invents it:** a character already decides how a hero
stands — Editorial centres it, Bold splits it, Brutalist pushes it left — and today the only
way to change that is to load a whole different character, which takes the colour, the type
and the spacing with it. The real want behind the handoff's row is narrow: *"I like this
design, but new heroes should be split."*

**If that want ever becomes real**, the answer is not a design decision that looks live and
is not. It is a control that says what it does — **"new sections start like this"** — beside
the characters, where composition already lives, and it would have no reason to stop at the
hero: a character sets width, rhythm, alignment and edges for every block type the same way.

Until somebody actually asks for it, this is a feature refused rather than deferred, which is
the rule about not building an abstraction before a second real caller — applied to a
feature.


### D-070: The picture opens on a width the column can carry

**Status:** built 2026-09-21 — the two rules of the handoff's §2.3 I had left out, and said
so when the owner asked what was left.

- **A new width comes with Fit.** Zoom belongs to the width it was chosen for: 100% of a
  desktop page in this column is a corner of it, and carrying that over to the phone shows a
  390px page at twice its size. Fit is the only answer that means the same thing at every
  width.
- **The screen opens on the widest viewport this column can actually carry** above the 0.5
  floor — and only until the owner picks one, after which their choice is theirs. Desktop
  needs 640px of stage; below that it opens on Tablet, then Phone.

**Measured when the size is KNOWN, not once at load.** A stage still being laid out reports
zero, and a screen that picked its width from that would open on the phone every time. A
`ResizeObserver` on the stage fires when there is something to measure and again whenever the
column changes — the admin's rail folding, a window resized. The handoff warned about exactly
this: its prototype went through two rounds of a silently clipped preview because it trusted
a hardcoded width instead of measuring.

Measured on the copy: a 1600px window opens on Desktop at 0.81, an 1100px window — 540px of
stage — opens on Tablet at 0.65, and switching to Phone at 50% zoom comes back at Fit.


### D-071: The screen really fills the window — three faults found in one screenshot

**Status:** 2026-09-21. The owner put our screen beside the handoff's and said the layout
behaves differently, that a dead band had appeared under it, and that the prototype's
controls look denser. All three were right, and the first two were the same class of fault:
**something escaping the box that was supposed to hold it.**

- **A three-row grid on a two-row screen.** `.appearance` named rows for the bar, a message
  and the body; the message is usually absent, so the body landed in the `auto` row and took
  the height of its own CONTENT. The page then grew past the window, the columns stopped
  scrolling on their own, and under them sat a band of the admin's background. It is a flex
  column now, which does not care how many children there are. Measured before and after at
  a 620px window: 699px of document, then 620.
- **Absolutely positioned things escaping a scrolled column.** The radio a segment is built
  on is clipped to a pixel and positioned absolutely; with no positioned ancestor it was
  measured against the PAGE, from inside a column that scrolls, and stretched the document.
  The two columns are `position: relative` now — one rule, rather than hunting each hidden
  label down.
- **Horizontal scrollbars under both columns.** A visually hidden label sixteen pixels wider
  than its button, and a text input asking for its default twenty characters. Both fixed at
  the source, and the columns are `overflow-x: clip`: their width is the layout's to give, so
  a sideways scrollbar is never the right answer, only a report that something is too wide.
- **Density.** The group's name is a label now — small, quiet, in capitals, as the handoff
  draws it — the hints are at footnote size, and the paddings are tighter. The hints stay:
  they are what teaches the screen, and hiding them is the owner's call, not mine.

**Why the browser suite had not caught it:** it ran at one window height, and both faults
only show when the content is taller than the window. It checks a short window now.


### D-072: The screen measures itself, not the window

**Status:** 2026-09-21. From `docs/ispravci.md` §A, which is a reading of this screen against
the handoff's prototype. The owner's own words were that "extra sidebars open"; §A found six
faults behind it, and five of them were one mistake made five ways: **the screen was reasoning
about a width nobody had measured.**

- **The admin rail could be opened as a fourth column.** `bare` screens ship with the rail
  folded, but `admin-nav.js` still showed the button that unfolds it — one click laid a 216px
  rail over three columns that were already full, and because nothing is remembered on a
  `bare` screen the next load undid it again. The button is not drawn on those screens now.
- **Every threshold here is a `@container`, not a `@media`.** How much room this screen has
  depends on the admin rail beside it — 3.25rem folded, 13.5rem open — which a media query
  cannot see: the same 1100px window gave the screen 1048px once and 884px the next time and
  the CSS behaved identically both times. That is also what created a band between 1000 and
  1024px with a wide rail and stacked columns: two thresholds measuring different things.
- **Three columns, then two, then one — never three to one.** 12.25 + 30 + 19.5rem is 988px
  of content, so three stopped fitting long before the old 64rem let go of them. What gives
  way first is the LIBRARY OF CHARACTERS, which is a place to start from, not the inspector,
  which is what the screen is used with: under 74rem the rail folds into a panel opened from
  the bar (`appearance-rail.js`), under 56rem everything stacks with the picture first.
  Without a script there is no panel to open, so the screen stacks at 74rem instead — one
  column earlier is a fair price for not drawing a button the browser cannot press.
- **The picture no longer changes width under the owner's hand.** `fitsTheColumn()` ran from
  every `ResizeObserver` tick, so dragging the window walked the preview from desktop to
  tablet to phone and back, and nothing said why. It chooses once, at the first real
  measurement; when the room runs out afterwards the strip SAYS so and the width stands.
- **The frame owns the height.** It was `calc(100dvh - var(--ui-bar-height))`, true only
  while the strip is exactly that tall — and on Croatian labels it wraps to two rows. The
  window is handed down instead: frame, column, main, screen. Only above the phone width,
  where the frame really is two columns; under it a clipped frame would put the whole screen
  out of reach for anyone without a script.
- **The stage scrolled 404px onto nothing.** Found while checking §A3's "exactly one vertical
  scroll": the frame is laid out at full size and scaled down, and the scrollable overflow
  the stage reports is the frame BEFORE the transform — 1280×1176 of it while 840×772 is what
  is drawn. Sideways only now, which is the one direction that ever has anything to reach.

The strip over the picture also says what size it is — `1280×950 · 81%` — into a slot that
had been in the markup since the first day with nothing ever writing to it.

**Checked by dragging, not by reasoning:** the suite now sweeps ten widths from 1600 to 820
and asserts the relations, because every one of these faults is a relation between two widths
and none of them shows at a single one.

**And the stylesheet was split, because it was twice the limit.** `admin-appearance.css` had
reached 1050 lines — past the point where a file is searched rather than read, and CLAUDE.md's
hard limit is 500. It is five now, along the seam the screen already has: the screen itself,
then one per column (`-rail`, `-picture`, `-inspector`), and `-widths.css` holding every
threshold, loaded last so its overrides win. Twelve dead rules went with it — the old wide
layout's `.library*`, `.preview-bar*` and `.slider*`, none of them in any template. **Proved
pure rather than asserted:** every computed property of all 13,965 elements on the screen, at
three widths × five tabs, before and after — zero differences.


### D-073: The preview changes its stylesheet, not its page

**Status:** 2026-09-21. From `docs/ispravci.md` §B. Every change set `preview.src`, so the
picture was a full document reload every 250ms while a slider was being dragged: a white
flash, the scroll position lost, and the fonts and pictures fetched again each time. The
prototype felt different because nothing in it was ever reloaded.

The frame is same-origin, so the preview's own stylesheet link can be swapped instead —
the old one stays in force until the new one has loaded, so there is no unstyled moment.
**Compare became the same swap**, which matters more than it sounds: it is a button meant to
be held down, and it was reloading the document on every press.

**Which decisions need the page back was MEASURED, not reasoned.** The preview was fetched
with each of the form's 52 controls moved in turn and the HTML compared. Nine change it: the
two bleeds, which menu the header draws, four of the chrome's layout choices, and the
footer's small print. `boxed` does **not** — an unboxed page is a frame of zero rather than a
different sheet, deliberately (D-067) — and that is the one this would have got wrong by
reasoning about it, because `docs/ispravci.md` lists it as needing a reload. Three of the
chrome's choices measured as token-only and are on the list anyway: they are choices about a
thing built out of markup, and a needless reload is only the old behaviour while a missed one
is a screen showing something the site will not do.

**The list is a guard, not a comment.** The browser suite reads the regex out of
`appearance.js` itself, asks the server which controls change the markup, and fails if any of
them is missing from it. A second check marks the frame's own `<html>` and watches whether
the mark survives — token-only changes keep the document, a header bleed replaces it.

**Two old verdicts were re-aimed, not relaxed.** "Compare shows the published site" and
"choosing a value refreshes the preview by itself" both watched the frame's `src`, which
stood for "the picture changed" only while every change was a reload. With the swap the
address is deliberately identical before, during and after — and the Compare check then
passed by comparing two identical strings, which is the worst way for a check to survive.
Both read the colour and the spacing the page is actually painted in now.

`appearance.js` passed 300 lines with this and split along the seam it already had:
`appearance-readouts.js` writes the server's answer onto the screen — messages, palette,
gauge, readouts — and knows nothing about the preview, the debounce or the reload list.


### D-074: The palette is one list

**Status:** 2026-09-21. From `docs/ispravci.md` §C1. The colour tab held the same palette
twice: a list of fifteen colours that could not be touched, and, folded away under it, a
panel with the seven that could. **The half that can be changed was the half that was
closed**, and the two could not be read against each other at all.

It is one list of sixteen roles now, in the order the palette works them out. A role the
owner may take over carries a colour input — **the swatch IS the picker**, because a colour
square beside a button that opens a colour picker is two things where there is one — and the
nine that are chosen against whatever they sit on say `computed` beside their hex.

- **Giving a colour back is an ACTION, not a box to untick.** Each hand-set colour still
  needs its switch in the form, because a colour input always carries some colour and "is
  this mine" cannot be read off its value (D-065) — but that is mechanism. The row shows one
  button that says what it does; the switch is clipped to a pixel beside it, never faded,
  because opacity is what D-012 forbids for making a control quiet. `Reset all` is the same
  action over every role, and the server **re-validates** after clearing: the palette's own
  colour can fail a pair the owner's passed, and a screen that stopped saying so would give
  up the guarantee D-063 moved from derivation to checking.
- **Both buttons appear and go by CSS, not by a render.** The switches flip under the
  owner's hand as colours are picked, so `:has(input:checked)` is what shows the revert and
  `Reset all` — a button the server decided about would always be one round trip behind.
- **Measured:** sixteen rows, seven with a colour input, nine saying `computed`, no old list
  anywhere on the screen, every swatch square. That last one was a real fault, not a
  formality: every field's input carries a 2.5rem floor so a text box is comfortable to hit,
  a floor beats a height, and the squares came out 24 wide by 40 tall.

**One browser verdict was re-aimed, not relaxed.** It used to click the two checkboxes
directly; they are clipped now, and a check that reaches for something nobody can see stops
being a check of the screen. It presses `Reset all`, which is what the owner presses. What it
asserts — that a colour can be given back — is unchanged.

**And one PHP test was split, not adjusted.** "the screen offers the seven colours, folded
away until one is the owner's" asserted two things at once. The first — that the seven are
offered with their switches — is untouched and keeps its own case. The second is the rule
this change deliberately drops, so what replaces it is asserted as its own: one list with
every role in it, no folded panel and no read-only list beside it, and a taken colour
offering to go back, with the site untouched until Publish.


### D-075: The specimen is the sizes, drawn

**Status:** 2026-09-21. From `docs/ispravci.md` §C2 and §C5, with §D5.

**The specimen was fixed at 1.6rem.** Dragging the scale moved the number beside each line
and the lines themselves did not move at all — which took away the only thing a specimen is
for, the RELATION between the sizes. It is drawn at the page's own sizes now, all four
shrunk by the SAME factor so the largest fits the column, and set in the pairing being
chosen: the cards above already take the site's faces, because they are what is being
chosen, and a specimen of a pairing has to show both halves of it.

- **The arithmetic is not redone in JavaScript.** The pixels in each readout are the
  server's, from the one formula that owns them (`Derived::sizeOf`, D-066); the script only
  multiplies all four by one factor. A second copy of that formula is two answers waiting to
  differ, which is the bug D-066 exists to prevent.
- **The numbers stay real.** The pixels beside each line are what the site gets, not the
  shrunken ones.
- **Two judgements, and both were corrected by looking.** The handoff suggested capping the
  largest line at 34px: at the default character that put the body and the small print at 8
  and 6, a pair of smudges. 40px keeps them legible at the sizes people actually choose. And
  a 9px floor — my own — made 17px and 13px land on the SAME size, which is the one thing
  this must never do: two steps drawn identically say the design has no step at all.
- **Without a script the four fixed sizes remain** as the fallback, so the specimen is still
  four different sizes rather than four identical lines.

**The tab strip is five equal columns**, not a wrapping row. Five tabs share about 280px,
which fits in English and does not in Croatian — "Boja · Tipografija · Oblik · Stranica ·
Zaglavlje" broke into two ragged rows, and a strip that wraps unevenly reads as two strips.
A name too long for its column gives way at the end, with the whole of it in the `title`.

**The gauge's sample is 28×18** rather than 40×24: twelve of them share a 312px column, and
the sample only has to show a pair of colours against each other.


### D-076: Or a colour of your own

**Status:** 2026-09-21. From `docs/ispravci.md` §C3, the last of the eleven corrections. No
migration: `design_tokens` is one row per decision with a JSON value, so a new decision needs
none — the same reasoning `0024_design_library` is built on.

**Three places take a shade of the palette, or a colour the owner picked**:
`page_background_colour`, `header_colour`, `footer_colour`. Empty until they set one, and a
hex when they have — the convention the seven hand-set roles already use (D-063), because a
default nobody chose is not a value.

**This is not the free colour per section that §5.4 refuses,** and it does not open it. A
section takes its surface from the palette because a page of arbitrary bands is a page with
no palette left. These are three places, decided once for the whole site, and each is
answered:

- **The frame around a boxed page carries no text at all,** so it needs nothing but the
  colour and the gauge gains no pair. That is the reasoning `Tokens` has always given for
  the decision beside it.
- **The ink on the header and the footer is DERIVED from the colour,** by the same function
  the contrast surface has always used (`Palette::inksOn`): the readable one of the palette's
  two inks, a muted tone beside it, a raised tone for what sits on top. So a colour of its
  own brings its own text rather than standing under whatever the palette happened to hold.
- **And it is still measured.** Four more pairs when a colour is set, so a surface neither
  ink can be read on is refused by name. Measured: of 52 greys, two are.

**A fixed step of 0.42 in lightness was wrong at the ends of the range.** It is a comfortable
muted tone for a surface in the middle, which every contrast surface the five characters ship
is — and against a colour the owner picks it broke at both ends: a pure black header put the
muted text at 2.48:1 and a pure white one at 4.29:1, so Boxlet refused the two colours
anybody is likeliest to choose. The derivation now walks further until the pair reads and
stops at the first step that does. A surface already passing at 0.42 is returned at 0.42, so
nothing that works today moves — measured: the five characters' compiled stylesheets are
byte-for-byte what they were.

**A transparent header is excluded,** because the two choices contradict each other: that
layout exists to paint nothing and take the colours of the section beneath it. The layout
wins, and it wins by the rule not matching.

**No class, and no rule generated per site.** `chrome.css` reads each value through
`var(--chrome-header-bg, <what the surface class gave>)`, and the tokens exist only when a
colour was set — so with none set the whole block computes to exactly what `sections.css` put
there. The `--section-*` properties are redefined on the CHILD, never on the element: a custom
property whose own value appears in its fallback is a cycle, and a cycle in CSS is not a
warning but the property silently becoming invalid.

**Two files were split on the way, both forced and both along a seam they already had.**
`admin-appearance-inspector.css` passed the 500-line hard limit, so how a COLOUR is shown and
taken over left for `admin-appearance-colour.css`; what stays is how a DECISION is made.
(The contrast gauge joined it from `admin-design.css` when that file went — D-077.)

### D-077: The old Design screen is gone

**Status:** 2026-09-22. Closes O-22 and O-24, which the owner asked for together. They are
two halves of one thing: the screen that merged into Appearance (D-059) still had an address
and still had a stylesheet.

**The two addresses are removed, not redirected for ever.** `/admin/design` and
`/admin/chrome` were redirects for one release, which is the grace a bookmark gets; a
redirect kept for ever is a second address for one screen, the arrangement the merge exists
to end. `AppearanceController::moved()` goes with them. **The ⌘K entry never named them** —
it names Appearance and carries `design character colours fonts type header footer chrome`
among its words, so searching for either still finds the screen. Measured before touching
anything, not assumed.

**`admin-design.css` is gone, and what was alive in it moved to where it belongs:** the
contrast gauge to `admin-appearance-colour.css`, because the gauge is the palette measured;
`.design-form` and `.derived` to `admin-appearance-inspector.css`, because the form is this
screen's form and the derived line is one of its readouts; `.preview-actions` to
`admin-appearance-picture.css`. Everything else in the file — the character cards, the
workspace, the preview frame and its notes — named classes no screen has.

**Which four were alive was measured, not read off the old note.** A substring grep says
`derived` is used and it is wrong: the match is `role-derived`. Counting whole class tokens
in the markup and in the scripts is what gives the real answer.

**The move is proved, not asserted.** Every computed property of every element on the screen,
at three widths and on all five tabs, before and after: **8,699,295 comparisons, 0 differing**
(16,275 elements × 535 properties). The only difference in the raw capture was the order in
which custom properties enumerate, because one `<link>` is gone — no value moved. This
mattered more than usual: the rules landed LATER in the cascade than they had been, since
`admin-design.css` was loaded first of six.

### D-078: Four things the owner saw

**Status:** 2026-09-22. All four reported from the screen, all four measured before being
touched, and none of them was where the report pointed.

**The picture scrolled sideways when it fitted.** The frame is laid out at FULL size and
then scaled, and `transform` does not shrink a layout box: at 1040px of stage the box was
still 1280px, so the stage reported a scrollable width it could not use — a horizontal
scrollbar under a picture that fitted, with a band of nothing to the right of it. **Measured
at every width the screen has, not only on a resize**, which is where the owner noticed it.

- **The asymmetry was there to read in `draw()`:** the height is already divided by the
  factor so it lands back on the stage; the width never was. `admin-appearance-picture.css`
  even claimed "the frame's own rectangle is exactly the stage's" — true of one axis.
- **Negative end margins on both axes**, so the margin box is what is drawn. The transform
  stays: the frame's inner viewport has to be the width the owner chose, which is what the
  three viewport buttons are for, and only the OUTER box was ever wrong.
- **The vertical one is not decoration.** A horizontal scrollbar takes height off the stage,
  which changes the frame's height, which fires the ResizeObserver again. Taking the
  scrollbar away takes the loop with it. That argument came from the owner's designer; I had
  decided to skip the vertical margin because `overflow-y: hidden` clips it, and they were
  right that clipping the symptom leaves the loop.

**Two controls were the browser's, not the admin's.** The zoom select carried a padding and a
font size and nothing else; the name for a design carried only a width. Both drew as the
browser's own dark-mode controls — measured at `rgb(107, 107, 107)` on `rgb(133, 133, 133)`
with square corners — among chips and fields that are the admin's. The name's default
`content-box` with `width: 100%` is also how it came to touch the edge of its column: it is
in a `.field` now, so there is ONE answer about what an input in this admin looks like.

**The hints are off until they are asked for.** A line of explanation under every control is
what teaches this screen and also what fills a 312px column — forty-four of them. The owner
asked for the space. They are remembered in the browser and nowhere else: it is how one
person likes to work, not a decision about the site, so it is not a setting and never
reaches the database. **Without a script they show and the button is not there**: the state
that explains itself is the safe one, and a toggle that cannot toggle is worse than none.
Scoped to this screen — every other screen keeps its hints, because it is this column that
is short of room.

**And one of my own claims was wrong.** D-075 said the gauge's sample went from 40×24 to
28×18. It shrank the SVG's `viewBox` and left the CSS box at `2.5rem × 1.5rem`, and an SVG
scales its contents to the box it is given — so nothing on screen moved. The box moves now,
and the twelve pixels go to the name beside it.

**One new verdict was wrong about the INSTRUMENT three times running** while the screen
measured the same every time: it compared the two controls with `.appearance`'s own
background, which paints nothing; then read `--ui-panel` off `documentElement`, where it is
not, because the admin's tokens are declared on `.admin`; then compared with the first
viewport chip, which is the pressed one and carries the accent. CLAUDE.md's rule held every
time — a verdict that fails is a claim about the instrument until the instrument is checked —
and it is written into the scenario so the next reader does not spend the same three rounds.

### D-079: Undo in the page editor

**Status:** 2026-09-22. First slice of the page-editor work. The plan behind it, and what it
deliberately does not do, is in the section on the editor redesign below.

Removing a block dropped its field group out of the form, and the only way back was to leave
without saving — which took every other change on the page with it. A Columns block with
twelve filled items was one click from gone. That is the hole this closes.

**One path for every structural change.** Insert, remove, duplicate, move and drag all call
`api.commit()` before they act, which pushes a snapshot onto a stack of twenty. An undo that
covered removal but not a move would be worse than none, because nobody could predict it.
There is no redo: twenty steps back covers the mistake this exists for, and a redo stack is a
second thing to reason about for a case nobody has asked for.

**The state is already in one place,** which is what keeps this small: the field groups in the
form and the sections in the canvas are the same page seen twice. A snapshot is the inner HTML
of both; a restore puts both back and raises the editors inside again.

**A way back in words, not only a shortcut.** After a removal a strip appears at the foot of
the canvas — the block's name and an Undo button — for six seconds. The shortcut is not
discoverable, and the people who most need it are the ones who do not know it is there. It
sits over the canvas rather than in the panel because that is where the block was when it
went. `Esc` deselects.

**⌘Z belongs to a field the author has typed in, not to a field that merely has focus.** The
first rule written was the obvious one: ignore the shortcut whenever something editable has
focus, so TipTap and every text input keep their own undo. Driving it proved that rule useless
for three of the five actions — `api.show()` focuses the first field of whatever block is
selected, so after every insert, duplicate and move the cursor is already sitting in one.
Measured: duplicate, ⌘Z, nothing happened. A field that has not been typed in has an empty
undo stack of its own and loses nothing by letting the page have the keystroke; from its first
keystroke it keeps it. Typing is tracked from the `input` event and forgotten on every focus
change.

**Two defects the browser found that reasoning had not.**

- **A snapshot is HTML, and HTML carries a field's value in an ATTRIBUTE while typing changes
  a PROPERTY.** So `innerHTML` serialised what the server had rendered, not what the author had
  written: typing into one block and then undoing another block's removal restored the block
  and threw the typing away — the precise loss this file exists to prevent. `sync()` writes
  every live value, checked state and selected option into the markup before the snapshot is
  read. Rich text needs nothing extra: `richtext.js` keeps its hidden input current on every
  keystroke, and a hidden input is an input like any other.
- **The strip was a tall empty box.** `.builder-canvas` is a grid, so a second child took a row
  of its own and stretched to all the height the canvas was not using, with the sentence lost
  at the bottom of it. Both children are now named `grid-area: 1 / 1` — the iframe as well,
  because auto-placement steps *around* an explicitly placed item, and naming only the strip
  pushed the page into a second row and put the strip above it instead of over it.

**Reachable from outside for a reason.** `unsetRichText()` and `unsetPicker()` existed so a
duplicated group's dead editors could be raised again; a restored group's are dead for exactly
the same reason, which is the second caller that justifies `api.unsetLive()`. `api.forget()`
bumps a counter that `redraw()` checks, so an answer still in flight cannot land on a page that
no longer exists.

**Where it lives.** `public/assets/builder-undo.js`, its own file: `builder.js` is the shell —
selection, the panel's modes, the device width — and this is the page's history. `builder.js`
carries a no-op `api.commit()` stub so callers say what they mean without asking whether the
file is present.

**Checked** in `tools/browser-suite/scenarios/04-builder.mjs`, five verdicts, last in the
scenario and after the last save so a check that fails part-way cannot leave an extra block in
a form about to be submitted.

**A seam to watch:** `builder-blocks.js` is at 345 lines. Like `Blocks.php` at D-041, it is
under the 300-line guidance but not past the hard limit, and the split when it comes is
insert/remove/move on one side and the redraw conversation with the server on the other.

### D-107: The Section panel is rows of buttons, and the arrangement is a shape

**Status:** 2026-09-23. The owner, after D-106: *"možemo li na dodavanje ili odabir sekcije
imati automatski ovakav section property kao na artifaktu?"*

**A `<select>` HIDES THE ANSWER TO THE QUESTION IT IS ASKING.** The one thing an owner wants
to know while looking at a page is what else this band could be, and every field cost a click
to find out. The closed sets are now rows of buttons — **the control the Appearance screen
has had since D-065**, because this is the same question asked about a band instead of about
the site. Radios, so the form still submits without a script, the names and values posted are
unchanged, and a screen reader is told it is a radio group rather than a listbox.

**AND THE ARRANGEMENT IS A DIAGRAM**, which `SectionLayout::LAYOUTS` has said its weights
were for since it was written: *"the weights are what the panel draws as a little diagram, so
the owner picks a shape rather than a word"*. Each button draws its columns in proportion,
with the notation the page outline already uses under it — and it is drawn **from the
weights**, so an arrangement added to that list arrives here already drawn rather than needing
a rule of its own.

**THE MARKUP MOVED TO A HELPER, because there were now two callers.** `segmented_group()` in
helpers.php; Appearance calls it too, keeping its own readout and "still following the
character" state around it. One definition of a control is one definition of its behaviour.

**MOVING IT EXPOSED TWO NAME CLASHES THAT WERE ALWAYS THERE.** `.segmented` already meant a
row of LINKS in `admin-parts.css`, and `.field-row` already meant a COLUMN of stacked fields
inside the builder's panel (`builder-inspector.css`). Neither had ever met the Appearance
version, because no screen loaded both — and putting the control in `admin.css`, which every
screen loads, made every screen load both. The seven arrangements came out in one unreadable
row and the readout fell to a line of its own. **Two controls, two names:**
`.segmented-choice` and `.choice-head` / `.choice-value`.

**`short_label()`** takes `style.<key>.short.<value>` where one is written and the ordinary
label where it is not, so "Stack them, top to bottom" stays the right sentence in a hint and
"Stack" is what goes on the button. A test refuses any value over fourteen characters, so a
value added to a closed set is caught here rather than on the screen.

**AND THE CONTROL BROUGHT A BUG NOBODY HAD NEEDED TO THINK ABOUT: an unchecked radio is not
an answer.** A group of radios shares ONE name, so a serialiser that writes every field into
a map by name leaves the LAST option standing. Changing the arrangement redraws the whole
band on the server, so every other choice was posted — and posted wrong: the band came back
gradient, centred, full width and curve-edged, every closed set's final value at once. The
owner watched his page turn purple: *"provjeri zašto se nakon promjene broja kolumni cijela
sekcija stilizira"*.

**Three places serialised a group of fields and each had decided separately what to do about
an unchecked control** — one skipped checkboxes, one skipped nothing, one skipped checkboxes
in one of its two loops. Fixing the first left the bug alive in the other two, and it was
still there on the next run. They are one function now, `collect()`, because the rule is one
rule. `44-sections` asserts that changing the arrangement leaves every other class the band
had, and that check was proved to fail against the bug — `surface-contrast` to
`surface-gradient`, `width-wide` to `width-full`, `divider-line` to `divider-curve`.

**A cleanup that could not find its own control shrugged.** `44-sections` set the band back to
one column through `if (select) { … }`, and with the <select> gone it found nothing, did
nothing and reported success, leaving the page arranged. It fails loudly now — the rule
[[probe-cleanup-is-not-optional]] already states, met again in its quietest form.

### D-106: Seven things the owner saw while testing, and what they had in common

**Status:** 2026-09-23, reported on sight while opening the editor to test D-105.

> *"na prvu vidim da dodavanje sekcije i blok prostor prelaze izvan granice sekcije —
> preview layout je kratak po visini"*

**ONE ROOT CAUSE FOR THE FIRST: an empty column had no height.** On a page an empty column
is nothing and should be; in the EDITOR it is the place a block goes, and at nought pixels
everything belonging to it had to be drawn somewhere else — the "+ Block" area and the
"+ Section" pill then stood outside the band's own edge. Measured on a freshly added band:
band 128px tall, column 0. Given `min-block-size: 6rem` in the canvas only, the band is
224px, the slot sits inside it, and nothing crosses an edge.

**AND "+ Section" IS NOW A STRIP ON THE SEAM, not a pill across it.** It was centred on the
boundary, so half of it stood in the band above and half in the one below, and it cut
through the dashed outline of a selected band. It now wears exactly what a column's
"+ Block" wears — a hairline strip the width of the page with the control centred in it —
which is also what the design artifact draws, and reads AS the seam rather than as something
poking through it. The two-tone rule is unchanged: ink inside, white outside, full opacity
at rest.

**THE SECOND WAS A GRID LEFT IMPLICIT.** `.builder-canvas` had two rows and named neither,
so both were `auto` — and a grid's `align-content: normal` behaves as **stretch**, which
shares the spare height between them. Measured: 708px of page and **242px of breadcrumb**,
which is why the page looked short and the words sat a screenful below it. Now
`grid-template-rows: minmax(0, 1fr) auto`: the page takes what is left, the trail is one line.

**Three more came in the same breath, and one of them was the worst defect in the editor.**

**A CLICK ON THE PAGE OPENED THE NEXT BLOCK'S FIELDS.** *"klik na blok Questions mi otvara
postavke Image and text bloka"*. Selection travelled between the canvas and the panel as a
**position**, and a position names a block only while both sides count the same order. Adding
one block ends that: the canvas draws it where it stands on the page, its field group is
appended at the END of the form. Measured — canvas `[n0, k0, k1 …]` against form
`[k0, k1 … n0]` — so every click was off by one. **The key was on both sides all along and
agreed; only the counting did not** (D-094 said exactly this and the message never carried
it). Selection now travels as a key in both directions, with the position as a fallback for
a message that carries none.

It is the same shape as the bug that deleted the wrong block twice. That one was fixed where
it was found; this one was the same mistake one message away, and nothing looked for it.

**THE LIBRARY STAYED OPEN OVER A SELECTED BAND.** *"umjesto postavki sekcije ... imamo blokove
desno"*, and then *"sekcija se nakon dodavanja ne može označiti da bi se vidjele njene
postavke"* — two reports, one cause: the band's own fields were rendered UNDER the whole
library, measured at 3,393px down a panel nobody scrolls that far. They were reachable and
invisible, which is not a distinction worth having. A selected band now shows its settings
and hides the library; the library is what you see once you have AIMED somewhere (D-099), and
pressing "+ Block" in a column brings it back.

**AND THE SEAM LOST THE EDGE I HAD JUST GIVEN IT.** The strip above was drawn with a dashed
border, and over a band that is itself outlined the two sets of dashes read as one confused
thing: *"preljevaju te linije preko sekcije"*. It is a place, not a frame — a neutral veil
marks the seam and the control on it is what has to be legible, which is how the artifact
draws it and what `contrast_test` now guards in both directions: the two tones on the control,
and no edge on the strip.

**A BAND COULD NOT BE PRESSED IN THE PAGE.** Found while checking his third report rather
than reported: the canvas could be TOLD which band was marked but could never say so itself,
so a band was reachable only from the outline — and an empty one, which is what you have the
moment you add a section, has nothing else to press. Pressing it cleared the selection, which
is the opposite of what pressing a thing means. It now sends the band's key.

**AND "+ Block" HAD NOWHERE TO STAND, which took three wrong answers.** It is a line under
what a column holds; "+ Section" lies across the band's bottom edge; a band whose content
reaches that edge has room for neither. Raising the line covered the words — *"sada dodavanje
bloka prelazi prema gore preko sadržaja blokova"* — and shrinking it clipped its own label.
The owner asked the question none of the three answered: *"zašto u prevju ne bi mogli imati
dole mjesta kao u artifaktu?"* **A preview may keep room for the editor's own controls.**
Every column now holds 3.5rem under its content in the canvas, which is enough for the strip,
the gap and the seam's upper half.

That is the second place the canvas is deliberately not pixel-identical to the page, after
the empty column, and the rule behind both is the same: **an editor has to show the places
things go, and a place with no size is not one.** Where D-097 proved the canvas moved
nothing, that was about the PAGE's own drawing; the editor's marks were always allowed their
own space, and this is where that becomes explicit.

**Every one of the seven is checked in `44-sections` and `45-add-section`, and every check
was proved to fail**
against the behaviour it replaces — column 0; a 702px page under a 244px trail; the library
open with the fields 3,347px down; and `n0 opened k2, k2 opened k3, k3 opened k4` — because a
check written after a fix that cannot fail is not a check.

### D-105: The nine blocks — what people were making out of the wrong block

**Status:** 2026-09-23, the second half of the review's step 5. The owner chose the set:
*"Reviewovih osam + Slika"* — the review's §2.3 eight, plus Picture.

**quote, gallery, accordion, cta, stats, embed, divider, logos, picture.** Every one of them
is something an owner is doing TODAY with a block that was never meant for it: a testimonial
typed into a Text block as italics, a row of client marks pushed through a Gallery that crops
them, a closing "get in touch" built out of a Hero, which is the block that OPENS a page. The
set is not "more blocks"; each one is a misuse with a name.

**PICTURE ONLY BECOMES POSSIBLE WITH COLUMNS.** Alone in a full-width band a picture is a
poster, and `image_text` already covered a picture with words beside it. In a column it is
the commonest thing there is, and nothing in the set could say it — which is why it was worth
adding to the review's eight and was not worth adding before D-100.

**`embed` IS THE ONE WITH A SECURITY SHAPE, and the rule is not validation.** The pasted
address is NEVER the iframe's src. `App\Support\Embed` reduces it to a provider and an id
and BUILDS the src from a fixed template, so whatever is stored — a typo, an old paste, a
string put there through a database somebody had access to — the only thing that can ever be
framed is one of four addresses with an id of that provider's own shape. A test asserts that
property over hostile pastes on the right hosts, not merely a list of bad strings. The frame
is sandboxed without `allow-popups` or `allow-top-navigation`, and YouTube goes through
`youtube-nocookie.com`.

**The four are YouTube, Vimeo, OpenStreetMap and Google Maps.** A fifth needs its own
decision. An `http`/`https` scheme is required: a scheme-less paste could not be dangerous,
since the src is rebuilt regardless, but a paste out of a browser's address bar always has
one, so refusing it costs an owner nothing and is a rule that fits in a sentence.

**AND IT IS NOT VALIDATED ON SAVE, DELIBERATELY.** The field-type set is closed (SPEC §5.3)
and holds no type that could check a provider, and a block-by-block validation hook with one
caller is the abstraction this project refuses. An address nothing recognises draws a note in
the CANVAS — `blocks.css` hides it on the page, `canvas.css` shows it, the same device the
Columns block uses for an empty column — which puts the message where the owner is looking at
the moment they paste one.

**`accordion` CARRIES NO JAVASCRIPT.** `<details>`/`<summary>` fold by themselves, fold
before any script has loaded, print open, are opened by the browser's own find-in-page, and
are what a screen reader already knows. `open` is written from the block's own `start`
setting and is never state: the page a visitor is handed looks the same every time.

**`logos` IS NOT A GALLERY WITH A SWITCH.** A gallery crops so a grid lines up; cropping a
logo is the one thing nobody is allowed to do to one. The marks are set to one height and
keep their own width, and a mark with a name and no picture shows the NAME in the site's own
type — which is a legitimate way to run the block, not a fallback, because half the marks a
small studio can show never arrived as a file.

**THE DEMO GAINED A FIFTH PAGE, AND THE OTHER FOUR GAINED REAL USES.** `tests/demo_test.php`
requires every block in every layout to appear on a published demo page, which is what makes
the demo a visual fixture rather than a sample. Rather than relax it: the home page's
blockquote-in-a-Text-block became a Quote and its closing Hero became a CTA (the exact misuse
the block was written for), About gained the numbers, a quotation and the MAP under its own
address, Services gained the questions and a closing CTA — and a fifth page, `/blocks`,
sweeps up the shapes that no studio page would honestly have. Keeping the two kinds apart is
the point: a demo where every page is a catalogue teaches nobody what a page looks like.

**Each block has its own icon.** Nine blocks sharing `image` is nine cards nobody can tell
apart, and the library card is read by its picture before its name. `tools/icons/build.php`
gained eight Lucide names and the sprite was rebuilt.

**AND A NEW RULE ABOUT TOKENS, because three of the ones this slice used did not exist.**
`--text-l` for `--text-lg`, `--text-s` for `--text-sm`, `--leading-tight` for
`--leading-heading`. A `var()` naming a token nobody defines is **silent** — the declaration
is dropped and the element inherits — so the Quote block shipped set at body size and looked
merely underwhelming. The existing rule asks only that a size come FROM a custom property,
never that the property exists. `blocks_test` now reads what the design layer compiles over
every preset, plus what the stylesheets define for themselves, and refuses any `var()`
**with no fallback** naming something outside that set. Only unfallbacked reads:
`var(--chrome-header-bg, …)` is the documented shape of a token emitted only when the owner
set one (D-076).

**How it got through is worth more than the typo.** The token list was grepped out of a
stylesheet that already held the edit being checked, so `--text-l` counted as evidence that
`--text-l` existed. An instrument that includes the thing under test measures nothing; that
is the same rule as "fix the instrument before judging the subject", one step earlier.

**The browser suite gained `47-new-blocks`**, which adds all nine by hand on a page of its
own and deletes the page afterwards. It earned its place on its first run: a page of nine
untouched blocks is REFUSED, because five declare a required field and two of those are
required once per repeater row. That is the blocks being honest — a `<summary>` with nothing
in it is an unpressable control — so the scenario fills them as an owner would.

**And six scenarios went red for reasons that were not defects, two of them lying about what
they pressed.** `44-sections` and `45-add-section` took a block by reading the library's
buttons and picking the first whose words began with "text" — the Text CARD until D-104 put a
row of SHELVES above the cards, one named Text, which D-105 then gave a block so it is always
offered. They pressed the filter, added nothing, and the failure surfaced three verdicts later
as "the block landed in the wrong column". **A check that names a control by its words hands
the next reader a wrong diagnosis**; they now aim at `data-add-type`, which is what the card
IS. `32-contact` pressed a card on an empty page, which D-101 deliberately made do nothing.
Three carried a literal band count and now count the page at the start. `20-page-links` was
wrong rather than stale and predates this work: a link applied across a selection spanning two
paragraphs is two anchors, so "exactly one more" could never hold for the block it picks.

**WHAT THE DEMO STILL CANNOT SHOW: a section with columns.** `DemoSite::seed()` writes one
band per block, so nothing in the demo exercises D-100 to D-104 — the arrangement work of
this whole stretch is visible only to somebody who opens the editor and builds one. The
seed's data shape has no grouping to say it with. Recorded as **O-28**.

### D-104: A block says which shelf it sits on, and the library can be searched

**Status:** 2026-09-23. Closes **O-15**, and it is the first half of the review's step 5 —
the half that has to exist before eight more blocks arrive.

**Why now.** One column of cards is a list you read; with thirteen it is a list you scroll
past. The filter and the shelves narrow the SAME cards — there is no second list, nothing is
fetched, nothing is rebuilt — so without a script the library is exactly what it has always
been.

**A closed set of five:** text, media, layout, marketing, embed, the ones the design artifact
names. A free string would let one block say "Media" and the next "media", and the library
would grow a shelf for each. **Only the shelves that have a block on them are offered**, so
the filter never finds nothing by being asked a question nobody can answer.

**WHAT IS SEARCHED is what the server wrote onto the card** — the name, the shelf and the
line saying what the block is for, lower-cased once. Searching the rendered text instead
would reach into the preview's iframe, which holds the demo's own words, and would match a
block for something it merely happens to say.

**AND THE KEY IS OPTIONAL, WHICH IS NOT A LOOPHOLE.** The site's chrome goes through the same
validator: a header and a footer are blocks by every other measure — fields, layouts, a
template — and they are the two that can never be ADDED, so a shelf is a thing they cannot
have. They say so by leaving it out. A page block that left it out would fall off the filter
silently, so a test walks `app/Blocks` and refuses one without a shelf, and walks
`app/Chrome` and refuses one WITH. It cost a 500 on the copy to notice they share a
validator at all.

**The other half arrived as D-105:** nine new block types, and `embed` with the security
shape this predicted — a closed list of providers, the id parsed out of a pasted URL, a
sandboxed iframe. Free HTML is refused, for the reason `SectionStyle` refuses a free colour.

### D-103: The editor draws columns everywhere, and a drag has two levels

**Status:** 2026-09-23, the last of the review's step 4.

**Dragging a block between columns** — which is what makes a column a place rather than a
label. Bands reorder among themselves; blocks move within and between columns. The column
sortables share one group name, which is the whole of what lets a block cross: Sortable's
own mechanism rather than machinery of ours.

**AND A DRAG IS REPLAYED AS PLACES, NOT AS AN ORDER.** A flat list of keys could say that
two blocks swapped; it could not say that one of them crossed into another column — and the
undo, which compared orders, called such a move "nothing happened" and made it un-undoable.
Each block now says which band and which column it stands in, and the form is put in that
order.

**THE EDITOR'S CANVAS ALWAYS DRAWS COLUMNS.** A band of one block is drawn on a PAGE as the
element it has always been — there the shape can be proven not to have moved, and D-097 says
why. In the editor that band had no column, so there was nowhere to drop a block into and
nowhere to hang its `+`; every part of the editor would have needed a branch for it. One
boolean on `SectionRender::draw()`, so the two shapes stay described in one place. Nothing a
stylesheet reads changes between them — D-093's measurement — so the page looks the same
either way.

**What that cost, and each of these is a real change a person will notice:**

*A library card now needs somewhere to land.* Pressing one with nothing aimed at used to add
the block as a band of its own at the end of the page — a guess, at the one place nobody
meant. It now says where to press, which is what the library's own line has said since, and
what the design artifact says.

*A block's arrows move it within its column.* They used to move it one place on the PAGE,
which is what a flat list had. In a tree the next place is in the same column, and stepping
outside it would drop the block into a neighbouring band — one band emptied and another
holding something nobody put there. Every block of every page today is alone where it
stands, so both arrows are unavailable and the BAND's arrows are what move it down the page.

*A duplicate stands beside the block it was copied from*, in the same column, rather than
becoming a band of its own at the next position.

*The tool bar goes in the gap above what it belongs to*, and which gap that is now depends:
a band, or a block first in its column, straddles the BAND's top edge — the geometry every
page had while a block was a band — and a block standing under another sits wholly above its
own, in the gap the column keeps. Reading the block's own edge for the first block put half
the bar on the first line, measured on the development site.

**Eight scenario reads moved, none weakened.** They counted `> section` where they meant
blocks, clicked a band where they meant a block, looked for a surface class on a block that
carries it on its band, and pressed a library card with nowhere aimed. Every one of them was
true while a block WAS a band.

### D-102: Where you are, and what you can do to a band

**Status:** 2026-09-23, the rest of the artifact's navigation.

**A breadcrumb over the page:** `Section 2 › Column 1 › Text`. A page used to be a list of
blocks, and the block under the cursor said everything there was to say about where it
stood. On a page of bands and columns it does not — the same block can be the whole of one
band or one of four things in another. The column is named only where there is more than
one, the same rule the outline follows. It sits ABOVE the canvas rather than over it, so it
never covers the first band, which is the one a person looks at most.

**A band's own four**, the same four a block has, acting on the band and everything standing
in it: move up, move down, duplicate, remove. A band is three things at once — an element on
the canvas, a group of fields, and the blocks it holds — so each of them moves all three.

**AND THE BUG THIS FOUND, which nothing was asking about.** `Page::update()` writes the
bands in the order they are SUBMITTED, and that is the order their field groups stand in the
form. A band added in the middle of the page had its group appended at the END — so the
canvas said middle, the outline said middle, and the save said last. Three views, two
answers, and the two that agreed were the two you can see.

**Fixed by deriving rather than maintaining.** The canvas is where a band's place on the page
actually is, so the groups are put into the canvas's order after every structural change,
instead of every change being careful to insert in the right spot. Nothing is moved when
nothing differs, which keeps the keyboard where it was. The scenario now adds a band in the
MIDDLE for exactly this reason: adding at the end cannot tell a right answer from a wrong
one.

**Still to come:** dragging a block between columns, and then the library's filter and groups
with the new block types.

### D-101: A section is a thing you add, select and shape

**Status:** 2026-09-23. The owner asked it in one sentence — *"imamo li sad opciju dodavanja
sekcija u koje onda možemo dodavati blokove?"* — and the answer was no. The `+` between bands
added a BLOCK, which then quietly brought a band with it. A section existed in the database
and nowhere on the screen.

**What changed, following the design artifact rather than a reduction of it.**

- **`+ Section`** between bands and at the end. It adds an EMPTY band of one column and
  selects it with the Section tab open, so the next thing on the screen is its arrangement —
  which is the next thing a person wants. The controls carry the word: a line that says only
  `+` leaves you to find out by pressing it.
- **`+ Block`** in every column, not only an empty one. An empty column is the place itself,
  so the control is the whole box; a column that holds something gets a line UNDER what it
  holds, because a box over the content would cover the words.
- **A band can be SELECTED**, which is not a block being selected. The panel has shown one
  block at a time since it existed, and a band with nothing in it has no block to show. So it
  is its own state: every block group hidden, the band's own group shown, the Section tab
  turned to, a dashed edge on the canvas where a block gets a solid one — the band is a
  container, and its selection is about what is inside it.

**AN EMPTY BAND IS DRAWN IN THE EDITOR AND NEVER ON THE PAGE.** To a visitor it is a surface
and a rhythm around nothing, and `Sections::prune()` removes it on the next save. To an
author it is the band they just added and are about to fill, and a `+ Section` that appeared
to do nothing would be the editor lying about what it had done.

**Three things this turned up, none of which a test was asking about.**

*A block added into a column could not be typed into.* The panel stayed on the Section tab,
where a block's fields are `display: none` — so the block existed, was selected, and its own
field could not even be focused. Adding a block now turns to the Content tab, because what
you do with a new block is write in it.

*A band of one block has no column element to hang a `+` on.* `SectionRender` gives it the
shape it has always had — the band IS the block — so looking for `.section-column` found a
place to add a block in exactly the bands that did not need one. The band's own `.container`
is that column, numbered 0, which is what the save calls it too.

*The `+ Section` pill and the `+ Block` strip met on a band's edge* where the padding is
small, and the last one drawn won. The smaller target has to be the one that can be hit, so
the pill sits above.

**And the outline had to learn about empty bands.** It ordered bands by walking the BLOCKS,
so a band with nothing in it vanished — which is precisely the band somebody has just added
and is looking at. The page order is the canvas's; the outline reads it from there.

**Still to come:** a breadcrumb over the canvas, a band's own tools (move, duplicate, remove
a section), dragging between columns, and then the library's filter and groups with the new
block types.

### D-100: The page outline, which the review asked for in the same slice and got later

**Status:** 2026-09-23, after the owner opened the editor and said it plainly: *"ovo je
prebugovito i neintuitivno, ovo je upravo onaj trenutak na koji sam upozorio, na kojem puca
development i gubi se volja."* He was right, and the diagnosis is not that the direction was
wrong.

**Where the direction was NOT wrong.** The storage, the rendering, the seven layouts, the
style on the section, the stable keys — that is the review's own plan, in the review's own
order, step by step.

**Where it was.** The review says of the outline: *"I would build it in the same slice as the
tree, not later."* I built the tree and shipped it without the outline, without the panel's
breadcrumb, without insertion points inside a column, and without two-level drag. Those four
are not decoration: they are how a person navigates a tree. Without them the page became
sections of columns in the database and nothing on the screen said so — which is why the
owner asked whether sections could be added at all. In the editor a section was not a thing;
it was a tab that appeared when a block was clicked.

**The lesson, and it is about cutting rather than about direction.** I broke the review's one
slice into small steps so each could be proved, and shipped the half that has no handles.
Every step was green and the whole was unusable. A slice ends with something a person can
use, not with a mechanism that works.

**What the outline is**, following the design artifact the owner sent, which is the shape to
build toward rather than my own reduction of it:

| row | what it says |
| --- | --- |
| Section N | its arrangement as notation — `1`, `1/2`, `2/3+` |
| Column N | how many blocks stand in it, and only when the band has more than one |
| the block | its own name, and its key — `b42`, `n7`, what the rest of the editor calls it |

Pressing a block row selects that block in the panel and on the canvas; pressing a band row
turns to the Section tab, where that band's own fields are. The row you are on is marked
with a tint AND a bar down its leading edge, because a tint alone is a colour difference and
a bar is a shape.

**DRAWN BY THE SERVER, rebuilt from the PANEL.** The rows arrive rendered, so the tree is
right before a script has run, after a rejected save and with JavaScript off. The script
rebuilds them from the field groups — which already say which band they stand in, what the
block is called and which icon it wears — because that is the same data the save posts, and
an outline built from anything else is a second copy of the page's shape to keep in step.

**A key is not a key.** `data-block-key` on a group is the key the CANVAS is paired by; the
block's own name is in its field names. Reading the first for the second drew an outline
whose rows named blocks nothing could find, and it took a click that did nothing to notice.

**Still to come, and this is the rest of the artifact:** `+ Section` between bands and at the
end, with the layout chosen as a diagram rather than a word; `+ Block` under the content of
EVERY column rather than only an empty one; a breadcrumb over the canvas; selecting a band on
the canvas rather than its first block; dragging between columns; then the library's filter
and groups, and the new block types.

### D-099: You choose the arrangement first, and the empty column asks to be filled

**Status:** decided 2026-09-22 by the owner, before the visible half of D-093 step 3 was
built.

**How a block comes to stand beside another.** You give the section an arrangement — two
columns, thirds, a wide column with a narrow one — and the empty column appears with the
same `+` that already adds a block to a page. Press it, choose a block, and it lands in that
column. Moving a block that already exists into a column is DRAGGING, and that is step 4.

**Why this and not the other two.** *A control on the block saying "put beside the block
above"* is a sentence you read where the other is a shape you see, and what it will produce
has to be imagined. *Dragging straight away* is the most natural of the three and it is also
the largest piece of work — two-level selection on the canvas and every tool rewritten — so
choosing it would mean the longest wait before anything at all is visible.

**What it means for the build.** The insert endpoint stops meaning "at position N on the
page" and starts meaning "into this section, this column, at this place" — which is the same
address the renderer and the save already use, so the `+` in an empty column is the first
control that speaks the new shape end to end.

**Built 2026-09-23, and it dragged one more thing in with it.** An empty column draws a
dashed slot with the same `+` the page already has, and pressing it inserts a block into
that column: `data-bx-section` names the band on the canvas, `[data-insert-into]` and
`[data-insert-column]` are the address, and `PageBlockController` draws the block with
SectionRender's wrapper-less shape because the band around it already draws the surface.

**THE CANVAS HAD TO START COUNTING BLOCKS INSTEAD OF BANDS,** and this was NOT optional.
Everything in the editor pairs a field group with a canvas element by ordinal, and that
element was `[data-bx-blocks] > section` — a band. Clicking a block in the second column
selected the band, which pairs with the band's FIRST block, so the tools acted on the wrong
one: pressing Remove deleted the block beside the one that was clicked. It did, on the copy,
twice, and the second time was the cleanup written after the first. So `blocks()` and
`api.sections()` now return one element per block — the band itself where a band holds one,
which is every page written before this, so nothing about existing pages changes — and the
tool bar is positioned from a rect rather than `offsetTop`, because a block in a column has
its band as its offset parent and the bar was landing at the top of the band.

The `+` BETWEEN bands still counts bands, because what it adds is a band; and `place()`
counts how many blocks stand in the bands above, because a band's position and a group's
position stopped being the same number.

**`44-sections` is the scenario**, and it is the one that saves. The others deliberately do
not: 24-columns leaves the editor without saving, because the canvas is the server's drawing
of what the form would post and so agrees with itself whether the save is right or not. Only
the served page settles whether a block that DRAWS in a column is STORED in one. Three
defects hid behind that distinction in one evening.

**AND IT NEEDS ONE THING FIRST, found while starting it (2026-09-23).** The section's fields
are rendered INSIDE each block's group (`block.php`, where they have always been). That is
exact while a section holds one block. The moment a second block joins, its group renders
the same fieldset again: two `<select name="sections[s7][style][surface]">` on one form,
editing one does not move the other, and the last one in the document decides what is saved.
Nobody would see it until they had set the surface on the wrong half of a band.

**Done 2026-09-23.** Before a block can be added to a column, **the section's fields moved
into a group of their own** — `[data-section-group="s7"]` beside `[data-block-groups]` — and the Section tab
(D-086) shows the group belonging to the selected block's section instead of a fieldset
folded into the block's own. That is the split D-086 described, done properly rather than by
an attribute; it removes the duplication rather than managing it, and it is what makes
"select any block in a band and set the band's style once" true. The plain editor keeps its
single scroll: there the section group is rendered where the fieldset is today, above the
blocks it holds.

### D-098: The editor carries a flat list of blocks and a map of sections, not a tree

**Status:** 2026-09-22, while building the second half of D-093 step 3. The model half is
done: `Page::update()` can be told a page's sections. The editor does not speak it yet.

**The choice.** The obvious shape for a tree is a tree — `sections[s7][blocks][b42][field]`.
It was refused, and the alternative is the one the FRONT END already uses: a flat list of
blocks, each naming the section it stands in and the column it stands in, beside a separate
map of sections. `Sections::group()` joins the two at the moment of drawing.

**Why, and it is not taste.** The nested shape renames every field on the screen, and the
names are load-bearing in five places that have nothing to do with sections:
`admin.js:nameGroup()` rewrites four regexes all anchored at the start of the name;
`repeater.js` pulls a block's key out of `blocks[…]` and rewrites an item index *in the
middle* of the name, safe today only because both editors' regexes stop before it *by
construction*; `views/admin/item.php` builds a third prefix from the same string;
`builder-blocks.js:values()` strips `blocks[…]` to talk to the redraw endpoint; and
`builder-save.js` derives the `_unchanged` marker's name from the id input's. A tree in the
field names puts all five in play at once, for a fact that fits in two hidden inputs.

So a block gains `blocks[<key>][section]` and `blocks[<key>][column]`, and a section's own
five style keys plus its layout and stack are typed at `sections[<skey>][…]` — a NEW prefix
beside the old one rather than a wrapper around it.

**A section is named, like a block (D-094):** `s7` for one the database knows, `m0` for one
made in this session. A different letter from `b`/`n` on purpose — the same JavaScript reads
both, and `s7` meaning a section while `b7` means a block is a difference a reader can see.

**Silence means "leave it", everywhere.** `Sections::save()` takes a null layout, a null
stack and now a null style, each meaning "as it was". Three callers need it and each would
otherwise do damage: a save from the plain page editor would flatten a section arranged in
the builder; an older form or a hand-made request would set `one` by saying nothing; and a
section holding a block this installation cannot draw would be repainted with the defaults,
losing the only record of what it was. That last one replaces `Page::keepSection()`, which
did it with a second read and a second write.

**A block naming a section nobody sent is given one of its own** at the end of the page,
rather than dropped or attached to a neighbour. Visible, obviously wrong, and nothing lost.

**What `Page::update()` now writes.** Sections first, in submitted order, which is the
page's order; then each block with its section, its clamped column, and its place counted
DOWN that column. A block's `sort` finally means what SPEC §5.0 has said since 0026.

**Built on top of it, the same evening:** `editable()` says where each block stands and
`editableSections()` says what the bands are; both controllers parse `sections[…]` and carry
it through every re-render; `pending_canvas` carries the arrangement; **the canvas draws the
page the way the page draws itself** — the same `Sections::group()` + `SectionRender::draw()`
loop `PageController::show()` runs, which until now it did not, so the editor and the site
agreed only by the accident of every section holding one block; and the Section panel has
Columns and On a narrow screen.

**Two things this cost, both found by measuring rather than by reading:**

*Every block added in a session went into one band.* `nameGroup()` rewrites names, and
`[section]` is a hidden input whose VALUE names the section — a clone kept the template's,
so `m0` meant all of them. It now mints a section key beside the block key.

*Choosing two columns replaced every section on the page.* The form posted the arrangement
and the save ignored it, because `BlockForm::parse()` never read `[section]` off a block:
every block named nothing, every block got the "give it a band of its own" fallback, and six
section rows were quietly replaced by six new ones. Nothing looked wrong on the screen. It
was found by reading the ids out of the copy's database after the save, and it is why the
round trip now has a test that asserts the section row is **the same row**.

**And the revisions carry it too (2026-09-23).** `data_json` holds the bands beside the
blocks, so restoring a page that had two columns when it was recorded puts the columns back
and not only the words — without it, a restore was last Tuesday's content poured into this
week's bands, which is neither one page nor the other. A revision written before this has no
bands and says so by their absence: null reaches `Sections::save()` as "leave it", the only
honest answer, since that revision does not know what the arrangement was and guessing "one
column each" would flatten a page whose columns were never what the restore was about.

**TWO REGRESSIONS THE OWNER FOUND BY OPENING THE EDITOR, on 2026-09-23**, both from moving
the band's fields into a group of their own, and neither visible to any check I had written.
He said: *"vidim samo kad odaberem neki blok da ima tab Section, ali što god tu odaberem ne
radi ništa."*

*Nothing chosen in the Section tab reached the canvas.* The live redraw listens for `change`
on `[data-block-groups]`, and the band's fields had moved to `[data-section-groups]`. It had
listened in the right way to the wrong container ever since.

*And typing one letter took the band's look off the canvas.* A block redraw posts the
group's fields to the insert endpoint, which drew the block with the character's composed
style because no style arrived — so a tinted, airy, wide, centred band went plain, normal,
narrow and left on the screen while the database held the truth. Nothing was lost; the
editor simply stopped telling the truth about the page, which is the one thing it is for.

**Fixed both, measured before and after.** A redraw carries the band's fields (`section[…]`
beside `block[…]`), and the Section tab acts on the canvas the moment something is chosen:
the five style keys are swapped straight onto the band's own `<section>`, which is instant
and which a block redraw cannot reach when a band holds several, and the block is redrawn
besides. `44-sections` now holds both, and both were watched failing before the fix.

**WHAT STILL NEEDS A SAVE TO BE SEEN: the number of columns.** Changing "One column" to
"Two columns" rearranges the band's markup, which no class swap can do and which the block
redraw endpoint cannot draw — it draws one block, not a band. The honest fix is an endpoint
that draws a BAND from its own fields and its blocks', reusing `SectionRender::draw()` so
the two shapes stay declared once. Until then the canvas catches up on save.

**Still to come:** the band redraw above, dragging a block between columns (step 4), and
then the eight new blocks (step 5).

### D-097: Seven column layouts, and the section draws them

**Status:** 2026-09-22. Third step of D-093, first half: a section can hold blocks in
columns and knows how to draw them. The editor cannot yet make one — that is the second
half, and until it lands no page on any site can reach the new shape.

**The closed set, as D-093 promised:** `one`, `halves`, `thirds`, `quarters`, `wide-left`
(2fr+1fr), `wide-right`, `sidebar` (3fr+1fr). Never a percentage, for the reason
`SectionStyle` refuses a free colour: a column at 37% cannot promise a readable measure and
cannot say what it does when the screen halves. One knob for narrow screens — `stack`,
`stay`, `reverse` — and `reverse` earns its place because a picture left of text reads
correctly side by side and wrongly stacked.

**TWO SHAPES, DELIBERATELY.** A section of one column holding one block is still the same
element as that block: `<section class="block block-hero layout-center surface-tinted …">`,
the markup every page has had since the first commit. Anything else draws
`.section-cols` inside the container and gives each block a `<div class="block-{type}
layout-{layout}">` of its own.

The tidier design is one shape everywhere, and it was refused on evidence rather than taste.
`compare.mjs` can only compare elements both captures have, so a wrapper added around every
block on every page is the one change it cannot check — and this is the step where the
owner's site must be provably unmoved. The cost is five lines in `SectionRender::draw()`.
The cost is *only* five lines because of what D-093's measurement found: every rule reading
`layout-*` is a descendant selector and `block-{type}` is a selector only for chrome, so
both classes may sit a level lower and no stylesheet has to know which shape it is looking
at.

**PROVED, NOT ASSERTED.** `node compare.mjs diff before after`: **3,848,190 property
comparisons, 0 differing**, 7,050 elements before and after, across five characters, four
pages and three widths. The "before" capture was taken from a stashed tree at `5c4952a`, so
it is the released code and not my memory of it.

**What the columns look like** was checked on the copy, not reasoned about: a text block and
a form put into one section, screenshotted at 1400 and at 390, and the probe put the rows
back by exact id in the same run (memory: probe cleanup is not optional — and the first
restore missed `stack`, which is why the check is the reading and not the intention).

The three narrow-screen behaviours were MEASURED rather than argued from specificity, which
is what I would otherwise have done, `.stack-reverse` and `.section-cols` both weighing one
class:

| | at 1400 | at 390 |
| --- | --- | --- |
| `halves` + `reverse` | grid, `626px 626px`, both columns level | `flex`, `column-reverse`; the second block is drawn ABOVE the first |
| `quarters` + `stay` | grid, `299px` × 4, the two empty columns drawn | grid, `153px` × 2 — `stay` keeps proportions only where there are any left to keep |

**Two declarations of one fact, with a guard over them.** The proportions are grid tracks in
`sections.css` and weights in `SectionLayout::LAYOUTS`, because CSS is not generated from PHP
here. `tests/section_columns_test.php` reads the stylesheet and refuses a layout with no rule,
a rule laying out a different number of tracks, and a `.cols-*` rule for a layout PHP does not
know.

**A block remembers a column its section no longer has.** Narrowing a section from four
columns to two draws those blocks in the last column and leaves the stored value alone, so
widening it again puts them back. Nothing is silently dropped and nothing is silently
rewritten.

**What migration 0027 does not do:** it moves no data. `column_index` defaults to 0, which is
where every existing block already stands. It is named `column_index` and not `column`,
because `column` is reserved in MySQL and a name that only works quoted is a trap.

**A correction to this decision, made the same day.** `Composition::apply()` first returned
the number of BLOCKS it had touched, and I wrote in the commit message that this kept the
admin's number meaning "blocks restyled". The message it feeds says *"…:count **sections**
were reset to the … composition"*, which I had not read. While a section held one block the
two counts were the same number, so nothing could say which it was; the moment a section
holds two they part, and a number that quietly means something other than the sentence
around it is worse than no number. It counts sections.

**And running the WHOLE browser suite, not the scenarios this looks like it touches, found
four that had been broken since D-094** — when a block's key became a name (`b12`) instead
of a position (`1`). `14-front`, `03-design` and `16-slice5-accept` each built a field name
out of a number and had matched nothing since; `14-front`'s *"the first section is eager and
every later one is lazy"* was not running at all, which is the check that matters most on
that page. Fixed here by reading the key off the form instead of guessing it. **Not a test
adjusted to match new output:** the rule changed deliberately in D-094 and these probes were
never told. The fourth is `12-picker`, which fails identically at `5c4952a` and is recorded
as O-27 rather than fixed in a slice it has nothing to do with.

424 checks, 4 failing before; 3 fixed here, 1 recorded.

**The half that is missing, and it is not small.** `Page::update()` still writes one section
per block, so the page editor would flatten a section's columns back into a row of bands.
Nothing can create one yet, so nothing is at risk today — but `Page::editable()`,
`Page::update()`, `BlockForm`, the canvas, `pending_canvas` and `PageRevision` all still hold
a flat list of blocks, and all of them have to hold sections before the editor may offer a
layout. That is the next commit and it is the one the owner will actually see.

### D-096: A character composes a section from the type its blocks agree on

**Status:** decided 2026-09-22 by the owner, before step 3 of D-093 needed it. **This is the
question D-093 left open and the review never raised**, and it is the real cost of the tree.

**The collision.** `Composition::style($character, $blockType)` derives `surface` and
`divider` from a BLOCK TYPE — "Editorial gives a hero a tinted surface, a form a plain one".
That is exact while a section holds one block. A section holding a hero and a form has no
single type, so the sentence has no answer.

**The rule chosen.** A character composes a section from the type its blocks agree on. One
block, or several of the same type, compose as that type does today. A section whose blocks
disagree composes from the character's own `section` defaults, with no per-type surface or
divider laid over them.

**Why, in the owner's terms.** Every one of today's pages keeps exactly the look it has, and
that is not an argument from taste: every existing section holds one block, so "the type they
agree on" is that block's type and nothing is composed differently. The ritmo of tinted and
plain bands that makes a character feel designed survives, because a band of three text
blocks still composes as text. Only the genuinely ambiguous case — two different kinds of
block side by side — falls back, and it falls back to something the character itself states
rather than to a guess.

**Three that were offered and refused.** *From the first block's type*: moving a block to the
front would silently repaint the whole band, a change the author did not ask for and cannot
see the cause of. *The character composes sections only, types give a starting value at
creation*: the cleanest rule, and it throws away the band rhythm on every "Apply character".
*Skip mixed sections*: nothing is lost but part of the page stays outside the character,
which is visible and unexplained.

**Where it lives.** `Composition::style()` keeps its per-type signature, and a second entry
point takes the list of types a section holds and reduces it: one distinct type composes as
that type, more than one composes as the character's `section`. The same reduction decides
whether the Section panel reports the style as changed from the character's (D-086), so the
panel and "Apply character" can never disagree about what a section should look like.

### D-095: The section is a record, and it owns the style

**Status:** 2026-09-22. Second step of D-093. Nothing anybody can see changes; where the
layer-2 style lives does.

**`page_sections (id, page_id, sort, layout, style_json, …)`** and `page_blocks.section_id`,
migration 0026. Every existing block becomes its own one-column section carrying that block's
style, so every page renders exactly as it did.

**The migration gives a section the id of the block it was made from:**

```sql
INSERT INTO page_sections (id, page_id, sort, layout, style_json, created_at, updated_at)
SELECT id, page_id, sort, 'one', style_json, created_at, updated_at FROM page_blocks;
UPDATE page_blocks SET section_id = id;
```

Both databases take an explicit value in an auto-increment column and continue above it, so
the pairing is exact without a temporary column and without trusting the order rows come
back in.

**The old copy is emptied.** `page_blocks.style_json` stays — a committed migration is not
edited and dropping a column is not portable — but it is set to `'{}'`, because a second,
plausible, silently stale copy is worse than an empty one: a reader nobody updated would
return last week's surface for ever, where an empty one gives the defaults and is noticed. A
**source guard** keeps it that way: the only place `page_blocks` and `style_json` may be
named together is the one INSERT that satisfies the NOT NULL column.

**What the plan said and what was built are not the same, deliberately.** The approved plan
had this step also splitting the renderer — `Sections::render()` and a `'none'` wrapper. While
building it, that turned out to be unnecessary here: **while a section holds one block,
`Blocks::render()` already draws exactly the right thing**, it is simply handed a style that
came from another table. So this step is storage only, and "nothing is visible" stops being a
risk to be proved and becomes a consequence of not having changed a line of rendering. The
renderer split moves to step 3, where a section can hold more than one block, where it is
needed, and where it can be tested. It costs nothing extra there.

**Proved, with an instrument that was itself checked.** `tools/browser-suite/compare.mjs` is
new and kept, because D-077's proof was a throwaway that had to be rebuilt from its
description. It captures every demo page under all five characters at three widths — 60
captures — as raw HTML and every computed property of every element. Before and after:
**3,848,190 property comparisons, 0 differing.**

Both halves of the instrument were checked first, because "0 differing" is also what a broken
comparison says: two identical runs gave 0 of 3,848,190, and a deliberate
`letter-spacing: 0.0001em` gave 6,835 differences, each named by element path and property.

**What that proof does NOT cover, said plainly:** the harness applies a character with
"design only", which never touches section styles, so `Composition::apply()` is not exercised
by it. Its own test is, and passes.

**The tests caught something the proof could not.** Order. The page's order moved to the
section, and a block's own `sort` became its place inside its section — which is 0 while a
section holds one. Six places still read `page_blocks.sort` as the page's order, including
`Translations::create()`, which would have given every translated page a shuffled copy. They
failed, which is what they are for.

**`Composition::apply()` is where the design layer and sections meet**, and it is the file
that will have to answer D-093's open question. A character composes layer 2 from a BLOCK
TYPE — `surfaces[$type]`, `dividers[$type]` — so the two only meet through the block a section
holds. While a section holds one block that is exact. When it can hold several, "Editorial
gives a hero a tinted surface" has no single answer for a section holding a hero and a form.
**The review does not mention this at all**; it is the real cost of the tree and it is the
owner's decision when step 3 comes.

**Also moved:** `MediaLibrary`'s `candidates()` and `usage()` now read the section's
`style_json`. That is not cosmetic — `usedBy()` is what refuses to delete a picture, so a
missed source would have unlinked one still painted behind a section.

### D-094: A block is named, not numbered

**Status:** 2026-09-22. First step of D-093, and the one that changes nothing anybody can
see. It is O-25, deferred in D-082 with the words *"worth doing when something else needs it
— a tree"*.

**`blocks[b42][heading]` instead of `blocks[3][heading]`,** and errors keyed `b42.heading`.
`b{id}` for a block the database knows, `n{n}` for one added in this session. The key is
**derived, never stored**: a saved block's key is its id, and a new block's only has to last
until the save that gives it one.

**The order still comes from the order the groups appear in the request** — which is what
carried that meaning all along. The index never did; it only looked as if it did, because
two editors rewrote every name on the page after every add, move and remove to keep it
looking true.

**Two of the three renumbering regexes are gone.** `builder.js` now sets one attribute —
which group the panel shows — and `admin.js` renumbers nothing at all. The third, in
`repeater.js`, still has real work: an item's place inside its block genuinely is positional.
What replaces them is one function that names a group **once, when it is born**, shared by
both editors because both have the same job when they clone a template or place a block from
the server.

**A key is minted above the highest one already on the page**, not from zero, because the
server renders a new block as `n0` and a second `n0` would be two blocks with one name.

**The no-JS actions name a block too** — `up-b42`, `item-add-b42-items` — so a form rendered
before something moved acts on the block it meant rather than on whatever has taken that
slot since. A key that names nothing does nothing, which is the honest answer to a stale
button.

**Found while driving it, and worth more than the slice itself:** `place()` was reading the
key off a DocumentFragment, which throws, and the caller's `catch` reported that as *"the
block could not be added. Check your connection."* A network message for a programming
mistake, and it would have hidden the next one too. The catch now also says what happened,
the way `redraw()`'s already did.

**SPEC §5.3 changed deliberately**, the third time this month and the second on this day: the
index in field names is a key. Recorded there.

**Tests changed, not adjusted.** Thirteen assertions named a block by its position because
that was the rule; the rule changed, so they name it by its key. One became stronger on the
way: `builder_test` now reads the keys out of the rendered page and asserts they are the
stored ids in order, which proves more than the three literals it replaced.

### D-093: A page becomes sections of columns — reversing D-008

**Status:** decided 2026-09-22 by the owner, after testing the finished flat editor. Not yet
built. **This reverses `docs/SPEC.md` §5.3 and D-008**, which say a page stays a flat,
ordered list of blocks with no nesting, and it is written down here rather than worked
around, as the review asked.

**The shape.** A page becomes a list of SECTIONS. A section has a layout and holds blocks in
its columns. **Depth is exactly two: section → column → block.** A section cannot hold a
section and a block cannot hold a block.

**Why this and not "let blocks nest".** The block contract is untouched: a block stays a leaf
with fields, so every definition, every template, `BlockForm::field()`, the repeater, media
resolution all stay as they are. That is what makes it affordable, and it is also the claim
the whole estimate rests on.

**What it buys, in one sentence each.** Section style finally belongs to a section: a tinted
band with three text blocks in it is one setting instead of three that have to be kept in
step. A gallery can sit in one column with a form in the next, which no arrangement of the
present `columns` block can express. And `columns` stops being a layout pretending to be a
block — it stays as the repeating card grid it is genuinely good at.

**What the review costed, and three things it did not.** It names storage, rendering,
identity, the canvas, the panel and an outline. Measured here: `page_blocks` is read or
written in seven files and `style_json` in six. Not in the review:

- **Translations copy blocks across locales by `sort`.** Sections have to be mirrored the
  same way, or a Croatian page gets an English arrangement.
- **Three block templates hard-code `sizes="… 50vw"`,** because a block has always been as
  wide as its container. Inside a one-third column that is the wrong picture downloaded.
- **Revisions (D-088) and `pending_canvas` both hold blocks** and will have to hold sections.

**The order, which is the review's own** (the owner chose it over building the eight blocks
first), and every step leaves the editor working:

1. **Stable keys replace positional indices**, page still flat. Nothing visible changes.
   This is O-25, deferred in D-082 with the words *"worth doing when something else needs it
   — a tree"*. That day has come.
2. **`page_sections` + migration + `Sections::render()`**, every section holding one column.
   Every existing page must render byte-identically, proved the way D-077 was proved. Section
   style moves from the block to the section here.
3. **Column layouts**, blocks addressable by column, insertion and drag within a column.
4. **Drag between columns, and the page outline.** The panel's section/block split is already
   built (D-086).
5. **The eight new blocks** from the review's §2.3, and with them the library's groups and
   filter (O-15).

**Column widths are a closed set**, never free percentages: `one`, `halves`, `thirds`,
`quarters`, `wide-left` (2/3+1/3), `wide-right`, `sidebar` (3/4+1/4). The same argument
`SectionStyle` makes about colour — a column at 37% takes the design system's guarantee with
it, and a percentage cannot say how it collapses on a phone. One knob for small screens
(stack / stay / reverse), because reversing on mobile is the one thing owners need and cannot
otherwise express.

**`embed` is the one block with a security shape:** a closed list of providers with the id
parsed out of a pasted URL and rendered in a sandboxed iframe. A free-HTML block is refused,
for the reason `SectionStyle` refuses a free colour.

### D-092: Undo is a control, so it is visible at rest

**Status:** 2026-09-22. The owner, testing: *"Undo gumb ne vidim, on radi preko tipkovnice?"*

It did. Undo had a keyboard shortcut and a strip that appeared for six seconds after a
removal — so somebody who had not removed anything never learnt that undo existed at all.
**CLAUDE.md: no control is ever invisible at rest**, and a notification that comes and goes
is not a resting state. D-079 reasoned carefully about the strip being discoverable for the
people who most need it, and missed that the feature itself was not.

A button in the editor's bar, left of the device sizes, with the Lucide `undo-2` icon.
**Disabled when there is nothing to undo, never hidden**: a control that disappears teaches
nobody that it is there, and being found is the whole point of this one. Not rendered
without a script, because then nothing can undo anything.

### D-091: A row size asks for its columns

**Status:** 2026-09-22. The owner, testing: *"odabir broja kolumni na 4 ne dodaje unos
sadržaja za četvrtu kolumnu i ne prikazuje ju u bloku, samo gurne postojeće 3 u lijevo."*

Exactly so. `layout` is how many columns share a row and `items` is the content, and the two
could disagree: three items at four in a row drew three columns and an empty cell, with no
fourth field to type into. The label already said "Four in a row" and was honest; the
behaviour was still not what anybody means when they choose it.

**The block says what a layout asks for.** A repeater may declare `per_layout`, a map of
layout name to how many items it wants — a deliberate addition to the frozen SPEC §5.3,
recorded there, and the second one today after `sample`. Nothing generic has to guess that a
layout called "four" means four: `BlockForm::parse()` reads the number the block wrote down,
and the view writes it onto each `<option>` as `data-wants` so the editor reads it off the
option that was chosen.

**In the parser, not only in the save**, because the canvas redraws through the same parser:
the column appears as the row size is chosen rather than after the page is saved. And in the
panel too, by pressing the repeater's own Add — one way to add a row, the one the fallback
editor uses, which already knows about numbering, the maximum and the empty state.

**It only ever tops up.** Going back to "two in a row" keeps all four columns: a row size is
a choice about arrangement, and throwing away what somebody wrote is not one of its
consequences. A row already full is left alone — seven columns at four in a row is a full row
and a short one, which is ordinary.

**Checked by driving it:** three columns, choose four, the panel has four fields and the
canvas four columns; choose two, still four.

### D-090: A check that writes owns what it writes

**Status:** 2026-09-22. Found by looking at the development site rather than at the suite.

`04-builder` adds a block and SAVES it, and it never took it away again. Run eight times in
one session it left **seventeen text blocks on the home page where the demo has one** — on
the site the owner opens to look at his own work. Measured by fetching the page and counting
sections, not by reading the scenario.

The scenario now removes what it added and saves once, and says so as a verdict. **It removes
by the id of the field group, found by the exact heading it typed** — never by matching the
words in a block, because the demo's own text block would answer a search for "text" and a
cleanup that deletes by resemblance is how a real page goes.

**This is the same rule as `probe-cleanup-is-not-optional`, one level up.** A throwaway probe
has to restore what it changed; so does a scenario that is run hundreds of times. A suite that
writes without owning what it writes quietly becomes the thing that ruins the site it tests.

**Two things on that page I cannot account for**, and they are recorded rather than quietly
patched: it is missing its `form` block, and its first `hero` and `columns` have swapped
places, against what `app/Modules/Demo/pages.php` seeds. Both are probably from a scenario
run of mine today; I cannot prove which, and the page's own history (D-088) only reaches back
to 14:59, by which time it was already so. Putting demo content back BY HAND on the site the
owner judges is his call, not mine — the honest choices are to reinstall the demo there, or
to leave it, and he decides which.

### D-089: The rich text paste note was already there, and I nearly wrote a third one

**Status:** 2026-09-22. The last item on D-080's list, closed by looking rather than by
building.

D-080 asked for *"one line of advice beside the rich text about what is kept when you
paste"*. I wrote it: an attribute on the field, a string, a transient note shown for eight
seconds after a paste, and a stylesheet rule. It worked — pasted text arrived with its bold
and without its colours, and the note appeared.

**Then I looked at the field.** Under it already stood *"Ctrl+Shift+V pastes without
formatting."* and, under that, `pages.field.richtext_hint`: *"Kept when you save: paragraphs,
bold, italic, links, headings, quotes and lists. Anything else pasted in — colours, fonts,
tables — is removed, so the text always wears the site's design."* The item had been done
long before the plan asked for it. Mine would have been the **third** sentence about pasting
under one field.

Reverted, every line of it.

**Why it read as missing:** `richtext_hint` is a `.hint`, and D-087 — hours earlier, the same
day — turned hints off until asked for. So the explanation did not vanish; it moved behind a
toggle, on the very morning the plan's list was being worked through. That is a real
consequence of D-087 and it belongs to the decision the owner said he would make after using
it: whether hints stay off, and whether some of them are not hints at all, as D-088's note
about Restore is not.

**The lesson, which is CLAUDE.md's in another form.** *A weak feature is fixed before
anything is built on top of it* has a twin: a feature that already exists is found before
anything is built beside it. Both the plan and I took "the editor does not explain pasting"
on trust, from a review, and neither of us opened the screen to check. Measuring the subject
would have cost one look.

### D-088: What the page was before the last few saves

**Status:** 2026-09-22. Eighth and last slice of D-080. `page_revisions` has been in
`docs/SPEC.md` §5.2 since the beginning and nothing had ever built it, so this needs no
change to a frozen contract — only the migration that was always implied.

**What it is for.** Undo (D-079) covers the editing session and dies with the tab. Everything
before that save was unrecoverable: a heading rewritten last Tuesday, a paragraph deleted and
saved, a block removed and saved — gone, with no way back short of a database backup nobody
on shared hosting knows how to read. It does not change what the editor can do. It changes
what the owner dares do with it.

**A revision is an editing event, not a row rewrite**, which is why recording one is the
controller's job and not `Page::update()`'s. The demo seed calls `update()` too, and a fresh
install does not want four pages of history nobody made.

**It is read from the database, never from the request.** With D-081 a save may carry only
the blocks that changed, and "what the page was" has to be true of the whole page whatever
arrived.

**Restoring is an ordinary save.** The revision holds the page in the shape
`Page::editable()` returns, so putting it back runs the same validation, the same media
resolution, the same sitemap refresh and the same activity line as any other save. A restore
with a path of its own would be the one path nobody exercises until the day it matters. And
it records the current page first, so **a restore can itself be undone** — pressing it by
mistake must not be the one action in this editor with no way back.

**What is on screen is discarded by a restore**, deliberately: restoring to an earlier
version while keeping the edits that are open would be neither one page nor the other.

**Five per page, pruned on write**, the same reasoning that gave undo twenty steps: enough to
cover the mistake this exists for, and a limit at all because a site's whole history on
hosting sold by the gigabyte is not a kindness. Pruned with two statements rather than a
DELETE with a subquery over the same table, which MySQL refuses outright (1093) while SQLite
allows it — SPEC §5.0's portability rule is easiest to keep by not writing the clever version.

**A revision belongs to its page**, checked in `PageRevision::find()` rather than by the
caller, because this is reached from a request and "restore revision 41 into page 3" must not
be able to pour another page's blocks into this one. A row that is not JSON, or not a page, is
refused rather than half-applied; a block whose type has gone since is left out rather than
restored as a hole.

**Two things the screen decided.**

- **The sentence that makes Restore safe to press is not a hint.** Hints are off until asked
  for (D-087), and behind that toggle *"restoring one is itself a save, so it can be undone"*
  would never be read by the person deciding whether to press it. A hint describes a field;
  this states what an action does to the page, and that belongs in front of somebody at the
  moment they choose. It is the one line in the panel that ignores the toggle, and the reason
  is written where it is drawn.
- **Five saves in one working session all read "14:15".** A list that cannot tell its own rows
  apart is not a list, so `Dates` gained `localToSecond()` beside `local()` — a second format,
  not a changed one, because every other screen shows a date and this one shows an event.

**The update gate showed itself working.** The copy answered 503 until its migration was run,
which is D-019 refusing to serve a site whose schema is behind its code. On a real site the
owner presses the button; here `php migrations/migrate.php` from the copy's own directory.

### D-087: Hints on demand, in the page editor too

**Status:** 2026-09-22. The owner, seeing D-086's Section tab: *"Sakrij ih za sada ali kasnije
ćemo odlučiti kad sve testiram u praksi."* So they are hidden, with a way to ask for them, and
the decision stays open until he has used it.

Six section controls, each with a line of explanation as tall as the control itself. With the
hints off the whole of Section fits on one screen — measured, 781px of panel down to 558px.

**One implementation, two screens, which is what made it a file of its own.** D-078 built this
for the Appearance screen; `appearance-hints.js` becomes `hints.js`, and any element carrying
`data-hints-root` that contains a `data-hints-toggle` gets the behaviour. The attribute's value
is the name the preference is stored under, so a preference set on one screen cannot quietly
empty the other. A third screen needs the attribute and nothing else. **This is the second
caller CLAUDE.md asks for before an abstraction exists** — the first version was deliberately
tied to one screen, and the generalisation waited until there was something to generalise.

The two strings moved with it: `appearance.hints_show` and `appearance.hints_hide` became
`hints.show` and `hints.hide` in `lang/en/hints.php`, because a string named after one screen
is wrong on the other. The button's own style and the rule that hides a hint moved from
`admin-appearance-inspector.css` to `admin.css`, scoped by the attribute rather than by the
screen — every other screen in the admin keeps its hints, because it is these two narrow
columns that are short of room.

**`data-hints-root` sits on the whole panel**, not on the selected-block header: the field
groups are that header's SIBLING, and the rule that hides a hint has to reach them.

**And a correctness fix the hints work uncovered.** A field group can arrive as fresh markup —
an undo puts all of them back, an insert brings a new one from the server — and such a group's
style `<details>` is open only when `block.php` happened to render it open, which is when its
style differs from the character's. On the Section tab a closed one shows nothing. The first
check said it survived an undo; it survived by that accident, on a block whose style did
differ. The tab is now re-applied on every selection, which is the rule rather than the luck.

### D-086: Content and Section, side by side

**Status:** 2026-09-22. Seventh slice of D-080, and the other half of what it called the
small costs. It is not small.

**The measurement.** With the panel's first field on screen at y=287, the first
section-style control sat at y=1308 on a Hero, y=1603 on an Image and text and **y=4296 on a
Columns block** — four screens down, past twelve repeater items — in a window 1000px tall.
Afterwards, on that same Columns block: **y=340.** The thing most often changed while looking
at the page had been the hardest thing in the editor to reach.

**The split is a class on the panel and two rules in the stylesheet, not a rearrangement of
the field group.** `views/admin/block.php` is the PLAIN editor's view too, and there the
group stays one scroll with the style folded at its foot. Moving markup around would have
meant two shapes of one thing, and the plain editor is the fallback that has to keep working.
The shared view gains exactly one attribute, `data-panel-part="section"`; everything else
hangs off the panel. Without a script nothing sets `data-panel-tab`, nothing matches, and
both halves show — which is the group as it has always been.

**The trap from D-080 is intact and now has a verdict of its own.**
`builder-inspector.css` hides the plain editor's move and remove controls inside the panel,
and that is what makes the branch in `PageEditorController::again()` safe. The rules added
here hang off `.block-body`, and those controls are its sibling, so they stay hidden — and a
browser check asserts it on both tabs rather than leaving it to be noticed later.

**Which tab is open is remembered for the session, not per block.** Somebody adjusting how a
page looks moves from block to block doing the same thing, and being thrown back to Content
on every selection would undo that.

**Two things the screen found that the measurement had called fine.**

- **The Section tab was EMPTY.** The style is a `<details>` because the plain editor folds
  it; behind a tab its summary is hidden, and a closed `<details>` with no summary shows
  nothing. The probe said otherwise — `getBoundingClientRect()` on a field inside the closed
  `<details>` reported a box 40px tall at a plausible y, because the browser lays out what it
  does not paint. **The screenshot was right and the measurement was wrong.** Every check
  here now asserts the height of the `<details>` itself, and the scenario says why.
- **The six controls had no space between them**, every hint touching the next field's label.
  `.block-style-grid` takes its `display: grid` and `gap` from `admin-pages.css`, which the
  builder does not load, so only `grid-template-columns` was arriving and the grid it
  templates never existed. Exactly the same fault, and the same fix, as `.block-body` above
  it in the same file — which did not show while the style was folded at the foot of a long
  scroll, and did the moment it had a tab of its own.

**Left deliberately, for the owner:** every section field shows its hint, and the hints are
as tall as the controls. D-078 made hints something you ask for on the Appearance screen; the
same could be true here. It is his to say, and it is not free — the hints are the only thing
explaining what Rhythm or Top edge mean.

**A seam to watch:** `builder.js` is at 304 lines, four past the guidance and well under the
limit. The real seam when it comes is the device-width controls, which have nothing to do
with selection, the panel's modes or the canvas conversation.

### D-085: The block's controls move off its text, and keep the keyboard

**Status:** 2026-09-22. Sixth slice of D-080, and the first half of the "small costs" it
listed. Every claim here was measured three times before it was believed, because the first
two measurements were of the wrong thing.

**Where the bar sits.** It was drawn 12px inside the block's top, over its first line. The
review said it "sits over content on a full-bleed section and collides with the insertion
control on the first block". Measured:

- Against the block's `.container`, everything overlapped — a container spans the whole
  block, so that measurement could only ever say yes.
- Against the elements holding the text, most blocks overlapped — an `h2`'s box runs the
  full width even when its words do not.
- Against **the line boxes of the text itself**, through `Range.getClientRects()`: the bar
  covered real words on **2 of the demo page's 7 blocks**, the ones whose heading reaches
  the right edge. And it never touched an insertion control, on any block. So half the
  review's claim was true and narrower than stated, and half was not true at all.

The bar now straddles the block's top edge, in the gap between blocks, at the right where
the centred insertion controls are not. The first block had no gap above it, so the canvas
gained `padding-top` — the mirror of the `padding-bottom: 4rem` that has been there since
the beginning for exactly the same reason, the last insertion control. Measured afterwards:
no words covered on any of the seven, no insertion control touched, nothing clipped.

**Who has the keyboard.** Pressing a control rebuilt the bar and left focus on the document
body, so moving a block twice needed the mouse twice. Two separate causes, and fixing only
the first changed nothing, which is how the second was found:

- `drawInserts()` empties the whole overlay, so by the time `drawTools()` looked for the old
  bar it was already gone. Reading which button had focus has to happen before the clearing,
  which is why it is a function of its own rather than two lines inside `drawTools()`.
- **The parent was reaching into the canvas and taking the keyboard out of it.**
  `api.show()` focuses the first field of the selected block — right when a block is chosen
  from the library, wrong when the selection is a consequence of something done in the
  canvas. It is the same call that made ⌘Z after a duplicate do nothing in D-079. When focus
  is inside the canvas the parent's `activeElement` IS the iframe, so the rule costs one
  comparison: **the editor never takes the keyboard away from where the person is working.**

**So there is no new shortcut for reordering.** The plan asked for arrow keys on the selected
section; with focus kept, the button IS the shortcut, and pressing it twice moves a block two
places without the mouse. A browser verdict presses Enter twice and reads the order.

**A test split rather than adjusted.** `23-block-tools` asserted "four controls on its top
right corner" — what the controls ARE and where they SIT, in one verdict. The second rule
changed deliberately, so the case is now two: the controls, and the position — where the
position is checked by what it is FOR, that no word of the block is underneath it.

**Measured and left for the next slice:** the section style is at the bottom of the panel's
scroll, past every content field. With the panel's first field on screen at y=287, the first
section-style control sits at y=1308 on a Hero, 1603 on an Image and text, and **4296 on a
Columns block** — four screens down, past twelve repeater items — in a window 1000px tall.
The thing most often changed while looking at the page is the hardest thing in the editor to
reach. That is the Content/Section split, with the trap named in D-080.

### D-084: The icon every block declared, and a line saying what it is for

**Status:** 2026-09-22. Fifth slice of D-080.

**`icon` was wiring, not design.** Every block definition has carried one since the first
block, `BlockDefinition` validates it, SPEC §5.3 requires it — and a grep through `app/`
found nothing that draws it. Worse, the five names (`hero`, `text`, `image-text`, `columns`,
`form`) were **none of them in the sprite**, so drawing them would have produced five empty
squares. Nobody could notice while nothing drew them.

The definitions now name Lucide icons — `panel-top`, `type`, `image`, `columns-3`,
`clipboard-list` — and `tools/icons/build.php` carries them, which is a line in a list and a
command, not a new dependency or a build step at install time. Renaming the definitions was
chosen over drawing icons of our own so the sprite stays the one source: a block added later
names a Lucide icon and the maintainer runs the builder.

**A test now stands where a comment stood.** `tools/icons/build.php` ends with *"a name that
is not in the sprite draws nothing, so check the screen"*, which is a rule nobody remembers
on the day they add a block. The sprite is a committed file, so a test reads it and refuses a
name it does not carry; the browser scenario measures that the drawn icon has a size, because
a `<use>` at a missing symbol renders an empty box of zero pixels and no error at all.

**One line under the name, saying what the block is for.** The picture shows the shape and
the name labels it; neither answers the question somebody scrolling a library is actually
asking. `block.{type}.summary` in `lang/en/pages.php`, written to finish "use this for…",
kept to one line because a card that needs a paragraph is a block that needs a better name.
A second test refuses a block with no summary — `t()` answers with the key when lang has no
string, so without it a card reads `block.hero.summary` on the screen — and refuses two
blocks sharing one.

**And the gap that let D-083's literal colour through.** `blocks_test` already forbids a
literal colour in a front-end stylesheet, and it missed `stroke='%23000'` because a data URI
percent-encodes the `#`. The rule was never "no `#` character", it was "no colour of its
own", so the guard now rejects `%23` followed by hex as well. Checked by putting the old
value back and watching it fail.

### D-083: The library shows the block, not a grey slab

**Status:** 2026-09-22. Fourth slice of D-080. Three separate lies on one panel, all fixed
without giving up the rule that a preview is RENDERED from the block and never drawn by hand.

**The card was a fixed window onto a block of any height.** `.library-frame` was
`aspect-ratio: 16 / 9` with the iframe at 400% scaled to a quarter. Measured on the demo
site — frame 397×223, iframe viewport 1588×893 — the blocks drew 201px (Text), 222 (Form),
342 (Columns), 459 (Hero) and 501 (Image and text). So three quarters of the Text card was
the empty document under the block, every card was the same size whatever it held, and the
one thing a picture of a block can say that its name cannot — how much room it takes — was
the one thing it could not say. **The card is now as tall as the block**, measured in the
browser and remembered in `localStorage` by the preview's file name, which is hashed against
the block and the stylesheet and so cannot go stale.

- **`documentElement.scrollHeight` was the obvious reading and it is useless here**: the body
  fills the viewport, so it answers 893 for every block on the list. What has a height is the
  section the block rendered into. The plan said to use `scrollHeight`; measuring it is what
  found otherwise.
- **The scale is 0.4, not 0.25.** At a quarter the whole library was legible only as shapes.
  Both were rendered and looked at.
- **The fade at the foot is drawn only where something is cut off.** Unconditionally it
  covered most of a short card — the Text card is 3.5rem tall and the fade was 2.5rem of it —
  so five cards that fitted perfectly well all looked like they were dissolving.
- **An artefact that was mine, not the product's:** changing `--library-scale` from the probe
  after the measurement left a grey band under the taller cards, because the block reflows at
  a different viewport width. Setting it in the stylesheet and rendering again removed it.
  CLAUDE.md's rule held: fix the instrument before judging the subject.

**Every card said the same words.** `sampleFields()` mapped every text field to
`t('preview.heading')`. **A field may now declare `sample`, a language key** (SPEC §5.3, a
deliberate change to a frozen contract), and without one the generic sample still stands — so
a block added later has a preview for nothing, which was the point of generating them.

**The Form card showed no form**, under a hint that promises "the block as this site renders
it". A form block draws nothing when its form is missing, and a preview has no database to
take one from. A `form` field now samples to an id that `BlockPreview` resolves to a form of
three sample questions, shaped exactly as `FormBlocks` resolves a real one. **The special case
belongs to the field type, not to the block** — the `media` field above it already has one —
so any block that takes a form gets this. The card went from 87px to 224px and is the first
one that has ever shown what a Form block is.

**A flat rectangle reads as damage; a rectangle with a picture in it reads as a picture
area.** `.media-placeholder` gains a glyph drawn as a MASK filled from
`--section-placeholder-edge`, so it takes the colour of whatever surface it lands on and no
literal colour enters a front-end stylesheet. One declaration serves the page, the canvas and
the preview, because all three render the same block through the same class. **This is visible
to visitors** on a published page whose picture area is empty — it replaces a grey slab, and
the owner should say if he would rather have nothing there at all.

**The contrast guard was widened, deliberately, and measured afterwards.** The card's fade is
`linear-gradient(to bottom, transparent, var(--ui-panel))` — every colour in it an admin token
— and `tests/contrast_test.php` refused it, because the rule had been written for a bare token
or a `color-mix()` and had never met a gradient. It now applies the same arithmetic to any
value: it must name at least one `--ui-` token, and every colour reference in it must be one.
Requiring a token is what stops the widening letting through a value that names no colour at
all. Checked by breaking it four ways rather than by reading it — a bare literal, a literal
inside a gradient, a SITE token inside a gradient, and a bare `url()` — all four still fail.

**And a literal colour I had written into a front-end stylesheet.** The placeholder glyph was
a stroked SVG, which meant `stroke='%23000'` in `sections.css`. In a mask the colour cannot
reach the screen, which is exactly the kind of reasoning the rule exists to make unnecessary,
so the glyph is drawn with filled shapes that name no colour and let SVG fill them black by
itself. No test caught it; CLAUDE.md did.

**Two tests changed deliberately** rather than being adjusted to new output: `blocks_test`
asserted the exact normalised field shape, which gained `sample`; `columns_test` asserted
three sample columns through the generic string. What each test is for is unchanged, and the
second gained a companion that asserts the new rule — that two blocks do not say the same
thing, and that a field declaring nothing still gets a sample.

### D-082: Stable block keys are deferred, because the defect they were for does not exist

**Status:** 2026-09-22. Third slice of D-080, stopped before it was written.

The slice was to replace the positional index in field names with a stable key, `b{id}` for
a saved block and `n{n}` for a new one, and it carried a deliberate change to the frozen
`docs/SPEC.md` §5.3. The reason given was: *reorder the blocks, let the save fail validation,
and the errors follow positions rather than blocks.*

**That was reasoned, not measured, and it is wrong.** Driven through the real save path —
three blocks named ALPHA, BETA and GAMMA, submitted in the order GAMMA, ALPHA, BETA with
ALPHA's required body emptied — the 422 comes back with the message on ALPHA:

```
  group 0: GAMMA
  group 1: ALPHA  ERROR
  group 2: BETA
```

It is right because both halves speak the same language: `BlockForm::parse()` keys an error by
the position in the SUBMITTED order, and the rejected save re-renders the SUBMITTED blocks in
that same order. A position is only ambiguous when one side means the stored order and the
other means the submitted one, and nothing here does.

**So the contract is not changed.** CLAUDE.md permits changing SPEC §5 before v0.1 when it is
deliberate and recorded; it does not make it free. Renaming the index touches SPEC §5.3, four
`action` regexes in `PageEditorController`, the renumbering in `builder.js`, `admin.js` and
`repeater.js`, three views and every test that asserts `blocks[0][body]` — a large change
against a defect that turned out to be imaginary.

**What survives as a real, smaller argument, recorded as O-25 rather than acted on:** with
stable keys nothing would ever need renumbering, and the three mutually load-bearing renumber
regexes would go. That is a simplification, not a fix, and it is worth doing the day something
else needs it — a tree, or a second editor — and not before.

**The lesson is CLAUDE.md's own, and it cost a slice's worth of plan:** a claim is measured,
not reasoned. Writing the plan I checked the *cost* of the change against the code carefully
and took the *reason* for it on trust from the review.

### D-081: Only the blocks that changed send their fields

**Status:** 2026-09-22. Second slice of the page-editor work (D-080).

Every block of every page went into every save. One Columns block is around sixty fields,
so ten blocks pass PHP's default `max_input_vars` of 1000. `_end` already catches that
rather than letting PHP silently drop half the page — but the refusal arrives after an hour
of work, and *"your page is too big to save"* is not an answer.

**A block that did not change sends its id and the marker `_unchanged`, and nothing else.**
The server restores its content, style and layout from storage, so what `BlockForm::parse()`
returns is the same block it would have returned had every field arrived. Nothing downstream
— the canvas, a rejected save, the write — has to know which blocks did that. The block's
PLACE still comes from where its skeleton sits in the request, so reordering costs no fields
at all. Measured on the demo page: seven blocks, **115 block fields whole, 14 as skeletons**;
with one block edited, 27. A skeleton is two fields whatever the block is, so what an
untouched page costs stops depending on how big its blocks are.

**Which blocks changed is measured, not inferred from events.** The first design marked a
group dirty on `input`, `change` and `click` inside it. That is a guess about which gestures
mean "edited", and every gesture it fails to think of loses work silently — the rich text
toolbar writes its hidden input directly and fires nothing, and a repeater item can be
dragged. So each group is fingerprinted from the values it would submit, once when the page
loads and again when it is saved. A group whose fingerprint is unchanged cannot have changed,
because the fingerprint *is* what the submit would carry. The fingerprint counts named fields
only: raising a TipTap editor moves the name off the textarea onto a hidden input beside it,
and a fingerprint that counted every field would report every rich text block as edited the
moment the editor loaded.

**The error has a direction: send too much rather than too little.** A block with no id, a
group that appeared after the baseline was taken, anything uncertain — submits whole.

**The trap, found by asking what the screen means rather than by a failing test.** A save that
fails validation re-renders the SUBMITTED blocks, valid edits included, because a save is
refused whole. A baseline taken from that screen would call those blocks unchanged, and the
server would restore them from storage: the author fixes the one error, saves, and their other
block rolls back silently. So the server says whether the field groups are the stored page —
`data-blocks-stored` on the form — and without it `builder-save.js` stands down entirely and
the form submits as it always did. **The default is the unsafe answer's opposite:** `shell()`
takes `$fromStorage = false`, only `edit()` passes true, and anything added later that
re-renders submitted blocks is safe without knowing any of this exists. A test holds it open,
and it fails when the attribute is emitted unconditionally.

**The fields are disabled rather than removed.** A disabled field is not submitted, and if
anything stops the submit the form still holds every value. The id input is kept rather than
rebuilt, so the skeleton is addressed by the very name the rest of the group was using — the
index `renumber()` last wrote — instead of one counted again and able to disagree with it.

**A skeleton naming a block that is not this page's adds nothing**, rather than an empty
block: without the id there is nothing to restore it from, and an empty block here would be
content the author never wrote.

`_end` and the field count stay exactly where they are. They guard the wall; this moves the
wall further away. Without the script the form submits whole, as before, and so does the
fallback editor.

### D-080: The page editor redesign — what is in scope, and what the tree costs

**Status:** 2026-09-22. The owner's framing: *"zadnji veliki posao na cms-u"*. The ground was
a review written against the real code and an interactive prototype of the editor, both
brought by the owner. Neither is in the repository, and by his decision that is the rule for
all such material: working documents are how a decision was reached, and what survives of them
belongs here and in `docs/SPEC.md`, in our own words. `.gitignore` says so.

**The review's §1 holds and is not touched:** the canvas is the real page in an iframe, the
server owns what a block is and returns HTML rather than a schema, one form and one save path,
`_end` catches `max_input_vars`, `pending_canvas` keeps a refused save.

**Its §3 — a tree of sections with columns — contradicts `docs/SPEC.md` §5.3, which is a
frozen contract** and which refuses it in as many words: *"A page stays a flat, ordered list of
blocks: no zones, no columns, no nesting … It would also multiply every later feature
(translation, revisions, caching) by the nesting depth."* That reasoning was checked against
the code rather than taken on trust, and it is accurate: `Translations.php` copies blocks
across locales by `sort`, `TranslationStatus` pairs them by `block_group_id`,
`Composition::apply()` rewrites `style_json` across blocks in bulk, `MediaLibrary` finds a
picture's use with `style_json LIKE '%"image":N%'`, and the `sizes` strings in templates
(`columns/template.php`) assume a block is as wide as its container. The review mentions none
of them.

**The owner's decisions (2026-09-22):**

- **Scope now is everything except the tree.** The tree is deferred, not refused, and the
  reason is recorded above so it is argued from the joins that exist rather than from taste.
- **When sections gain columns, the `columns` block is retired.** Recorded now, executed then.
- **New blocks come after the structural work.** The eight the review proposes, and with them
  the library's groups and filter — O-15 already says a category filter over five items is
  furniture.

**The slices, in order, and why that order.** Undo first (D-079), because it makes every later
slice safe to try. Then the form wall — skeleton always submitted, content only from blocks
that changed, erring towards over-submitting, because losing a changed block is losing work
while re-sending an unchanged one is only waste. Then stable block keys `b{id}` / `n{n}` in
place of positions, so a validation error after a reorder follows the block instead of the
slot. Then the library's previews and cards, which are what is seen first. Then the small
costs — the toolbar's position, focus surviving a redraw, keyboard reordering, splitting
Content from Section in the panel. Revisions last; `page_revisions` is already in SPEC §5.2.

**Two SPEC §5.3 changes are planned and deliberate** (CLAUDE.md permits this before v0.1, said
out loud and recorded): the index in field names becomes a key rather than a position, and a
block definition gains an optional per-field `sample`. Neither touches the schema or the flat
list.

**A trap for whoever splits the panel:** `builder-inspector.css:170-173` hides the block
controls inside the panel, and that is precisely what makes the branch in
`PageEditorController::again()` safe — without it one press of Add on an item would replace the
canvas with the plain form. A CSS rule holding a controller upright is worth knowing about
before the panel is rearranged.

**`pending_canvas` was never written down** although it is one of the better things in the
editor: a save that fails validation keeps the submitted blocks in the session, read once, so
the canvas that comes back is what the author had rather than what the database still holds.

### Lessons from the browser checks (2026-09-16)

- **Trix and the admin CSP.** Trix injects a stylesheet at runtime, and the admin's
  Content Security Policy refuses it. Every rule it injects has to live in
  `admin-richtext.css`. The policy is never loosened for it.
- **A commit message is a claim.** One commit described a change its code did not contain.
  Before committing, the executor checks each claim in the message against the staged diff.
- **Check what the owner will do, not only what is stored.** The heading menu passed
  every storage check and failed the owner's first try: select text, change the level.
  Browser acceptance for anything interactive includes the owner's real actions.
- **Green means CI, not the local suite.** From 4a (`da428df`) to `b4818c2`, twelve commits
  were deployed to the demo while GitHub CI failed on every one, because this server has
  image extensions CI lacked. Nobody was reading CI in that stretch, the architect
  included. A deploy now requires that commit's CI conclusion to be "success", checked
  and reported by the executor and checked again by the architect.
- **The live demo's design is never changed by a script.** Screenshots under several
  characters are taken on the copy. Applying a preset rewrites the site's design, and with
  "reset section styles" it would rewrite every section too; a restore afterwards is not a
  safety net, because nothing recorded what was there first.
- **Checks run on a copy.** `env()` reads `$_ENV` and `$_SERVER`, and this PHP's
  `variables_order` leaves `$_ENV` empty. Browser checks therefore use a copied site tree
  with its own `.env` (`~/boxlet-browser/site`), never environment overrides, so nothing can
  reach `boxletcms`. The driver lives in `~/boxlet-browser`.

---

## 5. Open items

*O-1 and O-2 resolved by D-019 and D-020. O-22 and O-24 resolved by D-077. O-15 resolved by
D-104. O-25 resolved by D-094.*

**O-28. The demo cannot show a section with columns.** `DemoSite::seed()` writes one band
per block — the page data in `app/Modules/Demo/pages.php` is a flat list of
`[type, content, style, layout]` with nowhere to say which band a block stands in. So the
columns, the arrangements and the stacking of D-100 to D-104 appear nowhere a visitor or the
owner can see, and `tests/demo_test.php` — which is what keeps the demo a complete visual
fixture — cannot require them, because it can only check what the seed can express. Fixing
it means giving the seed a band shape, which is a change to the demo's data format and
deserves its own slice rather than being smuggled into one.

**O-27. `12-picker` chooses a picture that never arrives.** After the scenario clears the
control and reopens the panel, `pick()` clicks the first card and nothing is chosen: the
control still reads *"No picture"*, and the next `openPicker()` then TOGGLES the still-open
panel shut and times out. Two failures, one cause.

Measured, not guessed: it fails identically at `5c4952a`, so it is not D-097's doing, and
the same sequence driven by hand outside the scenario — open, Escape, open, click card 0 —
works, ending with `value=7`, the name `room-light` and the panel closed. So the defect is
in what the scenario does AROUND the choice, not in the picker. `controlsOnPanels()` is
read-only and cleared; the remaining suspects are `report.shot()`, which is `fullPage: true`
and resizes the page under the open panel, and the clearing click before it. Found
2026-09-22 by running the WHOLE browser suite rather than the scenarios a change looks like
it touches — which is also how the four D-094 breakages below came to light. Not fixed in
D-097 because a scenario's own weakness is not the slice's (O-26 says the same).

**O-26. `03-design` depends on a starting design it does not set.** Run against a copy that
earlier runs have left on a dark character, *"a page colour set by hand carries the palette
with it"* fails: the page is set to `#0d0d10` and the text it works out stays `#161422`,
because it was already dark. On a freshly installed copy the same check passes with
`#eeecff`. Nothing is wrong with the code — the scenario asserts a CHANGE without owning the
state it changes from, so a real defect and a stale copy look identical. It should set the
character it needs at the top, the way `applyCharacter()` already lets it. Found 2026-09-22
while checking D-087; not fixed there because a scenario's own weakness is not the slice's.

**O-25. Stable block keys in place of positions** (D-082). Nothing needs them: the defect they
were proposed for was measured and does not exist. What remains is that `builder.js`,
`admin.js` and `repeater.js` each renumber `blocks[n]` names with their own regex, and that
those three must agree. Stable keys would delete all three. Worth doing when something else
needs it — a section tree, or a second editor — and not for its own sake.

**O-21. Live links inside the design preview** (D-057). Now that the preview draws the real
header, the iframe contains a menu whose links WORK: clicking one navigates the frame to that
page, out of the preview and into the site. Block links could already do this; a menu makes it
likely. The CSP is not the lever — `form-action 'none'` covers forms, not navigation — so this
needs either a `<base target="_blank">`, intercepting clicks in the frame, or accepting it and
giving the frame a way back. Decided when the merged Appearance screen is built, not patched
now.

**O-20. Statistics, round 2** (D-051): done 2026-09-20.
Built: narrowing by clicking with the state in the address and a range of your own, counting
the addresses that are not there, the world map, the visitor's own address behind a proxy,
gathering the rows of one or two visitors, and export.

*Left, and why* — and both reasons were corrected on 2026-09-20, when the owner asked:

**The footer credit is built** (2026-09-20). Two different things were hiding behind one
phrase, and the owner's question separated them:

- *What the specification meant* by "atribucija u footeru" was **DB-IP's** credit, optionally
  on the public site. **Not built, and not needed.** CC BY 4.0 asks for attribution where
  the material is used, and DB-IP's data never reaches a visitor — only the admin is ever
  shown a country or a city. It is credited where it is seen: at the foot of the Statistics
  screen. Crediting a database in a client's public footer for something their visitors
  never see is noise.
- *What the owner wanted*, once the two were separated: a line saying what made the site.
  Built as `site_credit`, a switch in Settings → General, **off unless asked for**. It draws
  one quiet line in the site's own small print — "Made with Boxlet", in the page's language,
  linking to **boxlet.org**, which the owner named as the project's address. A site whose
  footer holds nothing else still draws one for it, or the switch would be on and the line
  nowhere.

The address lives in `BOXLET_SITE` in app/Support/helpers.php, beside ADMIN_LANG, rather
than in config/: every install credits the same project, a credit a site could point
anywhere is not a credit, and config/ is the part of a Boxlet its owner may have edited —
so it is the part an update cannot safely replace. The browser copy proved that the hard
way within the hour: it does not sync config/, and the line silently did not appear.

**Putting counts in is gone, and Plausible's format will not be built** (the owner,
2026-09-20, asked which was needed and answered his own question: neither). The import that
round 2 built has been taken out — the route, the action, the model, the screen's field and
its words.

The reason is better than the one it replaces. A file of numbers that ADDS to what a site
has counted is a way to make a site's statistics say something that never happened, and
nobody had asked for it. What is left is a promise worth more: **nothing but a visit ever
writes a count.** Taking counts OUT stays — a CSV of the table on screen, and one JSON file
of everything, for keeping a copy or reading in a spreadsheet.

For the record, since it was recorded wrongly before: Plausible's format never needed PHP's
zip extension. Measured — zip is on this server and on most shared hosting, it is optional
in PHP, and it does not matter: a ZIP is a local header, the data, a central directory and
an end record, and `crc32()` and `gzdeflate()` are zlib, which is everywhere.

**O-17. Downloads: documents and archives in Media.** The owner wants to offer visitors
files to download (PDF, ZIP, TAR and similar) from the same library, which is why it is
called "Media" rather than "Pictures". Not built now. To decide when it is scheduled:
which types are allowed (a whitelist such as pdf, zip, tar, gz, docx, xlsx, pptx, odt, ods,
csv, txt; never html, svg, or anything executable); where the files live and how they are
served (public under a safe generated name with `nosniff` and a download disposition, or
kept outside the web root and served through PHP, which also allows counting downloads
later); size limits; and how a download is placed on a page (a link from rich text, a
link field, or a small "file" block). *After Slice 5, before release.*

*O-4 resolved by D-050.*

*O-6 resolved by D-045.*

*O-7, O-8 and most of O-9 resolved by D-028; analytics remains open there.*

**O-18. WebP quality is ignored on this Imagick.** AVIF's half is resolved (2026-09-19):
the AVIF writer on ImageMagick 6.9.12-98 reads the wand's quality, not the image's, and
MediaWriter now sets both, so the one-retry size guard works on Imagick hosts too (and its
threshold is §8's 200 KB, down from 250). WebP still comes out byte-identical at 82 and 75
through both setters. Browsers that take AVIF never see the WebP, so this matters only to
the few that do not. Pictures uploaded before the fix keep their heavier AVIF until
variants can be regenerated (O-13). *Before release.*

*O-19 resolved by D-044.*

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

*O-12 resolved by D-043.*

*O-13 resolved by D-048.*

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
