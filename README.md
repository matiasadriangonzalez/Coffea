# Coffea landing

Static one-page landing for the fictional coffee shop **Coffea** (UADER, UX-UI 2026, TP4). Nine sections, responsive from 360 px to 1920 px, designed to be deployed on Netlify.

> **Status:** the site has **not** been deployed yet. See [Deploy to Netlify](#deploy-to-netlify).

## Stack and constraints

- HTML5 + CSS3 only. **No JavaScript**, frameworks, preprocessors or build step.
- All interactions are CSS-only: hover/focus states, scroll-snap carousels, favourite hearts (checkbox), mobile menu (`<details>`), testimonial pager (anchors).
- Accessibility target: WCAG 2.2 AA (skip link, landmarks, keyboard navigation, visible focus, reduced motion).
- Language: English (`lang="en"`).

## Project structure

```
.
├── index.html            Landing page (inline SVG icon sprite)
├── 404.html              Error page (same header and footer as the landing)
├── netlify.toml          Publish directory and headers
├── css/
│   ├── main.css          Dev-only entry: @import of every layer, in order
│   ├── tokens.css        Layer order, @font-face, design tokens (custom properties)
│   ├── base.css          Reset, typography, focus, reduced motion
│   ├── layout.css        .container
│   ├── components/       header, hero, categories, product-card, carousel,
│   │                     promo, testimonial-card, newsletter, footer, not-found
│   └── utilities.css     .visually-hidden, .skip-link, helpers
├── assets/
│   ├── img/              AVIF + WebP + JPG/PNG fallbacks
│   ├── icons/            Optimised SVG sources (the pages use the inline sprite)
│   └── fonts/            Self-hosted woff2 (latin subset, SIL OFL 1.1)
└── media/                LOCAL ONLY, git-ignored: original photos and exports
```

## CSS architecture

Cascade order is declared once, at the top of `css/tokens.css`:

```css
@layer tokens, base, layout, components, utilities;
```

Every other file wraps its rules in its own layer, so specificity never needs `!important` (the only exception is the `prefers-reduced-motion` block in `base.css`).

| Rule | Detail |
| --- | --- |
| Tokens | Every colour, font family, radius, spacing and size lives in `css/tokens.css` and nowhere else. |
| Naming | BEM: `.product-card`, `.product-card__title`, `.product-card--featured`. |
| Mobile-first | Base styles target 360 px; `min-width` queries go up. |
| Breakpoints | 768 px (`48em`), 1024 px (`64em`), 1440 px (`90em`). |
| Units | `rem` for type and spacing, `clamp()` for fluid headings. |
| Loading | One `<link>` per file, in layer order (no `@import` chain in production). |

## Run locally

Open `index.html` directly in a browser (the icon sprite is inline, so it works over `file://`). Note that `404.html` uses root-absolute URLs on purpose (Netlify serves it for any missing path), so view it through a server:

```sh
python3 -m http.server 8000     # then open http://localhost:8000 and /404.html
```

## Validate

W3C Nu HTML checker plus its CSS checker (needs Java 11+; on macOS with Homebrew `openjdk`):

```sh
PATH="$(brew --prefix openjdk)/bin:$PATH" \
  npx --yes vnu-jar --also-check-css index.html 404.html css/*.css css/components/*.css
```

No output and exit code 0 means no errors and no warnings.

## Lighthouse (local)

Serve the site and audit it with headless Chrome:

```sh
python3 -m http.server 8000 &
CHROME_PATH="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  npx --yes lighthouse http://127.0.0.1:8000/index.html \
  --chrome-flags="--headless=new" --view                 # mobile (default)
# add --preset=desktop for the desktop profile
```

`python3 -m http.server` speaks HTTP/1.1 without compression or cache headers, so it scores lower than a CDN. The acceptance numbers must be measured on the production URL after deploying (PRD RNF-D7).

## Assets pipeline

- Originals live in `media/` (local only, git-ignored; never commit them or the PDF brief).
- Deliverables are generated into `assets/img/`: AVIF and WebP plus a JPG (photos) or PNG (transparent art) fallback, served through `<picture>` with explicit `width`/`height`. The hero ships in 768, 1280 and 1920 px widths.
- Icons are optimised with SVGO; `index.html` and `404.html` embed them as an inline `<symbol>` sprite and reference them with `<use>`.
- Fonts (Outfit, Playfair Display, Poppins, Reggae One) are self-hosted as latin woff2 in `assets/fonts/`, declared in `css/tokens.css` with `font-display: swap`. Outfit, Playfair Display and Poppins 500 are preloaded.

## Deploy to Netlify

**Not done yet.** Instructions to follow when you are ready:

1. Push the repository to GitHub or GitLab.
2. In Netlify: **Add new site > Import an existing project**, pick the repository.
3. Build settings: **no build command**, **publish directory `.`** (both already set in `netlify.toml`).
4. Production branch: `main`. Every push to it publishes automatically.
5. Enable **Deploy Previews** (default): each pull/merge request gets its own preview URL for review.
6. HTTPS is automatic (Netlify-managed certificate, HTTP is redirected to HTTPS). Use the `*.netlify.app` URL or add a custom domain.
7. Netlify serves `404.html` for missing paths automatically.
8. Measure Lighthouse on the production URL (PRD thresholds: Performance >= 90 mobile, Accessibility / Best Practices / SEO >= 95).

### Immutable cache warning

`netlify.toml` serves `/css/*` and `/assets/*` with `Cache-Control: public, max-age=31536000, immutable`. Browsers will never re-request a file with the same URL for a year, so a changed file must be **renamed or versioned** (for example `main.v2.css`, `hero-1920.v2.avif`) and the HTML updated.
Tradeoff: that is the best repeat-visit performance, but forgetting to rename ships stale styles; if that is too fragile, relax the header (for example `max-age=3600, must-revalidate`) at the cost of more revalidation requests.

## Browser support

Target: Chrome / Edge, Firefox (latest 2), Safari macOS / iOS 16+, Samsung Internet (latest 2).

| Browser | Status |
| --- | --- |
| Chrome (desktop) | Checked during development |
| Firefox, Safari macOS, Safari iOS, Samsung Internet | **Pending: needs a manual check** |
| Real iOS and Android devices | **Pending: needs a manual check** |

## Open decisions / known limitations

- **Photo licence:** the stock photos (istockphoto) come from a Figma Community template; confirm the licence or replace them before publishing.
- **Currency:** prices read "Rs. 200" as in the design; decide whether to switch to `$` or another currency.
- **Placeholder copy:** hero paragraph, product name/description/price ("Lungo coffee", "Rs. 200", reused for desserts), testimonials ("James Smith" and lorem ipsum, one avatar reused), newsletter title ("Subscribe to our newsletter"), and footer link texts other than "Explore Our Products", "Our story", "Plans & pricing" and "Sell your products". Nav and footer links are `#` placeholders.
- **Dessert photos** are coffee photos (as in the Figma).
- **Contrast:** the filled star (`#FFB921`) on the sand card is about 1.4:1 and filled vs empty stars about 1.1:1 (the design's own colour; the rating is also exposed through `aria-label`). The borderless newsletter input against the sand band is about 1.2:1 (below 3:1 for non-text UI). Decide whether to darken them.
- **Carousel arrows** are visual-only from 1280 px, where all four cards fit (add a fifth card or hide them).
- **Mobile menu** does not close when a link is clicked (that needs JavaScript); the page links are placeholders anyway.
- **Testimonial pager** dots show no active state (needs JavaScript or scroll-driven animations).
- **Ratings** (4/5/3 stars) were assigned arbitrarily; the Figma did not make the mapping clear.
- **Newsletter form** posts to `action="#"` and validates natively (`:user-invalid`); Netlify Forms (`data-netlify="true"`) is not used. Error state is colour-only.
- **`og:image`** is missing until the final production URL is known (it needs an absolute URL).
- **Hero headline** is about 51 px at 768 px wide (PRD suggests about 56 px).
