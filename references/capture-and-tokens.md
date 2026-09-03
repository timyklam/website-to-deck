# Step 1 — Reading the Capture

## What to read, in order

### 1. Hero screenshot `capture/screenshots/scroll-000.png`

The most important image. Determine:
- Light or dark hero? (Enterprise sites often: dark hero + light `#F7F7F7` body)
- Dominant visual (photo? abstract background? product shot?)
- Which 2–3 accent colors jump out — cross-check against tokens.json

Then skim 2–3 more screenshots (they overlap ~30%) for section rhythm.

### 2. `capture/extracted/tokens.json`

Extract and note:
- **Colors** — the `colors` array is frequency-ordered. Typical mapping:
  - darkest hex = text/hero bg, mid blues = accents, `#F7F7F7`/`#FFFFFF` = surfaces
  - `colorStats` tells you role: high `textCount` = text color, high `areaBg` = background
- **Fonts** — families + weights (e.g. `Roboto (400,500,600,700)`). You'll declare
  them in deck.html and may load a woff2 from `capture/assets/fonts/` (verify the
  file exists; use a `latin` subset for Latin decks).
- **Headings** — `fontSize`/`fontWeight`/`color` of real headings; the deck's type
  scale should echo them (deck h1 ≈ 48–66px, h2 ≈ 42px at 1280×720).
- **Sections** — each has `heading`, `text`, `layout`, `assetUrls`. This is your
  slide inventory: hero → title slide; grid sections → card slides; etc.

### 3. `capture/extracted/visible-text.txt`

Lines like `[h1] Heading`, `[p] Body`, `[a] Link`, `[span] Item`.
- Headings = key messages; paragraphs = supporting copy; `li`/`strong` pairs =
  structured content (e.g. Challenge/Solution/Outcome lists).
- **Strip the `[tag]` prefix** when quoting into slides. Light copyediting is fine;
  inventing claims is not.
- Skip nav menus, portal lists, footer link farms (usually the first ~150 and last
  ~40 lines).

### 4. `capture/extracted/asset-descriptions.md`

- One line per downloaded asset with size.
- `logo-<hash>.svg` entries: the **brand logo** is usually the largest file and has
  `viewBox` around `0 0 1000 185`-scale. Confirm by reading the SVG (no `fill`
  attributes → recolorable with CSS `filter: brightness(0) invert(1)` on white).
- Icon-only PNGs (e.g. `expert-led-*.png`) are small card icons — usable but often
  better replaced by inline SVG icons matching the deck's stroke style.
- **Missing assets:** if `assetUrls` in tokens.json references URLs not present in
  `assets/`, the capture dropped them (size-floor / cap-reached). Follow
  partner-logo-recovery.md to fetch from the source.

## Brand summary to carry into Step 3 (state it explicitly)

Before writing any HTML, print:

- **Site:** name
- **Colors:** top 3–5 HEX with roles (hero bg / accent / surface / text)
- **Fonts:** families + weights
- **Sections:** count + slide mapping
- **Key assets:** logo file, hero image, 1–2 section photos
- **Vibe:** one sentence
