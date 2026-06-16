# Site Improvements Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add online booking, testimonials, and an FAQ to laurenhendriks.nl, plus three polish refinements (self-hosted fonts, subtle scroll motion, optimized images).

**Architecture:** Hugo static site with a custom theme (`themes/laurenhendriks/`). New content is data-driven (`data/*.yaml`) and rendered by isolated partials/layouts. Booking uses a click-to-load Calendly embed so no third-party connection happens until the visitor opts in. All work happens on one feature branch, one task per commit; each task is dispatched to a fresh subagent and reviewed before the next.

**Tech Stack:** Hugo extended v0.163.1, vanilla JS (no framework), plain CSS, Calendly, Formspree (existing).

**Testing approach:** This project has no unit-test harness. The verification analog for each task is: run `hugo --minify` (must build with no errors) and assert on the rendered `public/` output with `grep`. Plus a manual visual check via `hugo server` where noted. "Write the failing assertion first" means: grep for the expected markup *before* implementing and confirm it is absent.

**Branch & execution:** Before Task 1, create branch `feat/site-improvements` off `master`. Each task commits to this branch. Tasks are ordered to avoid shared-file conflicts (fonts → booking/nav/hero → testimonials → FAQ → motion → images); motion comes after the sections it animates exist. Execute with subagent-driven-development: one fresh subagent per task, review between tasks. At the end, open a PR to `master`.

---

## File Structure

**New files**
- `content/afspraak.md` — booking page content (front matter + placeholder intro)
- `content/veelgestelde-vragen.md` — FAQ page content
- `themes/laurenhendriks/layouts/_default/afspraak.html` — booking page layout
- `themes/laurenhendriks/layouts/_default/faq.html` — FAQ page layout
- `themes/laurenhendriks/layouts/partials/booking.html` — Calendly click-to-load widget (isolated)
- `themes/laurenhendriks/layouts/partials/testimonials.html` — testimonials section (isolated)
- `themes/laurenhendriks/layouts/partials/reveal.html` — loads the scroll-motion script
- `data/testimonials.yaml` — testimonial entries
- `data/faq.yaml` — FAQ entries
- `themes/laurenhendriks/assets/js/reveal.js` — IntersectionObserver scroll-reveal
- `themes/laurenhendriks/assets/fonts/*.woff2` — self-hosted font files

**Modified files**
- `hugo.toml` — add `calendlyUrl`, `hasCalendly` params
- `themes/laurenhendriks/layouts/_default/baseof.html` — drop Google Fonts, add reveal partial
- `themes/laurenhendriks/layouts/partials/nav.html` — Afspraak as CTA, Contact normal link
- `themes/laurenhendriks/layouts/index.html` — dual hero CTA + testimonials section + reveal attrs
- `themes/laurenhendriks/layouts/_default/aanbod.html` — intake button → `/afspraak/` + reveal attrs
- `themes/laurenhendriks/layouts/_default/over-mij.html` — responsive image pipeline
- `themes/laurenhendriks/layouts/partials/footer.html` — FAQ link
- `themes/laurenhendriks/assets/css/main.css` — @font-face, motion, booking/testimonials/FAQ/hero styles

---

## Task 0: Create the feature branch

**Files:** none

- [ ] **Step 1: Branch off master**

```bash
cd "C:/Users/snell/Documents/Code/business-website"
git checkout master
git pull --ff-only || true
git checkout -b feat/site-improvements
```

- [ ] **Step 2: Confirm clean build baseline**

Run: `hugo --minify`
Expected: builds with no errors; `public/` regenerated.

---

## Task 1: Self-host fonts

Replace the Google Fonts CDN `<link>` with locally served `woff2` files.

**Files:**
- Create: `themes/laurenhendriks/assets/fonts/cormorant-garamond-300.woff2`, `cormorant-garamond-400.woff2`, `cormorant-garamond-600.woff2`, `cormorant-garamond-300italic.woff2`, `cormorant-garamond-400italic.woff2`, `lato-300.woff2`, `lato-400.woff2`, `lato-700.woff2`
- Modify: `themes/laurenhendriks/layouts/_default/baseof.html:9-11`
- Modify: `themes/laurenhendriks/assets/css/main.css` (top of file)

- [ ] **Step 1: Download the font packages**

Use the google-webfonts-helper API (returns a zip of `woff2` files, latin subset):

```bash
cd "C:/Users/snell/Documents/Code/business-website"
mkdir -p themes/laurenhendriks/assets/fonts
cd themes/laurenhendriks/assets/fonts
curl -L -o cormorant.zip "https://gwfh.mranftl.com/api/fonts/cormorant-garamond?download=zip&subsets=latin&variants=300,regular,600,300italic,italic&formats=woff2"
curl -L -o lato.zip "https://gwfh.mranftl.com/api/fonts/lato?download=zip&subsets=latin&variants=300,regular,700&formats=woff2"
unzip -o cormorant.zip
unzip -o lato.zip
ls -1
```

Expected: several `*.woff2` files like `cormorant-garamond-v17-latin-300.woff2`, `lato-v24-latin-regular.woff2`, etc.

- [ ] **Step 2: Rename to canonical filenames**

Rename so the `@font-face` block can reference exact, stable names (adjust the source names to match what `unzip` produced):

```bash
cd "C:/Users/snell/Documents/Code/business-website/themes/laurenhendriks/assets/fonts"
mv cormorant-garamond-*-300.woff2        cormorant-garamond-300.woff2
mv cormorant-garamond-*-regular.woff2    cormorant-garamond-400.woff2
mv cormorant-garamond-*-600.woff2        cormorant-garamond-600.woff2
mv cormorant-garamond-*-300italic.woff2  cormorant-garamond-300italic.woff2
mv cormorant-garamond-*-italic.woff2     cormorant-garamond-400italic.woff2
mv lato-*-300.woff2                       lato-300.woff2
mv lato-*-regular.woff2                   lato-400.woff2
mv lato-*-700.woff2                       lato-700.woff2
rm -f cormorant.zip lato.zip
ls -1
```

Expected: exactly the 8 canonical files listed in **Files** above.

- [ ] **Step 3: Add @font-face rules to the top of main.css**

Prepend to `themes/laurenhendriks/assets/css/main.css`:

```css
/* Self-hosted fonts (latin subset). Replaces Google Fonts CDN. */
@font-face { font-family: 'Cormorant Garamond'; font-style: normal; font-weight: 300; font-display: swap; src: url('../fonts/cormorant-garamond-300.woff2') format('woff2'); }
@font-face { font-family: 'Cormorant Garamond'; font-style: normal; font-weight: 400; font-display: swap; src: url('../fonts/cormorant-garamond-400.woff2') format('woff2'); }
@font-face { font-family: 'Cormorant Garamond'; font-style: normal; font-weight: 600; font-display: swap; src: url('../fonts/cormorant-garamond-600.woff2') format('woff2'); }
@font-face { font-family: 'Cormorant Garamond'; font-style: italic; font-weight: 300; font-display: swap; src: url('../fonts/cormorant-garamond-300italic.woff2') format('woff2'); }
@font-face { font-family: 'Cormorant Garamond'; font-style: italic; font-weight: 400; font-display: swap; src: url('../fonts/cormorant-garamond-400italic.woff2') format('woff2'); }
@font-face { font-family: 'Lato'; font-style: normal; font-weight: 300; font-display: swap; src: url('../fonts/lato-300.woff2') format('woff2'); }
@font-face { font-family: 'Lato'; font-style: normal; font-weight: 400; font-display: swap; src: url('../fonts/lato-400.woff2') format('woff2'); }
@font-face { font-family: 'Lato'; font-style: normal; font-weight: 700; font-display: swap; src: url('../fonts/lato-700.woff2') format('woff2'); }
```

(The existing `font-family: 'Cormorant Garamond'` / `'Lato'` declarations elsewhere in the file are unchanged.)

- [ ] **Step 4: Remove the Google Fonts link and preconnects from baseof.html**

In `themes/laurenhendriks/layouts/_default/baseof.html`, delete lines 9–11:

```html
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Cormorant+Garamond:ital,wght@0,300;0,400;0,600;1,300;1,400&family=Lato:wght@300;400;700&display=swap" rel="stylesheet">
```

- [ ] **Step 5: Build and assert no Google Fonts reference remains**

```bash
hugo --minify
grep -r "fonts.googleapis.com" public/ ; echo "exit:$?"
grep -rl "fonts/cormorant-garamond-400" public/css/ ; echo "exit:$?"
```

Expected: first grep prints nothing and `exit:1` (no Google Fonts anywhere). Second grep lists the minified CSS and `exit:0` (font URL is bundled).

- [ ] **Step 6: Visual check**

Run: `hugo server` → open http://localhost:1313 → confirm headings render in Cormorant Garamond and body in Lato (not a system fallback). Stop the server.

- [ ] **Step 7: Commit**

```bash
git add themes/laurenhendriks/assets/fonts themes/laurenhendriks/assets/css/main.css themes/laurenhendriks/layouts/_default/baseof.html
git commit -m "perf: self-host fonts, drop Google Fonts CDN"
```

---

## Task 2: Booking page, nav, hero & aanbod link

Add the `/afspraak/` page with a click-to-load Calendly embed, repoint the
"plan een..." buttons, and make Afspraak the nav CTA.

**Files:**
- Create: `content/afspraak.md`
- Create: `themes/laurenhendriks/layouts/_default/afspraak.html`
- Create: `themes/laurenhendriks/layouts/partials/booking.html`
- Modify: `hugo.toml`
- Modify: `themes/laurenhendriks/layouts/partials/nav.html:9-14`
- Modify: `themes/laurenhendriks/layouts/index.html:7` (hero CTA)
- Modify: `themes/laurenhendriks/layouts/_default/aanbod.html:15`
- Modify: `themes/laurenhendriks/assets/css/main.css` (booking + hero CTA styles)

- [ ] **Step 1: Add Calendly params to hugo.toml**

In `hugo.toml`, under `[params]`, add:

```toml
  calendlyUrl = ''
  hasCalendly = false
```

- [ ] **Step 2: Create the booking content page**

Create `content/afspraak.md`:

```markdown
---
title: "Afspraak maken"
layout: afspraak
subtitle: "Plan je gesprek"
---

Plan hieronder eenvoudig een gesprek in. Kies een moment dat jou uitkomt — voor
een eerste kennismaking of een vervolgafspraak. *(Placeholder-tekst; vul samen
met Lauren aan.)*
```

- [ ] **Step 3: Create the booking partial (click-to-load)**

Create `themes/laurenhendriks/layouts/partials/booking.html`:

```html
{{ if and .Site.Params.hasCalendly .Site.Params.calendlyUrl }}
<div class="booking" data-calendly-url="{{ .Site.Params.calendlyUrl }}">
  <div class="booking__cover" id="booking-cover">
    <p class="booking__cover-text">Klik om de agenda te laden en een moment te kiezen.</p>
    <button type="button" class="btn btn--primary" id="booking-load">Plan je gesprek</button>
    <p class="booking__note">De agenda wordt geladen via Calendly zodra je klikt.</p>
  </div>
</div>
<script>
  (function () {
    var wrap = document.querySelector('.booking');
    if (!wrap) return;
    var btn = document.getElementById('booking-load');
    btn.addEventListener('click', function () {
      var url = wrap.getAttribute('data-calendly-url');
      var iframe = document.createElement('iframe');
      iframe.src = url;
      iframe.title = 'Calendly afsprakenplanner';
      iframe.className = 'booking__frame';
      iframe.loading = 'lazy';
      wrap.innerHTML = '';
      wrap.appendChild(iframe);
    });
  })();
</script>
{{ else }}
<div class="booking booking--fallback">
  <p>Online plannen is binnenkort beschikbaar. Neem voor nu gerust
     <a href="{{ "/contact/" | relURL }}">contact</a> met me op, dan plannen we samen een moment.</p>
</div>
{{ end }}
```

- [ ] **Step 4: Create the booking layout**

Create `themes/laurenhendriks/layouts/_default/afspraak.html`:

```html
{{ define "main" }}
<header class="page-header">
  <div class="container">
    <h1>{{ .Title }}</h1>
    {{ with .Params.subtitle }}<p class="page-header__sub">{{ . }}</p>{{ end }}
  </div>
</header>

<section class="booking-section">
  <div class="container">
    <div class="booking-section__intro">{{ .Content }}</div>
    {{ partial "booking.html" . }}
  </div>
</section>
{{ end }}
```

- [ ] **Step 5: Add booking + fallback styles to main.css**

Append to `themes/laurenhendriks/assets/css/main.css`:

```css
/* Booking page */
.booking-section { padding: 3rem 0 5rem; }
.booking-section__intro { max-width: 640px; margin: 0 auto 2.5rem; text-align: center; }
.booking { max-width: 720px; margin: 0 auto; }
.booking__cover { background: var(--surface, #E8DFD0); border-radius: 12px; padding: 3rem 1.5rem; text-align: center; }
.booking__cover-text { margin: 0 0 1.5rem; }
.booking__note { margin: 1rem 0 0; font-size: .85rem; opacity: .7; }
.booking__frame { width: 100%; min-height: 700px; border: 0; border-radius: 12px; }
.booking--fallback { background: var(--surface, #E8DFD0); border-radius: 12px; padding: 2.5rem 1.5rem; text-align: center; max-width: 720px; margin: 0 auto; }
```

(If the CSS uses literal hex rather than custom properties, substitute `#E8DFD0` for the surface colour directly to match the file's convention.)

- [ ] **Step 6: Repoint the aanbod intake button**

In `themes/laurenhendriks/layouts/_default/aanbod.html` line 15, change the href from `/contact/` to `/afspraak/` (keep the label "Plan een kennismaking"):

```html
      <a href="{{ "/afspraak/" | relURL }}" class="btn btn--light">Plan een kennismaking</a>
```

- [ ] **Step 7: Add the hero primary CTA**

In `themes/laurenhendriks/layouts/index.html`, replace line 7 (the single CTA) with a button pair:

```html
    <div class="hero__actions">
      <a href="{{ "/afspraak/" | relURL }}" class="btn btn--light">Plan een afspraak</a>
      <a href="{{ "/aanbod/" | relURL }}" class="btn btn--ghost">Bekijk het aanbod</a>
    </div>
```

Append the supporting styles to `main.css`:

```css
.hero__actions { display: flex; gap: 1rem; justify-content: center; flex-wrap: wrap; }
.btn--ghost { background: transparent; border: 1px solid rgba(255,255,255,.7); color: #fff; }
.btn--ghost:hover { background: rgba(255,255,255,.12); }
```

- [ ] **Step 8: Make Afspraak the nav CTA, Contact a normal link**

In `themes/laurenhendriks/layouts/partials/nav.html`, replace the `<ul>` list items (lines 10–13) with:

```html
      <li><a href="{{ "/" | relURL }}"{{ if .IsHome }} class="nav__link--active" aria-current="page"{{ end }}>Home</a></li>
      <li><a href="{{ "/over-mij/" | relURL }}"{{ if strings.HasPrefix .RelPermalink "/over-mij" }} class="nav__link--active" aria-current="page"{{ end }}>Over mij</a></li>
      <li><a href="{{ "/aanbod/" | relURL }}"{{ if strings.HasPrefix .RelPermalink "/aanbod" }} class="nav__link--active" aria-current="page"{{ end }}>Aanbod</a></li>
      <li><a href="{{ "/contact/" | relURL }}"{{ if strings.HasPrefix .RelPermalink "/contact" }} class="nav__link--active" aria-current="page"{{ end }}>Contact</a></li>
      <li><a href="{{ "/afspraak/" | relURL }}" class="nav__cta{{ if strings.HasPrefix .RelPermalink "/afspraak" }} nav__link--active{{ end }}"{{ if strings.HasPrefix .RelPermalink "/afspraak" }} aria-current="page"{{ end }}>Afspraak</a></li>
```

- [ ] **Step 9: Build and assert the booking page + routing**

```bash
hugo --minify
ls public/afspraak/index.html && echo "page-built"
grep -c "booking--fallback" public/afspraak/index.html   # hasCalendly is false → expect 1
grep -c "Plan een afspraak" public/index.html            # hero CTA → expect 1
grep -c 'href="/afspraak/"' public/aanbod/index.html     # intake button → expect 1
grep -c 'class="nav__cta' public/index.html              # Afspraak is nav CTA → expect 1
grep -c 'href="/afspraak/"' public/index.html            # nav CTA present → expect >=1
```

Expected: page-built printed; fallback count 1; the three route greps each ≥1.

- [ ] **Step 10: Visual check**

Run `hugo server` → confirm: hero shows two buttons; nav highlights "Afspraak" as the CTA; `/afspraak/` shows the fallback message (since `hasCalendly` is false); aanbod "Plan een kennismaking" navigates to `/afspraak/`. Stop the server.

- [ ] **Step 11: Commit**

```bash
git add hugo.toml content/afspraak.md themes/laurenhendriks/layouts/_default/afspraak.html themes/laurenhendriks/layouts/partials/booking.html themes/laurenhendriks/layouts/partials/nav.html themes/laurenhendriks/layouts/index.html themes/laurenhendriks/layouts/_default/aanbod.html themes/laurenhendriks/assets/css/main.css
git commit -m "feat: add /afspraak/ booking page with click-to-load Calendly"
```

---

## Task 3: Testimonials

Data-driven testimonials section on the Home page.

**Files:**
- Create: `data/testimonials.yaml`
- Create: `themes/laurenhendriks/layouts/partials/testimonials.html`
- Modify: `themes/laurenhendriks/layouts/index.html` (add section after pillars)
- Modify: `themes/laurenhendriks/assets/css/main.css` (testimonials styles)

- [ ] **Step 1: Create the data file with placeholder entries**

Create `data/testimonials.yaml`:

```yaml
# Vul aan met echte (geanonimiseerde) ervaringen. Alleen voornaam.
- quote: "Placeholder-ervaring. Vervang door een echte quote van een cliënt."
  name: "Voornaam"
  context: "na een leefstijltraject"
- quote: "Tweede placeholder-ervaring. Vervang door een echte quote."
  name: "Voornaam"
  context: ""
```

- [ ] **Step 2: Create the testimonials partial**

Create `themes/laurenhendriks/layouts/partials/testimonials.html`:

```html
{{ with site.Data.testimonials }}
<section class="testimonials">
  <div class="container">
    <h2 class="testimonials__heading">Ervaringen</h2>
    <div class="section-divider"></div>
    <div class="testimonials__grid">
      {{ range . }}
      <figure class="testimonial">
        <blockquote class="testimonial__quote">{{ .quote }}</blockquote>
        <figcaption class="testimonial__cite">
          {{ .name }}{{ with .context }} <span class="testimonial__context">· {{ . }}</span>{{ end }}
        </figcaption>
      </figure>
      {{ end }}
    </div>
  </div>
</section>
{{ end }}
```

(The `{{ with }}` guard means an empty/absent `data/testimonials.yaml` renders nothing.)

- [ ] **Step 3: Render the partial on the Home page**

In `themes/laurenhendriks/layouts/index.html`, add after the closing `</section>` of `.pillars` (before `{{ end }}`):

```html
{{ partial "testimonials.html" . }}
```

- [ ] **Step 4: Add testimonials styles to main.css**

Append:

```css
/* Testimonials */
.testimonials { padding: 4rem 0; background: var(--light, #FAFAF7); }
.testimonials__heading { text-align: center; }
.testimonials__grid { display: grid; gap: 2rem; grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); margin-top: 2rem; }
.testimonial { margin: 0; padding: 2rem; background: var(--background, #F5F0E8); border-radius: 12px; }
.testimonial__quote { margin: 0 0 1rem; font-family: 'Cormorant Garamond', serif; font-size: 1.3rem; font-style: italic; line-height: 1.5; }
.testimonial__cite { font-size: .95rem; font-weight: 700; }
.testimonial__context { font-weight: 400; opacity: .7; }
```

(Substitute literal hex for the custom properties if that matches the file's convention.)

- [ ] **Step 5: Build and assert**

```bash
hugo --minify
grep -c "class=\"testimonials\"" public/index.html   # expect 1
grep -c "testimonial__quote" public/index.html       # expect 2 (two entries)
```

Expected: section count 1, quote count 2.

- [ ] **Step 6: Empty-state check**

```bash
mv data/testimonials.yaml /tmp/testimonials.yaml
hugo --minify
grep -c "class=\"testimonials\"" public/index.html   # expect 0 (renders nothing)
mv /tmp/testimonials.yaml data/testimonials.yaml
hugo --minify
```

Expected: count 0 when the data file is absent, confirming the guard works. (File restored after.)

- [ ] **Step 7: Commit**

```bash
git add data/testimonials.yaml themes/laurenhendriks/layouts/partials/testimonials.html themes/laurenhendriks/layouts/index.html themes/laurenhendriks/assets/css/main.css
git commit -m "feat: add data-driven testimonials section on home"
```

---

## Task 4: FAQ page

Accordion FAQ at `/veelgestelde-vragen/`, data-driven, no JS.

**Files:**
- Create: `data/faq.yaml`
- Create: `content/veelgestelde-vragen.md`
- Create: `themes/laurenhendriks/layouts/_default/faq.html`
- Modify: `themes/laurenhendriks/layouts/partials/footer.html`
- Modify: `themes/laurenhendriks/assets/css/main.css` (FAQ styles)

- [ ] **Step 1: Create the data file**

Create `data/faq.yaml`:

```yaml
# Vul aan met echte vragen en antwoorden. 'answer' mag Markdown bevatten.
- question: "Wat is orthomoleculaire therapie?"
  answer: "Placeholder-antwoord. Vervang door een echte uitleg."
- question: "Hoe verloopt een eerste afspraak?"
  answer: "Placeholder-antwoord. Vervang door een echte uitleg."
- question: "Wordt een consult vergoed?"
  answer: "Placeholder-antwoord. Vervang door een echte uitleg."
```

- [ ] **Step 2: Create the content page**

Create `content/veelgestelde-vragen.md`:

```markdown
---
title: "Veelgestelde vragen"
layout: faq
subtitle: "Antwoorden op wat cliënten vaak vragen"
---
```

- [ ] **Step 3: Create the FAQ layout**

Create `themes/laurenhendriks/layouts/_default/faq.html`:

```html
{{ define "main" }}
<header class="page-header">
  <div class="container">
    <h1>{{ .Title }}</h1>
    {{ with .Params.subtitle }}<p class="page-header__sub">{{ . }}</p>{{ end }}
  </div>
</header>

<section class="faq">
  <div class="container faq__container">
    {{ with site.Data.faq }}
    {{ range . }}
    <details class="faq__item">
      <summary class="faq__question">{{ .question }}</summary>
      <div class="faq__answer">{{ .answer | markdownify }}</div>
    </details>
    {{ end }}
    {{ else }}
    <p>Er zijn nog geen vragen toegevoegd.</p>
    {{ end }}
  </div>
</section>
{{ end }}
```

- [ ] **Step 4: Add the footer link**

In `themes/laurenhendriks/layouts/partials/footer.html`, add inside the `.footer__links` nav, before the Privacyverklaring link:

```html
      <a href="{{ "/veelgestelde-vragen/" | relURL }}">Veelgestelde vragen</a>
```

- [ ] **Step 5: Add FAQ styles to main.css**

Append:

```css
/* FAQ */
.faq { padding: 3rem 0 5rem; }
.faq__container { max-width: 760px; }
.faq__item { border-bottom: 1px solid var(--surface, #E8DFD0); padding: 1.25rem 0; }
.faq__question { cursor: pointer; font-family: 'Cormorant Garamond', serif; font-size: 1.4rem; list-style: none; display: flex; justify-content: space-between; align-items: center; }
.faq__question::after { content: '+'; font-size: 1.5rem; line-height: 1; opacity: .6; }
.faq__item[open] .faq__question::after { content: '–'; }
.faq__question::-webkit-details-marker { display: none; }
.faq__answer { padding-top: 1rem; line-height: 1.7; }
```

- [ ] **Step 6: Build and assert**

```bash
hugo --minify
ls public/veelgestelde-vragen/index.html && echo "page-built"
grep -c "faq__item" public/veelgestelde-vragen/index.html       # expect 3
grep -c "veelgestelde-vragen" public/index.html                 # footer link on every page → expect >=1
```

Expected: page-built; 3 items; footer link ≥1.

- [ ] **Step 7: Visual check**

Run `hugo server` → open `/veelgestelde-vragen/` → confirm clicking a question expands/collapses it (native `<details>`), and the `+`/`–` marker toggles. Stop the server.

- [ ] **Step 8: Commit**

```bash
git add data/faq.yaml content/veelgestelde-vragen.md themes/laurenhendriks/layouts/_default/faq.html themes/laurenhendriks/layouts/partials/footer.html themes/laurenhendriks/assets/css/main.css
git commit -m "feat: add data-driven FAQ page with native accordion"
```

---

## Task 5: Subtle scroll motion

IntersectionObserver reveal, gated behind `prefers-reduced-motion`.

**Files:**
- Create: `themes/laurenhendriks/assets/js/reveal.js`
- Create: `themes/laurenhendriks/layouts/partials/reveal.html`
- Modify: `themes/laurenhendriks/layouts/_default/baseof.html` (load partial)
- Modify: `themes/laurenhendriks/assets/css/main.css` (reveal styles)
- Modify: `themes/laurenhendriks/layouts/index.html`, `_default/aanbod.html`, `partials/testimonials.html` (add `data-reveal`)

- [ ] **Step 1: Create the reveal script**

Create `themes/laurenhendriks/assets/js/reveal.js`:

```js
(function () {
  if (!window.matchMedia || window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
    return; // respect reduced-motion: leave everything visible
  }
  var els = document.querySelectorAll('[data-reveal]');
  if (!els.length || !('IntersectionObserver' in window)) return;
  els.forEach(function (el) { el.classList.add('reveal'); });
  var io = new IntersectionObserver(function (entries) {
    entries.forEach(function (entry) {
      if (entry.isIntersecting) {
        entry.target.classList.add('reveal--in');
        io.unobserve(entry.target);
      }
    });
  }, { threshold: 0.12 });
  els.forEach(function (el) { io.observe(el); });
})();
```

- [ ] **Step 2: Create the loader partial (fingerprinted, minified)**

Create `themes/laurenhendriks/layouts/partials/reveal.html`:

```html
{{ $js := resources.Get "js/reveal.js" | minify | fingerprint }}
<script src="{{ $js.RelPermalink }}" integrity="{{ $js.Data.Integrity }}" defer></script>
```

- [ ] **Step 3: Load it from baseof.html**

In `themes/laurenhendriks/layouts/_default/baseof.html`, add before `</body>` (after the footer partial):

```html
  {{ partial "reveal.html" . }}
```

- [ ] **Step 4: Add reveal styles to main.css**

Append:

```css
/* Scroll reveal (only applied when JS adds .reveal; reduced-motion users never get it) */
.reveal { opacity: 0; transform: translateY(18px); transition: opacity .6s ease, transform .6s ease; }
.reveal--in { opacity: 1; transform: none; }
```

- [ ] **Step 5: Tag sections with data-reveal**

In `index.html`: add `data-reveal` to the `<section class="intro">`, `<section class="pillars">` opening tags.
In `_default/aanbod.html`: add `data-reveal` to the `.intake-card`, `.options-section`, and `.programs-section` opening `<div>`s.
In `partials/testimonials.html`: add `data-reveal` to each `<figure class="testimonial">` opening tag.

Example (index.html intro):

```html
<section class="intro" data-reveal>
```

- [ ] **Step 6: Build and assert**

```bash
hugo --minify
grep -c "data-reveal" public/index.html        # expect >=2
grep -rl "reveal" public/js/ ; echo "exit:$?"   # fingerprinted reveal.<hash>.js exists → exit:0
grep -c "reveal.js\|reveal\." public/index.html # script tag present
```

Expected: data-reveal ≥2; a `public/js/reveal.*.js` file exists; script referenced on the page.

- [ ] **Step 7: Reduced-motion check**

Run `hugo server`. In the browser devtools, emulate `prefers-reduced-motion: reduce` and reload `/` → all sections must be fully visible immediately (no fade). Disable the emulation → sections fade in on scroll. Stop the server.

- [ ] **Step 8: Commit**

```bash
git add themes/laurenhendriks/assets/js/reveal.js themes/laurenhendriks/layouts/partials/reveal.html themes/laurenhendriks/layouts/_default/baseof.html themes/laurenhendriks/assets/css/main.css themes/laurenhendriks/layouts/index.html themes/laurenhendriks/layouts/_default/aanbod.html themes/laurenhendriks/layouts/partials/testimonials.html
git commit -m "feat: add subtle scroll-reveal motion with reduced-motion guard"
```

---

## Task 6: Optimized Over-mij image

Responsive, WebP, lazy-loaded image pipeline that activates when the real photo lands.

**Files:**
- Modify: `themes/laurenhendriks/layouts/_default/over-mij.html:13-21`
- Note: real photo will be dropped at `themes/laurenhendriks/assets/images/lauren-photo.jpg`

- [ ] **Step 1: Replace the hasPhoto branch with a processed-image pipeline**

In `themes/laurenhendriks/layouts/_default/over-mij.html`, replace the `{{ if .Site.Params.hasPhoto }} … {{ else }} … {{ end }}` block (lines 13–21) with:

```html
      {{ $img := resources.Get "images/lauren-photo.jpg" }}
      {{ if and .Site.Params.hasPhoto $img }}
      {{ $small := $img.Resize "400x webp q82" }}
      {{ $medium := $img.Resize "800x webp q82" }}
      {{ $large := $img.Resize "1200x webp q82" }}
      <div class="about__image">
        <img
          src="{{ $medium.RelPermalink }}"
          srcset="{{ $small.RelPermalink }} 400w, {{ $medium.RelPermalink }} 800w, {{ $large.RelPermalink }} 1200w"
          sizes="(max-width: 600px) 100vw, 400px"
          width="{{ $medium.Width }}" height="{{ $medium.Height }}"
          alt="Lauren Hendriks" loading="lazy" decoding="async">
      </div>
      {{ else }}
      <div class="about__image-placeholder">
        <p>Foto van Lauren<br>volgt binnenkort</p>
      </div>
      {{ end }}
```

(Note: the source moves from `static/images/` to `assets/images/` so Hugo can process it. The `and .Site.Params.hasPhoto $img` guard means: placeholder shows until both the flag is true AND the file exists — so the build never errors on a missing file.)

- [ ] **Step 2: Build and assert the placeholder still renders (no photo yet)**

```bash
hugo --minify
grep -c "about__image-placeholder" public/over-mij/index.html   # hasPhoto false → expect 1
grep -c "srcset" public/over-mij/index.html                     # expect 0
```

Expected: placeholder count 1, srcset count 0 (pipeline dormant until photo + flag).

- [ ] **Step 3: Smoke-test the pipeline with a temporary image**

```bash
mkdir -p themes/laurenhendriks/assets/images
# create a throwaway 1200px test jpg (uses ImageMagick if available; otherwise copy any jpg here)
magick -size 1200x1500 xc:#6B8F71 themes/laurenhendriks/assets/images/lauren-photo.jpg 2>/dev/null || true
# temporarily flip the flag
sed -i 's/hasPhoto = false/hasPhoto = true/' hugo.toml
hugo --minify
grep -c "srcset" public/over-mij/index.html    # expect 1 (pipeline active)
ls public/images/ | grep -c "webp" || true     # processed webp emitted
# revert the smoke test
sed -i 's/hasPhoto = true/hasPhoto = false/' hugo.toml
rm -f themes/laurenhendriks/assets/images/lauren-photo.jpg
hugo --minify
```

Expected: with the test image + flag on, `srcset` count is 1 and WebP files appear under `public/images/`. After revert, back to the placeholder. If `magick` is unavailable, skip the generation line and drop any real `.jpg` at that path manually for the smoke test.

- [ ] **Step 4: Commit**

```bash
git add themes/laurenhendriks/layouts/_default/over-mij.html
git commit -m "perf: responsive WebP image pipeline for Over-mij photo"
```

---

## Task 7: Final integration check & PR

**Files:** none

- [ ] **Step 1: Full clean build**

```bash
rm -rf public resources/_gen
hugo --minify
```

Expected: builds with no errors or warnings about missing partials/layouts.

- [ ] **Step 2: Manual walk-through**

Run `hugo server` and verify end to end:
- Home: dual hero CTA, testimonials section, fade-in on scroll.
- Nav: "Afspraak" is the CTA; active states correct.
- `/afspraak/`: fallback message (Calendly not configured yet).
- `/aanbod/`: "Plan een kennismaking" → `/afspraak/`.
- `/veelgestelde-vragen/`: accordion works; linked from footer.
- Fonts render locally (no network request to fonts.googleapis.com in the Network tab).
Stop the server.

- [ ] **Step 3: Push and open PR**

```bash
git push -u origin feat/site-improvements
gh pr create --base master --title "Site improvements: booking, testimonials, FAQ, polish" --body "Implements docs/superpowers/specs/2026-06-16-site-improvements-design.md — /afspraak/ click-to-load Calendly booking page, data-driven testimonials and FAQ, self-hosted fonts, subtle scroll motion, and a responsive WebP image pipeline.

🤖 Generated with [Claude Code](https://claude.com/claude-code)"
```

---

## Configuration follow-ups (not code — for the site owner)

These are post-merge, manual, and intentionally **not** part of any task:
- Set `calendlyUrl` and `hasCalendly = true` in `hugo.toml` once Lauren has a Calendly account.
- Replace placeholder testimonials in `data/testimonials.yaml` with real (anonymised) quotes.
- Replace placeholder Q&As in `data/faq.yaml`.
- Drop the real photo at `themes/laurenhendriks/assets/images/lauren-photo.jpg` and set `hasPhoto = true`.
