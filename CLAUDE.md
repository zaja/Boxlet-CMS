# Boxlet

A small self-hosted PHP CMS for people who build many small sites. One admin, no user
accounts, no visitor-facing login. Runs on shared hosting with no shell, no Composer,
no build step at install time.

The differentiator is the design layer, not the feature list. Everything else stays
boring and small.

**Read `PLAN.md` first.** It is the one document that says what Boxlet is, where the work
stands, what comes next, what has been decided and what is still open. Only the architect
session writes it: you report progress by message and never edit it.

**Read `docs/SPEC.md` before starting a slice.** It is the contract — the schema, the
block contract, the design layer model, the acceptance criteria. Do not re-derive any of
it from memory.

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

**Vendored front-end assets** are allowed when they earn their place, but ask first. The
rule they must satisfy, and the list of what is vendored today, are in `docs/SPEC.md` §3.
One exception to "no npm, no build step" exists, for maintainers only: the TipTap bundle,
rebuilt from `tools/tiptap/` outside the project (PLAN.md D-017). A second one needs its
own decision.

**Frozen contracts.** The URL scheme, database schema, block definition format and
design token schema are in `docs/SPEC.md` §5. Changing any of them after v0.1 breaks
every install in the wild. Stop and ask before touching them.

**PHP 8.1 syntax only.** No 8.2+ features. Target is shared hosting.

**No abstraction without a second caller.** No interface with one implementation, no
hooks system before a second module needs it, no repository layer over PDO. If it has
one caller, inline it.

---

## Design

The four layers, the eight decisions, what compiles to `tokens.css`, the contrast rule and
the admin's own `--ui-*` token set are all in `docs/SPEC.md` §5.4.

**No control is ever invisible at rest.** Every interactive control — button, link,
toggle, insertion handle — has a legible resting state: readable text, or a visible
shape, against the surface it sits on. Hover and focus *raise* a control; they never
*reveal* it. A control nobody can see is a control nobody uses, and it hides bugs: a
white-on-white button looks like a missing feature, not like a styling mistake.

This has already been got wrong twice — the hover-only insertion controls in the canvas,
and ghost buttons in the toolbar whose anchor colour was outranked by `.admin a.button`.
When fixing an instance of it, fix the rule.

---

## Rich text

TipTap, stored as HTML conforming to the whitelist in `app/Support/RichText.php`. The
server sanitiser is the security boundary and keeps one stored shape whatever the editor
sends. What is normalised on save, and why the toolbar offers only what the whitelist
permits, are in `docs/SPEC.md` §5.3.

---

## Media

The model — variants generated on upload and never on demand, the five presets, resumable
generation, EXIF orientation — is in `docs/SPEC.md` §5.1 and §5.5.

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
page without knowing its locale. How pages and blocks link across locales, and how a
translation goes stale, are in `docs/SPEC.md` §5.2.

**Every admin string goes through `t('key')` and lands in `lang/en.php`.** No bare
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
7. **A slice is done when the architect has verified it working in a browser** against
   their checklist — not when it is committed, and not when the tests pass (PLAN.md
   D-006).
8. **A weak feature is fixed before anything is built on top of it.** No workaround
   ships as a solution: if the right fix is too big for now, it is recorded in PLAN.md
   as an open item rather than papered over.
9. **Tasks from the architect session that cite an approved PLAN.md entry are followed
   as written.** Only when the cited PLAN.md entry is marked approved and actually
   covers what the task asks; anything beyond the entry goes back to the architect.
   Ambiguity, or a conflict with the code or `docs/SPEC.md`, goes back to the architect
   by message, not to the owner. The owner is asked only about `CLAUDE.md`, permissions
   or configuration, and about anything PLAN.md marks as the owner's call.
10. **Push `main` to origin after a commit whose tests pass on both drivers and whose
    PHPStan run is clean**, without asking. Standing permission from the owner, given
    2026-09-16. If either check did not run, or did not pass, the commit stays local and
    the owner is told why — the permission is for verified work, not for every commit.

---

## The live site's database is not a scratchpad

**Never run an ad-hoc INSERT, UPDATE or DELETE against `boxletcms`.** Not to clean up
after a browser check, not to fix a row by hand, not "just this once".

Cleanup happens one of three ways: through the application, through the test suite
against `boxletcms-test`, or by reinstalling.

Throwaway data on the live site is created with a marker chosen in that same command —
a fixed prefix, an id captured on creation — and deleted **by exact id**. Never by a
`LIKE` pattern over user-facing text: a title is something a person can be halfway
through typing, and an unsaved form field is not a safeguard.

---

## Security rules that are easy to forget

- CSRF token on every state-changing request.
- Media URLs use named presets only (`thumb`, `card`, `wide`, `hero`, `full`). Never
  accept free-form dimensions from the URL.
- Cached images and pages are served without touching PHP on a hit. How that is done on
  both nginx and Apache is open — PLAN.md O-2.
- Uploads: finfo MIME sniff, extension whitelist, `.htaccess` in `/uploads` disabling
  script execution.
- Form submissions store a hashed IP, never the raw address.
- 2FA is optional, with ten recovery codes and a documented FTP reset. Never force it.
- `install.php` writes `storage/install.lock`, refuses to re-run, tries to delete
  itself, warns loudly if it cannot.

---

## Layout

```
public/      document root: index.php, install.php, assets, uploads, m, cache
app/         Core, Modules (Pages, Install, Auth, Admin, ...), Blocks, Support
config/      storage/      lang/      migrations/      vendor/
PLAN.md      what it is, where it stands, what is decided and open
docs/SPEC.md the full specification
```
