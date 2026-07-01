# CLAUDE.md

Guidance for AI assistants (and humans) working in this repository.

## What this is

**LE PALMIER** is the marketing website for a fast-food restaurant (kebab,
tacos, pizza, shawarma) located at *10 Rue de Bayard, 31000 Toulouse, France*.
It is a **single-page, static "vitrine" (brochure) site** in French, deployed
to Netlify. There is **no backend, no database, no API** — the site is
informational only (no online ordering).

Live URL (canonical): `https://lepalmier-toulouse.netlify.app/`

## Repository layout

The tracked git tree is currently minimal:

```
.
├── README.md                        # placeholder only
├── CLAUDE.md                        # this file
└── lepalmier-deployendiojvn.zip     # the actual deployable site (see below)
```

⚠️ **Important — the real site lives inside the zip.** The entire deployable
site is packaged in `lepalmier-deployendiojvn.zip`, **not** extracted into the
working tree. Inside the archive, everything sits under a `lepalmier-deploy/`
directory:

```
lepalmier-deploy/
├── index.html          # the whole site — HTML + inline CSS + inline JS (~150 KB, ~327 lines)
├── netlify.toml        # security headers + caching rules (belongs at deploy root)
├── robots.txt          # allows all crawlers, points to sitemap
├── sitemap.xml         # single URL (the homepage)
├── og-image.jpg        # Open Graph / social share image (1200×630)
└── images/             # 43 hashed .webp photos (img-01-*.webp … img-43-*.webp)
```

When deployed, the **contents of `lepalmier-deploy/` become the site root** —
i.e. `index.html`, `netlify.toml`, `images/`, etc. are served from `/`. Image
references in the HTML use absolute paths like `/images/img-03-7a437dff76.webp`.

To inspect or edit the site, extract the zip first:

```bash
unzip lepalmier-deployendiojvn.zip
# edit lepalmier-deploy/index.html ...
```

## Tech stack & conventions

- **Pure static site.** No framework, no build step, no package manager, no
  dependencies, no CI config. Everything is hand-written HTML/CSS/JS.
- **Single file.** `index.html` contains the full markup, **all CSS in inline
  `<style>` blocks**, and **all JS in inline `<script>` blocks** (several small
  vanilla-JS IIFEs). There are no external `.css` or `.js` files.
- **Vanilla JS only** — no jQuery, React, etc. Interactivity is done with plain
  DOM APIs and `IntersectionObserver`.
- **Language:** all user-facing copy, class names, comments, and IDs are in
  **French** (e.g. `conteneur`, `carte`, `apropos`, `trouver`, `reveler`,
  `charbon`, `creme`, `terracotta`). Keep new code consistent with this.
- **External resources** (allowed by the CSP): Google Fonts
  (`Anton`, `Work Sans`, `Playfair Display`; the code also references
  `Barlow`/`Barlow Condensed` via CSS vars) and an embedded Google Maps iframe.
- **Accessibility is intentional:** ARIA roles/labels, `sr-only` helpers,
  focus-visible outlines, and `prefers-reduced-motion` guards throughout.
  Preserve these when editing.

### CSS design system

Colors are defined as CSS custom properties on `:root` (note there are a few
overlapping/legacy palettes declared across the inline styles). Core tokens:

- `--charbon` `#141410` (background), `--charbon-2` `#1c1c16`
- `--terracotta` / `--or` `#f5c800` (yellow accent), `--terracotta-vif` `#ffd740`
- `--creme` `#fff8e1` (text), `--creme-doux` `#c8c480` (muted text)
- `--palme` `#4caf50` (green), `--ciel` `#5bc8f5` (blue accent)

Reusable class prefixes: `lph-` (header/nav), `lp-hero-` (hero), `mp-` (menu
section), `gal-` (gallery carousel), `tca-` (reviews/testimonials carousel),
`ml-` (mentions légales overlay), `palm-`/`hero-palmier` (animated palm SVGs),
`reveler` (scroll-reveal animation).

## Page structure (`index.html`)

Fixed header (`#lph`) with anchor nav + mobile drawer, then these `<section>`s:

| ID            | Purpose                                                          |
|---------------|-----------------------------------------------------------------|
| (hero)        | Full-viewport hero image + brand title                          |
| `#sec-carte`  | The menu — tab UI (Burgers / Pizzas / Paninis / Tex-Mex / Assiettes) + a "La Carte" modal showing a full menu photo |
| `#sec-maison` | "About the house" story section                                 |
| `#sec-galerie`| Photo carousel of the premises (auto-advancing, cloned slides, swipe/dots) |
| `#sec-avis`   | Google reviews carousel (schema.org `LocalBusiness`, ★4,2 / 221 avis) |
| `#sec-acces`  | Contact info (address, phone, hours) + embedded Google Map      |
| footer        | NAP details + link that opens the **Mentions légales** overlay  |

The **Mentions légales** (legal notice) is a full-screen overlay (`#ml-overlay`)
toggled by JS, not a separate page. `sitemap.xml` contains a commented-out note
about splitting it onto its own `/mentions-legales` page in the future.

### JS behaviors (all inline IIFEs at the bottom of the file)

- Header scroll state + active-section highlighting; mobile drawer open/close.
- Menu tab switching + staggered card reveal animation.
- Gallery carousel (infinite loop via clones, autoplay, arrows, dots, touch swipe).
- Generic `.reveler` scroll-reveal via `IntersectionObserver`.
- "La Carte" image modal open/close (Escape-aware).
- Mentions légales overlay open/close (Escape-aware, scroll-lock).

## SEO / metadata

- Extensive `<meta>` tags: description, canonical, Open Graph, Twitter cards.
- JSON-LD `Restaurant` structured data in `<head>` (cuisine, address, phone,
  hours 11:00–00:00 daily, aggregateRating 4.2 / 221 reviews).
- `robots.txt` + `sitemap.xml` reference `lepalmier-toulouse.netlify.app`.

## Deployment (Netlify)

`netlify.toml` (must live at the deploy root) configures:

- **Security headers** on `/*`: `X-Content-Type-Options`, `Referrer-Policy`,
  `X-Frame-Options`, `Permissions-Policy`, and a **Content-Security-Policy**
  that whitelists Google Fonts and the Google Maps iframe. If you add a new
  third party (analytics, another embed, etc.), you **must** update the CSP or
  the resource will be blocked — verify in the browser console after deploy.
- **Caching**: `/images/*` is cached `immutable` for 1 year (safe because
  filenames are content-hashed); `robots.txt` and `sitemap.xml` cache 1 hour.

There is no build command — Netlify just serves the static files.

## Working on this repo

- **Development branch:** commit work to `claude/claude-md-docs-rbcjdu` (create
  it from the latest default branch if needed). Do not push to `main` without
  explicit permission. Push with `git push -u origin <branch>`.
- **Editing the site:** extract the zip, edit `lepalmier-deploy/index.html` (and
  assets), then re-package if the zip is meant to remain the source of truth.
  Confirm the intended workflow with the maintainer before restructuring the
  repo (e.g. extracting the site permanently into the tree).
- **Testing:** it's a static file — open `lepalmier-deploy/index.html` directly,
  or serve locally (`python3 -m http.server` from inside `lepalmier-deploy/`) so
  the absolute `/images/...` paths resolve. There is no test suite or linter.
- **When adding images:** use content-hashed `.webp` filenames under `images/`,
  set explicit `width`/`height`, add `loading="lazy"`, and write descriptive
  French `alt` text (matching the existing SEO-oriented alt style).

## Known inconsistencies (verify before relying on them)

These conflicting details exist in the current source — flag or fix them if a
task touches the relevant area, but confirm the correct value with the
maintainer first:

- **Hosting:** `netlify.toml` and the canonical/OG URLs say Netlify, but the
  Mentions légales "Hébergeur" line names **Vercel Inc.** and references the
  domain **`www.lepalmier-toulouse.fr`** (not the `.netlify.app` URL used
  everywhere else).
- **Postal code:** the site and JSON-LD use **31000**, while the legal
  owner-card uses **31100**.
