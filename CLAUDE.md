# Boxlet

A small self-hosted PHP CMS for people who build many small sites. One admin, no user
accounts, no visitor-facing login. Runs on shared hosting with no shell, no Composer,
no build step at install time.

The differentiator is the design layer, not the feature list. Everything else stays
boring and small.

**Read `PLAN.md` first.** It says what Boxlet is, where the work stands, what comes next,
what has been decided and what is still open. Keep it current as you go: a decision that
is not written there did not happen.

**Read `docs/SPEC.md` before starting a slice.** It is the contract — the schema, the
block contract, the design layer model, the acceptance criteria. Do not re-derive any of
it from memory.

---

## How this project is worked on

One session, working directly in this checkout, which is the development site at
https://boxlet.svejedobro.hr. There is no separate development clone and no deploy step:
what you save is what the site runs. Commit and push to origin as you go.

The site is a demo with no real content. It may be reinstalled. The database is
`boxletcms`; `boxletcms-test` belongs to the test suite and is wiped by it.

**Pending migrations are applied from the command line:** `php migrations/migrate.php`
(`--check` lists them and exits 1). After adding a migration, or pulling code that carries
one, run it at once. The admin's "Database update needed" screen (PLAN.md D-019) is for
real sites; the owner never presses it for development.

Ask the owner about: anything they will see and judge (layout, wording, look), product
decisions, this file, permissions and configuration. Everything else is yours — decide it,
do it, and record the decision in `PLAN.md`.

---

## Non-negotiable

**Runtime dependencies are a closed list.** `composer require` holds only
nikic/fast-route, vlucas/phpdotenv, symfony/mailer, monolog/monolog,
spomky-labs/otphp, bacon/bacon-qr-code. Nothing else. No framework, no ORM, no Twig,
no imaging library, no Tailwind, no Alpine, no HTMX. If you think something needs a
new runtime dependency, stop and ask.

**Dev tools live in `require-dev`.** They never ship in the release ZIP and never exist on
a user's server. Currently: PHPStan (level 8, `vendor/bin/phpstan analyse`).

**Vendored front-end assets** are allowed when they earn their place, but ask first. The
rule and the list are in `docs/SPEC.md` §3. One exception to "no npm, no build step"
exists, for maintainers only: the TipTap bundle, rebuilt from `tools/tiptap/` outside the
project (PLAN.md D-017). A second one needs its own decision.

**Frozen contracts.** The URL scheme, database schema, block definition format and design
token schema are in `docs/SPEC.md` §5. Changing any of them after v0.1 breaks every
install in the wild. Before v0.1 they may change deliberately: say so, update SPEC, and
record it in PLAN.md.

**PHP 8.1 syntax only.** No 8.2+ features. Target is shared hosting.

**No abstraction without a second caller.** No interface with one implementation, no hooks
system before a second module needs it, no repository layer over PDO. If it has one
caller, inline it.

---

## Design

The four layers, the eight decisions, what compiles to `tokens.css`, the contrast rule and
the admin's own `--ui-*` token set are all in `docs/SPEC.md` §5.4.

**The design layer is the product.** When a choice is between another feature and making
the design layer richer or the admin better to look at, the design wins.

**No control is ever invisible at rest.** Every interactive control — button, link,
toggle, insertion handle — has a legible resting state: readable text, or a visible shape,
against the surface it sits on. Hover and focus *raise* a control; they never *reveal* it.

**The admin has its own fixed design system** (`--ui-*`), never the site's tokens. A
front-end stylesheet with a literal colour is a bug; an admin stylesheet reading a site
token is a bug. `tests/contrast_test.php` enforces the contrast rules.

---

## Rich text

TipTap, stored as HTML conforming to the whitelist in `app/Support/RichText.php`. The
server sanitiser is the security boundary and keeps one stored shape whatever the editor
sends. What is normalised on save, and why the toolbar offers only what the whitelist
permits, are in `docs/SPEC.md` §5.3.

---

## Media

The model — variants generated on upload and never on demand, the six presets, resumable
generation, EXIF orientation, originals outside the web root — is in `docs/SPEC.md` §5.1
and §5.5.

---

## Multilingual

Locale is in the router from the first commit. There is no code path that renders a page
without knowing its locale. How pages and blocks link across locales is in `docs/SPEC.md`
§5.2.

**Every admin string goes through `t('key')` and lands in a file under `lang/en/`.** No
bare English in a template. The files are split by concern; `t()` merges them, so a new
concern gets a new file rather than growing an existing one.

---

## Working practice that has already cost time

**Fix the instrument before judging the subject.** The headless browser misreports both
what it captures and what it types; a test copy can be running older code; a probe can
report success it never earned. Before believing a defect, check the tool: slow the
driver, drive through real input, and use a control — the same input through a plain
element with no library involved.

**State when you exceed an instruction.** Extending a rule is initiative and is welcome;
doing it silently is drift. Say which it is.

**Do not adjust a test to match new output.** If a test contradicts intended behaviour,
change the rule deliberately and say so, splitting the case if it covered two things.

**A claim is measured, not reasoned.** Do not write in a commit message, a comment or a
report anything you have not checked. Several times a rule that never applied was believed
because a number moved for another reason.

---

## How to work

1. **Vertical slices.** Every slice ends with something visible in a browser.
2. **Migration first**, then model, then controller, then view. All SQL portable between
   MySQL and SQLite (SPEC §5.0); a committed migration is never edited, only followed by
   a new one.
3. **After any schema change**, run `php tests/run.php` with `.env.test` configured, so
   migrations run on both drivers.
4. **Keep code files small.** Past 300 lines, split along a real seam of concern; never
   shorten comments or code just to fit. Hard limit 500. Tests, language files and the
   browser suite are exempt.
5. **Every slice adds tests for what it builds**, starting from the acceptance criteria in
   SPEC §8.
6. **Before each commit**, run `php tests/run.php` and `vendor/bin/phpstan analyse`, and
   check every claim in the commit message against `git diff --cached`. Push to origin.
7. **CI must stay green.** Read the conclusion without admin rights:
   `curl -s https://api.github.com/repos/zaja/Boxlet-CMS/actions/runs?per_page=1`.
   A red run is fixed before new work; failures are readable in the check-run annotations.
8. **Browser checks are a reusable suite** in `tools/browser-suite`, run with one command,
   one scenario file per area. A new feature adds its scenario. Verify in proportion:
   a small logic change needs the test suite; a visual change needs a screenshot; the
   whole suite belongs at the end of a slice. Scenarios that install a site or lock the
   account run against a copy, never the development site.
9. **NOT CHECKABLE means a limit of the environment.** Missing test data is a failure.
10. **A weak feature is fixed before anything is built on top of it.** No workaround ships
    as a solution: if the right fix is too big now, record it in PLAN.md as an open item.
11. **Never end a turn with a list of next steps. Run them.** A turn ends when the task is
    done, when you are blocked on the owner, or when you are reporting. A background
    process such as a dev server is not work in progress: stop it when the check that
    needed it is done.
12. **Show the owner what they will judge.** Anything visual: a screenshot, and for design
    work more than one character.

---

## The database is not a scratchpad

**Never run an ad-hoc INSERT, UPDATE or DELETE against `boxletcms`.** Not to clean up after
a browser check, not to fix a row by hand, not "just this once". Cleanup happens through
the application, through the test suite against `boxletcms-test`, or by reinstalling.

Throwaway data is created with a marker chosen in that same command and deleted **by exact
id**. Never by a `LIKE` pattern over user-facing text.

---

## Security rules that are easy to forget

- CSRF token on every state-changing request.
- Media URLs use named presets only (`thumb`, `card`, `natural`, `wide`, `hero`, `full`). Never accept
  free-form dimensions from the URL.
- Cached images and pages are served without touching PHP on a hit. How that is done on
  both nginx and Apache is open — PLAN.md O-2.
- Uploads: finfo MIME sniff and extension whitelist. Originals live outside the web root in
  `storage/uploads/`; only generated variants are public.
- Form submissions store a hashed IP, never the raw address.
- 2FA is optional, with ten recovery codes and a documented FTP reset. Never force it.
- `install.php` writes `storage/install.lock`, refuses to re-run, tries to delete itself,
  warns loudly if it cannot.

---

## Layout

```
public/      document root: index.php, install.php, assets, m, cache
app/         Core, Modules (Pages, Media, Design, Menus, Chrome...), Blocks, Support
config/      storage/      lang/      migrations/      tools/      vendor/
PLAN.md      what it is, where it stands, what is decided and open
docs/SPEC.md the full specification
```
