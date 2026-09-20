# Handoff: Boxlet CMS admin — "Workbench" (v2)

## Overview

The second of two redesign directions for the Boxlet CMS admin (repo: `zaja/Boxlet-CMS`, branch `main`).

**v1 "Studio"** (see `design_handoff_admin_redesign/`) keeps the top bar, opens with a prose summary, and carries a warm ember accent for things needing attention.

**v2 "Workbench"** — this package — is the restrained, information-dense take. A grouped left rail instead of a top bar. No prose, no warm accent: strictly Nocturne's mono blurple on neutrals. The dashboard is a log, not a poster. Media is a table, not a grid. Settings is a label/control ledger with its own sub-navigation. Monospace for every number, path and timestamp.

Pick one direction and implement it; they are not meant to be merged wholesale. If you want a hybrid, the natural seam is v2's left rail + v1's prose dashboard.

Four screens: **Overview**, **Pages**, **Media library**, **Site settings**, plus a command palette.

## About the design files

`Boxlet Admin v2.dc.html` is a **design reference created in HTML** — a prototype of intended look and behavior, not production code.

How to read it:
- It uses a small streaming-template runtime (`<sc-for>`, `<sc-if>`, `{{ }}` holes) with a `class Component` block at the bottom holding all sample data. **Ignore the runtime** — read it as markup plus a data object.
- **All styling is inline `style="…"`** by constraint of the prototyping environment. Port to whatever the CMS already uses (Blade/Twig partials + a stylesheet, Tailwind, CSS modules).
- `style-hover="…"` means a `:hover` rule.
- `nocturne-tokens.css` is the design-system stylesheet. **Port this first**: its `:root` variables are the source of truth for every value below, and its component layer (`.btn`, `.input`, `.field`, `.seg`, `.tag`, `.table`, `.card`, `.dialog`) is plain CSS on plain HTML — it drops into a PHP/Blade admin nearly as-is. The `.table` class in particular is already used unchanged here.

The task is to **recreate these screens in the CMS's existing template environment** using its established patterns.

## Fidelity

**High-fidelity.** Colors, type sizes, spacing and interaction states are final. Where this document and the HTML disagree, the HTML wins.

Exceptions: media thumbnails are CSS gradient placeholders; the metrics, activity log and visitor figures are sample data.

## Design tokens

From `nocturne-tokens.css` (`:root`). Use the variables, not the literals.

| Token | Value | Use |
| --- | --- | --- |
| `--color-bg` | `#161826` | content ground |
| `--color-surface` | `#232532` | input backgrounds, preview squares |
| `--color-text` | `#e9e9ed` | body text |
| `--color-accent` | `#9184d9` | the only accent — actions, links, focus, active nav |
| `--color-divider` | `#e9e9ed` @ 16% | rules |
| `--color-neutral-100…900` | `#f3f5fe` … `#292b31` | surfaces, borders, muted text |
| `--color-accent-100…900` | `#f5f4ff` … `#2b2741` | accent tints, pressed states, `.tag-accent` |
| `--shadow-sm/md/lg` | hairline edge + ambient | elevation |
| `--radius-sm/md/lg` | 4 / 8 / 14px | v2 mostly uses 5–6px (see below) |
| `--space-1…8` | 2.8 / 5.6 / 8.4 / 11.2 / 16.8 / 22.4px | density 0.70× |

**Literals used here that are not yet tokens** — add them when you port:

| Literal | Role |
| --- | --- |
| `#131522` | left rail ground |
| `#191b28` | metric tile ground |
| `#1a1d2b` | control ground (rail search, palette surface) |
| `#7fb98a` | healthy / online dot |
| `#a6aabb` | inactive nav label |
| `#8e93a5` | top-strip icons, lede-adjacent muted text |
| `#75798c` | kickers, meta, hints (= `--color-neutral-600`) |
| `#9397ab` | table secondary cells (= `--color-neutral-500`) |
| `#6f7385` | tertiary meta (metric notes, activity type) |
| `#b2b6ca` | filename in a picker row (= `--color-neutral-400`) |
| `#a7a1db` | monospace addresses |
| `#565a69` | rail group kickers |
| `#4b4f5e` | drag handles, breadcrumb slash |
| `#595d6c` | overflow-menu icons (= `--color-neutral-700`) |
| `#333644` | avatar square |
| `#3f424d` | hairline insets (= `--color-neutral-800`) |
| `#b5abfc` | "needs attention" icon (= `--color-accent-400`) |

**Radii** — v2 is tighter than the token defaults on purpose: 4px (thumbnails, avatar), 5px (palette rows, breadcrumb hit area), 6px (nav rows, buttons, inputs, metric grid, panels), 8px (palette shell).

**Type** — Inter, `--font-heading` = `--font-body`, heading weight **500 (never bolder)**. Base font-size **13px** on the root container.

| Role | Size / spacing |
| --- | --- |
| Page title (h1) | 21px / −0.015em |
| Page lede | 12.5px / `#8e93a5` / max-width 70ch |
| Metric value | 22px / weight 500 / −0.02em / `tabular-nums` |
| Section kicker | 11px / uppercase / 0.13em / `#75798c` |
| Rail group kicker | 9.5px / uppercase / 0.14em / `#565a69` |
| Column header | 9.5px / uppercase / 0.13em |
| Rows, tables | 12.5px |
| Meta, hints | 11–12px |
| Monospace | 11–11.5px, `ui-monospace, SFMono-Regular, Menlo, monospace` |

**Monospace rule:** every number, address, byte size, timestamp, shortcut and dimension is monospace. Prose is never monospace. This single rule does most of the "professional" work in v2 — do not skip it.

**Icons** — [Phosphor](https://phosphoricons.com), regular weight, loaded as the web font:

```html
<link rel="stylesheet" href="https://unpkg.com/@phosphor-icons/web@2.1.1/src/regular/style.css">
<i class="ph ph-gauge"></i>
```

Glyphs used: `ph-gauge` (Overview), `ph-file-text` (Pages), `ph-image` (Media), `ph-list` (Menus), `ph-list-checks` (Forms), `ph-palette` (Design), `ph-layout` (Header & footer), `ph-gear-six` (Settings), `ph-users` (Users), `ph-chart-bar` (Statistics), plus `ph-magnifying-glass`, `ph-caret-down`, `ph-arrow-square-out`, `ph-sign-out`, `ph-warning-circle`, `ph-upload-simple`, `ph-dots-three-vertical`. Self-host the font in production.

## Screens

### 1. Chrome

`display: grid; grid-template-columns: 216px minmax(0,1fr); height: 100vh; overflow: hidden`.

**Left rail** — ground `#131522`, right edge as `box-shadow: inset -1px 0 0` divider (not a border). Four stacked regions:

1. **Brand row**, 46px tall, `inset 0 -1px 0` divider below: a 16px accent square (radius 4px), "Boxlet" 13.5px weight 500, and a right-aligned build tag `v2.4` in 10px `#595d6c`. The version number matters — an admin that tells you which build you are on reads as maintained.
2. **Search button**, full width, `#1a1d2b`, 1px `#e9e9ed`@9% border, radius 6px, 12px: magnifier + "Search" + a monospace `⌘K`. Hover: border → accent, text → `--color-accent-300`.
3. **Nav**, scrollable, 13px gaps between groups. Groups: **Site** (Overview) · **Content** (Pages 5, Media 15, Menus 1, Forms 2) · **Presentation** (Design, Header & footer) · **Administration** (Settings, Users 3, Statistics). Group kicker, then rows: padding `6px 8px`, radius 6px, 14px icon at 0.8 opacity, label, and an optional right-aligned monospace count. Inactive `#a6aabb`; hover `#e9e9ed` on `#e9e9ed`@6%; **active = `#e9e9ed` label over an absolutely-positioned overlay with `accent@15%` background and `box-shadow: inset 2px 0 0 var(--color-accent)`** — a left bar, not a filled pill.
   - The grouping is the point: this scales to 12+ sections, which the current top bar does not. Keep the groups even if a group has one item.
4. **User row**, `inset 0 1px 0` divider above: a 22px `#333644` rounded-square avatar with initials, then name 12px over role 10px `#75798c`, then a caret. Opens the account menu.

**Top strip** — 46px, `inset 0 -1px 0` divider, inside the content column:
- Left: a **site switcher** — a 5px green dot + hostname + caret, in a hoverable `5px/8px` hit area — then a `#4b4f5e` slash and the current section in `#75798c`. This replaces the page-title-only header; multi-site is where a CMS admin usually starts to hurt.
- Right: a monospace clock/zone stamp (`Europe/Zagreb · 12:41`) in `#595d6c`, then 28px icon buttons for open-live-site and sign-out (radius 6px, `#8e93a5`, hover `#e9e9ed` on `#e9e9ed`@7%).

**Content column** — `max-width: 1240px`, padding `22px 26px 56px`, left-aligned (not centered — Nocturne is asymmetric by direction).

**Page header** — a row: left, an h1 (21px) over a 12.5px `#8e93a5` lede at `max-width: 70ch`; right, `.btn-secondary` + `.btn-primary`, both 12.5px at `padding: 5px 10px`.

| Screen | Title | Lede | Secondary | Primary |
| --- | --- | --- | --- | --- |
| Overview | Overview | Site status, what changed lately, and what is waiting on someone. | Export report | New page |
| Pages | Pages | Ordered within each language and under their parent. The order is what menus and page lists start from. | Import | New page |
| Media | Media library | Every picture is kept in several sizes, made on upload. Descriptions are used by visitors who cannot see the image. | Remake sizes | Upload |
| Settings | Site settings | What this site is called, the time it keeps, and the pictures that stand for it. | Discard changes | Save settings |

**Primary buttons are a 1px accent outline on transparent — never filled.** This is a Nocturne rule and it is what stops the admin looking like a bootstrap panel.

### 2. Overview

**a. Metric strip.** `grid-template-columns: repeat(auto-fit, minmax(142px,1fr)); gap: 1px`, on an `#e9e9ed`@8% background with a matching 1px border and `border-radius: 6px; overflow: hidden` — the gap becomes hairline dividers, so there are no card borders anywhere. Six tiles, each `#191b28`, padding `12px 14px`, three lines: uppercase kicker / 22px value / 11px note.

| Kicker | Value | Note |
| --- | --- | --- |
| Pages | 5 | all published |
| Pictures | 15 | 8.4 MB |
| Visitors · 7d | 1 284 | +12% vs prior |
| Form entries | 8 | 3 unread |
| Languages | 2 | EN main, HR |
| Design | Brutalist | unchanged 12 days |

The note line is what makes this different from the current four stat cards: every number carries its context. Never ship a bare number.

**b. Two columns** — `minmax(0,1.7fr) minmax(0,1fr); gap: 26px; align-items: start`.

**Recent activity** (left) — a kicker row with a right-aligned "Full log" link, then **grid rows, not a `<table>`**: `grid-template-columns: 78px minmax(0,1fr) 86px; gap: 12px; align-items: baseline; padding: 9px 4px`. Cells: monospace relative time `#75798c` / the description (ellipsised) with a small uppercase type label (`Page`, `Media`, `Menu`, `Form`, `Design`) pushed to its right / the actor in `#9397ab`. Hover tints the row at `#e9e9ed`@4%.

This is the single most useful addition for a multi-editor site: one place that answers "what changed, and who did it".

**Needs attention** (right top) — 2–4 outlined cards: 1px `#e9e9ed`@9%, radius 6px, padding `8px 10px`, a `ph-warning-circle` in `#b5abfc`, then the issue in 12.5px over its location in 11px `#75798c` ("Media · wall-planks, workshop"). Hover: border → accent, background → accent@9%. Each card links to the fix. **Empty-state this list ("Nothing needs attention") rather than padding it.**

**Most read · 14 days** (right bottom) — four rows, each a name + right-aligned monospace hit count, with a 3px progress track below (`#e9e9ed`@8% ground, accent fill at 0.8 opacity, width as % of the top page).

**The fading rule.** All row separators in this design are a Nocturne signature — a 1px rule that fades to transparent at both ends, painted as a background layer on the row itself:

```css
background: linear-gradient(to right,
    transparent,
    color-mix(in srgb, #e9e9ed 8%, transparent) 30px,
    color-mix(in srgb, #e9e9ed 8%, transparent) calc(100% - 30px),
    transparent) no-repeat bottom / 100% 1px;
```

Hover adds a flat tint as a *second* background layer so the rule keeps painting. (Nocturne's `.table` class already does this per row — the grid rows reproduce it by hand.) Box outlines and in-control separators stay solid.

### 3. Pages

**Filter row** — a 250px `.input` "Filter by title or address" (32px min-height, 12.5px), a compact `.seg` **All / EN / HR**, and a right-aligned monospace `5 rows`.

**The table** — Nocturne's `.table` class, `font-size: 12.5px`, **`table-layout: fixed`**, inside a `min-width: 0; overflow-x: auto` wrapper. (Both are required: `.table` is `width: 100%`, and on `table-layout: auto` the fixed `th` widths plus the title column's min-content width push the table past its grid track and over the next column.)

Columns: drag handle 26px · Title · Address 170px · Lang 60px · Status 96px · Last edited 132px · overflow 28px.

- Drag handle `⠿` in `#4b4f5e`, `cursor: grab`, centered.
- Title, ellipsised, with an optional **Home** badge: 9.5px uppercase 0.08em, `#9397ab`, 1px `#3f424d` border, radius 3px.
- Address in monospace `#a7a1db`.
- Lang as a bare `EN` / `HR` in `#9397ab` — a code, not a spelled-out language name; the column is 60px because of it.
- Status as `.tag.tag-neutral` at 10px — **not green**. In v2, color is reserved for action and for problems; "Published" is the normal case and should be quiet. A draft or scheduled row is where a tinted tag earns its color.
- Last edited in monospace (`19 Sep 06:23`).
- A `ph-dots-three-vertical` overflow menu in `#595d6c` — **replaces the always-visible trash icon** in the current admin. Destructive actions go behind it.

### 4. Media library

A **table, not a grid** — this is the main structural difference from v1, and the reason is maintenance: a grid shows you pictures, a table shows you which ones are a problem.

**Filter row** — a 230px `.input`, a `.seg` **All / Unused / No description**, and a right-aligned monospace `15 files · 8.4 MB`.

**The table** — same `.table` + `table-layout: fixed` + scroll-wrapper rules. Columns: thumb 52px · File · Dimensions 118px · Size 78px · Used on 88px · Description 120px · overflow 28px.

- Thumb: a 38 × 28px rounded rect, radius 4px, `inset 0 0 0 1px #3f424d`, `object-fit: cover`.
- File name with extension, ellipsised.
- Dimensions and Size in monospace `#9397ab`.
- **Used on**: "2 pages" / "1 page" / "—". This is new and it is the most valuable column in the screen — it makes the "Unused" filter trustworthy and deletion safe.
- **Description**: `.tag.tag-accent` "Missing" when there is no alt text, otherwise a quiet `Set` in `#6f7385`. Should be inline-editable from this cell.
- Overflow menu per row.

**Drop zone** — one compact row *below* the table (not a 140px box above it): radius 6px, 1px **dashed** accent@40%, a `ph-upload-simple` in accent, then "Drop files anywhere, or [browse] — JPEG, PNG, WebP, GIF, AVIF up to 256 MB". The whole page is the drop target — implement the page-level dragover state (tint the content column), and this row is just the affordance and the format contract.

### 5. Site settings

`grid-template-columns: 168px minmax(0,1fr); gap: 30px; align-items: start; max-width: 940px`.

**Sub-navigation** (left, sticky) — **General / Branding / Languages / Maintenance**, styled exactly like the rail rows (radius 6px, accent overlay + `inset 2px 0 0` bar when active). Settings pages grow; give them their own axis from day one instead of one long scroll.

**The ledger** (right) — one row per setting: `grid-template-columns: 176px minmax(0,1fr); gap: 22px; align-items: start; padding: 14px 4px`, separated by the fading rule.

- **Left cell**: the label in 12.5px `#e9e9ed` over its hint in 11px `#75798c` at line-height 1.4.
- **Right cell**: one of three control types (mutually exclusive — render one, do not hide the others):
  - **text** — a `.input` at `max-width: 320px`, 32px min-height, 12.5px.
  - **file** — an outlined picker row (1px `#e9e9ed`@9%, radius 6px, `max-width: 400px`): 32px preview square (`#232532`, `inset 0 0 0 1px #3f424d`), filename in 12px `#b2b6ca` (ellipsised), and a `.btn-ghost` "Change" / "Choose".
  - **tags** — a wrapping row: `.tag.tag-accent` "English · main", `.tag.tag-neutral` "Hrvatski · /hr/", and a small `.btn-secondary` "Add".

Rows, in order: Site name (text) · Time zone (text) · Logo (file) · Favicon (file) · Sharing image (file) · Languages (tags).

**This label-left / control-right ledger is the core fix** for the current settings page, where every field carries its own paragraph of help text and the page becomes a wall of prose. The hint sits beside the control, small and permanent, and the eye can scan the label column alone.

**Footer** — a `.btn-primary` "Save settings" plus a 11.5px `#75798c` line: "Last saved 19 Sep 2026, 12:41 by Marko Kovač". Say who and when; it is cheap and it settles arguments.

### 6. Command palette (⌘K)

- Opens on `⌘K` / `Ctrl+K` (preventDefault), closes on `Escape` or backdrop click.
- Backdrop `rgba(9,10,17,.6)` + `backdrop-filter: blur(3px)`, content pushed down `12vh`.
- Panel `min(540px, 92vw)`, `height: fit-content`, radius 8px, `#1a1d2b`, `--shadow-lg`.
- Search row: padding `11px 13px`, `inset 0 -1px 0` divider; a borderless transparent 14px autofocused input, placeholder "Search pages, pictures, settings, actions", and a monospace `esc` on the right.
- Result rows: `7px 9px`, radius 5px, 12.5px, hover `accent@15%`. Each row = a 46px uppercase kind label (`Go` / `Action` / `Page` / `Setting`) + label + right-aligned monospace hint (count, shortcut, age, parent section).

Search across pages, pictures, menus, form entries, **settings fields**, and actions in one ranked list. Arrow keys + Enter. The `Setting` kind is worth the extra work: "time zone" should land you on the field, not the page.

## Interactions & behavior

| Trigger | Behavior |
| --- | --- |
| Rail row click | switch section; accent overlay + left bar move, label brightens |
| Site switcher click | site list dropdown |
| `⌘K` / `Ctrl+K` | open palette, focus input |
| `Escape` / backdrop click | close palette |
| Row hover (activity, tables) | `#e9e9ed`@4% tint layered over the fading rule |
| Outlined card hover (issues) | border → accent, background → accent@9% |
| `.btn-primary` hover / active | accent@12% / accent@22% behind the outline |
| `.btn-secondary` hover / active | text@7% / text@14% |
| Keyboard focus | `2px solid var(--color-accent)` at `outline-offset: 2px` — **never the browser default** |

Not in the prototype, required in the real thing: drag-reorder on Pages, upload queue with progress, dirty-state tracking and validation in Settings, arrow-key navigation in the palette, inline alt-text editing in the Media table, empty states for every list, and a responsive breakpoint — below ~1000px collapse the rail to icons only; below ~760px the Overview two-column grid stacks and the tables' lower-priority columns (By, Used on, Lang) should drop rather than scroll.

## State

Prototype state is `{ screen, palette }`. Real state:
- Route / current section, and the Settings sub-section.
- Palette open + query + selected index.
- Dirty-form tracking for Settings (drives Discard and the "Last saved" line).
- Upload queue for Media; drag state for Pages.
- Overview payload: six metrics with their context notes, the activity log (paged — the screen shows six, "Full log" shows all), the issue list, the top-pages series.
- Site list for the switcher.

## Assets

- **Icons**: Phosphor regular (see the glyph list above). Self-host.
- **Fonts**: Inter 400/500/600/700, pulled from Google Fonts in `nocturne-tokens.css`. Self-host in production.
- **Media thumbnails**: gradient placeholders in the prototype.
- The brand square is a plain accent rounded rect; the existing Boxlet logo mark works in its place. Unlike v1, v2 has **no warm accent** — if you use the orange mark, it is the only warm pixel in the interface, which is fine, but do not let it spread.

## Screenshots

`screens/` — captured at ~900px wide, i.e. near the narrow end. At full desktop width the columns are wider and the content column caps at 1240px.

| File | Screen |
| --- | --- |
| `screens/01-overview-dark.png` · `-light.png` | Overview, both themes |
| `screens/02-pages-dark.png` · `-light.png` | Pages |
| `screens/03-media-dark.png` · `-light.png` | Media library |
| `screens/04-settings-dark.png` · `-light.png` | Site settings |
| `screens/05-command-palette-dark.png` | Command palette |
| `screens/05-overview-light-switch.png` | Overview in light, theme switch selected |

References for composition and tone; use the tables above and the HTML for exact values.

## Both themes

The prototype now carries a **light (warm paper) theme and a dark theme**, switched from the
three-way control in the top strip (sun / moon / match-system, persisted in localStorage).

Every colour in the prototype resolves through a `--ui-*` custom property — there is no hex
literal left in the markup — so a theme is a block of values, not a second stylesheet. Two
things to carry over, both learned the hard way here:

- **A token whose name states a direction lies in one of the two themes.** Nocturne's ramp
  steps (`-800` = "dark tint") had to be inverted for the light block, or accent tags render
  as navy chips on paper. In `admin.css` the equivalents are `--ui-accent-dark` and
  `--ui-on-accent`.
- **Anything styled outside the token system wins and breaks the theme.** In the prototype
  that was inline `style` on the switch buttons; in the repo it will be any literal colour in
  a per-screen stylesheet. Grep for `#` outside `admin.css` first.

`IMPLEMENTATION-theme-switch.md` has the full plan for the repo: where the two token blocks
go, the dark `--ui-*` values, the form-POST switch (no JS, no flash, CSP-safe), the
per-user migration, and what to check.

## Files

- `Boxlet Admin v2.dc.html` — the design reference. Markup + inline styles + a logic class holding sample data.
- `IMPLEMENTATION-boxlet.md` — how this lands in `zaja/Boxlet-CMS`: the real files, and five repo rules the prototype breaks. **Read before the README's styling advice.**
- `IMPLEMENTATION-theme-switch.md` — adding the light/dark themes and the switch to the repo.
- `nocturne-tokens.css` — the Nocturne stylesheet: `:root` tokens plus the `.btn` / `.input` / `.field` / `.seg` / `.tag` / `.card` / `.table` / `.dialog` / `.hr` / `.lighten` component layer. Port this first.

## Design-system rules to honor

From Nocturne's own guidance:

- Left-aligned and asymmetric: headings flush left, content hugs the left edge, whitespace on the right. Center nothing.
- **Primary buttons are outlined, never filled.**
- Headings never exceed weight 500 — hierarchy is size and space.
- No pure black, no pure white; every value comes from the ramps.
- One accent, used as a line, a bar and a glow — never a flood. Do not introduce a second hue for status; use the neutral ramp and reserve the accent for action and for problems.
- Elevation on a dark ground is a hairline edge plus ambient darkness. Do not stack shadows.
- Accent-on-ground is ~3:1 — fine for icons, chrome and large text, **not for paragraph copy**. For body text in accent use `--color-accent-300`.
- Rules fade at their ends; short marks (the active-nav bar) stay solid.
