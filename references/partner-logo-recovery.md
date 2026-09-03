# Partner Logo Recovery (and as-is logo treatment)

## Why

Captures routinely **drop partner/brand logos** (size-floor, cap-reached) and the
`asset-descriptions.md` won't list them. Faking logos with styled text ships
off-brand and misrepresents marks — never do it. Always recover the real files.

## Recovery procedure

### 1. Locate the partners section in the captured DOM

```bash
grep -o 'model-our-partners/[^"]*' capture/extracted/page.html | sort -u
```

Generalize: find the section heading in `page.html`, slice the HTML between it
and the next section heading, then extract `<img>` tags from the slice.

### 2. Map URL → brand via `alt` (never guess)

```python
import re
html = open('capture/extracted/page.html').read()
for name in ['Palo Alto','Fortinet','CrowdStrike']:          # brands from visible-text.txt
    m = re.search(name, html, re.I)
    seg = html[max(0,m.start()-1200):m.end()+200]
    for im in re.findall(r'<img[^>]*>', seg):
        print(im[:400])   # read alt="..." next to src="..."
```

Beware lazy-loading: match on `data-src` **or** `src`. Beware repeated
nearby images bleeding into the slice (e.g. one section's logo appearing next to
another's name) — the `alt` attribute is the source of truth for the mapping.

### 3. Download into the capture tree

```bash
mkdir -p capture/assets/partners
curl -sS "<url>" -o capture/assets/partners/<brand-slug>.png
```

Rename opaque hashes (`4_2b0f6f86….png`) to human slugs. Verify with `sips -g
pixelWidth -g pixelHeight -g hasAlpha` (typical: ~210×120 transparent PNGs) and
an image read (confirm it's the right mark, note icon-only vs full wordmark).

## As-is logo treatment on dark slides

Transparent logo PNGs on a dark gradient are illegible. Put them on **white
cards**:

```css
.partner{width:300px;height:200px;background:#fff;border-radius:18px;
  display:flex;flex-direction:column;align-items:center;justify-content:center;gap:14px;
  box-shadow:0 14px 32px rgba(0,0,0,.4)}
.partner img{width:180px;height:104px;object-fit:contain}
.partner span{font-size:12.5px;letter-spacing:.2em;text-transform:uppercase;color:#5E6B7A}
```

- Keep the mark unmodified (no recolor filters on third-party logos).
- Caption below the logo **inside the card**; if a logo is icon-only (e.g. the
  Palo Alto glyph without wordmark), the caption must carry the full brand name.
- If the user asks for full wordmark lockups that the source site doesn't ship,
  say so explicitly — upgrading requires vendor brand-asset pages, which is a
  separate fetch the user should approve.
