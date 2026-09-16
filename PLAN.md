# Boxlet — architectural decisions

The decision log. Written by the architect session only; the owner approves every entry
before it is acted on. A decision is not real until it is here.

- `docs/SPEC.md` is the technical contract. `docs/plan.md` is the executor's progress
  report. `docs/product.md` is the functional description. This file does not copy them;
  each decision names which of them must be updated to reflect it ("Reflect in").
- Status is one of: **proposed**, **approved** (by the owner, with date), **superseded**
  (by a later entry, named).

---

## Context from the owner (2026-09-16)

- Boxlet serves two users equally: the owner's own client sites, and a public release.
  Frozen contracts (SPEC §5) are therefore genuinely frozen once v0.1 ships, and a safe
  upgrade path for existing installs is required, not optional.
- Hosting: **nginx is the primary target; full Apache compatibility is required.**
- Admin 2FA must be in the build order (not now), with a file-based unlock over FTP if
  access is lost.
- Work runs in two sessions: architect (planning) and executor (code). Architectural
  messages to the executor are sent only after the owner confirms.

---

## D-001 — Where decisions are recorded

**Status:** approved 2026-09-16

This file, in the repository root, holds architectural decisions only. The architect
writes it; the executor reads it and reflects each decision into the documents it names.

**Trade-offs.** One more document, when documents already drift from code. Accepted
because it separates who writes what: the owner, who does not review code, can follow
decisions here without reading the spec. Drift is contained by the "Reflect in" line on
every entry and by the architect checking that the reflection happened.

**Reflect in:** `CLAUDE.md` (a pointer to this file beside the existing pointers to
`docs/SPEC.md` and `docs/plan.md`).

---

## D-002 — The role of each database

**Status:** approved 2026-09-16

- `boxletcms-test` is for the automated test suite only. The suite drops every table
  before each test, so it can never hold anything a person looks at.
- `boxletcms` (https://boxlet.svejedobro.hr) is a demo and acceptance install with no real
  content. It may be reinstalled. The existing rule stands: no ad-hoc INSERT, UPDATE or
  DELETE against it.
- No third database for now.
- Revisit when the first real client site is built on Boxlet.

**Trade-offs.** Reinstalling on a schema change is cheap today but wipes the demo and the
chosen design each time. That cost is what open item O-1 is about.

**Reflect in:** `CLAUDE.md` names the test database `boxletcms_test`; the real name is
`boxletcms-test`.

---

## D-003 — Media variants live in `public/m/`

**Status:** approved 2026-09-16

Generated image sizes are stored under `public/m/`, the same path as their address
`/m/{preset}/{id}-{slug}.{ext}`. The web server can then send the file straight from disk
without running PHP. The `/public/cache/media/` path in SPEC §5.5 was left over from the
abandoned on-demand model.

**Trade-offs.** Generated files sit outside `public/cache/`, so "clear the cache" does not
clear them, which is correct: they are only regenerated deliberately (Slice 8).

**Reflect in:** SPEC §5.5.

---

## D-004 — Per-page meta title and description belong to Slice 5

**Status:** approved 2026-09-16

Two fields per page, defaulting to the page title, emitted in `<head>`. Nothing more (no
sharing image, no robots, no sitemap).

**Trade-offs.** Slightly widens Slice 5, which is already large. The alternative, moving
it to the site settings slice, would leave every page without its own search title for
several more slices, when `docs/plan.md` and `docs/product.md` already promise it now.

**Reflect in:** SPEC §8 Slice 5.

---

## D-005 — The spec does not list code that has no caller yet

**Status:** approved 2026-09-16

SPEC §4 stops naming `Hooks` in Core (it would break "no abstraction without a second
caller"), and marks `Cache` as arriving with Slice 8. A document that names parts that do
not exist misleads whoever reads it next, human or agent.

**Reflect in:** SPEC §4.

---

## Open items — not decided

**O-1. Upgrading an existing install.** The Migrator runs only from the installer, so a
new migration never reaches an existing install until Slice 8. The live install has not
applied `0011_media.sql` or `0012_media_meta.sql`. Candidate: bring forward a minimal
"apply pending migrations" step before more tables arrive.

**O-2. nginx-first serving and caching.** SPEC §8 Slice 8 (page cache) and the CLAUDE.md
security rules rely on `.htaccess` file checks, which nginx ignores. Needs a design that
works on both servers.

**O-3. Order after Slice 5.** Candidates: site chrome (header, navigation, footer),
repeater blocks, nested page addresses. SPEC §9 and `docs/product.md` disagree.

**O-4. 2FA.** SPEC §6 describes it (optional, ten recovery codes, `storage/disable-2fa`
via FTP) but no slice in SPEC §8 owns it. To decide: which slice, and confirm that it
stays optional rather than forced.

**O-5. Documentation that disagrees with the code.** Handed to the executor 2026-09-16,
with D-003..D-005; awaiting the commit and the architect's check.

1. `docs/plan.md` §4 says the media migration is uncommitted; it is committed (`517cfbe`).
2. SPEC §8 Slice 5 still says "lazy variant generation" and ".htaccess direct serving",
   contradicting §5.1 (variants generated on upload).
3. SPEC §5.5 writes variants to `/public/cache/media/`; the repository uses `public/m/`.
4. SPEC §4 lists `Cache` and `Hooks` in Core; neither exists.
5. SPEC §9 cites `{{menu:main}}` as a §5.6 tag; §5.6 does not list it.
6. `docs/product.md` §3.4 lists paste-as-plain-text as existing; `docs/plan.md` says it is
   unverified.
7. Per-page meta title and description: owed by Slice 5 per `docs/plan.md` and
   `docs/product.md`, absent from SPEC §8.
8. SPEC §9 `Config::get()` typing was to be revisited after Slice 3; it was not.
