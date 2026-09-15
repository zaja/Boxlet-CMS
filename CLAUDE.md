# Boxlet

A small self-hosted PHP CMS for people who build many small sites. One admin, no user
accounts, no visitor-facing login. Runs on shared hosting with no shell, no Composer,
no build step at install time.

The differentiator is the design layer, not the feature list. Everything else stays
boring and small.

**Read `docs/SPEC.md` before starting a new slice.** It holds the full schema, the block
contract, the design layer model and the build order. Do not re-derive any of it from
memory.

---

## Non-negotiable

**Runtime dependencies are a closed list.** `composer require` holds only
nikic/fast-route, vlucas/phpdotenv, symfony/mailer, monolog/monolog,
spomky-labs/otphp, bacon/bacon-qr-code. Nothing else. No framework, no ORM, no Twig,
no imaging library, no Tailwind, no Alpine, no HTMX. If you think something needs a
new runtime dependency, stop and ask.

**Dev tools live in `require-dev`.** They never ship in the release ZIP (built with
`--no-dev`) and never exist on a user's server. They are allowed when they earn their
place, but ask first. Currently in use: PHPStan (level 6, `vendor/bin/phpstan analyse`).

**Frozen contracts.** The URL scheme, database schema, block definition format and
design token schema are in `docs/SPEC.md` §5. Changing any of them after v0.1 breaks
every install in the wild. Stop and ask before touching them.

**PHP 8.1 syntax only.** No 8.2+ features. Target is shared hosting.

**No abstraction without a second caller.** No interface with one implementation, no
hooks system before a second module needs it, no repository layer over PDO. If it has
one caller, inline it.

---

## The design layers

This is the product. Four layers, applied in order:

```
0  Character   one preset (Editorial / Minimal / Bold / Soft / Brutalist)
1  Tokens      eight decisions: seed colours, type pairing, scale ratio,
               spacing unit, radius character, shadow character, container width
2  Section     per block instance: surface, rhythm, width, align, divider
3  Layout      per block type, from its block.php 'layouts'
```

Layers 0 and 1 compile to `public/cache/tokens.css`. Layers 2 and 3 render as class
names on the section wrapper.

**A block template containing a hard-coded colour, pixel value, font or shadow is a
bug.** Everything goes through CSS custom properties.

Generated palettes must pass a WCAG AA contrast check on every text/background pair
and refuse combinations that fail, with a message saying which pair failed.

---

## Multilingual

Locale is in the router from the first commit. There is no code path that renders a
page without knowing its locale.

- Pages link across locales via `content_group_id`, blocks via `block_group_id`.
- Each translated row stores `source_hash`. When the source changes, the hash no longer
  matches and that block alone shows as stale. Re-translation is never all-or-nothing.
- Every admin string goes through `t('key')` and lands in `lang/en.php`. No bare
  English in a template.

---

## How to work

1. **Vertical slices.** Every slice ends with something visible in a browser. Do not
   build all of Core before anything renders.
2. **Migration first**, then model, then controller, then view.
3. **After any schema change**, run `php migrations/seed.php` and load the demo site.
   That is the primary regression check.
4. **Code files under 300 lines.** Applies to PHP, templates, CSS and JS, not to
   documentation such as `docs/SPEC.md`. A controller past that means the feature is
   too big.
5. **Every slice adds tests for what it builds.** The acceptance criteria in SPEC §8
   are the starting point for what to assert. Run `php tests/run.php`; see SPEC §10.
6. Commit at the end of each slice, message naming the slice.

---

## Security rules that are easy to forget

- CSRF token on every state-changing request.
- Media URLs use named presets only (`thumb`, `card`, `wide`, `hero`, `full`). Never
  accept free-form dimensions from the URL.
- Cached images and pages are served by an .htaccess file check, bypassing PHP on hit.
- Uploads: finfo MIME sniff, extension whitelist, `.htaccess` in `/uploads` disabling
  script execution.
- Form submissions store a hashed IP, never the raw address.
- 2FA is optional, with ten recovery codes and a documented FTP reset. Never force it.
- `install.php` writes `storage/install.lock`, refuses to re-run, tries to delete
  itself, warns loudly if it cannot.

---

## Layout

```
public/      document root: index.php, install.php, assets, uploads, cache
app/         Core, Modules, Admin, Blocks, Support
config/      storage/      lang/      migrations/      vendor/
docs/SPEC.md the full specification
```
