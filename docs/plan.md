# Boxlet — state of play

Where the work stands, what is half-finished, and what comes next. `docs/SPEC.md` is the
contract; this file is the progress report against it. Written at the end of Slice 5's
first part, with a substantial amount of work uncommitted.

**Read this together with `CLAUDE.md`** (rules that must not be rediscovered) and
`docs/SPEC.md` §8 (the build order) and §9 (deferred questions).

---

## 1. Current state

| | |
| --- | --- |
| Last commit | `eae20c6` — Slice 5 (1/n): editor fixes |
| Tests | 259 passing, both drivers (SQLite + MySQL), 23 test files |
| PHPStan | clean at level 8 |
| CI | green on 8.1, 8.2, 8.3, 8.4 as of `eae20c6` |
| Live site | https://boxlet.svejedobro.hr, MySQL, demo site installed, **Brutalist** character active |
| Admin login | `acceptance@example.com` — password rotated in Slice 5; it is in the session log, not in the repository |

Slices 1 through 4.6 are committed and done. Slice 5 is in progress.

---

## 2. Uncommitted work

This is the part most easily lost. Two independent threads are in the working tree.

### Thread A — rich text (Trix)

| File | State |
| --- | --- |
| `public/assets/vendor/trix.umd.min.js`, `trix.css` | vendored, v2.1.19, MIT, provenance headers |
| `app/Support/RichText.php` | `div→p`, `h1→h2`, `h4–h6→h3`, attachment stripping |
| `tests/richtext_test.php` | cases pinned to captured Trix output + idempotence |
| `app/Modules/Pages/views/admin/block.php` | Trix toolbar + textarea, restricted buttons |
| `public/assets/richtext.js` | editor setup, plain toggle, Ctrl+Shift+V, attachment refusal |
| `public/assets/admin-richtext.css` | Trix restyled in admin tokens |
| `app/Modules/Admin/AdminView.php`, `views/layout.php` | per-screen `scripts` hook |
| `PageBuilderController`, `PageEditorController` | load Trix on both editors |
| `public/assets/builder-blocks.js` | rescan inserted blocks |

### Thread B — media (foundations only)

| File | State |
| --- | --- |
| `migrations/0011_media.sql`, `0012_media_meta.sql` | tables per SPEC §5.2, migrate on both drivers |
| `app/Modules/Media/MediaPresets.php` | five presets + focal-point crop geometry |
| `tests/media_presets_test.php` | geometry pinned, including no-upscale and clamping |
| `public/uploads/.htaccess`, `public/m/.gitkeep`, `.gitignore` | upload guard, variant directory, ignores |
| `app/Modules/Install/Requirements.php`, `lang/en.php` | upload limit reporting |

### Also uncommitted

- `public/assets/admin-ui.css` — the invisible-control fix (`.admin a.button` outranked
  `.button-ghost`, so ghost buttons were white on white).
- `app/Modules/Pages/views/admin/builder.php` — Preview opens in a new tab.
- `CLAUDE.md` — live-database rule, visible-control rule, rich text and media contracts,
  the working-practice section, a pointer to this file.
- `docs/SPEC.md` — Trix in §3; the richtext editing contract in §5.3; "no free colour per
  section" in §5.4; three §9 entries (site settings and SEO, regenerating media variants,
  repeater fields); the changelog entry recording the Trix reasoning.
- `README.md` — Trix in the vendored assets table.
- `docs/plan.md` — this file.

---

## 3. Rich text — decided, mostly built, not finished

**Decision: Trix.** Superseded an earlier recommendation of Wysi. The correction that
matters: an editor's *internal document model* is not lock-in, only its *storage format*
is. Quill is excluded because its ecosystem stores Delta. Trix stores HTML. Wysi
satisfied replaceability but has a very small user base, and a rich-text editor lives on
edge cases — paste from Word, nested lists, undo, IME, mobile — that only a large user
base surfaces. Replaceability protects content; it does nothing for experience.

Wysi remains the fallback if Trix proves unworkable in this admin.
Ruled out and not to be revisited: Quill and Editor.js (proprietary storage format),
TinyMCE and CKEditor (licence model and weight), TipTap (build step), Summernote (jQuery).

### Verified by observation, not documentation

Trix was driven through keyboard and toolbar in a headless browser, served from
`127.0.0.1` because `navigator.clipboard` needs a trustworthy origin.

| Construct | Trix emits | Stored as |
| --- | --- | --- |
| paragraph | `<div>` | `<p>` |
| bold / italic | `<strong>` / `<em>` | unchanged |
| link (toolbar dialog) | `<a href>` | unchanged |
| heading | `<h1>` | `<h2>` |
| quote | `<blockquote>` | unchanged |
| lists, nested 3 deep | real `<ul><li><ul>` nesting | unchanged |
| strike, code | `<del>`, `<pre>` | **buttons removed** |
| attachments | `<figure data-trix-attachment>` | **disabled + stripped** |

**Against a control** — the same Word HTML pasted into a plain `contenteditable` keeps
`MsoNormal` classes, inline styles, `<font>` and a whole `<table>`. Trix reduces it to
`strong`, `em`, `a`, real lists and `div` blocks. Trix is dramatically better than the
browser's own behaviour, not worse.

### Verified in the running admin

Fallback editor (`/admin/pages/{id}/form`): Trix upgrades, toolbar carries exactly the
eleven permitted controls, `name` moves off the textarea onto a hidden input, typing
reaches the named field, plain toggle works both ways.

### NOT yet verified — pick up here

1. **§5d round-trip, the priority.** Load existing demo content into Trix, save without
   editing, confirm the stored HTML is byte-identical. PHP-level idempotence is already
   proven (`sanitize(sanitize(x)) === sanitize(x)`), but that only shows the sanitiser
   does not change its mind. It says nothing about whether Trix hands back what it was
   given. If they differ, report what changed rather than adjusting the test.
2. **Visual editor interaction.** The structural check passed; the interaction check was
   never completed because the probe selected the first `[data-richtext]`, which belongs
   to a hidden block group. Scope to the visible group.
3. **Editors built inside `display:none`.** Every unselected block group is hidden at page
   load, so `richtext.js` initialises Trix inside a hidden subtree. This is a known source
   of broken sizing and dead selection once revealed. Unchecked.
4. **`boxletRichText.scan`** on newly inserted blocks — written, never exercised.
5. **Ctrl+Shift+V** paste-as-plain-text — implemented, never verified.

### Documentation for rich text — done

`CLAUDE.md`, SPEC §3, SPEC §5.3, SPEC §5.4, the changelog and the README all record the
decision, the reasoning and the normalisation. Nothing is left to write; what remains is
the verification in the list above.

---

## 4. Media — decided model, almost nothing built

### The model (SPEC §5.1, §5.5, settled in Slice 4.5)

Variants are generated **on upload, never on demand**. A request for a variant is always a
request for a file that exists, so serving never touches PHP and needs no server rule the
user may be unable to add. On-demand generation does not work on this server at all:
nginx answers a request for a missing `.webp` from disk with its own 404 and PHP never
runs.

Presets: `thumb` 200×200, `card` 600×400, `wide` 1200×630, `hero` 1920×1080 (all crop),
`full` max 2400 wide, no crop. URL shape `/m/{preset}/{id}-{slug}.{ext}`.

### Measured on this server

GD 2.3.3 (JPEG/PNG/WebP/GIF, **no AVIF**); Imagick 6.9.12 (**AVIF, WebP, HEIC**); `exif`
and `fileinfo` present. No `cwebp`/`avifenc`/`convert` binaries.

On a synthetic 2400×1600 source: GD WebP 1200×630 **129 ms / 29 KB**; Imagick WebP
**261 ms / 21 KB**; Imagick AVIF **385 ms / 9 KB**; Imagick WebP 1920×1080 **455 ms**.
Imagick is therefore the primary encoder, GD the fallback, AVIF best-effort.

**Not yet measured, and required:** a real large photograph, and the GD-only path forced,
so the shipped figure is honest rather than extrapolated from one synthetic image.

### Built

`MediaPresets` — the five presets and the crop arithmetic: largest rectangle of the
preset's proportions, slid so the focal point is central then pushed inside the edges,
**never enlarging**. Because output size is not always the preset size, the crop function
returns the true dimensions, which `<picture>` needs to emit `width`/`height` and avoid
layout shift. Tested.

Migrations for `media` and `media_meta`, upload `.htaccess`, installer limit reporting.

### Not built — the whole pipeline

1. **Encoder** (`ImageFile`): open source, apply **EXIF orientation before cropping**,
   draw the crop, encode WebP/AVIF/original. Imagick preferred, GD fallback.
2. **Upload**: finfo MIME sniff *and* separate extension check, explicit rejection of
   anything PHP-adjacent, sha1 dedup, original stored untouched, readable failure when
   PHP truncates an oversized post.
3. **Resumable variant generation** — required by the brief, and the machinery Slice 8's
   regeneration becomes a thin wrapper around:
   - generate in priority order: `thumb`, `card` first (the library needs them), then
     `wide`, `hero`, `full`;
   - **record which variants exist** per item — needs a column on `media`
     (the migration is uncommitted, so amend it rather than adding another);
   - check remaining execution time before each encode and stop cleanly;
   - an incomplete item is marked as such, shown that way in the library, and offers to
     finish.
   Rationale: up to fifteen encodes per image on a slow shared host exceeds
   `max_execution_time`, PHP is killed mid-pipeline, and because nothing is generated on
   demand the missing variants 404 for ever.
4. **Library screen**: grid newest-first, search by filename, replace/delete/focal point,
   per-locale alt and caption. **Deletion blocked while in use**, naming the pages.
5. **Picker in the inspector**, replacing the raw numeric id field; also wired into
   section style so `surface: image` can choose an image.
6. **Front-end**: `<picture>` AVIF → WebP → original, `width`/`height` always, focal point
   honoured, alt from `media_meta` for the locale, empty alt for decorative images.
7. **Demo images**: generate with GD at seed time, or source CC0 photographs if approved.

**Explicitly not to be added:** a free background colour per block. The constrained set
(plain, tinted, contrast, image, gradient) is the answer; an open colour field would make
the contrast-checked palette decoration. Record in SPEC §5.4.

---

## 5. Also owed by Slice 5, not built

- **Per-page meta title and description** (`pages.seo_json`, in the schema since Slice 3
  with no interface, so every page currently renders with no `<title>` override and no
  meta description). Two fields in the page settings panel with character counts,
  defaulting to the page title, emitted in `<head>` beside the existing canonical link.
  Nothing else: no OG image, no schema, no robots, no sitemap.
- **A contrast sweep over every admin control**, so the visible-control rule is enforced
  by a check rather than by memory. Promised, not written.
- SPEC §8 marking Slice 5 done, changelog, README upload limits and the nginx caveat.

---

## 6. Deferred, recorded in SPEC §9

Blog/news content · nested page addresses · header, logo, navigation and footer ·
block library categories · **site settings and SEO** · **regenerating media variants**
(Slice 8) · **repeater fields and the block library** (its own slice, immediately after
Slice 5 — a team grid is one block with a repeating item, not a generic column container).

---

## 7. How to pick this up

1. Run `php tests/run.php` and `vendor/bin/phpstan analyse` — expect 259 green and clean.
2. Finish the rich-text verification in §3, **round-trip first**.
3. Commit rich text as its own part, with the reasoning in the changelog.
4. Then media, in the brief's order: upload and variants, library and picker, front-end
   rendering — committing each part separately.

The live site is a real installation with a demo site on it. Everything in `CLAUDE.md`
about not writing to its database by hand applies.
