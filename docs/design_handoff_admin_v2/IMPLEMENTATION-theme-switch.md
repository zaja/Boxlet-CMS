# Light theme + theme switch — Boxlet CMS

Companion to `IMPLEMENTATION-boxlet.md`. Read after it.

Good news: `public/assets/admin.css` already scopes its whole `--ui-*` set to `.admin { … }`,
so a second theme is a second token block — not a second stylesheet, and not a single line of
new CSS anywhere else. If a literal color appears in any admin stylesheet other than
`admin.css`, that is the bug to fix first; a theme switch is only possible if every color in
the admin resolves through a token.

## 1. Where the themes live

Keep `.admin { … }` as the **default (light / warm paper)** block — it already is one, and it
stays the fallback for a request with no preference and no JS.

Add two blocks below it in the same file:

```css
/* the dark set — same token names, dark values */
.admin[data-ui-theme="dark"] { --ui-bg: #161826; … }

/* follow the OS when asked to */
@media (prefers-color-scheme: dark) {
  .admin[data-ui-theme="system"] { --ui-bg: #161826; … }
}
```

Two blocks, one value list. Put the dark values in a CSS custom-property list you write once
and `@media`-duplicate — there is no way around the duplication in plain CSS, and the repo
takes no build step, so duplicate it honestly with a comment saying the two must move
together. A `tests/` check that the two blocks declare the same token names is cheap and
worth writing.

Dark values, translated into the existing `--ui-*` names (measure before commit, see §5):

```css
--ui-bg: #161826;          --ui-panel: #1a1d2a;       --ui-panel-sunken: #191b28;
--ui-ink: #e9e9ed;         --ui-ink-muted: #9397ab;   --ui-ink-faint: #75798c;
--ui-line: #2a2d3c;        --ui-line-strong: #3f424d; --ui-control-line: #595d6c;
--ui-control-soft: #3f424d;
--ui-accent: #9184d9;      --ui-accent-dark: #b5abfc; --ui-accent-soft: #2b2741;
--ui-on-accent: #161826;
--ui-bar: #12141f;         --ui-bar-ink: #e9e9ed;     --ui-bar-ink-muted: #a6aabb;
--ui-bar-raised: #262a3a;
--ui-success: #7fb98a;     --ui-danger: #e08d84;      --ui-warning: #d9a85c;
--ui-mark: #ff6b3d;
```

Three notes on those:

- **`--ui-accent-dark` gets lighter, not darker.** On paper the hover step sinks; on a dark
  ground it must rise. The token name now lies in one of the two themes — rename it
  `--ui-accent-hover` while you are in there.
- **`--ui-on-accent` flips** from paper-white to the dark ground, because the accent itself
  lightens. Any place that assumed white-on-accent breaks.
- **`--ui-mark` (the orange) stays.** It is the brand, it reads on both grounds, and it is the
  one warm value in either theme. Do not re-tune it per theme.

Shadows need per-theme values too: on paper they are ink-tinted drops; on the dark ground
they are a hairline edge plus ambient darkness (`0 0 0 1px` + a low-alpha black). Same token
names, different composition.

## 2. The switch, without breaking the no-JS rule

`CLAUDE.md` forbids a control that only works with JS, and the CSP forbids inline script, so
the switch is **a form, server-rendered, three radios**:

```html
<form method="post" action="/admin/appearance" class="theme-switch">
  <?= csrf_field() ?>
  <button name="theme" value="light"  aria-pressed="<?= $theme === 'light'  ? 'true' : 'false' ?>">…</button>
  <button name="theme" value="dark"   aria-pressed="<?= $theme === 'dark'   ? 'true' : 'false' ?>">…</button>
  <button name="theme" value="system" aria-pressed="<?= $theme === 'system' ? 'true' : 'false' ?>">…</button>
</form>
```

Three buttons, not two — **Light / Dark / Match system**. A two-state toggle cannot express
"follow the OS", which is what most people actually want, and a toggle whose label changes
meaning as you press it ("Dark" — is that the current state or the thing it does?) is exactly
the kind of control `CLAUDE.md` calls invisible at rest.

Server side:

1. `POST /admin/appearance` validates against `['light','dark','system']`, stores it, and
   redirects back to the referring admin page. A full page load on theme change is completely
   acceptable here — it happens roughly once per person per install.
2. `layout.php` reads it and prints `data-ui-theme="<?= e($theme) ?>"` on the existing
   `.admin` element. That is the whole render path.
3. Where to store it: **per user**, in the `users` table (`ui_theme` varchar, default
   `'system'`) — a new migration `0022_user_ui_theme.sql`. Not a site setting: two editors on
   one site will disagree, and the admin chrome is not site content. If the admin has no
   session for it yet, a cookie is an acceptable fallback, but the column is the right home.

**Progressive enhancement (optional).** A tiny listener in `admin-nav.js` can flip the
attribute client-side and post in the background, so the change is instant. The buttons must
keep working with the script absent — which they do, because they are form submits.

**No flash.** Because the attribute is server-rendered into the layout, there is no
first-paint flash and no need for the usual blocking inline script — which the CSP would have
blocked anyway. This is the one place the repo's strictness pays you back.

Place the switch in the bar's right cluster, next to sign-out. Not in Settings: it is a
personal view preference, not site configuration. (A line in Settings → General pointing at
it is fine.)

## 3. Icons and copy

Add to `ICONS` in `tools/icons/build.php`, then re-run it: `sun`, `moon`, `contrast`
(Lucide's name for the half-filled circle — the "system" glyph).

New keys in `lang/en/shell.php`:

```php
'appearance'        => 'Appearance',
'appearance_light'  => 'Light',
'appearance_dark'   => 'Dark',
'appearance_system' => 'Match system',
'appearance_hint'   => 'Only changes how the admin looks to you.',
```

The hint matters. People fear that a control in the admin changes their public site; say that
it does not.

## 4. What breaks, and what to check

- **Anything with a literal color outside `admin.css`.** Grep the per-screen stylesheets for
  `#`. Each hit is a theme bug.
- **`--ui-on-accent` assumptions** — filled buttons, the accent-backed badges.
- **The dark bar** stops being "the one dark surface" in dark mode; it now needs to be
  *darker than* the ground (`#12141f` against `#161826`) or the frame disappears. That is why
  `--ui-bar` is not simply reused.
- **Images and the media grid.** Thumbnails were composed against paper. Photographs with
  white backgrounds will glare on the dark ground; give `.media-thumb` a per-theme ground and
  check a few real uploads.
- **The chrome preview** (`Settings/views/chrome.php`) renders the *site's* colors inside the
  admin. It must not inherit the admin theme — scope it so admin tokens stop at its edge, or
  the preview lies. `tests/chrome_look_test.php` should be extended to assert this under both
  themes.
- **`tests/contrast_test.php` is the gate.** It pairs every ink with every surface; it must
  now run the matrix **twice**, once per theme, and both must pass. Expect the faint inks to
  be what fails — `--ui-ink-faint` on a `-soft` tint is the classic casualty.
- **Screenshots**: `tools/browser-suite/` scenarios should capture both themes for any
  screen you touch, or the second theme silently rots.

## 5. Order of work

1. Grep for literal colors outside `admin.css`; fix them. Nothing else can start before this.
2. Make `contrast_test.php` theme-aware (it will pass — one theme, unchanged).
3. Add the dark block + the `prefers-color-scheme` block. Run the contrast test. Fix the
   failures in the token values, not by excluding pairs.
4. Migration + controller + the `data-ui-theme` attribute in the layout.
5. The switch in the bar, with `t()` keys and the two new icons.
6. Screenshots of both themes; note the decision in `PLAN.md`.

Steps 1–3 are worth doing even if the switch never ships: a themeable admin is a well-built
admin, and the test that proves it is the deliverable.
