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
| Tests | 668 on both drivers, PHPStan clean at level 8 (2026-09-18) |
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
   - Left: the Slice 5 acceptance check from SPEC §8 (a ~4 MB photograph, served under
     200 KB, second request served without PHP).
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
     done (6a, `e141816`); it adds no browser scenario until Columns uses it. See O-11.
7. **Slice 6, languages**, including adding a language from the admin. See O-12.
8. **Slice 7, forms and mail:** form builder, `{{form:slug}}`, submissions, SMTP and
   Resend, admin notification, autoreply, honeypot, test-mail button. See O-6.
9. **Slice 8, operations:** page cache, backup, update by ZIP upload, revisions,
   sitemap, regenerating media variants (O-13), and 2FA (O-4).
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

*O-1 and O-2 resolved by D-019 and D-020.*

**O-17. Downloads: documents and archives in Media.** The owner wants to offer visitors
files to download (PDF, ZIP, TAR and similar) from the same library, which is why it is
called "Media" rather than "Pictures". Not built now. To decide when it is scheduled:
which types are allowed (a whitelist such as pdf, zip, tar, gz, docx, xlsx, pptx, odt, ods,
csv, txt; never html, svg, or anything executable); where the files live and how they are
served (public under a safe generated name with `nosniff` and a download disposition, or
kept outside the web root and served through PHP, which also allows counting downloads
later); size limits; and how a download is placed on a page (a link from rich text, a
link field, or a small "file" block). *After Slice 5, before release.*

**O-4. 2FA.** SPEC §6 describes it: optional, ten single-use recovery codes, and a reset by
placing `storage/disable-2fa` on the server over FTP. To confirm: it stays optional rather
than forced (a single admin with forced 2FA and a lost phone means a lost site).
*Step 9.*

**O-6. Resend transport.** Slice 7 promises Resend, but no Resend package is on the closed
dependency list. Options: Resend's SMTP endpoint through symfony/mailer, or its HTTP API
called directly. *Step 8.*

*O-7, O-8 and most of O-9 resolved by D-028; analytics remains open there.*

**O-18. AVIF quality is ignored on this Imagick.** Measured on ImageMagick 6.9.12-98: AVIF
and WebP come out byte-identical at quality 10, 40 and 82, while JPEG honours the number, so
the setting reaches the encoder and the delegate discards it. Every AVIF made here and on CI
is therefore at the delegate's own default, and the one-retry size guard can only help where
GD does the encoding. Options when this is picked up: encode AVIF through GD when it is
available, look for a build or delegate that honours quality, or accept the default and set
the size budget from measurement. Decide it with real hosts in view, not this one machine.
*Before release.*

**O-19. Front-end text has no translation mechanism.** `t()` is the admin's. Visitor-facing
strings are written by the site owner, except for the few the product itself supplies — the
404 page, the language switcher's label, and now chrome (small print, button labels). Today
each is a small per-locale list in the code. Slice 6 should decide whether that becomes a
mechanism, and where a site owner overrides it. *Slice 6.*

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
defaulting to hidden from navigation and a 404 on a direct hit. Also for Slice 6: the
fallback chain between locales, including what populates `locales.fallback` (nothing does
today) and how it applies to alt text. Until then a picture with no alt text in the page's
language renders with an empty alt, which for a picture that carries meaning is worse than
the source language's alt. *Step 7.*

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
