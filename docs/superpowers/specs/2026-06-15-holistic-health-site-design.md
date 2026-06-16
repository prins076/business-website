---
name: holistic-health-site-design
description: Design spec for Lauren Hendriks' orthomolecular therapist website — laurenhendriks.nl
metadata:
  type: project
---

# Lauren Hendriks — Website Design Spec

**Domain:** laurenhendriks.nl  
**Language:** Dutch  
**Hosting:** GitHub Pages with custom domain  
**Hugo theme:** Custom (`themes/laurenhendriks/`)  
**Date:** 2026-06-15

---

## 1. Architecture

Custom Hugo theme built from scratch. The ananke submodule is removed (clutter).

### File structure

```
my-business-website/
├── hugo.toml
├── content/
│   ├── _index.md         ← Home
│   ├── over-mij.md       ← About
│   ├── aanbod.md         ← Services
│   └── contact.md        ← Contact
└── themes/laurenhendriks/
    ├── layouts/
    │   ├── _default/baseof.html      ← shared shell (nav + footer)
    │   ├── index.html                ← Home layout
    │   ├── page/single.html          ← layout for all other pages
    │   └── partials/
    │       ├── header.html
    │       ├── nav.html
    │       ├── footer.html
    │       └── contact-form.html     ← isolated for easy Calendly swap
    ├── assets/css/main.css
    └── static/
        └── images/
            └── lauren-photo.jpg      ← drop real photo here when ready
```

---

## 2. Pages & Content

### Home (`_index.md`)
- **Hero:** Full-width, deep green background, white fern SVG pattern overlay
  - Lauren's name as H1
  - Tagline: *"voeding · gezondheid · leefstijl"* (from her business cards)
  - CTA button → Aanbod
- **Intro:** 2–3 sentences about her approach, warm and personal
- **Pillars row:** 3 short anchors, e.g. *Persoonlijk / Natuurlijk / Wetenschappelijk*

### Over mij (`over-mij.md`)
- Photo (placeholder; real photo dropped into `static/images/lauren-photo.jpg`)
- Bio: personal story, background, philosophy
- Credentials / opleiding section

### Aanbod (`aanbod.md`)
Structured around Lauren's actual client process — not a sales funnel.

1. **Intake / Kennismakingsgesprek** — prominently first; every client journey starts here
2. **Wat kan er volgen?** — depending on the intake, Lauren advises on:
   - Voeding & supplementen
   - Leefstijlveranderingen
   - Een begeleidingstraject (one of her programs)
3. **Trajecten** — 2–3 programs presented as options that *might* suit you, not products to push
   - Framed as *"Voor wie een dieper traject past"*
   - No fixed duration mentioned — can be weeks or months depending on the client
   - Tone throughout: warm, informative, non-pressuring

### Contact (`contact.md`)
- Short intro: *"Neem contact op"*
- Form fields: naam, e-mailadres, bericht, verzenden
- Backend: Formspree (free tier) — endpoint URL configured in `hugo.toml`
- Isolated in `contact-form.html` partial → swap for Calendly embed in one line when ready

---

## 3. Visual Design

### Color palette

| Role       | Hex       | Usage                                      |
|------------|-----------|--------------------------------------------|
| Primary    | `#2D4A3E` | Nav, hero background, buttons, headings    |
| Secondary  | `#6B8F71` | Accents, hover states, dividers            |
| Background | `#F5F0E8` | Page backgrounds                           |
| Surface    | `#E8DFD0` | Cards, alternating sections                |
| Text       | `#2C2C2C` | Body copy                                  |
| Light      | `#FAFAF7` | Contrasting sections                       |

### Typography
- **Headings:** Cormorant Garamond (Google Fonts) — elegant serif, botanical feel
- **Body:** Lato (Google Fonts) — clean, readable sans-serif

### Fern motifs
- White fern SVG as subtle repeating pattern in the hero background
- Small decorative fern accent used as section dividers or beside headings
- Tasteful — texture, not wallpaper

### Layout
- Generous whitespace and padding
- Soft rounded corners on cards
- No harsh lines — calm and grounded feel

---

## 4. Deployment

### GitHub Pages + Custom Domain
- `baseURL` in `hugo.toml` set to `https://laurenhendriks.nl/`
- GitHub Actions workflow: on push to `main` → `hugo build` → deploy to GitHub Pages
- Custom domain configured in GitHub Pages settings
- DNS at registrar pointed to GitHub's servers (A records + CNAME)

### Contact Form (Formspree)
1. Register free account at formspree.io
2. Create a form, copy the endpoint URL
3. Paste endpoint into `hugo.toml` as `[params] formspreeEndpoint = "..."`
4. Emails arrive in Lauren's inbox

### Photo placeholder
Drop `lauren-photo.jpg` into `themes/laurenhendriks/static/images/` — it will appear on the Over mij page automatically.

---

## 5. Future considerations

- **Calendly integration:** Replace `contact-form.html` partial with a Calendly embed snippet — one file change, no other impact
- **Blog:** Not in scope; can be added later as a new content type if needed
- **i18n:** Not in scope; site is Dutch-only
