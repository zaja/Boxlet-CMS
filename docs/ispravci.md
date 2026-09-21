# Appearance — što još treba ispraviti

Usporedba implementacije na `zaja/Boxlet-CMS@main` (commit tree `466baa4ab00c`, čitano 21. 9. 2026.) s prototipom `Boxlet Appearance.dc.html` i handoffom u istoj mapi.

Pročitano: `app/Modules/Appearance/views/appearance.php`, `views/tabs/*.php`, `AppearancePreview.php`, `public/assets/admin-appearance.css`, `appearance.js`, `appearance-tabs.js`, `appearance-stage.js`, `admin-shell.css`, `admin-rail-compact.css`, `admin-nav.js`, `app/Modules/Admin/views/layout.php`.

**Kratko:** struktura je točna — traka, tri stupca, pet tabova, jedan `<form>` oko svega, `readouts` sa servera, gauge. Ono što ne valja je (A) ponašanje layouta pri promjeni širine prozora, (B) to što se preview reloada na svaku promjenu, i (C) nekoliko mjesta gdje je kontrola iz prototipa nestala ili je posložena drukčije nego što je specificirano.

Prioritet: **A → B → C → D**. A i B su ono zbog čega ekran "ne izgleda kao prototip"; C su funkcije koje nedostaju.

---

## A. Layout u tri stupca — zašto se "otvaraju dodatni sidebarovi"

### A1. Dijagnoza

Ekran zapravo ima **četiri** stupca, ne tri: admin rail (`.admin-rail`) pa tek onda naša tri. Prototip je pretpostavljao da Appearance zauzima cijeli prozor bez admin raila. Tri stvari se onda tuku:

| Izvor | Prag | Što radi |
| --- | --- | --- |
| `admin-rail-compact.css` | `max-width: 62.5rem` (1000px) | admin rail pada na ikone (3.25rem), iznad toga je 13.5rem |
| `admin-appearance.css` | `max-width: 64rem` (1024px) | naša tri stupca padaju u **jedan** |
| `admin-shell.css` | `max-width: 47.5rem` (760px) | admin rail postaje ladica |

**Bug 1 — gumb za otvaranje raila postoji na ovom ekranu.** `layout.php:102` daje `bare` ekranima klasu `rail-compact`, što je točno. Ali `admin-nav.js` svejedno prikazuje `[data-rail-toggle]` i na `bare` ekranu, pa jedan klik skine `rail-compact`, doda `rail-wide` i **otvori 216px širok rail kao četvrti stupac** preko ekrana koji je već pun. To je doslovno "otvori se dodatni sidebar". Jednom otvoren, na `bare` ekranu se ne pamti u cookieju, pa se stanje vraća na reload — što izgleda još proizvoljnije.

**Bug 2 — prazan pojas 1000–1024px.** Spuštanjem prozora prvo (na 1024) tri stupca padnu u jedan, a tek onda (na 1000) admin rail skupi na ikone. U tom pojasu imamo najgoru moguću kombinaciju: široki admin rail i tri stupca naslagana jedan ispod drugog, iako bi u toj širini dva stupca još stala.

**Bug 3 — media query mjeri prozor, a ne stupac.** Širina koja je nama na raspolaganju ovisi o stanju admin raila (52px ili 216px), a `@media` to ne vidi. Isti prozor od 1100px jednom ima 1048px sadržaja, drugi put 884px, a CSS se ponaša identično.

**Bug 4 — preview mijenja širinu ispod ruke.** `appearance-stage.js` zove `fitsTheColumn()` iz `ResizeObserver`-a na svaku promjenu. `picked` čuva samo ručni odabir; dok vlasnik nije ništa kliknuo, svako sužavanje prozora prebaci preview s Desktop na Tablet pa na Phone i natrag. Vlasnik gleda desktop layout, otvori devtools i dobije tablet — pa misli da je promijenio dizajn.

**Bug 5 — dvostruki scroll u složenom stanju.** Ispod 64rem `.appearance` ide na `height: auto` (stranica skrola), a `.preview-stage` zadržava `block-size: 70vh` s vlastitim skrolom. Dva skrola jedan u drugom.

**Bug 6 — `minmax(0, 1fr)` na sceni.** `grid-template-columns: 12.25rem minmax(0, 1fr) 19.5rem` dopušta sceni da se stisne na nulu prije nego breakpoint uopće opali. Scena nema donju granicu.

### A2. Popravci

**1) Sakrij `rail-toggle` na `bare` ekranima.** U `admin-nav.js`, odmah nakon `var editor = frame.hasAttribute('data-rail-editor');`:

```js
// Ekran koji sam sebi puni prozor (Appearance, uređivač stranice) nema mjesta za širok
// rail: otvaranje bi dodalo četvrti stupac preko tri koja su već tu. Rail ostaje na
// ikonama, a gumb koji to može poništiti ne postoji.
if (editor) {
  fold.hidden = true;
  return;
}
```

Alternativa ako se gumb želi zadržati u uređivaču stranice: uvedi zasebni atribut `data-rail-locked` i postavi ga samo na Appearance (`AppearanceController` već šalje `'bare' => true`).

**2) Mjeri stupac, ne prozor — container query.** U `admin-appearance.css`:

```css
.appearance {
  container-type: inline-size;
  container-name: appearance;
  /* ... postojeće ... */
}
```

pa sve pragove ovog ekrana prepiši u `@container appearance (max-width: …)`. Time ekran postaje neosjetljiv na to je li admin rail 52px ili 216px, i Bug 3 i Bug 2 nestaju zajedno.

**3) Tri stupca → dva → jedan, ne tri → jedan.**

```css
.appearance-body {
  grid-template-columns: 12.25rem minmax(30rem, 1fr) 19.5rem;
}

/* Nema mjesta za biblioteku karaktera kao stupac: ona postaje gumb u traci koji
   otvara isti popis kao ploču. Inspektor ostaje — on je ono što se koristi. */
@container appearance (max-width: 74rem) {   /* ~1184px sadržaja */
  .appearance-body { grid-template-columns: minmax(24rem, 1fr) 19.5rem; }
  .appearance-rail { display: none; }        /* vidi točku 4 */
  .appearance-rail[data-open] { … }
}

/* Jedan stupac: slika, pa kontrole, pa karakteri. */
@container appearance (max-width: 56rem) {   /* ~896px */
  .appearance { height: auto; }
  .appearance-body { grid-template-columns: minmax(0, 1fr); }
  .appearance-stage-column { order: -1; }
}
```

**4) Rail kao ploča u srednjem rasponu.** Kad `.appearance-rail` prestane biti stupac, u `.appearance-bar` se pojavi gumb „Karakteri" (`aria-expanded`, `aria-controls="appearance-rail"`) koji isti `<div class="appearance-rail">` prikaže kao ploču ispod trake (`position:absolute; inset-inline-start:0; inline-size:min(18rem, 90vw); z-index:2`). Nema duplog markupa — rail je i dalje u istom `<form>`, samo drukčije pozicioniran. Bez skripte gumb je `<a href="#appearance-rail">` i rail je na dnu stranice, kao sada.

**5) U složenom stanju scena nije `70vh` nego omjer.**

```css
@container appearance (max-width: 56rem) {
  .preview-stage {
    block-size: auto;
    aspect-ratio: 16 / 10;
    min-block-size: 22rem;
    overflow: hidden;   /* skrola stranica, ne scena */
  }
}
```

**6) `fitsTheColumn()` samo jednom.** U `appearance-stage.js`:

```js
var chosenOnce = false;

function fitsTheColumn() {
  var room = stage.clientWidth;
  if (room <= 0 || picked || chosenOnce) { return; }
  chosenOnce = true;
  // ... postojeći odabir ...
}
```

`ResizeObserver` i dalje zove `draw()` na svaku promjenu (zoom „fit" se mora preračunati), ali širina se bira jednom, na prvom mjerenju koje nije nula. Ako scena postane preuska za odabranu širinu, **ne mijenjaj je** — pusti `scale()` ispod `SMALLEST` i u traci scene ispiši „premalo mjesta, odaberi užu širinu". Tiha promjena je gora od poruke.

**7) Visina ekrana.** `height: calc(100dvh - var(--ui-bar-height))` radi samo dok traka ima točno `--ui-bar-height`. Sigurnije je vlasništvo nad visinom dati okviru:

```css
.admin-frame:has(.appearance) { height: 100dvh; overflow: clip; }
.admin-frame:has(.appearance) .admin-main-bare { min-height: 0; overflow: clip; }
.appearance { height: 100%; }
```

Time nestaje i ovisnost o tome hoće li se `.appearance-bar` prelomiti u dva reda (a hoće, na hrvatskim labelama).

### A3. Provjera nakon popravka

Povuci prozor od 1600 do 700px u koracima i provjeri da:
- admin rail nikad ne mijenja širinu (uvijek 3.25rem),
- nema stanja u kojem je rail širok a stupci složeni,
- preview ostaje na širini koju je vlasnik vidio na početku,
- nigdje se ne pojavi vodoravni skrol ni mrtvi pojas ispod ekrana,
- u jednostupčanom stanju postoji točno jedan okomiti skrol.

---

## B. Preview se reloada na svaku promjenu

`appearance.js` na svaku promjenu radi `preview.src = previewUrl + '?' + params`. To je puni reload dokumenta svakih 250ms dok se vuče klizač: bijeli bljesak, izgubljena pozicija skrola, ponovno učitavanje fontova i slika. Prototip nije imao taj osjećaj jer se ništa nije ponovno učitavalo.

**Popravak:** razdvoji promjene koje mijenjaju samo tokene od onih koje mijenjaju markup.

- **Samo tokeni** (boja, tipografija, oblik, stranica — sve što ide kroz `TokenCompiler`): okvir je same-origin, pa se izmijeni samo `href` stylesheeta unutar njega:

  ```js
  var doc = preview.contentDocument;
  var sheet = doc && doc.querySelector('link[rel="stylesheet"][data-design-tokens]');
  if (sheet) { sheet.href = stylesheetUrl + '?' + params; return; }
  ```

  Za to `AppearancePreview::page()` mora tom `<link>`-u dodati `data-design-tokens` (kroz `Url::useStylesheet`). Stara vrijednost ostaje aktivna dok se nova ne učita, pa nema bljeska.

- **Markup** (učitavanje karaktera, `header_menu`, riječi zaglavlja/podnožja, `boxed`, `header_bleed`, `footer_bleed`, sve iz Chrome taba): puni reload, kao sada, ali sa zapamćenom pozicijom skrola:

  ```js
  var at = preview.contentWindow ? preview.contentWindow.scrollY : 0;
  preview.addEventListener('load', function once() {
    preview.removeEventListener('load', once);
    preview.contentWindow.scrollTo(0, at);
  });
  ```

Popis odluka koje traže reload drži u jednoj konstanti na vrhu `appearance.js`, uz komentar zašto — inače će se raspasti pri sljedećoj novoj odluci.

Usput: `Compare` (`appearance-stage.js`) mijenja `src` okvira pri svakom pritisku, što znači puni reload na svako držanje gumba. S gornjim razdvajanjem compare postaje zamjena `href`-a na objavljeni stylesheet — trenutačno i bez bljeska.

---

## C. Razlike prema prototipu, po tabovima

### C1. Boja — dvije liste umjesto jedne (važno)

Prototip: **jedan** popis uloga, svaki redak = swatch + naziv + hex + strelica za vraćanje, plus „reset all" iznad. Vlasnik vidi paletu i mijenja je na istom mjestu.

Implementacija: `colour.php` ima `<ul class="swatches">` s **petnaest** izvedenih boja koje se ne mogu dirati, i zasebno `<details class="by-hand">` sa **šest** koje se mogu — sklopljeno, ispod. Ista informacija je na dva mjesta, a mijenja se na onom koje je zatvoreno.

**Popravak:** spoji ih u jedan popis uloga. Svaki redak:
- uloga iz `Palette::BY_HAND` → `<input type="color">`, hex u mono, gumb za vraćanje kad je preuzeta;
- izračunata uloga (tinte, tekst na boji) → swatch i hex, bez inputa, oznaka „izračunato" u `--ui-ink-faint`.

Zadrži `data-by-hand-switch` logiku (uzimanje uloge pri promjeni boje) — ona je dobro riješena i rješava utrku s `refresh()`. Samo je gumb za vraćanje ono što je vidljivo, a checkbox ostaje skriveni mehanizam, ili ga zamijeni `value=""` semantikom.

„reset all" iznad popisa, prikazan samo kad je barem jedna uloga preuzeta.

### C2. Tipografija — uzorak ne pokazuje ono što mijenja (važno)

`.specimen-4xl { font-size: 1.6rem }` je **fiksan**. Klizač `scale` mijenja samo broj s desne strane, a uzorak izgleda jednako. To mu oduzima jedini razlog postojanja: odnos veličina.

Fontovi su na ovom ekranu već učitani (`AppearancePreview::typefaces()` poslužuje sve pairinge), pa obrazloženje „admin ne smije uzeti pismo stranice" ne stoji — kartice pisama ga već uzimaju.

**Popravak:** uzorak se crta u odabranom pismu, s veličinama proporcionalnim `scale`-u, skaliran da stane u 19.5rem:

```
hero     = round(base * scale^3 * k)
naslov   = round(base * scale^2 * k)
tekst    = round(base * k)
sitno    = round(base / scale * k)
```

gdje je `k` faktor uklapanja (npr. tako da hero ne prijeđe 34px). Pikseli u readoutu ostaju **stvarni** brojevi sa servera, ne skalirani — oni su ono što stranica dobiva. Dodaj `aria-hidden` kako i sada jest.

Isti `data-readout` mehanizam ostaje; dodaje se samo `style` s veličinom, koji se piše kroz CSSOM (`element.style.fontSize = …`), jer politika brani `style` atribut.

### C3. Stranica — nedostaje vlastita boja

Prototip ima uz `page_background`, boju zaglavlja i boju podnožja i **„Ili odaberi svoju"** — `<input type="color">` s vraćanjem na nijansu palete. `page.php` ima samo segmentirani izbor između nijansi palete.

**Popravak:** dodaj tri opcionalne odluke — `page_background_colour`, `header_colour`, `footer_colour` — svaka prazna dok je vlasnik ne postavi, validirane kroz `Tokens::validate()` kao hex, s obveznom provjerom kontrasta (tekst na toj podlozi mora proći, inače `errors`). Kontrola ide neposredno ispod odgovarajuće segmentirane grupe, ne u zaseban odjeljak.

### C4. Oblik i Chrome — u redu

`shape.php` i `chrome.php` odgovaraju specifikaciji. „Following" stanje po grupi (`readout-following` + prvi segment imenovan po tome što karakter daje) je bolje riješeno nego u prototipu — zadržati.

Jedina zamjerka: `header_menu` je `<select>` na ekranu čije je vlastito pravilo (D-065) da je zatvoren skup red gumba. Izbornika može biti proizvoljno mnogo, pa je `<select>` ovdje opravdan — ali onda i `zoom` smije ostati `<select>`, ili oba postaju gumbi. Trenutačno su nedosljedni jedan prema drugome (vidi D3).

### C5. Tabovi se prelamaju na hrvatskom

`.tab-strip` ima `flex-wrap: wrap`, a pet tabova u 19.5rem − padding ≈ 280px stane tek na engleskom. „Boja · Tipografija · Oblik · Stranica · Zaglavlje" se sigurno prelomi u dva reda, i to neravnomjerno.

**Popravak:** `grid-template-columns: repeat(5, minmax(0, 1fr))` umjesto flex-wrapa, `font-size: var(--ui-text-xs)`, `white-space: nowrap`, `text-overflow: ellipsis`, puni naziv u `title`. Dva ravnomjerna reda su prihvatljiva, neravnomjerna nisu.

---

## D. Sitni ispravci

1. **`[data-stage-size]` se nikad ne ispisuje.** `appearance-stage.js` nigdje ne dira taj element, pa traka nad slikom uvijek ima prazno mjesto na desnoj strani. Ispiši `width + '×' + Math.round(tall/factor) + ' · ' + Math.round(factor*100) + '%'` unutar `draw()`.

2. **Widthovi i zoom su u gornjoj traci, a traka scene je prazna.** Prototip ih ima uz sliku jer pripadaju slici, a ne ekranu. Premjesti `.preview-tools` u `.stage-strip`; u gornjoj traci ostaju samo stanje, Revert i Publish. Time gornja traka prestaje prelamati na srednjim širinama.

3. **Zoom `<select>` vs. vlastito pravilo.** Ako ostaje select, ukloni `title`-less `visually-hidden` label u korist vidljivog „Zoom"; ako postaje red gumba (kao u prototipu), četiri gumba `Fit · 100 · 75 · 50` stanu u traku scene.

4. **`preview-actions` u traci.** `margin-inline-start: auto` na `.preview-tools` i na `.preview-bar-row .preview-actions` znači da se pri prelamanju stanje odvoji od gumba. Kad se alati presele (D2), ovo nestaje samo od sebe.

5. **Gauge u 312px.** Petnaest redaka s SVG uzorkom je previše za inspektor. Zadrži šest otvorenih + sklopljeno, ali uzorak smanji na 28×18 i redak u jedan red bez preloma.

6. **`.rail-card` tools vs. cover button.** `z-index` je riješen, ali cover `<button>` unutar `<form>` bez `type` nasljeđuje `submit` — to je ovdje namjerno (`name="action" value="preset:…"`), samo dodaj eksplicitno `type="submit"` da se ne izgubi pri refaktoru.

7. **Svjetlo/tamno.** Prototip je crtan u Nocturne tamnoj paleti, admin je svijetao (`--ui-*`). To je i dalje otvorena odluka vlasnika iz `IMPLEMENTATION-theme-switch.md` — nije bug, ali je razlog zašto ekrani „ne izgledaju isto" i vrijedi ga zatvoriti prije nego što se troši vrijeme na finu tipografiju.

---

## E. Redoslijed

| # | Zadatak | Datoteke | Težina |
| --- | --- | --- | --- |
| 1 | A1/A2 — sakrij rail toggle na `bare` | `admin-nav.js` | trivijalno |
| 2 | A2 — container query + 3→2→1 stupca | `admin-appearance.css`, `appearance.php` | srednje |
| 3 | A2 — `fitsTheColumn()` jednom | `appearance-stage.js` | trivijalno |
| 4 | A2 — visina preko `.admin-frame` | `admin-appearance.css` | malo |
| 5 | D1, D2 — traka scene: veličina, alati | `appearance-stage.js`, `appearance.php`, `admin-appearance.css` | malo |
| 6 | B — zamjena stylesheeta umjesto reloada | `appearance.js`, `AppearancePreview.php` | srednje |
| 7 | C1 — jedan popis boja | `tabs/colour.php`, `appearance.js` | srednje |
| 8 | C2 — živi uzorak tipografije | `tabs/type.php`, `appearance.js` | srednje |
| 9 | C5 — tabovi bez preloma | `admin-appearance.css` | trivijalno |
| 10 | C3 — vlastite boje stranice | `tabs/page.php`, `Tokens.php`, `Derived.php` | veće, nova odluka |
| 11 | D5, D6 — gauge i karte | `tabs/colour.php`, `admin-appearance.css` | malo |

Koraci 1–5 su pola dana i rješavaju sve što je vlasnik prijavio kao „otvaraju se dodatni sidebarovi". Korak 6 je ono što ekran čini da djeluje kao prototip. 7–9 su vjernost specifikaciji. 10 je jedina stavka koja dodaje novu odluku u `Tokens` i nosi migraciju.

---

## Napravljeno

Svih jedanaest koraka je napravljeno, u redoslijedu iz tablice gore. Odluke su zapisane u
PLAN.md kao D-071 (`466baa4`), D-072 (`ca6d336`), D-073 (`be6caff`), D-074 (`b2b7167`),
D-075 (`508b992`) i D-076.

**Jedna tvrdnja iz tablice gore nije točna, i to je izmjereno, a ne pretpostavljeno:** korak
10 **ne nosi migraciju**. `design_tokens` je tablica s jednim retkom po odluci i JSON
vrijednošću (`0010_design_tokens.sql`), pa nova odluka ne traži promjenu sheme — isto
obrazloženje koje `0024_design_library.sql` već zapisuje za svoj `decisions_json`. Nova
odluka dobiva praznu zadanu vrijednost u `Presets::get()` i time je gotova.

Dvije stvari napravljene su drukčije nego što dokument predlaže, obje namjerno i obje
zapisane u PLAN.md:

- **§C2, granica uzorka.** Dokument predlaže da najveći redak ne prijeđe 34px. Pri zadanom
  karakteru to je tijelo teksta spustilo na 8px i sitni tisak na 6px — dvije mrlje. Granica
  je 40px (D-075).
- **§C3, boja zaglavlja i podnožja.** Dokument traži provjeru kontrasta. Tinta na vlastitoj
  boji se **izvodi** iz nje, pa jamstvo drži izvođenje; provjera ostaje kao zaštita i hvata
  jedino boju na kojoj se nijedna od dvije tinte ne može čitati (D-076).
