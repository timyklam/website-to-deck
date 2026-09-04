# Step 3 — deck.html Template Conventions

Single file, no build step, relative asset paths (`capture/assets/...`) so the
project directory is portable and repo-ready.

## Architecture

```
<div id="progress">            top progress bar (gradient, width = i/n)
<button #prev / #next>         fixed circular nav buttons
<div #hud>                     "3 / 8" counter
<div id="stage">               1280x720, absolutely centered
  <section class="slide s1 dark active"> ... </section>
  ...
</div>
```

## Non-negotiables

### Scaling stage
```css
#stage{position:absolute;top:50%;left:50%;width:1280px;height:720px;
  transform:translate(-50%,-50%)}
/* JS on resize: */
const s = Math.min(innerWidth/1280, innerHeight/720) * 0.97;
stage.style.transform = `translate(-50%,-50%) scale(${s})`;
```

### Slide switching + re-triggerable entrances
- `.slide{opacity:0;visibility:hidden;transition:opacity .45s}` `.slide.active{...}`
- Entrance pattern: `.up{opacity:0;transform:translateY(26px)}`
  `.active .up{animation:up .7s forwards}` with delay classes `.d1`(0.08s) …
  `.d6`(0.68s). Because the class is tied to `.active`, animations replay every
  time a slide is revisited — this is the point.

### Keyboard + controls
```js
addEventListener('keydown', e => {
  if(['ArrowRight','PageDown',' '].includes(e.key)) show(i+1);
  else if(['ArrowLeft','PageUp'].includes(e.key)) show(i-1);
  else if(e.key==='Home') show(0);
  else if(e.key==='End') show(slides.length-1);
  else if(e.key.toLowerCase()==='f') document.documentElement.requestFullscreen?.();
});
```

### Theme
```css
:root{ --navy:…; --deep:…; --ink:…; --blue:…; --cyan:…;
       --ice:…; --bg:…; --text:…; --muted:…; }
```
Dark slides (title/partners/closer):
`background:radial-gradient(1100px 600px at 78% -10%,<accent-soft> 0%,<deep> 46%,#04101F 100%)`

## Component patterns (proven, reuse as-is)

- **Kicker** — small uppercase eyebrow with gradient dash before it.
- **Stat cards** — white, 1px `#E3EAF2` border, 14px radius, big blue number +
  small uppercase label.
- **Feature card grid (2×2)** — white card, 56px gradient icon tile with inline
  SVG (stroke `#fff`, width 1.9, round caps), ink-colored h3, muted body.
  Hover: `translateY(-4px)` + deeper shadow.
- **Timeline steps** — white rows with `border-left:4px solid var(--blue)`,
  numbered circles in `--ice`; final step green (`#12B76A`) + ✓.
- **Partner cards** — see partner-logo-recovery.md for the white-card treatment.
- **Related-product cards** — white, `border-top:5px solid var(--blue)`, small
  uppercase tag, h3, lead line, then a 2×2 mini-tile grid previewing the target
  page (see below), "Learn More →" pinned at the bottom.
- **Watermark SVG fill (empty card corners)** — shared gradient def once per
  deck, then one topic-matching icon per card. Size to the card: ~128px only in
  tall cards; **in small cards/tiles (< ~180px) use 40–60px, fully inside**
  (`right:12px;bottom:10px`), opacity ~0.45, and give the card `min-height`
  (~106px tiles / ~158px cards) plus `padding-right` (~30px) on body text so
  copy never runs under the mark. Off-corner clipping reads as a bug — users
  will ask you to fix it twice:
  ```css
  .card{position:relative;overflow:hidden}
  .card h3,.card p{position:relative;z-index:1}   /* text above the mark */
  .card .wm{position:absolute;right:14px;bottom:12px;width:56px;height:56px;
    z-index:0;stroke:url(#wmgrad);fill:none;stroke-width:1.3;
    stroke-linecap:round;stroke-linejoin:round;opacity:.45}
  ```
  ```html
  <svg width="0" height="0" style="position:absolute"><defs>
    <linearGradient id="wmgrad" x1="0" y1="0" x2="1" y2="1">
      <stop offset="0" stop-color="#0085FF"/><stop offset="1" stop-color="#10C7F1"/>
    </linearGradient>
  </defs></svg>
  ```
  Pick the icon from the card's topic (headset for helpdesk, storefront for
  retail, network nodes for SD-WAN, gauge for NOC …). Simple stroke paths only.
- **Icon/gradient panels** — when a side panel needs an image but the captured
  asset is an icon PNG (not a photo), build the panel as a flex column:
  gradient background, icon centered in a `flex:1` zone, caption in its own
  padded bar at the bottom (`background:rgba(1,36,67,.45)` on gradients).
  Never absolute-position icon and caption over each other.
- **Spreading text to kill white space** — don't shrink gaps, distribute:
  card gets `display:flex;flex-direction:column`; the fill block gets
  `flex:1` + `justify-content:space-evenly` (lists) or `center` + larger
  `gap`/padding/line-height (mini-tiles). Bump body font ~1px at the same time.

## Look ahead: previewing linked pages

Cards that link out ("Learn More") look empty with only the landing page's
one-liner. Fetch the target page and pull its real intro bullets in:

```bash
# webfetch the "Learn More" URL, then quote its overview section
```

Pattern: keep the lead sentence from the hub page, then one mini-tile per
pillar from the target page's "Why choose / advantages" section (title + one
line each, 2×2 grid of tinted chips). This keeps the no-invented-copy rule —
the preview is quoted from the page the button actually opens. Size check:
mini-tile body ≈ 11px/1.6 line-height fits ~3 lines; trim copy to match.
- **FAQ** — `<details class="faq">` with custom `summary::before` "?" chip;
  hide default marker (both `list-style:none` and `::-webkit-details-marker`).
- **Closer CTA** — pill button with gradient bg + colored shadow; real contact
  metadata from the page footer (email/address).

## Asset rules

- Always relative paths: `capture/assets/...`, `capture/assets/partners/...`.
- Never hotlink remote URLs — download first (keeps the repo self-contained).
- Logo recolor trick: `filter:brightness(0) invert(1)` renders a no-fill SVG
  white on dark slides.
- Photos: `object-fit:cover` + `border-radius:18px` + `linear-gradient(180deg,
  transparent 60%, rgba(1,36,67,.55))` bottom shade; depth via
  `box-shadow:0 24px 48px rgba(<navy-rgb>, .25)`.

## Post-build checklist

```bash
for f in <every path referenced in deck.html>; do [ -f "$f" ] || echo "MISSING $f"; done
# outbound links: verify each against the captured DOM, never guess
# (NOT IN CAPTURE = review manually; the capture may store the path relative,
#  or the link may come from a fetched sub-page — confirm before shipping)
for u in <every href in deck.html>; do rg -qF "$u" capture/extracted/page.html && echo "OK $u" || echo "REVIEW $u"; done
```
Then `open deck.html` and click through all slides once.

### Visual QA loop (no playwright needed)

Screenshot EVERY slide headlessly and read the shots before delivering —
clipped hero crops, dead space, and over-bold watermarks never show in code
review. Get the system Chrome path from `npx hyperframes doctor`.

```bash
# 1. hash-jumper in a TEMP copy (inject after show(0);):
sed 's|  show(0);|  show(0); const h=parseInt(location.hash.slice(1)); if(h>0) show(h-1);|' \
  deck.html > deck-shot-tmp.html
# 2. run FROM the project dir — relative asset paths break otherwise
mkdir -p /tmp/deck-shots
for n in $(seq 1 10); do
  "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --disable-gpu \
    --screenshot=/tmp/deck-shots/s$n.png --window-size=1312,740 --hide-scrollbars \
    --virtual-time-budget=3000 "file://$PWD/deck-shot-tmp.html#$n" >/dev/null 2>&1
done
rm deck-shot-tmp.html
```

Read every shot; fix layouts; **re-render only the changed slides** after each
edit round (2–3 iterations is normal). `--virtual-time-budget=3000` lets the
staggered entrance animations settle.
