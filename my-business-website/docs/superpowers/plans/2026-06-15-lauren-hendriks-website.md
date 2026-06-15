# Lauren Hendriks Website — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a 4-page Dutch Hugo website for orthomolecular therapist Lauren Hendriks, deployed to GitHub Pages at laurenhendriks.nl.

**Architecture:** Custom Hugo theme `laurenhendriks` built from scratch. Each page has its own layout template pulling structured content from markdown front matter. The ananke submodule is removed. Contact form uses Formspree; the partial is isolated so it can be swapped for Calendly later.

**Tech Stack:** Hugo (static site generator), HTML/CSS (no JS frameworks), Google Fonts (Cormorant Garamond + Lato), Formspree (contact form backend), GitHub Actions + GitHub Pages (CI/CD + hosting).

---

## File Map

```
.github/workflows/deploy.yml         ← GitHub Actions deploy pipeline
static/CNAME                          ← custom domain for GitHub Pages
hugo.toml                             ← site config, Formspree endpoint, photo flag
content/
  _index.md                           ← Home content + pillar data
  over-mij.md                         ← About content + credentials list
  aanbod.md                           ← Services + programs list
  contact.md                          ← Contact intro text
themes/laurenhendriks/
  layouts/
    _default/
      baseof.html                     ← HTML shell: <head>, nav, footer block
      over-mij.html                   ← About page layout
      aanbod.html                     ← Services page layout
      contact.html                    ← Contact page layout
    index.html                        ← Home page layout
    partials/
      nav.html                        ← Fixed top navigation bar
      footer.html                     ← Site footer
      contact-form.html               ← Formspree form (swap for Calendly here)
  assets/css/main.css                 ← All styles (CSS custom properties, responsive)
  static/images/fern.svg             ← White fern SVG for hero background pattern
```

---

## Task 1: Remove Ananke & Configure Hugo

**Files:**
- Modify: `hugo.toml`
- Create: `static/CNAME`
- Remove: `themes/ananke` submodule

- [ ] **Step 1: Remove the ananke git submodule**

```bash
git submodule deinit -f themes/ananke
git rm -f themes/ananke
rm -rf .git/modules/my-business-website/themes/ananke
```

Expected: `themes/ananke` directory is gone. `.gitmodules` may be removed automatically.

- [ ] **Step 2: Replace hugo.toml**

Overwrite `hugo.toml` with:

```toml
baseURL = 'https://laurenhendriks.nl/'
languageCode = 'nl'
title = 'Lauren Hendriks'
theme = 'laurenhendriks'

[params]
  description = 'Orthomoleculair therapeut | voeding · gezondheid · leefstijl'
  formspreeEndpoint = 'https://formspree.io/f/YOUR_FORM_ID'
  hasPhoto = false
```

> When Lauren's photo is ready, set `hasPhoto = true` and drop `lauren-photo.jpg` in `themes/laurenhendriks/static/images/`.  
> When Formspree is set up, replace `YOUR_FORM_ID` with the actual form ID from formspree.io.

- [ ] **Step 3: Create CNAME file**

Create `static/CNAME` with content:

```
laurenhendriks.nl
```

Hugo copies `static/` to `public/` at build time, so this file ends up in the right place for GitHub Pages.

- [ ] **Step 4: Verify config is valid**

Run: `hugo config`  
Expected: outputs the parsed config with `baseURL = https://laurenhendriks.nl/`, no errors.

- [ ] **Step 5: Commit**

```bash
git add hugo.toml static/CNAME
git commit -m "feat: configure Hugo for laurenhendriks.nl, remove ananke"
```

---

## Task 2: Create Theme Shell

**Files:**
- Create: `themes/laurenhendriks/layouts/_default/baseof.html`
- Create: `themes/laurenhendriks/layouts/partials/nav.html`
- Create: `themes/laurenhendriks/layouts/partials/footer.html`

- [ ] **Step 1: Create theme directory structure**

```bash
mkdir -p themes/laurenhendriks/layouts/_default
mkdir -p themes/laurenhendriks/layouts/partials
mkdir -p themes/laurenhendriks/assets/css
mkdir -p themes/laurenhendriks/static/images
```

- [ ] **Step 2: Create `themes/laurenhendriks/layouts/_default/baseof.html`**

```html
<!DOCTYPE html>
<html lang="nl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{ if .IsHome }}{{ .Site.Title }} — voeding · gezondheid · leefstijl{{ else }}{{ .Title }} — {{ .Site.Title }}{{ end }}</title>
  <meta name="description" content="{{ .Site.Params.description }}">
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Cormorant+Garamond:ital,wght@0,300;0,400;0,600;1,300;1,400&family=Lato:wght@300;400;700&display=swap" rel="stylesheet">
  {{ $css := resources.Get "css/main.css" | minify }}
  <link rel="stylesheet" href="{{ $css.RelPermalink }}">
</head>
<body>
  {{ partial "nav.html" . }}
  <main>
    {{ block "main" . }}{{ end }}
  </main>
  {{ partial "footer.html" . }}
</body>
</html>
```

- [ ] **Step 3: Create `themes/laurenhendriks/layouts/partials/nav.html`**

```html
<nav class="nav">
  <div class="nav__container">
    <a href="{{ "/" | relURL }}" class="nav__logo">Lauren Hendriks</a>
    <ul class="nav__links">
      <li><a href="{{ "/" | relURL }}"{{ if .IsHome }} class="nav__link--active"{{ end }}>Home</a></li>
      <li><a href="{{ "/over-mij/" | relURL }}"{{ if strings.HasPrefix .RelPermalink "/over-mij" }} class="nav__link--active"{{ end }}>Over mij</a></li>
      <li><a href="{{ "/aanbod/" | relURL }}"{{ if strings.HasPrefix .RelPermalink "/aanbod" }} class="nav__link--active"{{ end }}>Aanbod</a></li>
      <li><a href="{{ "/contact/" | relURL }}" class="nav__cta{{ if strings.HasPrefix .RelPermalink "/contact" }} nav__link--active{{ end }}">Contact</a></li>
    </ul>
  </div>
</nav>
```

- [ ] **Step 4: Create `themes/laurenhendriks/layouts/partials/footer.html`**

```html
<footer class="footer">
  <div class="container">
    <p class="footer__name">Lauren Hendriks</p>
    <p class="footer__tagline">voeding · gezondheid · leefstijl</p>
    <p class="footer__copy">&copy; {{ now.Year }} Lauren Hendriks</p>
  </div>
</footer>
```

- [ ] **Step 5: Commit**

```bash
git add themes/laurenhendriks/
git commit -m "feat: add theme shell (baseof, nav, footer)"
```

---

## Task 3: Build CSS

**Files:**
- Create: `themes/laurenhendriks/assets/css/main.css`

- [ ] **Step 1: Create `themes/laurenhendriks/assets/css/main.css`**

```css
/* ─── Variables ─────────────────────────────────────────── */
:root {
  --color-primary:   #2D4A3E;
  --color-secondary: #6B8F71;
  --color-bg:        #F5F0E8;
  --color-surface:   #E8DFD0;
  --color-text:      #2C2C2C;
  --color-light:     #FAFAF7;
  --color-white:     #FFFFFF;

  --font-heading: 'Cormorant Garamond', Georgia, serif;
  --font-body:    'Lato', sans-serif;

  --max-width: 1100px;
  --radius: 8px;
}

/* ─── Reset ─────────────────────────────────────────────── */
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

/* ─── Base ──────────────────────────────────────────────── */
html  { scroll-behavior: smooth; }
body  { font-family: var(--font-body); color: var(--color-text); background: var(--color-bg); line-height: 1.7; }
h1, h2, h3, h4 { font-family: var(--font-heading); font-weight: 400; line-height: 1.2; }
h1 { font-size: clamp(2.4rem, 6vw, 4.5rem); }
h2 { font-size: clamp(1.7rem, 4vw, 3rem); }
h3 { font-size: clamp(1.1rem, 3vw, 1.6rem); }
p  { font-size: 1.05rem; }
a  { color: inherit; text-decoration: none; }
img { max-width: 100%; height: auto; display: block; }

/* ─── Utilities ─────────────────────────────────────────── */
.container {
  max-width: var(--max-width);
  margin: 0 auto;
  padding: 0 1.5rem;
}

.section-divider {
  width: 50px;
  height: 2px;
  background: var(--color-secondary);
  margin: 1.25rem auto 2rem;
}
.section-divider--left { margin-left: 0; }

/* ─── Buttons ───────────────────────────────────────────── */
.btn {
  display: inline-block;
  padding: 0.75rem 2rem;
  border-radius: var(--radius);
  font-family: var(--font-body);
  font-size: 0.9rem;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  transition: all 0.2s ease;
  cursor: pointer;
  border: 2px solid transparent;
}
.btn--primary {
  background: var(--color-primary);
  color: var(--color-white);
  border-color: var(--color-primary);
}
.btn--primary:hover  { background: var(--color-secondary); border-color: var(--color-secondary); }
.btn--light {
  background: transparent;
  color: var(--color-white);
  border-color: rgba(255,255,255,0.65);
}
.btn--light:hover { background: rgba(255,255,255,0.12); border-color: var(--color-white); }

/* ─── Nav ───────────────────────────────────────────────── */
.nav {
  position: fixed;
  inset: 0 0 auto 0;
  z-index: 100;
  background: rgba(45,74,62,0.97);
  backdrop-filter: blur(8px);
}
.nav__container {
  max-width: var(--max-width);
  margin: 0 auto;
  padding: 0 1.5rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 4rem;
}
.nav__logo {
  font-family: var(--font-heading);
  font-size: 1.25rem;
  color: var(--color-white);
  letter-spacing: 0.02em;
}
.nav__links {
  list-style: none;
  display: flex;
  gap: 2rem;
  align-items: center;
}
.nav__links a {
  color: rgba(255,255,255,0.8);
  font-size: 0.85rem;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  transition: color 0.2s;
}
.nav__links a:hover,
.nav__links .nav__link--active { color: var(--color-white); }
.nav__links .nav__cta {
  padding: 0.35rem 1.1rem;
  border: 1px solid rgba(255,255,255,0.45);
  border-radius: var(--radius);
}
.nav__links .nav__cta:hover { background: rgba(255,255,255,0.1); border-color: var(--color-white); }

/* ─── Hero ──────────────────────────────────────────────── */
.hero {
  min-height: 100vh;
  background-color: var(--color-primary);
  background-image: url('/images/fern.svg');
  background-size: 180px;
  background-repeat: repeat;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  padding: 6rem 1.5rem 4rem;
}
.hero__content { max-width: 700px; }
.hero__name {
  color: var(--color-white);
  font-weight: 300;
  font-style: italic;
  letter-spacing: 0.03em;
  margin-bottom: 1.25rem;
}
.hero__divider {
  width: 60px;
  height: 1px;
  background: rgba(255,255,255,0.45);
  margin: 0 auto 1.25rem;
}
.hero__tagline {
  color: rgba(255,255,255,0.8);
  font-size: 0.95rem;
  letter-spacing: 0.18em;
  text-transform: uppercase;
  margin-bottom: 2.5rem;
}

/* ─── Intro ─────────────────────────────────────────────── */
.intro { padding: 5rem 0; background: var(--color-light); }
.intro__text {
  font-family: var(--font-heading);
  font-size: clamp(1.1rem, 2.5vw, 1.45rem);
  text-align: center;
  max-width: 680px;
  margin: 0 auto;
  color: var(--color-primary);
  line-height: 1.65;
  font-style: italic;
}
.intro__text p { font-size: inherit; }

/* ─── Pillars ───────────────────────────────────────────── */
.pillars { padding: 4rem 0 5rem; background: var(--color-bg); }
.pillars__grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1.75rem;
}
.pillar {
  background: var(--color-surface);
  border-radius: var(--radius);
  padding: 2.5rem 2rem;
  text-align: center;
  border-top: 3px solid var(--color-secondary);
}
.pillar h3 { color: var(--color-primary); margin-bottom: 0.75rem; }
.pillar p  { font-size: 0.95rem; line-height: 1.6; }

/* ─── Page Header (inner pages) ─────────────────────────── */
.page-header {
  background: var(--color-primary);
  padding: 8rem 1.5rem 4rem;
  text-align: center;
}
.page-header h1     { color: var(--color-white); font-weight: 300; font-style: italic; }
.page-header__sub   { color: rgba(255,255,255,0.7); margin-top: 0.75rem; font-size: 1.05rem; }

/* ─── Over mij ──────────────────────────────────────────── */
.about { padding: 5rem 0; }
.about__grid {
  display: grid;
  grid-template-columns: 1fr 1.4fr;
  gap: 4rem;
  align-items: start;
}
.about__image { border-radius: var(--radius); overflow: hidden; }
.about__image img { width: 100%; object-fit: cover; aspect-ratio: 3/4; }
.about__image-placeholder {
  width: 100%;
  aspect-ratio: 3/4;
  background: var(--color-surface);
  border-radius: var(--radius);
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--color-secondary);
  font-family: var(--font-heading);
  font-style: italic;
  font-size: 1rem;
  text-align: center;
  padding: 2rem;
  border: 2px dashed var(--color-surface);
}
.about__content h2  { color: var(--color-primary); margin-bottom: 0.5rem; }
.about__content p   { margin-bottom: 1.25rem; }
.about__credentials { margin-top: 2.5rem; padding-top: 2rem; border-top: 1px solid var(--color-surface); }
.about__credentials h3  { color: var(--color-primary); margin-bottom: 1rem; font-size: 1.15rem; }
.about__credentials ul  { list-style: none; }
.about__credentials li  { padding: 0.4rem 0 0.4rem 1.4rem; position: relative; font-size: 0.97rem; }
.about__credentials li::before { content: '·'; position: absolute; left: 0; color: var(--color-secondary); font-size: 1.4rem; line-height: 1; top: 0.3rem; }

/* ─── Aanbod ────────────────────────────────────────────── */
.aanbod { padding: 5rem 0; }
.aanbod__intro { text-align: center; max-width: 640px; margin: 0 auto 4rem; }

.intake-card {
  background: var(--color-primary);
  border-radius: var(--radius);
  padding: 3rem;
  margin-bottom: 4rem;
}
.intake-card h2   { color: var(--color-white); margin-bottom: 1rem; }
.intake-card p    { color: rgba(255,255,255,0.82); margin-bottom: 1.5rem; }

.options-section  { margin-bottom: 4rem; }
.options-section > h2 { color: var(--color-primary); text-align: center; margin-bottom: 0.5rem; }
.options-section > p  { text-align: center; margin-bottom: 2.5rem; max-width: 580px; margin-left: auto; margin-right: auto; }
.options-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1.5rem;
}
.option-card {
  background: var(--color-surface);
  border-radius: var(--radius);
  padding: 2rem;
  border-left: 3px solid var(--color-secondary);
}
.option-card h3 { color: var(--color-primary); margin-bottom: 0.75rem; font-size: 1.1rem; }
.option-card p  { font-size: 0.95rem; }

.programs-section > h2 { color: var(--color-primary); text-align: center; margin-bottom: 0.5rem; }
.programs-section > p  { text-align: center; margin-bottom: 2.5rem; max-width: 580px; margin-left: auto; margin-right: auto; }
.programs-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 2rem;
}
.program-card {
  background: var(--color-light);
  border-radius: var(--radius);
  padding: 2.5rem 2rem;
  border: 1px solid var(--color-surface);
}
.program-card h3 { color: var(--color-primary); margin-bottom: 1rem; }
.program-card p  { font-size: 0.95rem; }

/* ─── Contact ───────────────────────────────────────────── */
.contact { padding: 5rem 0; }
.contact__grid {
  display: grid;
  grid-template-columns: 1fr 1.5fr;
  gap: 4rem;
  align-items: start;
}
.contact__info h2 { color: var(--color-primary); margin-bottom: 0.5rem; }
.contact__info p  { margin-bottom: 1.25rem; }
.contact-form { display: flex; flex-direction: column; gap: 1.25rem; }
.form-group   { display: flex; flex-direction: column; gap: 0.4rem; }
.form-group label {
  font-size: 0.85rem;
  font-weight: 700;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  color: var(--color-primary);
}
.form-group input,
.form-group textarea {
  padding: 0.75rem 1rem;
  border: 1.5px solid var(--color-surface);
  border-radius: var(--radius);
  background: var(--color-light);
  font-family: var(--font-body);
  font-size: 1rem;
  color: var(--color-text);
  transition: border-color 0.2s;
}
.form-group input:focus,
.form-group textarea:focus  { outline: none; border-color: var(--color-secondary); }
.form-group textarea { resize: vertical; min-height: 150px; }
.contact-form .btn  { align-self: flex-start; }

/* ─── Footer ────────────────────────────────────────────── */
.footer {
  background: var(--color-primary);
  color: rgba(255,255,255,0.72);
  text-align: center;
  padding: 3rem 1.5rem;
}
.footer__name    { font-family: var(--font-heading); font-size: 1.4rem; color: var(--color-white); margin-bottom: 0.25rem; }
.footer__tagline { font-size: 0.82rem; letter-spacing: 0.12em; text-transform: uppercase; margin-bottom: 1.5rem; }
.footer__copy    { font-size: 0.78rem; opacity: 0.55; }

/* ─── Responsive ────────────────────────────────────────── */
@media (max-width: 768px) {
  .about__grid   { grid-template-columns: 1fr; }
  .contact__grid { grid-template-columns: 1fr; }
  .intake-card   { padding: 2rem 1.5rem; }
  .nav__links    { gap: 1rem; }
  .nav__links a  { font-size: 0.78rem; }
}

@media (max-width: 480px) {
  .nav__links .nav__cta { display: none; }
  .hero { min-height: 90vh; }
}
```

- [ ] **Step 2: Commit**

```bash
git add themes/laurenhendriks/assets/
git commit -m "feat: add main.css with color palette, typography, all page styles"
```

---

## Task 4: Create Fern SVG

**Files:**
- Create: `themes/laurenhendriks/static/images/fern.svg`

- [ ] **Step 1: Create `themes/laurenhendriks/static/images/fern.svg`**

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 80 160" fill="none">
  <line x1="40" y1="155" x2="40" y2="15" stroke="white" stroke-width="1.5" stroke-linecap="round" opacity="0.55"/>
  <path d="M40 135 Q22 120 14 118" stroke="white" stroke-width="1" stroke-linecap="round" opacity="0.45"/>
  <path d="M40 115 Q20 98 10 94"  stroke="white" stroke-width="1" stroke-linecap="round" opacity="0.45"/>
  <path d="M40 95  Q20 77 11 72"  stroke="white" stroke-width="1" stroke-linecap="round" opacity="0.45"/>
  <path d="M40 76  Q22 58 15 52"  stroke="white" stroke-width="1" stroke-linecap="round" opacity="0.45"/>
  <path d="M40 57  Q25 41 20 35"  stroke="white" stroke-width="1" stroke-linecap="round" opacity="0.4"/>
  <path d="M40 40  Q28 26 25 20"  stroke="white" stroke-width="1" stroke-linecap="round" opacity="0.35"/>
  <path d="M40 135 Q58 120 66 118" stroke="white" stroke-width="1" stroke-linecap="round" opacity="0.45"/>
  <path d="M40 115 Q60 98 70 94"  stroke="white" stroke-width="1" stroke-linecap="round" opacity="0.45"/>
  <path d="M40 95  Q60 77 69 72"  stroke="white" stroke-width="1" stroke-linecap="round" opacity="0.45"/>
  <path d="M40 76  Q58 58 65 52"  stroke="white" stroke-width="1" stroke-linecap="round" opacity="0.45"/>
  <path d="M40 57  Q55 41 60 35"  stroke="white" stroke-width="1" stroke-linecap="round" opacity="0.4"/>
  <path d="M40 40  Q52 26 55 20"  stroke="white" stroke-width="1" stroke-linecap="round" opacity="0.35"/>
</svg>
```

- [ ] **Step 2: Commit**

```bash
git add themes/laurenhendriks/static/
git commit -m "feat: add white fern SVG for hero background"
```

---

## Task 5: Build Home Page

**Files:**
- Create: `themes/laurenhendriks/layouts/index.html`
- Create: `content/_index.md`

- [ ] **Step 1: Create `themes/laurenhendriks/layouts/index.html`**

```html
{{ define "main" }}
<section class="hero">
  <div class="hero__content">
    <h1 class="hero__name">Lauren Hendriks</h1>
    <div class="hero__divider"></div>
    <p class="hero__tagline">voeding · gezondheid · leefstijl</p>
    <a href="{{ "/aanbod/" | relURL }}" class="btn btn--light">Bekijk het aanbod</a>
  </div>
</section>

<section class="intro">
  <div class="container">
    <div class="intro__text">{{ .Content }}</div>
  </div>
</section>

<section class="pillars">
  <div class="container pillars__grid">
    {{ range .Params.pillars }}
    <div class="pillar">
      <h3>{{ .title }}</h3>
      <p>{{ .text }}</p>
    </div>
    {{ end }}
  </div>
</section>
{{ end }}
```

- [ ] **Step 2: Create `content/_index.md`**

```markdown
---
title: "Home"
pillars:
  - title: "Persoonlijk"
    text: "Elk traject begint bij jou — jouw klachten, jouw verhaal, jouw doelen."
  - title: "Natuurlijk"
    text: "Werken met voeding, suppletie en leefstijl als basis voor echte balans."
  - title: "Wetenschappelijk"
    text: "Orthomoleculaire inzichten als fundament voor gerichte ondersteuning."
---

Als orthomoleculair therapeut help ik je op zoek naar de oorzaak van jouw klachten — niet alleen de symptomen. Vanuit een holistische benadering kijken we samen naar voeding, suppletie en leefstijl om jouw gezondheid duurzaam te verbeteren.
```

- [ ] **Step 3: Verify in browser**

Run: `hugo server -D`  
Open: `http://localhost:1313`  
Expected:
- Dark green hero with repeating fern pattern
- "Lauren Hendriks" in large italic serif
- Tagline "voeding · gezondheid · leefstijl" below
- Intro text in italicized serif on cream background
- Three green-topped pillar cards

Press `Ctrl+C` to stop the server.

- [ ] **Step 4: Commit**

```bash
git add themes/laurenhendriks/layouts/index.html content/_index.md
git commit -m "feat: add Home page layout and content"
```

---

## Task 6: Build Over Mij Page

**Files:**
- Create: `themes/laurenhendriks/layouts/_default/over-mij.html`
- Create: `content/over-mij.md`

- [ ] **Step 1: Create `themes/laurenhendriks/layouts/_default/over-mij.html`**

```html
{{ define "main" }}
<header class="page-header">
  <div class="container">
    <h1>{{ .Title }}</h1>
    {{ with .Params.subtitle }}<p class="page-header__sub">{{ . }}</p>{{ end }}
  </div>
</header>

<section class="about">
  <div class="container about__grid">

    <div>
      {{ if .Site.Params.hasPhoto }}
      <div class="about__image">
        <img src="/images/lauren-photo.jpg" alt="Lauren Hendriks">
      </div>
      {{ else }}
      <div class="about__image-placeholder">
        <p>Foto van Lauren<br>volgt binnenkort</p>
      </div>
      {{ end }}
    </div>

    <div class="about__content">
      <h2>{{ .Params.subtitle | default "Wie ben ik?" }}</h2>
      <div class="section-divider section-divider--left"></div>
      {{ .Content }}

      {{ with .Params.credentials }}
      <div class="about__credentials">
        <h3>Opleiding &amp; certificering</h3>
        <ul>
          {{ range . }}
          <li>{{ . }}</li>
          {{ end }}
        </ul>
      </div>
      {{ end }}
    </div>

  </div>
</section>
{{ end }}
```

- [ ] **Step 2: Create `content/over-mij.md`**

```markdown
---
title: "Over mij"
layout: over-mij
subtitle: "Kennismaking met Lauren"
credentials:
  - "Opleiding orthomoleculaire therapie — [naam opleiding invullen]"
  - "Gecertificeerd orthomoleculair therapeut"
  - "Lid van [beroepsvereniging invullen]"
---

Hier kun je jouw persoonlijke verhaal kwijt. Wie ben je, wat drijft je, en hoe ben je bij orthomoleculaire therapie terechtgekomen?

Schrijf over wat jou op dit pad heeft gebracht — jouw eigen ervaringen met gezondheid, voeding en leefstijl maken het verschil voor bezoekers die jou willen leren kennen.

Vanuit een warme en niet-oordelende benadering begeleid ik mensen naar meer balans en vitaliteit. Ik geloof dat ieder lichaam de capaciteit heeft om zichzelf te herstellen — met de juiste bouwstenen.
```

> **Note for Lauren:** Replace the placeholder text above and fill in the `credentials` list in the front matter with your actual opleiding and certifications.  
> **Photo:** When the photo is ready, set `hasPhoto = true` in `hugo.toml` and drop `lauren-photo.jpg` into `themes/laurenhendriks/static/images/`.

- [ ] **Step 3: Verify in browser**

Run: `hugo server -D`  
Open: `http://localhost:1313/over-mij/`  
Expected:
- Green page header with "Over mij"
- Photo placeholder box on the left
- Bio text and credentials list on the right
- Nav "Over mij" link appears active

- [ ] **Step 4: Commit**

```bash
git add themes/laurenhendriks/layouts/_default/over-mij.html content/over-mij.md
git commit -m "feat: add Over mij page layout and content"
```

---

## Task 7: Build Aanbod Page

**Files:**
- Create: `themes/laurenhendriks/layouts/_default/aanbod.html`
- Create: `content/aanbod.md`

- [ ] **Step 1: Create `themes/laurenhendriks/layouts/_default/aanbod.html`**

```html
{{ define "main" }}
<header class="page-header">
  <div class="container">
    <h1>{{ .Title }}</h1>
    {{ with .Params.subtitle }}<p class="page-header__sub">{{ . }}</p>{{ end }}
  </div>
</header>

<section class="aanbod">
  <div class="container">

    <div class="intake-card">
      <h2>Kennismakingsgesprek &amp; intake</h2>
      <p>Elk traject begint met een persoonlijk kennismakingsgesprek. Hier leer ik je kennen: jouw klachten, jouw doelen en jouw leefstijl. Op basis van dit gesprek bepalen we samen welke stap het beste bij jou past — geen aannames, geen standaardoplossingen.</p>
      <a href="{{ "/contact/" | relURL }}" class="btn btn--light">Plan een kennismaking</a>
    </div>

    <div class="options-section">
      <h2>Wat kan er volgen?</h2>
      <div class="section-divider"></div>
      <p>Afhankelijk van de intake adviseer ik wat het beste bij jouw situatie past. Dat kan zijn:</p>
      <div class="options-grid">
        <div class="option-card">
          <h3>Voeding &amp; suppletie</h3>
          <p>Gerichte adviezen op het gebied van voeding en orthomoleculaire suppletie, afgestemd op jouw klachtenpatroon.</p>
        </div>
        <div class="option-card">
          <h3>Leefstijlbegeleiding</h3>
          <p>Praktische handvatten rondom slaap, stress, beweging en andere leefstijlfactoren die bijdragen aan jouw herstel en vitaliteit.</p>
        </div>
        <div class="option-card">
          <h3>Een begeleidingstraject</h3>
          <p>Voor wie meer diepgang zoekt: persoonlijke begeleiding over een langere periode, volledig op maat.</p>
        </div>
      </div>
    </div>

    {{ with .Params.programs }}
    <div class="programs-section">
      <h2>Trajecten</h2>
      <div class="section-divider"></div>
      <p>Voor wie een dieper traject past, bied ik de volgende begeleiding aan. Hoe lang een traject duurt verschilt per persoon — dat kan weken zijn, maar ook maanden.</p>
      <div class="programs-grid">
        {{ range . }}
        <div class="program-card">
          <h3>{{ .title }}</h3>
          <p>{{ .description }}</p>
        </div>
        {{ end }}
      </div>
    </div>
    {{ end }}

  </div>
</section>
{{ end }}
```

- [ ] **Step 2: Create `content/aanbod.md`**

```markdown
---
title: "Aanbod"
layout: aanbod
subtitle: "Hoe we samenwerken"
programs:
  - title: "Traject naam 1"
    description: "Beschrijving van dit traject. Wat houdt het in, voor wie is het geschikt en wat kun je verwachten? Vul dit in samen met Lauren."
  - title: "Traject naam 2"
    description: "Beschrijving van dit traject. Wat houdt het in, voor wie is het geschikt en wat kun je verwachten? Vul dit in samen met Lauren."
  - title: "Traject naam 3"
    description: "Beschrijving van dit traject. Wat houdt het in, voor wie is het geschikt en wat kun je verwachten? Vul dit in samen met Lauren."
---
```

> **Note for Lauren:** Replace the three program placeholders with the actual names and descriptions of your programs. Remove any entries you don't need.

- [ ] **Step 3: Verify in browser**

Run: `hugo server -D`  
Open: `http://localhost:1313/aanbod/`  
Expected:
- Green page header with "Aanbod"
- Dark green intake card with CTA button linking to `/contact/`
- Three option cards (voeding, leefstijl, traject)
- Three program cards with placeholder text

- [ ] **Step 4: Commit**

```bash
git add themes/laurenhendriks/layouts/_default/aanbod.html content/aanbod.md
git commit -m "feat: add Aanbod page layout and content"
```

---

## Task 8: Build Contact Page

**Files:**
- Create: `themes/laurenhendriks/layouts/partials/contact-form.html`
- Create: `themes/laurenhendriks/layouts/_default/contact.html`
- Create: `content/contact.md`

- [ ] **Step 1: Create `themes/laurenhendriks/layouts/partials/contact-form.html`**

```html
<form action="{{ .Site.Params.formspreeEndpoint }}" method="POST" class="contact-form">
  <div class="form-group">
    <label for="name">Naam</label>
    <input type="text" id="name" name="name" required placeholder="Jouw naam">
  </div>
  <div class="form-group">
    <label for="email">E-mailadres</label>
    <input type="email" id="email" name="email" required placeholder="jouw@email.nl">
  </div>
  <div class="form-group">
    <label for="message">Bericht</label>
    <textarea id="message" name="message" rows="6" required placeholder="Schrijf hier je bericht..."></textarea>
  </div>
  <button type="submit" class="btn btn--primary">Verstuur bericht</button>
</form>
```

> **To swap in Calendly later:** Replace the entire content of this file with a Calendly embed snippet. No other files need to change.

- [ ] **Step 2: Create `themes/laurenhendriks/layouts/_default/contact.html`**

```html
{{ define "main" }}
<header class="page-header">
  <div class="container">
    <h1>{{ .Title }}</h1>
    {{ with .Params.subtitle }}<p class="page-header__sub">{{ . }}</p>{{ end }}
  </div>
</header>

<section class="contact">
  <div class="container contact__grid">
    <div class="contact__info">
      <h2>Neem contact op</h2>
      <div class="section-divider section-divider--left"></div>
      {{ .Content }}
    </div>
    <div>
      {{ partial "contact-form.html" . }}
    </div>
  </div>
</section>
{{ end }}
```

- [ ] **Step 3: Create `content/contact.md`**

```markdown
---
title: "Contact"
layout: contact
subtitle: "Neem contact op"
---

Heb je een vraag of wil je een kennismakingsgesprek inplannen? Vul het formulier in en ik neem zo snel mogelijk contact met je op.

Ik reageer doorgaans binnen 1–2 werkdagen.
```

- [ ] **Step 4: Verify in browser**

Run: `hugo server -D`  
Open: `http://localhost:1313/contact/`  
Expected:
- Green page header with "Contact"
- Left column: intro text from markdown
- Right column: contact form with naam, email, bericht fields and send button
- Form `action` points to the Formspree placeholder URL

- [ ] **Step 5: Commit**

```bash
git add themes/laurenhendriks/layouts/partials/contact-form.html
git add themes/laurenhendriks/layouts/_default/contact.html
git add content/contact.md
git commit -m "feat: add Contact page with isolated Formspree form partial"
```

---

## Task 9: Add GitHub Actions Deployment

**Files:**
- Create: `.github/workflows/deploy.yml`

- [ ] **Step 1: Create `.github/workflows/deploy.yml`**

```bash
mkdir -p .github/workflows
```

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy Hugo to GitHub Pages

on:
  push:
    branches: [master]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: false

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build
        run: hugo --minify

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

> **After pushing:** In the GitHub repo settings → Pages → set Source to "GitHub Actions". Then add `laurenhendriks.nl` as the custom domain. Point DNS at your registrar:  
> A records → `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`  
> CNAME `www` → `<your-github-username>.github.io`

- [ ] **Step 2: Commit**

```bash
git add .github/
git commit -m "feat: add GitHub Actions workflow for GitHub Pages deployment"
```

---

## Task 10: Final Build Verification

- [ ] **Step 1: Run a clean production build**

```bash
hugo --minify
```

Expected output (no errors, page counts for all 4 pages):
```
                   | EN
-------------------+-----
  Pages            |  7
  Paginator pages  |  0
  Non-page files   |  0
  Static files     |  2
  Processed images |  0
  Aliases          |  0
  Cleaned          |  0

Total in XXXms
```

- [ ] **Step 2: Check the built output**

```bash
ls public/
```

Expected: `index.html`, `over-mij/`, `aanbod/`, `contact/`, `images/`, `CNAME`, `css/`

- [ ] **Step 3: Push to GitHub**

```bash
git push origin master
```

Then in GitHub: Settings → Pages → Source: GitHub Actions. The first deploy will run automatically.

- [ ] **Step 4: Set up Formspree**

1. Go to formspree.io → create a free account
2. Create a new form → copy the endpoint URL (looks like `https://formspree.io/f/abcdefgh`)
3. Edit `hugo.toml`, replace `YOUR_FORM_ID` with the actual ID
4. Commit and push — Formspree will send a confirmation email to Lauren's inbox on the first submission

---

## Post-launch checklist

- [ ] Replace program placeholders in `content/aanbod.md` with Lauren's actual programs
- [ ] Replace bio placeholder in `content/over-mij.md` with Lauren's actual text
- [ ] Fill in `credentials` list in `over-mij.md` front matter
- [ ] Set `formspreeEndpoint` in `hugo.toml` to real Formspree URL
- [ ] When photo is ready: set `hasPhoto = true` in `hugo.toml`, add `lauren-photo.jpg` to `themes/laurenhendriks/static/images/`
- [ ] Configure DNS at domain registrar (A records → GitHub IPs listed in Task 9)
- [ ] Enable GitHub Pages in repo settings
