---
name: website-to-deck
description: >
  Capture a website and turn it into a professional, brand-matched HTML slide
  deck (no video). Use when asked to "make a deck/presentation from this URL",
  "turn this page into slides", "build an HTML presentation from a site", or to
  fix/re-host an existing capture-based deck. Covers capture, brand-token
  extraction, slide authoring, partner-logo recovery, and GitHub Pages hosting.
  For video output (MP4, narration, TTS timing) use website-to-hyperframes
  instead.
---

# Website to HTML Deck

Turn a captured website into a static, keyboard-navigable 16:9 HTML deck that
inherits the site's brand — then optionally host it on GitHub Pages.

**No video, no narration, no timing.** If the user wants an MP4 or narration,
stop and use `website-to-hyperframes`.

## Workflow (4 steps)

### Step 1 — Capture

```bash
npx hyperframes capture <URL> -o <project-dir>/capture
```

Keep artifacts in `capture/`; the deck lives at `<project-dir>/deck.html`.
Note the counts printed (screenshots, assets, sections, fonts). If many assets
are dropped (`size-floor`, `cap-reached`), the capture is truncated — expect to
fetch some assets manually later (see partner-logo-recovery.md).

**Read:** [references/capture-and-tokens.md](references/capture-and-tokens.md)

Then read, in this order:
1. `capture/screenshots/scroll-000.png` (hero — light/dark, dominant visual)
2. `capture/extracted/tokens.json` (colors, fonts, headings, sections)
3. `capture/extracted/visible-text.txt` (tag-prefixed copy — source of truth for slide text)
4. `capture/extracted/asset-descriptions.md` (what images exist; check `svgs/logo-*.svg` for the brand logo — pick the largest, verify with an image read)

### Step 2 — Plan the deck (no file needed)

Map the page's sections to a slide inventory. Default 6–9 slides:

1. **Title** — hero image + headline + badges
2. **Overview** — pitch + 3 stat cards + photo
3. **Key points** — page's N advantages/features as a card grid
4. **Use case / business case** — photo + numbered step timeline (if present)
5. **Partners / logos** — real logo images, never text placeholders (see partner-logo-recovery.md)
6. **Related products** — 3 cards
7. **FAQ** — `<details>` accordions (if present)
8. **Contact / CTA** — dark closer with real email/address from page footer

Rule: quote the page copy (strip `[tag]` prefixes), don't invent claims. Trim
nav/footer boilerplate from `visible-text.txt` first.

### Step 3 — Build `deck.html`

**Read:** [references/deck-template.md](references/deck-template.md)

Single self-contained file. Must have: 1280×720 stage that scales to viewport,
slide navigation (←/→/Space/Home/End, `F` fullscreen, arrows + progress bar),
staggered entrance animations re-triggered per slide (`.up` + `.d1`–`.d6`).
Reference captured assets via **relative paths** into `capture/assets/` so the
directory ships as-is. After building: verify every referenced path exists
(`for f in <paths>; do [ -f "$f" ] || echo MISSING $f; done`), then
`open deck.html`.

### Step 4 — Host (only if asked)

**Read:** [references/hosting.md](references/hosting.md)

`git init` → commit deck.html + capture/assets → `gh repo create <name>
--public --source . --push` → enable Pages via API → give the user the
`username.github.io/<repo>/deck.html` URL. Warn about the CDN cache delay.

## Gotchas learned in practice

- **Logo SVGs:** candidates are named `logo-<hash>.svg`; the real brand mark is
  usually the largest and has no `fill` attributes (recolorable via CSS filter).
- **Partner logos are often dropped by the capture** — recover them from the
  source page HTML, never fake them with styled text (see
  partner-logo-recovery.md).
- **As-is logos on dark slides:** transparent PNGs need white cards + dark
  captions, or they vanish.
- **GitHub Pages serves stale HTML** for minutes after build — check the build
  API before assuming the push failed; hard-refresh / `?v=` busts cache.
- **`#F7F7F7`-style light body + dark hero** is a common enterprise pattern;
  keep the deck's section rhythm matching the site's.
- **Double-check every outbound link against the captured DOM** — don't guess
  hrefs from URL patterns. `rg -o 'href="[^"]*<slug>[^"]*"' capture/extracted/page.html |
  sort -u` gives the real paths (Nuxt sites often nest one level deeper than
  guessed, e.g. `.../ems/solution/emsconnect`).
- **Look ahead for content:** when a slide feels thin, fetch the page a CTA /
  "Learn More" links to and pull its intro bullets into the card as a preview
  (2×2 mini-tiles). Real downstream copy beats padding or invented text.
- **Fill white space, don't leave it:** two proven fills — (1) spread existing
  text out (bigger line-height, `justify-content:space-evenly`/`center` +
  `flex:1` so content distributes across card height), or (2) add a relevant
  oversized watermark SVG (gradient-stroked, bottom-right, clipped by the card)
  per card — one icon that matches that card's topic (see deck-template.md).
- **Watermark SVGs overwhelm small cards** (tiles/cards < ~180px tall): they
  overlap text, and pushing them off-corner clips them into unreadable
  fragments. In small cards: 40–60px, fully inside the card (no negative
  offsets), opacity ~0.45, plus `min-height` on the card and `padding-right`
  on body text for clearance. Icon-panel pattern (icon zone + caption bar as
  flex rows, never absolute-positioned over each other) can't collide.
- **Icon-card PNGs are not photos:** assets from `model-card-icons/` style
  paths (e.g. `seamless-connectivity-across-*.png`) are huge flat glyphs —
  `object-fit:cover` turns them into a blown-up icon. For panels/hero cards use
  a brand-gradient background + inline stroke SVG instead.
- **Hero CTAs may be `<button>`s with no href** (Nuxt scroll/modal triggers).
  Wire them to verified alternatives instead of guessing: the page's
  `/contact-us` link, or the `mailto:` used by "Email Us" buttons on captured
  sub-pages (`rg -o 'mailto:[^"]*' page.html`).
- **Look-ahead fetches:** only run a full `hyperframes capture` (~2 min each)
  for pages whose *photo/screenshot assets* you need; for text-only extra
  slides, `webfetch` the URL (~10s). Run capture in the background (`nohup … &`)
  while webfetching the remaining look-ahead pages in parallel.

## Gate

- Every asset path in deck.html resolves locally.
- Slide text quotes the captured page, not invented copy.
- Every outbound href matches the captured DOM (grep page.html, don't guess).
- If hosted: `gh api repos/<o>/<r>/pages/builds/latest --jq .status` returns `built`.
