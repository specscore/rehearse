# Rehearse landing page

A modest, honest landing page for **Rehearse** — SpecScore's acceptance-evidence
layer. This replaces the "Coming Soon" stub now that Rehearse has real, shipped
capabilities to show.

It is a **scaffold**: a structural starting point, deliberately small. One clean
English page, no build step, no i18n framework. Polish and translation can come
later; honesty and simplicity come first.

## What's here

| File | Purpose |
|------|---------|
| `index.html` | The page. Self-contained; the only local asset is `style.css`. |
| `style.css`  | Editorial styling — sibling of specscore.md (paper/ink, hairlines, ~5 colours, no gradients, no dark mode). Rehearse's accent is `--mark-red`, the prompter's correction ink. |

The page describes only what actually works today (shipped in the
[`specscore-cli`](https://github.com/specscore/specscore-cli) runtime): real
executable Markdown scenarios (no glue, no mocks), reusable checks via
`**Use:**`, thin acceptance criteria + generated summary, markdown-native /
agent-authorable authoring, and self-hosting evidence. It states plainly that
Rehearse is **pre-v1, in active development** — there is no stable release.

## Preview locally

It's a static page — just open it, or serve the directory:

```bash
# Option A: open directly
open index.html            # macOS
xdg-open index.html        # Linux

# Option B: serve (fonts/relative links behave exactly as in production)
cd landing
python3 -m http.server 8000
# then visit http://localhost:8000/
```

## Deploy

Static hosting, no build. On **Cloudflare Pages**: point a project at this repo,
set the build output directory to `landing/` and leave the build command empty.
Any static host (GitHub Pages, Netlify, plain nginx) works the same way — upload
the two files.

External links (docs, GitHub) are absolute on purpose; the only local link is
`style.css`, so the page is fully portable.
