# RiadKit — Landing Page

Marketing landing page for **RiadKit**, a digital guest-experience platform for Moroccan riads.
One QR code gives guests Wi-Fi details, house rules, services, local recommendations and a way to
send requests; one dashboard gives staff a place to handle them.

> **Status:** standalone static site. It is **not** yet wired to any backend.
> The demo-request form and the login page are front-end previews only (see
> [Known gaps](#known-gaps--todo)).

---

## Contents

- [What this is](#what-this-is)
- [Run it](#run-it)
- [File map](#file-map)
- [Build pipeline](#build-pipeline)
- [Design system](#design-system)
- [Page structure](#page-structure)
- [Custom CSS components](#custom-css-components)
- [JavaScript behaviours](#javascript-behaviours)
- [Accessibility](#accessibility)
- [Assets & licensing](#assets--licensing)
- [What ships to production](#what-ships-to-production)
- [Known gaps / TODO](#known-gaps--todo)
- [Porting to Next.js](#porting-to-nextjs)

---

## What this is

A **vanilla** static site — hand-written HTML, one Tailwind-generated stylesheet, one
dependency-free JS file.

- No framework (no React/Vue/Svelte), no bundler (no Vite/webpack), no TypeScript
- `node_modules` exists **only** because Tailwind is a Node program; nothing in it reaches the browser
- Runtime third-party requests: **zero**. Fonts are self-hosted.

The pages:

| File | Purpose |
|---|---|
| `index.html` | The full landing page (1,240 lines) |
| `login.html` | Login screen — **preview only**, not connected to an account system |

## Run it

The site is plain files, so opening `index.html` directly in a browser works. A local server is
nicer because it avoids `file://` quirks:

```powershell
npx http-server -p 8080 -c-1     # not installed locally; pulls from network on first run
```

Any static server works — `python -m http.server`, VS Code Live Server, etc.

## File map

```
riadkit/
├── index.html              The landing page
├── login.html              Login (preview only)
├── css/
│   └── styles.css          BUILT — do not edit by hand
├── src/
│   ├── input.css           Source stylesheet (@tailwind directives + component classes)
│   └── fonts.css           @font-face declarations for self-hosted Manrope
├── js/
│   └── main.js             All interactions, no dependencies
├── assets/
│   ├── fonts/              Manrope woff2 — 400/500/600/700/800 × latin + latin-ext
│   └── images/             Riad photography (see licensing)
├── tailwind.config.js      Brand tokens + theme extensions
├── package.json            Only for running the Tailwind build
└── package-lock.json
```

---

## Build pipeline

`css/styles.css` is **generated**. Never edit it — changes will be overwritten.

```
src/input.css  ─┐
index.html     ─┼──►  tailwindcss CLI  ──►  css/styles.css  (≈38 KB, minified)
tailwind.config.js ─┘
```

Tailwind scans the HTML as plain text to learn which class names are actually used, then emits
only those rules. That's why the output is 38 KB instead of the whole framework.

```powershell
npm run build     # one-shot, minified
npm run watch     # rebuild on change
```

**Important:** because Tailwind generates rules from scanned content, a class used **only** in a
file missing from `content` in `tailwind.config.js` will silently never be emitted. Every page
that ships is currently listed there — keep it that way.

---

## Design system

All tokens live in `tailwind.config.js`. They are the single source of truth for colour.

### Colour

| Token | Hex | Role |
|---|---|---|
| `ivory` | `#FAF8F5` | Page background |
| `primary` | `#A63D40` | Brand terracotta — CTAs, accents, icons |
| `primary-dark` | `#8A3134` | CTA hover |
| `primary-deeper` | `#6E272A` | Gradient depth |
| `primary-soft` | `#C4696B` | Light accent on dark surfaces |
| `primary-50` / `primary-100` | `#FBF1F1` / `#F6E3E4` | Tinted fills, status chips |
| `sand` | `#E9D8C5` | Warm neutral, on-dark text |
| `sand-light` / `sand-dark` | `#F3E9DC` / `#D8C2A8` | Panels, dividers |
| `ink` | `#2B2B2B` | Primary text |
| `muted` | `#666666` | Secondary text |
| `line` | `#EEE6DD` | Hairline borders |
| `night` | `#211B1A` | Dark section background |
| `night-soft` / `night-card` | `#2C2422` / `#332A28` | Dark surfaces |

### Type

**Manrope**, self-hosted (`assets/fonts/`), weights 400–800, `latin` + `latin-ext` subsets,
`font-display: swap`. Weights 700 and 800 latin are preloaded.

### Spacing, radius, elevation

- Content container: `.shell` → `max-width: 72rem`, centred, `px-5` / `sm:px-8`
- Radii run large by intent — cards `rounded-3xl`, hero decoration `rounded-[3rem]`, phone `2.75rem`
- Shadows: `shadow-card` (subtle), `shadow-lift` (elevated hover), `shadow-phone` (mockup)

### Moroccan motif

Three hand-built SVG background patterns, inlined as data URIs in `src/input.css` so they cost no
requests: `pattern-zellige`, `pattern-zellige-light`, `pattern-arches`. Used as low-opacity
decoration behind dark sections and step cards.

---

## Page structure

`index.html`, top to bottom. Section IDs are anchor targets used by the nav and footer.

| # | Section | ID | Notes |
|---|---|---|---|
| 1 | Navbar | `site-nav` (header) | Fixed. Gains a blurred surface on scroll. Contains `mobile-menu` |
| 2 | Hero | `top` (main) | Two-column: copy + layered phone mockup with floating cards |
| 3 | Benefits | — | Four cards, staggered reveal |
| 4 | Product showcase | `platform` | Dark. Guest portal + staff dashboard mockups |
| 5 | How it works | `how-it-works` | Three numbered steps |
| 6 | For riads | `for-riads` | Photography + overlapping content panel |
| 7 | Feature bento | `features` | Asymmetric 6-column grid |
| 8 | Guest journey | `journey` | Before/after comparison |
| 9 | Interactive demo | `demo` | **Real tabs** — 4 portal screens in a phone frame |
| 10 | CTA + form | `request-demo` | Dark, photo background. Form id `demo-form` |
| 11 | FAQ | `faq` | 8-item single-open accordion |
| 12 | Footer | — | Dark. Links + photo attribution |

Nav links point at `#features`, `#how-it-works`, `#for-riads`, `#faq`.

**Sample content:** every riad reference — "Riad Dar Zaman", guest "Amira", Wi-Fi
`DarZaman_Guest`, room numbers, request texts — is fictional placeholder copy for the mockups. The
demo section is explicitly labelled as a front-end preview with no real guest data.

---

## Custom CSS components

Defined in `@layer components` in `src/input.css` using `@apply`. Treat these as the project's
component library — prefer them over re-assembling utilities inline.

| Class | What it is |
|---|---|
| `.shell` | The centred content container |
| `.eyebrow`, `.eyebrow-on-dark` | Small uppercase label with a leading hairline |
| `.btn` | Base pill button — `.btn-primary`, `.btn-secondary`, `.btn-sand`, `.btn-ghost-light` |
| `.card` | White rounded panel with hairline border |
| `.phone`, `.phone-screen`, `.phone-notch` | The device mockup used in hero and demo |
| `.reveal` / `.is-visible` | Scroll-entrance animation (delays via `--reveal-delay`) |
| `.faq-item`, `.faq-trigger`, `.faq-answer`, `.faq-icon` | Accordion. Open state = `.is-open` |
| `.nav-link`, `.is-active`, `#site-nav.nav-scrolled` | Nav styling |
| `.pattern-*` | Moroccan SVG motifs |

Utilities: `.text-balance`, `.mask-fade-b`.

`prefers-reduced-motion: reduce` disables all transitions/animations and forces `.reveal` visible.

---

## JavaScript behaviours

`js/main.js` — one IIFE, `"use strict"`, **no imports, no dependencies**. Seven sections:

| # | Behaviour | Notes |
|---|---|---|
| 1 | Navbar | Adds `.nav-scrolled` past 24px. Active link via `IntersectionObserver` (`rootMargin: -40% 0px -55% 0px`) |
| 2 | Mobile menu | Toggles `aria-expanded` + `.is-open` + `body.menu-open`. Closes on link click, Escape, or resize ≥1024 |
| 3 | Reveal on scroll | Adds `.is-visible`, then unobserves. Bails to "all visible" when motion is reduced or IO is missing |
| 4 | FAQ | Single-open accordion. Opening one closes any other open item |
| 5 | Demo tabs | Full ARIA tab pattern with roving `tabindex`. Arrow keys, Home/End |
| 6 | Demo form | Client-side validation only. No backend — see [Known gaps](#known-gaps--todo) |
| 7 | Footer year | Fills `[data-year]` |

Progressive enhancement is **intended** but currently incomplete — see Known gaps.

---

## Accessibility

Things already done well; keep them if you edit:

- Tabs use a proper `tablist` / `tab` / `tabpanel` pattern with `aria-controls`, `aria-selected`
  and roving `tabindex`
- FAQ triggers are real `<button>`s with `aria-expanded` + `aria-controls`
- Menu button toggles `aria-expanded`, `aria-controls`, and swaps its `aria-label`
- Form errors use `role="alert"` targets (`data-error-for="<name>"`); status uses `role="status"`
  + `aria-live="polite"`
- Global `:focus-visible` ring, with a `.dark-surface` variant that recolours it on dark sections
- All `<svg>` decoration is `aria-hidden="true"`; decorative images use `alt=""`
- All images carry `width`/`height` (no layout shift) and `alt`. All six are
  `loading="lazy"` — including the hero phone-mockup image, which sits above the fold and so
  doesn't need it (minor nit, harmless)
- `prefers-reduced-motion` respected throughout

Known issues are listed below.

---

## Assets & licensing

> ⚠️ **Attribution removed at the owner's request.** The photography is from Wikimedia Commons
> under **CC BY-SA**, which requires crediting the authors. `credits.html` and the footer link
> were deleted. If these photos stay on a public site, attribution must live somewhere — a
> one-line footer credit is the minimum. Alternatively swap in CC0/Unsplash images and this
> concern disappears.

| File | Used in | Size |
|---|---|---|
| `riad-courtyard.jpg` | For-riads section, OG image | 343 KB |
| `riad-night.jpg` | For-riads inset | 350 KB |
| `table-detail.jpg` | Hero card, feature bento, demo panel (×3) | 239 KB |
| `riad-arch-detail.jpg` | CTA section background | 96 KB |
| ~~`riad-pool-night.jpg`~~ | **unused** | 308 KB |
| ~~`riad-interior.jpg`~~ | **unused** | 305 KB |

Fonts: Manrope, SIL Open Font License, self-hosted.

---

## What ships to production

Deploy only these — everything else is build tooling:

```
index.html
login.html
css/
js/
assets/
```

**Do not ship:** `node_modules/`, `src/`, `package.json`, `package-lock.json`, `tailwind.config.js`.
The site never touches them at runtime.

---

## Known gaps / TODO

Open items, in rough priority order.

1. **`login.html` is a dead end** — it tells visitors sign-in "isn't enabled in this preview". Once
   the app exists, point this at the real login URL.
2. **Demo form has no backend** — the success message now says honestly that nothing was sent.
   Wire it to an endpoint (needs a Laravel endpoint) and restore real confirmation copy.
3. **`og:image` domain is a placeholder** — set to `https://riadkit.app/...` in `index.html`.
   Replace with the real production URL before launch (scrapers need an absolute URL).
4. **Two unused images** — `riad-pool-night.jpg`, `riad-interior.jpg` (~613 KB). Delete, or use them.
5. **Photo attribution was removed** — see [Assets & licensing](#assets--licensing). The CC BY-SA
   licence still requires it; revisit before going public.

### Fixed (keep the pattern)

- `credits.html` was removed entirely (it was also missing from Tailwind's `content`, which left
  it partially unstyled — moot now).
- The corrupted CJK glyph in the services demo ("Home-cooked dinner" icon) was replaced with a
  food emoji, matching its `☀ ✽ ✈` siblings.
- Reveal animation is gated behind an `html.js` class set by an inline head script — with JS off,
  all content stays visible and the mobile menu renders as plain links (the dead toggle button is
  hidden via `html:not(.js)`).
- Inline SVG favicon on `index.html` + `login.html` (no more `/favicon.ico` 404).
- `og:image` / `twitter:image` are absolute URLs (placeholder domain — see item 3 above).
- `#demo-form` has `scroll-mt-28` like every other anchor target.
- Footer copyright text raised from `text-white/45` (~4.3:1) to `text-white/60` (~5.5:1) for AA.
- `whitespace-nowrap` removed from the hero highlight so it can wrap at 320–360px.
- Collapsed FAQ answers now use `visibility: hidden` (delayed to the end of the collapse
  transition), so they leave the accessibility tree and tab order — `grid-template-rows: 0fr`
  alone only hid them visually.
- Hero mockup image is no longer `loading="lazy"` (it's above the fold).
- Dead config removed: `riseIn` keyframes, `.mask-fade-b`, `.scroll-x-none`.
- `http-server` is a real devDependency; `npm run serve` works offline.

---

## Porting to Next.js

Deferred by choice — finish and iterate on the page here first. When it's time:

### Mechanical HTML → JSX conversion

- `class=` → `className=`
- `for=` → `htmlFor=` (the form labels)
- Self-close void elements: `<img>`, `<input>`
- `style="--reveal-delay: 350ms"` → `style={{ "--reveal-delay": "350ms" }}`
- `<!-- comment -->` → `{/* comment */}`
- `<head>` meta/OG/Twitter tags → the Metadata API or `next/head`
- The JSON-LD block → `dangerouslySetInnerHTML`

### Keep it lean

This page is currently **38 KB CSS + 9.5 KB JS, zero React**. Don't lose that. Put the static
markup in a Server Component and isolate only what's interactive (`demo tabs`, `FAQ`, `form`,
`mobile menu`) as small Client Components — or keep `main.js` as a plain `<script>` and skip
React for those entirely.

### Consolidate

- Move the colour tokens into the app's shared `tailwind.config.ts` so marketing and dashboard
  read as one brand
- Route groups: `app/(marketing)/page.tsx` alongside `app/(dashboard)/…`
- Replace `login.html` with the real app login route
- `og:image` must become an absolute URL once the domain is settled
