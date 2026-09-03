# Step 4 — Hosting on GitHub Pages

Only when the user asks to host/publish the deck.

## Steps

```bash
cd <project-dir>
git init -b main
git add deck.html capture/assets        # NOT screenshots/extracted/, unless asked
git commit -m "<Deck name> with captured assets"
gh repo create <repo-name> --public --source . --push
gh api -X POST repos/<owner>/<repo>/pages -f "source[branch]=main" -f "source[path]=/"
```

Ask public vs private first — hosting (GitHub Pages for free accounts) requires
**public**; confirm before creating. Default name: the project folder name.

## What to include

| Include | Exclude |
|---|---|
| `deck.html` | `capture/screenshots/` (11MB+ of scroll shots) |
| `capture/assets/` (images, svgs, fonts, partners/) | `capture/extracted/` (page.html, tokens.json…) |
| | `capture/AGENTS.md`, `CLAUDE.md`, contact sheets |

## Verify + deliver

```bash
gh api repos/<owner>/<repo>/pages/builds/latest --jq '{status,commit}'
curl -s "https://<owner>.github.io/<repo>/deck.html?v=$RANDOM" | grep -o '<unique-new-string>'
```

Deliver: repo URL + `https://<owner>.github.io/<repo>/deck.html`.
First build takes ~1–2 min.

## Cache gotcha (report this to users)

After any later edit + push, the live page often serves **stale HTML for
minutes** even when the build already says `built`:

1. Check the build API first — if `status: built` and `commit` matches the
   pushed SHA, deployment succeeded.
2. Confirm content with `curl` + `?v=$RANDOM` query (CDN cache-bust).
3. The user's browser likely has the old copy cached → hard refresh
   (Cmd+Shift+R) or a `?v=` URL. Don't re-push or "fix" what isn't broken.

## Updating an already-hosted deck

Edit locally → `git add <changed>` → `git commit` → `git push`. Never re-create
the repo. New assets must live under `capture/assets/` to preserve structure.
