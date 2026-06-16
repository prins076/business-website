---
name: site-improvements-design
description: Design spec for the next round of laurenhendriks.nl improvements — booking page, testimonials, FAQ, and polish (self-hosted fonts, motion, optimized images)
metadata:
  type: project
---

# Lauren Hendriks — Site Improvements Design Spec

**Domain:** laurenhendriks.nl
**Date:** 2026-06-16
**Builds on:** `2026-06-15-holistic-health-site-design.md`

This round adds three new capabilities (online booking, testimonials, FAQ) and
three polish refinements (self-hosted fonts, subtle motion, optimized images).

**Explicitly out of scope:** blog, SEO/structured-data work, conversion tactics,
and editing any placeholder copy.

---

## 1. Information architecture & navigation

A new **booking page** is introduced. It is generic ("afspraak") rather than
kennismaking-specific, because returning clients may book follow-up calls — not
only a first kennismakingsgesprek.

- **New page:** `/afspraak/` — page title "Afspraak maken".
- **Navigation** becomes: Home · Over mij · Aanbod · **Afspraak** (CTA-styled) ·
  Contact. *Afspraak* takes the CTA slot (primary action); *Contact* becomes a
  normal link.
- **Hero** gains a primary CTA **"Plan een afspraak"** → `/afspraak/`, with the
  existing **"Bekijk het aanbod"** → `/aanbod/` demoted to a secondary button.
- **Aanbod intake card** button keeps its contextual label **"Plan een
  kennismaking"** but its link changes from `/contact/` → `/afspraak/`.
- **Contact page** is unchanged in purpose: it keeps the contact form. Booking
  and the form are deliberately separate pages.

---

## 2. Booking page — Calendly, click-to-load

The booking page embeds Calendly, but **loads it only on user interaction** so no
connection is made to Calendly's servers (and no cookies set) until the visitor
opts in. This keeps the privacy story consistent with the existing
privacyverklaring and keeps initial page load fast.

### Behaviour
- Page header + short intro paragraph (placeholder copy — not edited here).
- A styled **cover card** with a "Plan je gesprek" button.
- On click, a small inline script replaces the cover with the Calendly inline
  embed iframe (built from the configured URL). No Calendly script/iframe is
  present in the DOM until then.
- Wrapped so keyboard activation works (real `<button>`).

### Configuration
- `hugo.toml` `[params]` gains:
  - `calendlyUrl = ""` — the scheduling URL.
  - `hasCalendly = false` — toggle, mirroring the existing `hasPhoto` pattern.
- **Fallback:** when `hasCalendly` is false (URL not yet set), the page renders a
  friendly message linking to the contact form instead of the cover card. The
  site therefore builds and reads correctly before the real Calendly link
  exists.

### Isolation
- All booking markup lives in a `booking.html` partial (parallel to
  `contact-form.html`), rendered by an `afspraak` layout. Swapping providers
  later touches one partial.

---

## 3. Testimonials

Client quotes, managed as data so non-technical edits touch one tidy file.

- **Data:** `data/testimonials.yaml` — a list of entries:
  ```yaml
  - quote: "..."
    name: "Voornaam"
    context: "optioneel — bv. 'na een leefstijltraject'"
  ```
- **Component:** a `testimonials.html` partial reading `site.Data.testimonials`.
  If the file is missing or empty, the partial renders **nothing** — safe to ship
  before real quotes exist.
- **Placement:** a calm section on the **Home** page, below the pillars. Quote
  styling with a subtle fern accent; no heavy card chrome.
- **Privacy note:** only first names / non-identifying context, consistent with
  the privacyverklaring.

---

## 4. FAQ

- **Data:** `data/faq.yaml` — a list of `{ question, answer }`. `answer` is run
  through `markdownify` so links / emphasis are allowed.
- **Page:** new content page at slug **`/veelgestelde-vragen/`**, using a `faq`
  layout.
- **UI:** accordion built on native `<details>/<summary>` elements — accessible
  and keyboard-friendly with **no JavaScript required**. Styled to match the
  calm, rounded aesthetic.
- **Discoverability:** linked from the **footer**. Not added to the top nav, to
  keep navigation uncluttered.
- Renders nothing / an empty state gracefully if `faq.yaml` is empty.

---

## 5. Polish — self-hosted fonts

Stop loading fonts from Google's CDN (removes 3 external requests and the
visitor-IP-to-Google GDPR concern).

- Add the needed `woff2` files for **Cormorant Garamond** (300/400/600 + italic
  300/400) and **Lato** (300/400/700) under `assets/fonts/`.
- Define `@font-face` rules in `main.css` with `font-display: swap`.
- Remove the Google Fonts `<link>` and the two `preconnect` hints from
  `baseof.html`.
- Keep the existing CSS font-family declarations unchanged (same family names).

---

## 6. Polish — subtle motion

- A tiny `IntersectionObserver` script: elements marked `data-reveal` get a class
  when they scroll into view; CSS transitions opacity + a small `translateY`.
- **Reduced motion:** the whole effect is gated behind
  `prefers-reduced-motion: no-preference`. Visitors who opt out of motion see all
  content immediately, static — never hidden.
- Applied to: intro, pillars, option/program cards, testimonials, FAQ.
- Hover transitions on cards/buttons stay subtle (reuse existing patterns).
- Script lives in its own small file/partial, loaded once from `baseof.html`.

---

## 7. Polish — optimized images

Build a Hugo image-processing pipeline for the Over mij photo so it "just works"
when the real photo arrives.

- Photo source moves from `static/images/` to **`assets/images/`** so Hugo can
  process it.
- An `image` (or inline) pipeline resizes to multiple widths (e.g. 400 / 800 /
  1200), emits **WebP**, builds a `srcset` + `sizes`, and sets `loading="lazy"`
  and `decoding="async"`.
- The `over-mij.html` `hasPhoto` branch is updated to use the processed output;
  the existing placeholder branch is unchanged.
- No code change needed when the real `lauren-photo.jpg` is dropped into
  `assets/images/`.

---

## 8. Files touched (summary)

**New**
- `content/afspraak.md`
- `content/veelgestelde-vragen.md`
- `themes/laurenhendriks/layouts/_default/afspraak.html`
- `themes/laurenhendriks/layouts/_default/faq.html`
- `themes/laurenhendriks/layouts/partials/booking.html`
- `themes/laurenhendriks/layouts/partials/testimonials.html`
- `data/testimonials.yaml`
- `data/faq.yaml`
- `assets/fonts/*` (woff2 files)
- `assets/images/` (pipeline target; real photo dropped here later)
- small reveal-motion script (partial or `assets/js/`)

**Modified**
- `hugo.toml` — `calendlyUrl`, `hasCalendly` params
- `themes/laurenhendriks/layouts/partials/nav.html` — Afspraak as CTA, Contact normal
- `themes/laurenhendriks/layouts/index.html` — dual hero CTA + testimonials section
- `themes/laurenhendriks/layouts/_default/aanbod.html` — intake button → `/afspraak/`
- `themes/laurenhendriks/layouts/_default/over-mij.html` — image pipeline
- `themes/laurenhendriks/layouts/_default/baseof.html` — drop Google Fonts, add reveal script
- `themes/laurenhendriks/layouts/partials/footer.html` — FAQ link
- `themes/laurenhendriks/assets/css/main.css` — @font-face, motion, booking/testimonials/FAQ styles

---

## 9. Out of scope (confirmed)

Blog · SEO/structured data · conversion tactics · placeholder copy edits ·
top-nav FAQ link.
