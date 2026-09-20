# Handoff: the Appearance screen

A replacement for the Design screen (`/admin/design`) and the Header & footer screen
(`/admin/chrome`) in `zaja/Boxlet-CMS@main`: one preview-first screen, a writable preset
library, and a materially larger control surface — without giving up the thing that makes
the design layer worth having.

Read this file before the design reference. It was written after reading the real source
(`Design/views/design.php`, `Settings/views/chrome.php`, `Tokens.php`, `Presets.php`,
`Palette.php`, `TokenCompiler.php`, `DesignController.php`, `ChromeLook.php`,
`public/assets/design.js`, `migrations/`), so it names actual classes, columns and
constants. Where it and the prototype disagree, this file wins.

---

## 0. Why this change, stated plainly

The four-layer model (character → tokens → section style → layout) is the best idea in
Boxlet and nothing here touches it. Neither does anything here introduce a free value that
can produce an unreadable page. What changes is the **control surface and the feedback
loop**, which is where the owner's frustration actually lives:

1. **The owner cannot keep their own work.** Five characters can be *loaded*; nothing can be
   *saved*. An owner who spends an afternoon on colour and type has nowhere to put the
   result, and loading any character throws it away. This — not the number of controls — is
   the real source of "too few options". Twenty-four controls with no Save As is a dead end.
2. **Design and Header & footer are one screen split in two.** Both describe the appearance
   of one page. Chrome has seven choices and *no preview at all*; Design has a preview that
   deliberately renders **no chrome** (`DesignController::preview()` passes `headerHtml` and
   `footerHtml` as `''`, with a comment explaining why). Each screen is incomplete in exactly
   the way the other could fix.
3. **The preview lags the form.** `design.js` only refreshes on a 250 ms debounce *and* the
   screen still carries an "Update preview" button; the Save button sits at the bottom of a
   3000-px page, far from the thing being judged. People end up choosing by word ("airy",
   "slant") instead of by eye.
4. **Some decisions are too coarse to be useful.** `Tokens::CONTAINER` is four buckets.
   `boxed` is `no|yes` with the frame hard-coded to `spacing × 3` in `Tokens::page()`, so
   "boxed" has no look an owner can shape. The header cannot break out of a boxed sheet at
   all — structurally impossible today, because `--page-frame` pads the whole page.
   `footer_layout` is `simple|columns` with the column count fixed at three in the markup.
5. **CSS leaks onto the owner's screen.** `design.php` prints
   `implode(' · ', $derived['text'])` and `$derived['space']` — which renders
   `clamp(2.038rem, 1.508rem + 1.759vw, 2.827rem)` to someone who runs a bakery.

### What this does NOT change

Say this out loud in `PLAN.md`, because it is the part that is easy to lose:

- **No free colour per section, no per-block typeface.** The refusal in SPEC §5.4 stands.
- **Nothing is stored as a computed value.** The database keeps decisions;
  `Tokens::derive()` + `TokenCompiler` produce the stylesheet. A page saved before a
  decision existed still opens, with that decision at its default.
- **A seed is never silently nudged to pass.** `Palette`'s docblock is explicit about this
  and it is correct: a failing colour is *named*, not repaired.
- **Save is still the confirmation.** Loading a character changes nothing until saved, and
  applying a composition stays two explicit buttons.

---

## 1. The design reference

`Boxlet Appearance.dc.html` is a **prototype**, not production code. It uses a small
streaming-template runtime (`<sc-for>`, `<sc-if>`, `{{ }}`) with a `class Component` block
at the bottom holding all sample data and all derivation. Read it as markup plus a data
object; ignore the runtime.

Two things about it that are wrong for the repo on purpose:

- **Everything is inline `style=""`.** The admin sends `default-src 'self'` with no
  `'unsafe-inline'`, so not one of those attributes can ship. See §7 — the good news is the
  repo's architecture already solves this better than the prototype does.
- **The palette maths is an approximation in HSL.** The real thing is
  `Palette::colors()` in OKLCH. Do not port the prototype's `_hex`/`_hsl`/`_derived` — they
  exist only so the prototype can paint. Every number in the prototype's preview comes from
  a re-implementation; the server's own derivation is the truth.

The prototype's typefaces are also stand-ins (`Georgia` for Playfair, `Trebuchet MS` for
Nunito) because the sandbox blocks webfonts and the repo forbids a CDN. The real screen
reads `Typography::PAIRINGS` and the self-hosted faces via `Typography::fontFaces()`.

**Fidelity: high for layout, interaction and information architecture. Low for exact colour
values** (see above).

---

## 2. Screen anatomy

Three columns under one bar, `100vh`, nothing scrolls except the three columns
independently.

### 2.1 Top bar (46 px)

Left: title and a one-line subhead. Right, in order: **viewport switcher** (desktop 1280 /
tablet 834 / phone 390), **zoom** (Fit / 50% / 75% / 100%), **Compare**, a **state pill**,
**Revert**, **Publish**.

- The **state pill** has three states and they are ordered by severity:
  `Check contrast` (any pair below AA — amber, warning icon) → `Not published` (dirty) →
  `Published` (clean). Contrast outranks dirtiness because a failing design must not be
  publishable, and `Tokens::validate()` already refuses it.
- **Compare** swaps the preview to the currently-published design for as long as it is
  pressed. This is the cheapest possible answer to "is this actually better?" and the repo
  can serve it with the endpoints it already has (§6.4).
- **Publish** replaces the bottom-of-page Save. Keep the two-button composition choice
  (design only / design with composition) — surface it as a confirmation step on Publish
  when `Composition::hasBlocks()` is true and a character is loaded, exactly as
  `design.php` does today.

### 2.2 Left rail (196 px) — the preset library

Two lists.

**Characters** — the five built-ins from `Presets::ALL`, unchanged and not editable. Each
card: three swatches (accent / contrast / tint), the name, a live-badge when it is the
active composition (`Composition::active()`), and a one-line summary built from the
decisions themselves (`grotesk · 68rem · normal · full bleed`). Clicking loads it into the
workspace — it does not publish.

**Your designs** — saved presets. Each card has the same shape plus two icon buttons:
**overwrite** (save what is on screen into this preset) and **delete**. Below the lists, a
name field and **Save as a new design**.

This is the centre of gravity of the whole change. A character stops being a menu item and
becomes a starting point; the owner's own work becomes the thing they actually use.

### 2.3 Centre — the preview

A thin strip (hostname, page name, viewport, zoom, and a Compare indicator when active),
then the page itself: **header, hero, tinted text section, split section, footer**, rendered
at the chosen viewport width and scaled.

The preview is the largest element on the screen and has no Update button. That is the
point: the loop is change → see, with nothing in between.

Scaling rules that matter:

- `zoom` is floored at **0.5**. Below that the stage scrolls horizontally instead of
  shrinking further — a preview nobody can read is not a preview.
- Switching viewport returns zoom to Fit.
- If the column cannot carry the desktop width above the floor, the screen opens on Tablet
  or Phone instead. Measure the real container before painting: the prototype went through
  two rounds of a silently clipped preview because it trusted a hardcoded default width,
  so seed the width from the container itself and retry until the node exists.
- **The preview must reproduce the site’s own small-screen behaviour.** At ≤480 px the
  prototype swaps the wrapping menu for a menu button, hides the header CTA, and stacks the
  split hero and the footer columns. Without it the phone viewport shows a broken desktop
  header rather than the site, and judging `header_layout` on it is worthless. In the real
  screen this comes for free — the iframe loads the site’s own stylesheet at a real 390 px —
  which is one more reason not to fake the preview inside the admin document.

### 2.4 Right inspector (312 px) — five tabs

**Colour · Type · Shape · Page · Header**. Tabs rather than one long scroll, because the
current screen's 3375-px height is itself a defect. Every group is a row of segmented
buttons with a monospace readout of the resulting value on the right — the readout replaces
the `clamp()` text with the number a human wants (`Corners 12px`, `Air 20px`,
`Content width 56rem · 896px`).

---

## 3. The new decisions, one by one

The table below is the whole diff against today's model. **Storage** says where it lands in
`design_tokens` (layer 1, `Design::save`) or in the chrome settings
(`SiteChrome::saveLook`).

| New / changed | Today | Proposed | Storage |
| --- | --- | --- | --- |
| Per-role colour override | none — 15 roles all derived | any role may be set explicitly; the rest keep following the seed | layer 1, one JSON scalar per role |
| `base` type size | fixed 1rem | 13–21 px, 0.5 steps | layer 1 |
| `scale` | 6 named ratios | continuous 1.1–1.6 (named ratios remain as preset values) | layer 1 |
| Per-step nudges | none | `adjH1` −30…+40, `adjH2` −12…+20, `adjSm` −3…+5, `adjBody` | layer 1 |
| `heading_weight` | from the pairing | 400–800 override, empty = follow the pairing | layer 1 |
| `tracking` | from the pairing | tight / normal / wide override | layer 1 |
| `caps` | from the pairing | on / off override | layer 1 |
| `measure` | `CONTAINER` 4 buckets | 36–88 rem slider | layer 1 (**type change**, see §5.3) |
| `frame` | hard-coded `spacing × 3` | thin / narrow / normal / wide (1 / 2 / 3 / 5 units) | layer 1 |
| `sheet_radius` | none | square / soft / round | layer 1 |
| `sheet_shadow` | none | none / shadow / hairline | layer 1 |
| `header_bleed` | impossible | in the sheet / full width | layer 1 |
| `footer_bleed` | impossible | in the sheet / full width | layer 1 |
| `hero_layout` | composition only | left / centred / split, directly | layer 1 |
| `footer_columns` | fixed 3 in markup | 2 / 3 / 4 | chrome look |

Everything above is either a closed set or a bounded number. **No new decision can make
text unreadable that `Palette::failures()` would not catch** — with the single exception of
the colour overrides, which is precisely why the gauge in §4 must run on the final palette
and `Tokens::validate()` must keep refusing the save.

### 3.1 Colour overrides — the one genuinely risky addition

The model's guarantee comes from `Palette::colors()` deriving every role from one or two
seeds, and `Palette::failures()` checking eleven pairs. If an owner may set `text` and
`background` freely, the derivation no longer protects anyone — the *check* has to.

That is acceptable, and here is the precise reason: the check already exists, already runs
on save, already names the pair and the decision responsible, and already refuses. Nothing
in `failures()` cares whether a colour was derived or given. Overrides change which value
goes in; they do not weaken what comes out.

What must be true for this to be safe:

1. `Palette::colors()` gains a third input — the override map — and applies it **after**
   derivation, before returning. Signature, keeping the existing one working:
   ```php
   public static function colors(
       string $seed,
       string $secondary,
       string $surfaceContrast,
       array $overrides = [],   // role => #rrggbb
   ): array
   ```
2. **Derived roles that depend on an overridden one must be re-derived, not left stale.**
   `on-accent`, `on-contrast`, `muted-on-contrast`, `contrast-raised` and `on-gradient` are
   all computed *from* other roles by `readableOn()`. If the owner overrides `contrast`, the
   `on-contrast` that pairs with it has to be recomputed unless the owner also overrode it.
   Order: apply overrides to the eight base roles → recompute the dependent ones → apply any
   overrides to the dependent ones. Getting this order wrong is the most likely bug in the
   whole slice.
3. `Palette::failures()` runs unchanged on the final map. Attribute a failing pair to the
   **overridden role** when one is involved, so the error points at the control the owner
   just touched rather than at `surface_contrast`. That means `failures()` needs to know
   which roles were overridden — pass the override keys in, or return the role names and let
   `Tokens::validate()` map them.
4. Only override the roles the owner can see. The prototype exposes **eight**:
   `background, surface, tint, border, accent, contrast, text, muted`. Note `tint` does not
   exist in `Palette` today — the tinted surface is `surface` at a different OKLCH step.
   Either add `tint` as a real role or map the control onto `surface` + `SURFACE_STEPS`;
   the prototype assumes a real role, which is the cleaner answer.
5. `link` is currently `= $seed`. Once `accent` is overridable, decide whether `link`
   follows `accent` or becomes its own override. Recommendation: follows, with no control —
   two controls for one colour is a trap.

Storage: one JSON object, or one row per overridden role, in `design_tokens`. Prefer
**one scalar per role** (`color_text`, `color_background`, …) over a JSON blob, because
`TokenCompiler::css()` validates names against
`^[a-z][a-z0-9]*(-[a-z0-9]+)+$` and values against `~[;{}<>\\]~`, and a blob smuggles
unvalidated strings past the place that checks them. Absent row = derived. That also keeps
the forward-compatibility property: an old design has no override rows and renders exactly
as it does today.

### 3.2 Type: base, nudges and pairing overrides

`Tokens::typeScale()` today is `$ratio ** $step` with a `clamp()` for steps ≥ 3. Three
changes:

```php
private static function typeScale(float $ratio, float $base, array $adjust): array
```

- Multiply every step by `$base / 16` — or better, treat `$base` as the rem root and emit
  sizes in rem against it.
- Add the per-step nudge **in px, converted to rem**, after the ratio maths.
- The `clamp()` for large steps must fold the nudge into **both** the min and the max, or a
  nudged headline will snap back to its un-nudged size on a narrow viewport. Today:
  ```php
  $min = max(1.25, $size * 0.72);
  $slope = ($size - $min) / 0.45;
  ```
  With a nudge, compute `$size` first (ratio × base + nudge), then derive `$min` and
  `$slope` from that. Do not add the nudge to the finished `clamp()` string.

`heading_weight`, `tracking` and `caps` are overrides on the `heading` token group, which
`Tokens::derive()` currently fills from `Typography::PAIRINGS`:

```php
'heading' => [
    'weight'    => $decisions['heading_weight']  ?: $pairing['heading_weight'],
    'tracking'  => $decisions['tracking']        ? self::TRACKING[$decisions['tracking']] : $pairing['tracking'],
    'transform' => $decisions['caps']            ?: $pairing['transform'],
],
```

Empty string = follow the pairing. The inspector shows a "follow typeface" link whenever an
override is set, which is the same follow/override pattern `ChromeLook` already uses — and
is the best idea on the current Chrome screen (see §3.5).

### 3.3 The sheet, and the header breaking out of it

This is the structural change and it needs care.

`Tokens::page()` today returns `frame` as padding for the **whole page**, with the docblock
explaining that a zero frame is what makes `page_background` harmless. The header and footer
live inside the sheet, so they are inset by the frame and cannot reach the viewport edge.

New geometry — the frame wraps **only the sheet**:

```
.page                      background: var(--page-bg)
  header  (when header_bleed = full)     ← flush to the viewport edge
  .page-sheet-wrap         padding: var(--page-frame)
    .page-sheet            background: var(--page-sheet); radius; shadow
      header (when header_bleed = sheet)
      sections…
      footer (when footer_bleed = sheet)
  footer  (when footer_bleed = full)     ← flush to the viewport edge
```

New tokens from `Tokens::page()`:

| Token | Value |
| --- | --- |
| `--page-frame` | `boxed ? spacingUnit × frame : 0` |
| `--page-sheet-radius` | `boxed ? RADII[sheet_radius] : 0` |
| `--page-sheet-shadow` | `boxed ? (soft: a drop shadow; hard: a 1px hairline in border) : none` |
| `--page-header-width` | unchanged — where the logo and menu sit *inside* the band |

`header_bleed` / `footer_bleed` are **not tokens**; they decide which slot the chrome
renders into, so they are read by the layout template, not by CSS. That keeps the CSS free
of "is it boxed" conditionals, which is the property the existing docblock is proud of.

When `boxed = no`, `frame = 0` and both slots are identical — the sheet is already full
width. Say so in the hint (the prototype does): the control only does something on a boxed
page. Do not hide it; a control that disappears is worse than one that explains itself.

Keep `page_background` as a palette shade rather than a free colour. The reasoning in
`Tokens.php` — no text ever sits on it, so `failures()` gains no pairs — is still exactly
right, and it stays right only as long as no text lands there. A full-bleed header **does**
land there visually but paints its own background, so the pair to check is header-ink on
header-surface, which the gauge already covers.

### 3.4 Content width as a number

`Tokens::CONTAINER` maps four names to rem. Replacing it with a slider is a **type change on
a stored value**, and this is the one place the model's forward-compatibility story does not
save you: an existing row holds `"narrow"`, and `(float) "narrow"` is `0`.

Handle it at read time, not with a migration that rewrites data:

```php
$measure = is_numeric($v) ? (float) $v : (self::CONTAINER[$v] ?? self::CONTAINER['normal']);
```

Clamp to 36–88. Keep `CONTAINER` as the named values the five characters use, so a character
still reads as `'measure' => 56` and the presets stay legible.

`container-narrow` and `container-wide` in `Tokens::derive()` are currently `× 0.68` and
`× 1.3` of the base — keep those ratios against the new number.

### 3.5 Footer columns, and the follow/override pattern

`footer_columns` is a plain addition to `ChromeLook::OPTIONS` (`['2','3','4']`) plus one
entry per character in `ChromeLook::CHARACTER`. The footer partial reads it into a
`grid-template-columns: repeat(var(--footer-columns), minmax(0,1fr))`.

Show the control **only when `footer_layout` resolves to `columns`**. The prototype filters
it out otherwise.

While you are in `ChromeLook`: the empty-string-means-follow-the-character convention is the
best thing on the current Chrome screen, and the new screen should make it *visible* rather
than merely available. Today it is the first `<option>`
(`"As the character has it: Compact"`). In the new inspector it is a state: a group shows
`following` in accent when no override is set, and a `follow character` link when one is.
Same data, no hidden default.

---

## 4. The live contrast gauge

`DesignController::check()` already returns exactly what this needs, one step short:

```json
{ "errors": { "seed": "…" }, "colors": { "background": "#…", … } }
```

It returns **failures only**. The gauge wants **every pair with its ratio**, pass or fail, so
the owner can watch a colour approach the line instead of being told after the fact. Extend
the payload rather than adding an endpoint:

```json
{
  "errors": { … },
  "colors": { … },
  "pairs": [
    { "pair": "text_on_background", "decision": "surface_contrast", "ratio": 14.82, "required": 4.5, "passes": true },
    …
  ]
}
```

Implementation: factor the pair list out of `Palette::failures()` into a
`Palette::pairs(array $colors): list<array{…}>` that returns all eleven with ratios;
`failures()` becomes a filter over it. Both callers stay honest and there is one list of
pairs instead of two.

The prototype shows six rows (page text, tinted-section text, quiet text, button label,
header text, footer text) because eleven is too many for a sidebar. Pick the six the owner
can act on and put the full eleven behind a disclosure, or show only failures plus a
"6 of 11 checked, all pass" summary line. **Do not drop pairs from the server check** —
only from the display.

Keep `AA_BODY = 4.5` and keep the refusal on save. The gauge is information; validation is
the gate.

---

## 5. Server work, in dependency order

### 5.1 Migration `0024_design_presets.sql`

Migrations run to `0023_stats_places.sql`, so 0024 is next. The activity table already
exists (`0021_activity.sql`) and `Activity::record()` is already called by
`DesignController::save()` — reuse it for preset saves and publishes.

```sql
CREATE TABLE design_presets (
  id          INTEGER PRIMARY KEY AUTOINCREMENT,
  name        TEXT    NOT NULL,
  decisions   TEXT    NOT NULL,   -- JSON: the layer-1 decisions, including colour overrides
  chrome      TEXT    NOT NULL,   -- JSON: the ChromeLook overrides ('' entries omitted)
  created_at  TEXT    NOT NULL,
  updated_at  TEXT    NOT NULL
);
CREATE UNIQUE INDEX design_presets_name ON design_presets (name);
```

Notes on the shape:

- **JSON is acceptable here and not in `design_tokens`.** A preset is an inert record that is
  only ever read back through `Tokens::validate()` before use; the live design goes through
  `TokenCompiler`, which validates per name and per value. Validate a preset on load, not on
  save, and a hand-edited row cannot inject anything.
- No `user_id`. Presets are site-level: two editors on one site should see the same library.
  (The admin *theme* from `IMPLEMENTATION-theme-switch.md` is the opposite case — that one is
  per user.)
- Name is unique so overwrite-by-name is unambiguous and the UI can offer "replace?".

### 5.2 `Presets` becomes a repository

`Presets::ALL` and `Presets::COMPOSITION` stay as the five built-ins — they are code, they
ship with the product, they are not user data. Add a reader that merges:

```php
final class PresetLibrary
{
    /** @return list<array{name: string, label: string, builtin: bool, decisions: array, chrome: array}> */
    public function all(Db $db): array;
    public function save(Db $db, string $name, array $decisions, array $chrome): void;
    public function delete(Db $db, string $name): void;
}
```

Two rules that keep the composition layer coherent:

- **A saved preset carries no composition.** `Presets::COMPOSITION` describes how a character
  composes *blocks* (layers 2 and 3) and has no meaning for an arbitrary token set. A saved
  preset stores the character it was derived from, so "apply with composition" still works
  and still says which character's composition it would apply.
- Therefore `decisions` must include `character` — which is what
  `DesignController::save()` already tracks through the hidden `character` field.

### 5.3 `Tokens`

- `choices()` gains `header_bleed`, `footer_bleed`, `sheet_radius`, `sheet_shadow`,
  `hero_layout`, `caps`, `tracking`, `heading_weight` (with `''` allowed for the three
  overrides), and `frame` as an int set.
- `measure` leaves `choices()` and gets numeric validation with the string fallback of §3.4.
- `base`, `scale`, `adjH1`, `adjH2`, `adjBody`, `adjSm` get numeric validation with clamps.
  **Clamp, do not error**, for the numerics: they come from sliders, so an out-of-range value
  is a stale form, not a typing mistake — the same reasoning `ChromeLook::save()` already
  uses for unknown select values.
- Colour overrides validate as `#rrggbb` via `Color::normalizeHex()`, and an unparseable one
  is dropped back to derived rather than erroring.
- `derive()` gains the `page` tokens of §3.3 and the `heading` overrides of §3.2.

Every new decision needs a default that reproduces today's output, so an existing
`design_tokens` row renders byte-identically until the owner touches something. Write a test
for exactly that: load the five characters, derive, and compare against a committed
snapshot.

### 5.4 The preview must draw the chrome

This is the change with the most leverage and the most risk.

`DesignController::preview()` renders the home page's blocks (or a specimen) through
`Pages/views/page.php` with `headerHtml` and `footerHtml` empty. It needs to render real
chrome from `SiteChrome` + `ChromeLook::resolve()`, **with the submitted look overrides
applied**, so the seven-plus-three chrome choices are visible while they are being made.

Before writing that, read the comment in `preview()`. It says this method is the second
renderer of the site layout, that it has been caught out three times by missing layout
variables (`description`, `icon`, and the chrome of slice 5c), and that the real fix is one
source for those variables rather than a louder warning.

**This slice is where you pay that debt.** Adding chrome to the preview without fixing it
makes the fourth catching-out a certainty. Extract the assembly both callers need:

```php
final class PageLayoutData
{
    /** Every variable Pages/views/page.php reads, for a real page or for a preview. */
    public static function forPage(Db $db, int $pageId, string $locale): array;
    public static function forPreview(Db $db, array $lookOverrides, string $blocksHtml): array;
}
```

`PageController::render()` and `DesignController::preview()` both call it. A missing variable
then breaks one place, loudly, in a test.

The preview response already carries its own CSP with `frame-ancestors 'self'` and
`X-Frame-Options: SAMEORIGIN` — keep both exactly as they are.

### 5.5 Routes and navigation

- `/admin/appearance` — the new screen. `GET` shows it; `POST` publishes.
- `/admin/appearance/preview`, `/appearance/stylesheet`, `/appearance/check` — the three
  existing Design endpoints, renamed, with the chrome overrides added to the query they read.
- `/admin/appearance/presets` — `POST` save / overwrite, `POST` delete.
- Keep `/admin/design` and `/admin/chrome` as redirects for one release; bookmarks and the
  nav both point at them today.
- `Admin/views/layout.php`'s nav loses two items and gains one.

### 5.6 What stays on its own screen

**The per-language words do not move here.** `chrome.php` repeats four fields — button page,
button address, button label, footer text, small print — once per locale, and that is the
right shape for *content*: it is words, it is translated, and it grows with the number of
languages. The Appearance screen is about *look*, which is one decision for the whole site
in every language.

So: Appearance takes the `Look` panel (the seven choices) and the menu selection; the words
stay on a `/admin/chrome` that is now only words — or better, move each locale's words into
a collapsible per-language section on that screen, which also fixes the doc's complaint that
the form doubles with every language. Either way, do not drag ten translated text fields
into a preview-first screen; they will drown it.

The logo also stays in Settings → Branding (D-038). The Appearance screen should carry the
same honest link the current Chrome screen does.

---

## 6. Client work

### 6.1 No inline styles — and the architecture already handles it

The prototype fakes the preview inside one document, which is why it is wall-to-wall
`style=""`. The real screen does not have that problem, because the preview is **an
iframe with its own stylesheet endpoint**: `previewCss()` serves
`TokenCompiler::css(Tokens::derive($decisions))` with `Cache-Control: no-store`. Every live
value lives in that generated stylesheet, inside the frame.

So the admin page itself needs **zero** dynamic styling. The inspector is static markup and
static CSS; only the iframe's `src` changes. Port every prototype style into
`public/assets/admin-appearance.css` as classes. Where a value must be dynamic in the admin
chrome — the swatch fills next to each colour role, the specimen line sizes — use the
pattern `design.js` already uses for swatches (`rect[data-swatch]` + `setAttribute('fill')`)
or the `depth-N` class pattern from `Pages/views/admin/index.php`. Do not reach for a style
attribute, and do not add `'unsafe-inline'`.

### 6.2 `design.js` → `appearance.js`

Keep its shape: it is a good file. It already debounces at 250 ms, refreshes the frame,
fetches `check`, paints swatches and hex readouts, and guards `beforeunload` when dirty.
What to add:

- **Drop the debounce for the frame on discrete controls.** A segmented button is a decision,
  not typing — refresh immediately. Keep the 250 ms only for the colour inputs and sliders,
  where a drag fires continuously.
- The viewport switcher sets the iframe's width; zoom sets a CSS class or a custom property
  on its wrapper and the iframe keeps its real pixel width so the site's own breakpoints
  behave. **Scale the wrapper, never resize the frame to fake a zoom** — the page inside must
  lay out at 1280 to be judged at 1280.
- Compare: hold the button, and the frame's `src` swaps to the query built from the
  *published* decisions, which the screen already has server-rendered. Release restores.
- The gauge redraws from the extended `pairs` payload.
- Keep the whole thing optional. Without JavaScript: the tabs are anchors or a `<details>`
  group, the preview has its "Update preview" form back, and Publish is a normal submit.
  `CLAUDE.md` forbids a control that only works with JS, and every control here can be a
  form submit.

### 6.3 Keyboard and focus

Nothing in the prototype is keyboard-hostile, but nothing proves it either. The inspector is
five tab panels: give them real `role="tablist"` / `aria-selected` semantics with arrow-key
movement, and keep the `:focus-visible` accent ring the admin already defines rather than the
browser default.

---

## 7. What to check before committing

- `php tests/run.php`. Expect `tests/contrast_test.php` to be the first thing that fails on
  any palette change, and extend it: the override path needs its own cases, including the
  dependent-role recomputation of §3.1.2. A failing pair must be **attributed to the right
  decision** — assert that, not just that it fails.
- `vendor/bin/phpstan analyse` (level 8).
- `tests/chrome_look_test.php` will notice `footer_columns`.
- New test: the five characters derive to a committed snapshot, so the defaults of §5.3 are
  provably neutral.
- New test: a `design_tokens` row written before this slice still loads, with every new
  decision at its default and `measure` mapping from its old string.
- `tools/browser-suite/` gets an Appearance scenario per tab, plus one boxed-page shot with
  the header bled full width — that combination is new geometry and it is what will break.
- Record in `PLAN.md`: the merge of the two screens, the colour-override decision and why the
  contrast check makes it safe, the `measure` type change, and the `PageLayoutData`
  extraction. `CLAUDE.md`: a decision that is not written there did not happen.
- The colour-override addition is a deliberate softening of a SPEC §5.4 position. Say so in
  the commit message — "state when you exceed an instruction".

---

## 8. Suggested slicing

Each of these is shippable on its own and leaves the admin working.

1. **Preview draws the chrome.** `PageLayoutData` extraction + chrome in
   `DesignController::preview()`. Biggest single improvement, no new decisions, no
   migration. Also pays the three-times-caught-out debt.
2. **The loop closes.** Drop the Update button, refresh discrete controls immediately, move
   Save to a sticky bar, replace the `clamp()`/rem readouts with specimens and numbers.
   Still no new decisions.
3. **Merge the two screens.** Routes, nav, the five-tab inspector, viewport + zoom. The
   per-language words stay behind on their own screen.
4. **The preset library.** Migration 0024, `PresetLibrary`, the left rail, save / overwrite /
   delete.
5. **The finer controls.** Type base + nudges + pairing overrides, `measure` as a number,
   the sheet controls, `header_bleed` / `footer_bleed`, `footer_columns`, `hero_layout`.
6. **Colour overrides.** Last, deliberately: it is the one change that moves a guarantee from
   derivation to validation, and it wants the gauge from slice 2 and the tests from §7
   already in place.

Slices 1 and 2 are most of the felt improvement for the least work, and neither needs a
schema change or a decision from the owner.

---

## 9. Screenshots

`screens/`, captured from the prototype.

| File | What |
| --- | --- |
| `01-colour.png` | Colour tab: seed, second colour, surface separation, the eight overridable roles, the gauge |
| `02-type.png` | Type tab: typefaces, body size, ratio, per-step nudges, weight / tracking / caps, specimen |
| `03-shape.png` | Shape tab: corners, depth, air, content-width slider, spacing bars |
| `04-header.png` | Header tab: the ten chrome choices with follow / override state |
| `05-page-boxed.png` | Page tab with a boxed sheet and the header bled full width — the geometry of §3.3 |
| `06-phone.png` | The same design at the phone viewport, with the preview's own small-screen behaviour |

## 10. Files

- `Boxlet Appearance.dc.html` — the design reference. Read the markup; ignore the runtime and
  the approximated palette maths.
- `nocturne-tokens.css` — the admin-side design system the prototype is styled against. The
  repo has its own (`public/assets/admin.css`, the `--ui-*` set); see
  `IMPLEMENTATION-boxlet.md` in the other packages for how the two relate. Nothing in this
  screen requires adopting Nocturne — the layout and the information architecture are the
  deliverable.
