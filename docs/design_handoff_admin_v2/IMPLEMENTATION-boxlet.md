# Implementing the redesign in Boxlet CMS

Written after reading `zaja/Boxlet-CMS@main` (branch `main`, tree `d0a5685991c4`). This file
overrides anything in the two README handoffs where the repo's own rules disagree — the
prototypes were drawn before the code was read.

Read this first, then the direction's README (`design_handoff_admin_redesign/` = v1 "Studio",
`design_handoff_admin_v2/` = v2 "Workbench").

---

## 1. The good news: the admin already has this design system

`public/assets/admin.css` defines a complete `--ui-*` token set — the admin's own fixed
system, deliberately isolated from the site's compiled `tokens.css` (SPEC §5.4). It already
has:

- a full neutral ramp (`--ui-bg` `#f3f1ec` warm paper, `--ui-panel`, `--ui-panel-sunken`,
  `--ui-ink`, `--ui-ink-muted`, `--ui-ink-faint`, `--ui-line`, `--ui-line-strong`,
  `--ui-control-line`, `--ui-control-soft`)
- one accent (`--ui-accent` `#3b3fc4` indigo, `-dark`, `-soft`, `--ui-on-accent`)
- semantic colors (`--ui-danger`, `--ui-success`, `--ui-warning`, each with a `-soft`)
- the dark bar set (`--ui-bar` `#1c1b18`, `--ui-bar-ink`, `--ui-bar-ink-muted`,
  `--ui-bar-raised`, `--ui-mark` `#ff6b3d`)
- an 8-step spacing scale, a 6-step type scale, 3 radii, 2 shadows, a focus ring, a scrim
- self-hosted faces: **"Boxlet UI" (Inter)** for reading, **"Boxlet Display" (Space Grotesk)**
  for titles

**So this is a retune of `admin.css`, not a new stylesheet.** Most of the redesign is:
change token values in `.admin { … }`, then make a handful of structural changes to views
and the per-screen stylesheets. Do not add a parallel token set, do not import Nocturne's
`styles.css`, and do not put a literal color in any admin stylesheet other than `admin.css`.

Existing per-screen stylesheets to edit rather than replace: `admin-shell.css` (the bar),
`admin-ui.css` (buttons, notices, dialog, empty state), `admin-tables.css`,
`admin-forms.css`, `admin-dashboard.css`, `admin-pages.css`, `admin-media.css`.

---

## 2. Hard constraints the prototypes violate

Fix these before writing any code. All five come from `CLAUDE.md`, `docs/SPEC.md` or the
existing source.

### 2.1 No style attributes — CSP forbids them

From `app/Modules/Pages/views/admin/index.php`:

> the admin sends `default-src 'self'` with no `'unsafe-inline'`, so a style attribute is
> blocked

**Both prototypes are 100% inline styles.** Every value in them must land in a stylesheet as
a class. Where the prototype needs a dynamic value (a sparkline bar height, a progress-bar
width, a tree indent), the repo's own answer is already in that file: **a class, not a
custom property in a style attribute** — `page-name depth-<?= min($depth, 6) ?>`, with
`.depth-1 … .depth-6` declared in CSS and capped.

Apply the same pattern to:
- v1's 14-bar sparkline and v2's most-read tracks → bucket to `.bar-5` … `.bar-100` in steps
  of 5, or reuse `App\Modules\Stats\Chart::spark()`, which already renders a sparkline
  server-side (see `dashboard.php`) and is the right answer.
- v2's metric grid, v1's stat strip → static classes, no dynamic values needed.

### 2.2 Buttons stay filled — outlined primary is explicitly forbidden

`CLAUDE.md`:

> **No control is ever invisible at rest.** Every interactive control … has a legible
> resting state … Hover and focus *raise* a control; they never *reveal* it.

and `admin-ui.css`, on `.button-ghost`:

> No control in this admin is ever invisible or outline-only at rest (see CLAUDE.md).

Nocturne's "primary is an accent outline, never a fill" is a **direct conflict with the
owner's own rule, and the owner's rule wins.** Keep `.button` filled
(`background: var(--ui-accent)`, `color: var(--ui-on-accent)`), keep `.button-secondary` as
the panel-colored bordered variant, keep `.button-ghost` as accent-colored text.

Both READMEs say "primary buttons are outlined, never filled" — **ignore that line.** It is
the one place the design system and the product disagree, and this is the product's call.

Everything else in the two directions survives intact: the ledger settings layout, the
prose dashboard, the activity log, the Media "Used on" column, the ⌘K palette, the fading
rules, the monospace-for-numbers rule, the grouped rail.

### 2.3 No CDN, no new dependencies — use the existing icon sprite

`CLAUDE.md`: runtime dependencies are a closed list; no npm, no build step, no Tailwind.

v2's README tells you to load Phosphor from `unpkg.com`. **Do not.** The admin already has
an icon system: `public/assets/vendor/icons.svg`, a committed Lucide 1.47.0 sprite built by
`tools/icons/build.php`, used as `icon('name')` (`app/Support/helpers.php`). A name not in
the sprite draws nothing.

Currently in the sprite: `arrow-down`, `arrow-up`, `chevron-down`, `cloud-upload`, `copy`,
`crop`, `external-link`, `grip-vertical`, `image-up`, `languages`, `log-out`, `menu`,
`monitor`, `smartphone`, `tablet`, `pencil`, `plus`, `replace`, `search`, `settings`,
`trash-2`, `x`.

The redesign needs these added to `ICONS` in `tools/icons/build.php`, then a re-run:
`gauge`, `file-text`, `image`, `list`, `list-checks`, `palette`, `layout`, `users`,
`bar-chart-3`, `alert-circle`, `more-vertical`, `check`, `clock`. Lucide names, same
semantics as the Phosphor glyphs the prototypes drew.

### 2.4 Every string goes through `t()`

`CLAUDE.md`: *No bare English in a template.* The prototypes are full of literal copy —
"Site status, what changed lately, and what is waiting on someone", "Two pictures have no
description", "Last saved 19 Sep 2026, 12:41 by Marko Kovač", the column headers, the ⌘K
placeholder.

Each of those needs a key in the right `lang/en/*.php` file — and the files are split by
concern, so new concerns get new files:

| Copy | File |
| --- | --- |
| Dashboard lede, metric labels and notes, activity log, "Needs attention" | `lang/en/dashboard.php` (new — the dashboard currently borrows keys from `admin.*`) |
| Command palette: placeholder, kind labels, result hints | `lang/en/palette.php` (new) |
| Rail group headings (Site / Content / Presentation / Administration) | `lang/en/shell.php` (exists) |
| New Pages / Media column headers, filter labels | `lang/en/pages.php`, `lang/en/media.php` (exist) |
| Settings section headings and hints | `lang/en/settings.php` (exists — much of this copy is already there as `settings.*_hint`) |

Note the existing settings hints are already written in the voice both prototypes use; reuse
them rather than rewriting.

### 2.5 The status cell is a switch, not a label

`app/Modules/Pages/views/admin/index.php`:

> **THE STATUS IS THE SWITCH (D-039):** pressing "Published" makes the page a draft …
> A pill with an edge, so it reads as something to press at rest

v2's README says to make status "a quiet `.tag.tag-neutral`, not green". **That would remove
a control.** Keep it a `<button>` with a visible edge at rest. If you want it quieter,
quieten the *fill*, keep the border and the hover/active raise. Same for the per-row delete:
it is a real `<form>` + `<button>`; an "overflow menu" needs a `<details>` (scriptless, like
`admin-nav-group`) or it is a regression.

---

## 3. The open decision only the owner can make: light or dark

`admin.css` is deliberately a **warm paper** admin:

> Neutrals: a warm paper rather than a cool grey — the workbench, not the spreadsheet.

and the bar is:

> the one dark surface in the admin, so the tool reads as a frame around the work

Both prototypes are fully dark. That is not a token retune — it inverts a stated design
position and re-tests the entire contrast matrix. **Ask the owner before implementing it.**

Three honest options:

**A. Dark, as drawn.** Retune every `--ui-*` surface and ink. Highest impact, biggest
contrast-test churn, and it contradicts the comment above until the owner rewrites it.

**B. Keep paper, take the structure.** Implement the *layout* changes only — v1's prose
dashboard, v2's grouped rail, the settings ledger, the Media table with "Used on", the ⌘K
palette, the fading rules, monospace numerics — on the existing warm palette. Most of the
"not generic" complaint is structural, not chromatic; this gets ~80% of the redesign with
almost no contrast risk.

**C. Deepen the paper.** Keep the light ground, but take Nocturne's discipline: hairline
1px-gap grids instead of bordered cards, fading rules, tighter density, monospace numbers,
and the accent restricted to action + attention. A smaller, safer version of B.

**My recommendation: B, then revisit dark as its own slice.** The structure is what makes it
feel like a tool; the palette is what makes it risky.

If the owner picks A, these are the dark values to retune to (Nocturne, translated into
`--ui-*` names, and re-measured against `tests/contrast_test.php` before commit):

```
--ui-bg: #161826;         --ui-panel: #1a1d2a;      --ui-panel-sunken: #191b28;
--ui-ink: #e9e9ed;        --ui-ink-muted: #9397ab;   --ui-ink-faint: #75798c;
--ui-line: #2a2d3c;       --ui-line-strong: #3f424d; --ui-control-line: #595d6c;
--ui-control-soft: #3f424d;
--ui-accent: #9184d9;     --ui-accent-dark: #b5abfc; --ui-accent-soft: #2b2741;
--ui-on-accent: #161826;
--ui-bar: #12141f;        --ui-bar-ink: #e9e9ed;     --ui-bar-ink-muted: #a6aabb;
--ui-bar-raised: #262a3a;
--ui-success: #7fb98a;    --ui-danger: #e08d84;      --ui-warning: #d9a85c;
```

Two cautions on those: `--ui-accent-dark` gets *lighter* on a dark ground (the hover must
raise, not sink — Nocturne's rule and the repo's rule agree here), and `--ui-ink-faint`
at `#75798c` is ~4.6:1 on `#161826` but fails on the `-soft` tints; check it there first,
exactly as the existing comment on that token says it was derived.

---

## 4. Screen-by-screen: where the work lands

| Screen | Repo files | What changes |
| --- | --- | --- |
| Shell / nav | `app/Modules/Admin/views/layout.php`, `public/assets/admin-shell.css`, `public/assets/admin-nav.js` | v1: keep the bar; add the ⌘K trigger and the active-tab underline treatment (`.admin-nav [aria-current]` already does `inset 0 -2px 0 var(--ui-mark)` — the ember underline is **already there**). v2: replace the bar with a 216px rail — bigger change, and `admin-nav.js`'s phone fold needs rewriting as a collapse-to-icons. |
| Dashboard | `app/Modules/Admin/views/dashboard.php`, `public/assets/admin-dashboard.css` | v1: replace `.stat-grid` with the prose lede + the 3-panel strip; the controller must supply the sentence's parts (last-edited page + age, issue list). v2: replace it with the 6-metric grid + activity log. Both: the existing `$published/$drafts/$pictures/$menus/$character` payload already covers most metrics; `$stats` covers visitors. Activity log needs a new source — there is no audit table (see §5). |
| Pages | `app/Modules/Pages/views/admin/index.php`, `public/assets/admin-pages.css`, `public/assets/admin-tables.css` | Mostly CSS. The markup already has the drag handle, the group keying, the depth classes, the stale badge, the status switch and the delete form. Add: the language segmented filter (replacing the Locale column in v2), monospace on `.address` and `.date`, the quieter status fill, and the Home badge. |
| Media | `app/Modules/Media/views/admin/index.php`, `cards.php`, `public/assets/admin-media.css` | v1: restyle the existing card grid + add the No-alt badge and the All/Unused/No-description filter. v2: replace `cards.php` with a table — needs a new query for "used on N pages" (see §5). |
| Settings | `app/Modules/Settings/views/settings.php`, `public/assets/admin-forms.css` | The ledger layout: today each field is `label` + `input` + `.hint` stacked inside `.panel.stack`. Change to a 2-column grid row (label+hint left, control right) — this is a `.field` variant in `admin-forms.css` plus a wrapper class, not a rewrite. **Keep the four required `require` panels** (Languages, Mailer, Two-step, Stats) — the prototypes show none of them, and they are part of this screen. v2's sticky sub-nav should anchor to those panels. |
| ⌘K palette | new module or `app/Modules/Admin/` | New. See §5. |

**Note on the Design and Chrome screens:** `app/Modules/Design/views/design.php` (13.6 KB)
and `Settings/views/chrome.php` (9.8 KB) are the biggest admin screens and neither prototype
touches them. `CLAUDE.md` says *"The design layer is the product"* — so whichever direction
is chosen, the Design screen is the one that most deserves the treatment next. Budget for it.

---

## 5. What needs server work, not just CSS

Both prototypes show data the app does not currently have. Each is a real slice, with a
migration, per §"How to work" in `CLAUDE.md` (migration → model → controller → view → tests).

1. **The activity log** (v2's centrepiece, v1's "Pick up where you left off"). There is no
   audit table; `migrations/` runs to 0020 and none of them record who changed what.
   Needs `0021_activity.sql` — id, occurred_at, actor, kind, subject_type, subject_id,
   summary — written to from the existing controllers. Start with the five kinds the
   prototype shows (page, media, menu, form, design). **This is the single most valuable
   addition in either direction and also the most work; scope it as its own slice.**
   Cheap interim: derive a read-only "recent" list from `pages.updated_at` and
   `media.created_at` — no migration, no actor column, and it covers v1's four rows.

2. **"Used on N pages"** (v2's Media table). Requires counting media references in
   `page_blocks`. `App\Modules\Media\MediaReference` already exists and knows the reference
   shape — start there. Also makes the "Unused" filter trustworthy, which is what makes
   deletion safe.

3. **The "Needs attention" list.** Three cheap queries, no schema change: pictures with no
   `alt` (the `media_meta` / `media_alt_suggested` migrations already model alt text),
   `site_favicon` unset in settings, and stale translations — which **already exists** as
   `$stale` in the Pages controller. Reuse it.

4. **The ⌘K palette.** Needs one JSON endpoint (`/admin/search?q=`) over pages, media,
   menus, settings fields and a static action list, plus a small script. It must degrade:
   `CLAUDE.md` demands no control that only works with JS, so the trigger should be a real
   link to a `/admin/search` page that renders the same results server-side.

5. **Per-screen figures for the metric strip** (v2): form entries + unread count, language
   count, "design unchanged N days". All present in the schema (`forms`, `locales`,
   `design_tokens`); just not queried by the dashboard controller yet.

---

## 6. Before committing, per the repo's own rules

- `php tests/run.php` — and expect `tests/contrast_test.php` to be the one that fails first
  on any token change. It pairs every ink with every surface; a new token must be added to
  its matrix, not excluded from it.
- `vendor/bin/phpstan analyse` (level 8).
- `tests/chrome_look_test.php` and `tests/icons_test.php` exist and will notice bar and
  sprite changes.
- A visual change needs a screenshot; `tools/browser-suite/` gets a scenario per area.
- Record the decision in `PLAN.md`. Which direction was chosen, and the light/dark call from
  §3, are exactly the kind of thing `CLAUDE.md` says must be written there: *"a decision
  that is not written there did not happen."*
- The buttons-stay-filled resolution in §2.2 is a deliberate deviation from the design
  system. `CLAUDE.md`: *"State when you exceed an instruction."* Say it in the commit
  message.
