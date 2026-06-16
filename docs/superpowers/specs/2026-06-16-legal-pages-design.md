# Legal / policy pages — design

**Date:** 2026-06-16
**Status:** Approved, ready to implement

## Goal

Add the legal/policy pages the site currently lacks: a privacy statement, terms &
conditions, and a medical disclaimer — in Dutch, matched to how the site actually
works, and linked from the footer.

## Context

- Static Hugo site, hosted on **GitHub Pages** (US).
- Only personal data the site handles: the **contact form** (naam, e-mail, bericht),
  submitted directly to **Formspree** (US), which emails it to the owner.
- **No analytics, no tracking, no cookies** set by the site → no cookie banner
  required; a single "geen cookies" line in the privacy statement suffices.
- The practice is registered with a professional association and a Wkkgz
  disputes scheme — exact names/numbers to be filled in as placeholders.
- Theme uses a bespoke layout per page; there is **no generic single-page layout**.

## Scope

Three pages, Dutch:

| Page | URL |
|---|---|
| Privacyverklaring | `/privacyverklaring/` |
| Algemene voorwaarden | `/algemene-voorwaarden/` |
| Medische disclaimer | `/disclaimer/` |

Out of scope: separate cookie page (not needed), real legal review (owner's
responsibility), filling in business-specific placeholders.

## Approach

One reusable `_default/single.html` template renders a page title + markdown body in
a readable column (~720px), reusing the existing `.container` and olive accent. Each
legal doc is then a plain markdown content file. This also gives the site a reusable
template for any future simple text page.

Rejected alternatives: one combined page with anchors (can't link to a single doc,
messy); a bespoke layout per page (needless duplication for text content).

## Components

1. **`themes/laurenhendriks/layouts/_default/single.html`** — generic text-page
   layout (title, optional subtitle, markdown `.Content`).
2. **`themes/laurenhendriks/assets/css/main.css`** — add a `.legal` block: readable
   max-width, heading/paragraph/list spacing, a "laatst bijgewerkt" line. Match
   existing visual style.
3. **Content files** (`content/`):
   - `privacyverklaring.md`
   - `algemene-voorwaarden.md`
   - `disclaimer.md`
4. **`themes/laurenhendriks/layouts/partials/footer.html`** — add a footer links row
   for the three pages.

## Content outline

- **Privacyverklaring:** controller identity; data collected via contact form only;
  processors Formspree + GitHub Pages (US) with EU-transfer note; retention; no
  cookies/tracking; visitor rights (inzage, correctie, verwijdering); right to
  complain to the Autoriteit Persoonsgegevens.
- **Algemene voorwaarden:** toepasselijkheid, afspraken & annulering, betaling,
  aansprakelijkheid, klachten via the Wkkgz-geschilleninstantie. Placeholders for
  annuleringstermijn and tarieven.
- **Medische disclaimer:** orthomolecular therapy is complementary, not a replacement
  for medical care/diagnosis; consult a GP for medical complaints; references the
  beroepsvereniging.

## Placeholders (owner to complete before publishing)

`[volledige (bedrijfs)naam]`, `[KvK-nummer]`, `[contact-e-mailadres]`, `[adres]`,
`[beroepsvereniging]`, `[geschilleninstantie + nummer]`, `[annuleringstermijn]`,
`[tarieven / betaaltermijn]`. All listed at the bottom of the work as a checklist.

## Caveat

Drafted in good faith to be accurate to the site's actual data handling, but it is not
legal advice. The owner should review the text — especially the Algemene voorwaarden.
