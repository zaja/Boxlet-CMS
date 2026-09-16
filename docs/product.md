# Boxlet — what it is, and what it will do

A functional description of the product: the capabilities it has, the ones it is going to
have, and the ones it will deliberately never have. No code, no schema, no file names.

For the technical contract see `docs/SPEC.md`. For the state of the work in progress see
`docs/plan.md`. This document is the one to read to understand **what the thing is**.

---

## 1. The product in one paragraph

Boxlet is a small self-hosted CMS for people who build **many small sites** — freelancers
and small studios who currently reach for WordPress and then spend a day removing things.
It installs by uploading a ZIP and opening a page in a browser: no shell, no Composer, no
build step, no Node. One person runs it; there are no user accounts to manage. What it
sells is not a feature list but a **design layer**: you choose a character and a handful
of decisions, and the entire site takes on a coherent, distinct look without anyone
writing a line of CSS.

---

## 2. The idea that everything else follows from

**Constrained freedom.** Page builders hand the user unlimited control and users produce
ugly sites. Boxlet offers a bounded space of options in which every combination looks
acceptable. When a choice is in doubt, the knob is removed rather than added.

Three consequences, visible everywhere in the product:

1. **You cannot pick a colour for a section.** You pick from five surfaces — plain,
   tinted, contrast, image, gradient — all derived from the site's palette. An open
   colour field would let anyone put red text on orange, and the palette's guarantee of
   readable contrast would become decoration.
2. **You cannot build an arbitrary grid.** A "team grid" is one block that holds a
   repeating item, with variants for two, three or four across. It is not an empty
   container that any block can be dropped into.
3. **Every block already knows how to look good** in every design, on every surface, at
   every width. That is the block author's job, done once, so the site owner cannot get
   it wrong.

---

## 3. What exists today

Everything below is built and in the repository. Rich text (3.4) is the one part still
being checked in a running browser rather than only in tests; `docs/plan.md` lists
exactly which of its behaviours have been watched working and which have not.

### 3.1 Installing and running it

- Upload a ZIP, open `/install.php`, answer four screens: requirements, database,
  administrator, site details. The installer refuses to continue if the server cannot
  support the site, and says exactly what is missing rather than failing later.
- It checks the things that fail *silently* on cheap hosting — how many form fields PHP
  accepts, how large an upload may be — and reports them in plain language.
- It offers to install a demo site so the design layer can be judged immediately, rather
  than on an empty page.
- MySQL or SQLite. One administrator, protected by a rate limit that locks out repeated
  wrong passwords by both account and address.

### 3.2 The design layer — the differentiator

Four layers, applied in order, each narrower than the last:

1. **Character.** Five complete personalities: Editorial, Minimal, Bold, Soft,
   Brutalist. Choosing one sets everything below at once.
2. **Decisions.** Eight, not forty: two colours, a typeface pairing, a type scale, a
   spacing unit, corner character, shadow character, content width, surface contrast.
   Everything else — the full palette, every type size, the spacing scale — is derived
   and shown, not edited.
3. **Section style.** Per block: surface, vertical rhythm, width, alignment, and the
   shape of its top edge.
4. **Layout.** Per block: the arrangements that block offers, such as a hero with its
   text beside the image or below it.

What makes this more than a theme picker:

- **A character changes the composition, not just the paint.** Switching from Editorial
  to Brutalist changes the reading measure, the vertical rhythm, where headings sit, how
  sections are separated and how heroes are arranged. The same page becomes a different
  site.
- **Readable contrast is enforced, not advised.** Every text-and-background pair the
  palette can produce is checked, and a combination that fails is refused with the pair
  named. Chosen colours are never silently adjusted to pass.
- **Applying a character to a site that already has pages asks first**: change the design
  only, or also reset the per-section choices already made.
- **Typography is self-hosted.** No third-party font service, for both privacy and
  reliability.

### 3.3 Editing a page

- **A visual editor**: the real page on a canvas, exactly as a visitor will see it, with
  a library of blocks beside it. Each block in the library is shown as a picture of
  itself, rendered by the site in the site's own design, so nothing in the library can
  drift out of date.
- **Add a block by aiming at a position** — a "+" between any two sections — and then
  choosing one. **Reorder by dragging** on the page itself.
- **Select a block to edit it.** Its text, its section style, its layout and its controls
  appear beside the canvas, and the page updates as you type.
- **Page settings** in the same panel: title, address, parent page and whether it is
  published. The address follows the title while a page is a draft, until you change it
  yourself — a published page's address never changes behind your back.
- **A plain editor is always available** at its own address: every field of every block
  in one form, no JavaScript required. It is the way through when something breaks.
- **Nothing is saved until you press Save**, and leaving with unsaved changes warns you.
- Three block types today: hero, text, image-and-text.

### 3.4 Writing

- Rich text is edited with a real editor — bold, italic, links, headings, quotations,
  bulleted and numbered lists with nesting — and stored as plain HTML.
- **The toolbar offers only what the site can store.** There is no button whose result
  would be discarded when you save.
- **Pasting from Word or Google Docs is cleaned**, and there is an explicit
  paste-without-formatting for when it still comes out wrong.
- **Every rich text field can be switched to plain HTML** and back.

### 3.5 The administrator's experience

- One persistent navigation, a consistent layout, a comfortable reading width.
- **The admin has its own fixed appearance** and never takes on the site's design. You
  cannot be locked out of the screen that fixes an unreadable colour scheme by that
  colour scheme.
- Validation errors appear next to the field that caused them; saving confirms visibly.
- No control is ever invisible until hovered.

---

## 4. What is planned, in order

### 4.1 Media — next

- Upload one file or many, by choosing or by dropping onto the library.
- **Every size is produced at upload time**, not on demand, so a picture is always a file
  that already exists. This is what makes it work identically on every host.
- Five named sizes; a photograph is never served at its original weight by accident.
- **A focal point per image**, so cropping to a square never cuts off the head.
- **A photograph taken sideways on a phone is turned upright** before anything else
  happens.
- Modern formats served automatically, with the original as a fallback.
- Alt text and captions **per language**, because the same photograph needs different
  words in different languages. An image that is decoration gets no alt text rather than
  a filename.
- **An image in use cannot be deleted.** The library says which pages use it and lets you
  go and fix them first.
- A picker wherever an image is chosen, including as a section background.
- Large jobs resume: uploading a heavy photograph on a slow host cannot leave an image
  half-processed.

Also in this slice, and the only part of search-engine handling that gets built now: **a
title and description per page**, with the page's own title as the default. Nothing else
— no sharing images, no structured data, no robots file — until site-wide settings exist.

### 4.2 Repeating content

The missing shape for building a real site: a team grid of photograph, name and role; a
row of features; a strip of logos. One block holding a repeating item, with variants for
two, three or four across. It comes immediately after media, because a team grid without
photographs cannot be judged.

### 4.3 More than one language

- Each page exists once per language, linked to its siblings.
- **Translation is assisted**, with a glossary for terms that must not be translated.
- **Editing the original marks only the affected block as out of date**, in every
  translation. Re-translating is never all-or-nothing.
- A language switcher and correct `hreflang`.
- Two answers still to settle: what a visitor gets when a page has no translation yet
  (the leaning is to hide it from navigation and return not-found on a direct hit,
  configurable per site), and whether translation simply stays switched off until the
  owner supplies their own provider key — almost certainly yes, since the alternative is
  shipping someone else's bill.

### 4.4 Forms

Build a form, place it in a page, receive the submissions by email and keep them in the
admin. An automatic reply to the sender. Spam resistance without a third-party service or
a puzzle for the visitor.

### 4.5 Running a site over time

- **Page caching**, so a busy site serves without touching PHP.
- **Backup to a single file**, and restoring from it.
- **Updating from inside the admin** by uploading a new version, with a backup taken
  first automatically.
- **Revisions with rollback**, so an edit is never final.
- **Regenerating the picture sizes**, for when a size is added or changed after
  photographs are already in the library — resumable, and safe to run on a live site.
- A sitemap that knows about languages.

### 4.6 Ready to release

Six more blocks — gallery, features, call to action, accordion, testimonials, logo strip
— three starting templates, a demo site, and documentation. The test: a stranger
downloads it, uploads it to cheap hosting, and has a styled multilingual site in under
fifteen minutes.

---

## 5. Decided but not yet scheduled

These are real gaps, each with a decided answer and a place in the queue, recorded so
they are not forgotten or rediscovered. The reasoning behind each sits in `docs/SPEC.md`
§9.

- **Site chrome — the largest visible gap.** Pages currently render as a sequence of
  blocks with no header, no logo, no navigation and no footer. It is a design problem
  before it is a feature: a header is itself a design element, so a character should
  carry it — centred, left-aligned, transparent over a hero, sticky — the way it carries
  everything else. It waits on media only for the logo. *Its own slice, next in line
  after the current work.*
- **Site-wide settings.** Site name, favicon, time zone, default sharing image. Today
  only the installer writes them and there is no screen to change them afterwards.
  Analytics is the one item here that will be decided rather than simply added: embedding
  someone else's script contradicts the same posture that keeps the fonts self-hosted.
  *After site chrome.*
- **News or a blog.** Not a second content type with its own archives, feeds and
  pagination — that multiplies by language and brings a taxonomy the non-goals exclude.
  Instead one block that lists child pages in date order, as a list or a grid, with a
  ready-made "News index" starting page. The hierarchy and the publication dates already
  exist. Open within it: what a listed page contributes to its card, since a summary and
  a thumbnail are not yet things a page has.
- **Addresses that reflect the hierarchy**, so a child page can live under its parent's
  address — inseparable from remembering old addresses and forwarding them, because
  renaming a parent otherwise turns every page beneath it into a dead link. *Its own
  slice.*
- **Categories in the block library**, once there are enough blocks that a flat list
  stops being scannable — roughly fifteen. A filter that is quicker to ignore than to use
  is worse than none.

---

## 6. What Boxlet will never be

This list is as much a part of the product as the features:

- **No user accounts, roles or permissions.** One administrator.
- **No visitor accounts, registration or comments.**
- **No plugin marketplace**, and no plugin API in version one.
- **No e-commerce.**
- **No taxonomies** — no tags, no categories for content.
- **No page builder with free-form positioning.** A page is an ordered list of blocks.
- **No build step, ever** — on the server or for the person installing it.
- **No public API** in version one.
- **No third-party fonts, analytics or scripts** added casually. Anything that sends a
  visitor's data elsewhere is a deliberate decision, not a settings field.

---

## 7. What "finished" looks like

A freelancer takes a ZIP, uploads it to a cheap shared host, and within fifteen minutes
has a multilingual site that looks like it was designed — not like a template with the
logo swapped. They choose a character, adjust two or three decisions, write their pages
on a canvas that shows exactly what visitors will see, put in their photographs without
thinking about image sizes, and add a contact form that reaches their inbox.

They never open a CSS file. They never see a database. They never install a plugin.

And when the site needs changing a year later, they still recognise it.
