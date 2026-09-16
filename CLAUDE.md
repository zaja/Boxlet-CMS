# Boxlet

A small self-hosted PHP CMS for people who build many small sites. One admin, no user
accounts, no visitor-facing login. Runs on shared hosting with no shell, no Composer,
no build step at install time.

The differentiator is the design layer, not the feature list. Everything else stays
boring and small.

**Read `docs/SPEC.md` before starting a new slice.** It holds the full schema, the block
contract, the design layer model and the build order. Do not re-derive any of it from
memory.

**Read `docs/plan.md` to find out where the work actually stands** — what is committed,
what is half-finished in the working tree, and what was verified versus assumed.

`docs/product.md` describes the product in functional terms — what it does today, what
is planned, and what it will deliberately never do. It carries no technical detail and
decides nothing: where it and `docs/SPEC.md` disagree, the spec wins and the product
document is what needs correcting.

---

## Non-negotiable

**Runtime dependencies are a closed list.** `composer require` holds only
nikic/fast-route, vlucas/phpdotenv, symfony/mailer, monolog/monolog,
spomky-labs/otphp, bacon/bacon-qr-code. Nothing else. No framework, no ORM, no Twig,
no imaging library, no Tailwind, no Alpine, no HTMX. If you think something needs a
new runtime dependency, stop and ask.

**Dev tools live in `require-dev`.** They never ship in the release ZIP (built with
`--no-dev`) and never exist on a user's server. They are allowed when they earn their
place, but ask first. Currently in use: PHPStan (level 8, `vendor/bin/phpstan analyse`).

**Vendored front-end assets** are allowed when they earn their place, but ask first.
They must be MIT or similarly permissive, dependency-free, distributed as a single file,
committed to the repository, and loaded with a plain script or link tag. No npm, no
build step, no CDN. Record the version and source in the file header and in the README.

Currently vendored:

```
sortablejs 1.15.6   MIT   reordering blocks inside the editor canvas
trix       2.1.19   MIT   the rich text editor (see "Rich text" below)
```

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
0  Character   one preset (Editorial / Minimal / Bold / Soft / Brutalist):
               layer-1 values AND the composition it gives a page
1  Tokens      eight decisions: seed colours, type pairing, scale ratio,
               spacing unit, radius character, shadow character, container width
2  Section     per block instance: surface, rhythm, width, align, divider
3  Layout      per block type, from its block.php 'layouts'
```

Layers 0 and 1 compile to `public/cache/tokens.{hash}.css` on save. Layers 2 and 3
render as class names on the section wrapper. A character also sets the layer-2 and
layer-3 defaults new blocks start from; applying one to a site that already has pages
always asks whether to reset existing section styles.

**No control is ever invisible at rest.** Every interactive control — button, link,
toggle, insertion handle — has a legible resting state: readable text, or a visible
shape, against the surface it sits on. Hover and focus *raise* a control; they never
*reveal* it. A control nobody can see is a control nobody uses, and it hides bugs: a
white-on-white button looks like a missing feature, not like a styling mistake.

This has already been got wrong twice — the hover-only insertion controls in the canvas,
and ghost buttons in the toolbar whose anchor colour was outranked by `.admin a.button`.
When fixing an instance of it, fix the rule.

**The admin has its own fixed design system** and never links the site's tokens.css.
Its tokens are `--ui-*`, defined as literal values in `public/assets/admin*.css`. The
rule runs both ways: a front-end stylesheet or block template containing a literal
colour or size is a bug, and an admin stylesheet reading a site token is a bug.

**A block template containing a hard-coded colour, pixel value, font or shadow is a
bug.** Everything goes through CSS custom properties.

Generated palettes must pass a WCAG AA contrast check on every text/background pair
and refuse combinations that fail, with a message saying which pair failed.

---

## Rich text

Rich text is edited with **Trix**, and stored as HTML conforming to the whitelist in
`app/Support/RichText.php`. The editor is a convenience; **the server-side whitelist is
the security boundary and the storage contract**, and it sanitises on save regardless of
what arrives.

What that means in practice:

- **Trix's output is normalised on save**, in the sanitiser where every other rule lives:
  `div → p` (its block wrapper), `h1 → h2` (it offers one heading level, and the page's
  own title is the h1), `h4–h6 → h3`.
- `div → p` is **conditional**: a div holding a block is unwrapped instead, because a
  paragraph may not contain a list and renaming regardless would generate invalid markup
  ourselves.
- **Attachments are disabled entirely** — no drop, no paste, no button. Trix's attachment
  attribute carries JSON and is its one proprietary format. The sanitiser strips
  attachment markup as a backstop, so a later version cannot reintroduce it silently.
  Images come from the media library.
- **The toolbar offers only what the whitelist permits.** No strike (`del`), no code
  (`pre`). A button whose output is discarded on save is worse than no button.
- **The textarea is the real field.** It carries the `name`; JavaScript moves the name to
  a hidden input and puts Trix above it. Without JavaScript the field is still editable,
  and the plain-HTML toggle is simply what was underneath.

An editor's *internal document model* is not lock-in; only its *storage format* is. That
distinction is why Trix is acceptable and Quill is not, and it should not be relitigated
— see the changelog.

---

## Media

Variants are generated **on upload, never on demand**, so a request for one is always a
request for a file that exists and serving never touches PHP. This is not a preference:
a managed nginx answers a request for a missing `.webp` from disk with its own 404 and
PHP never runs.

Because of that, **a half-generated item is the worst outcome available** — its missing
variants 404 for ever. Generation is therefore resumable: priority order (`thumb`, `card`
first), a record of which variants exist, a check of remaining execution time before each
encode, and an item marked incomplete that offers to finish. Up to fifteen encodes per
image will exceed `max_execution_time` on a slow shared host.

Imagick is the primary encoder, GD the fallback, AVIF best-effort. EXIF orientation is
applied *before* cropping and stripped from the output.

---

## Working practice that has already cost time twice

**Fix the instrument before judging the subject.** The headless browser misreports both
what it captures and what it types. Screenshot artifacts have twice looked like product
defects, and in evaluating Trix, three of four "findings" — lost characters, shredded
text, broken redo — were the harness driving the editor faster than it re-renders. Code
written to work around a phantom survives for ever carrying a comment that explains the
wrong reason, which is worse than the bug because it looks deliberate.

So: slow the driver, drive through real input rather than an API, and **use a control** —
the same input through a plain element with no library involved. That is what turned
"Trix mangles Word paste" into "Trix is far better than the browser's own behaviour".

**State when you exceed an instruction.** Extending a rule (demoting `h4–h6` as well as
`h1`) is initiative and is welcome; doing it silently is drift. Say which it is.

**Do not adjust a test to match new output.** If a test now contradicts intended
behaviour, change the rule deliberately and say so — splitting the case if it covered two
things. Especially for round-trip and idempotence tests, whose whole value is catching
what the eye cannot see.

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
3. **After any schema change**, run `php tests/run.php` with the MySQL test database
   configured (`.env.test`), so migrations run on both drivers. Then add the demo site
   to an empty install (`php migrations/seed.php`, or the installer's demo option) and
   look at it under each design character.
   All SQL must be portable between MySQL and SQLite; see SPEC §5.0.
4. **Code files under 300 lines.** Applies to PHP, templates, CSS and JS, not to
   documentation such as `docs/SPEC.md`. A controller past that means the feature is
   too big.
5. **Every slice adds tests for what it builds.** The acceptance criteria in SPEC §8
   are the starting point for what to assert. Run `php tests/run.php`; see SPEC §10.
6. Commit at the end of each slice, message naming the slice.

---

## The live site's database is not a scratchpad

**Never run an ad-hoc INSERT, UPDATE or DELETE against `boxletcms`.** Not to clean up
after a browser check, not to fix a row by hand, not "just this once".

Cleanup happens one of three ways: through the application, through the test suite
against `boxletcms_test`, or by reinstalling.

Throwaway data on the live site is created with a marker chosen in that same command —
a fixed prefix, an id captured on creation — and deleted **by exact id**. Never by a
`LIKE` pattern over user-facing text: a title is something a person can be halfway
through typing, and an unsaved form field is not a safeguard.

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
app/         Core, Modules (Pages, Install, Auth, Admin, ...), Blocks, Support
config/      storage/      lang/      migrations/      vendor/
docs/SPEC.md the full specification
```
