# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page web app for tracking a personal LEGO minifig collection. No framework, no bundler — just `index.html`, `data.json`, and local images.

## Running locally

Open `index.html` in a browser, or serve it with any static file server (required for the `fetch('data.json')` call):

```bash
python3 -m http.server 8000
```

No npm install, no build step needed to view the app.

## Architecture

### Data layer
- `data.json` — source of truth for all minifig data: `name`, `series`, `itemNum`, `collected` (default), `img` (Rebrickable CDN fallback URL), `categories` (array, a fig can belong to multiple)
- Collected state at runtime is stored in `localStorage` key `minifig-collection-v3` (format: `{"Series 1:Caveman": true, ...}`)
- `build.js` — one-shot Node.js script that reads a Rebrickable CSV export and regenerates `data.json` with image URLs. The hardcoded CSV path is machine-specific. **To add new figs, edit `data.json` directly** — the collection array inside `build.js` is historical.

### Image layer
Images are local `.webp` files under `images/`. The path convention varies by category:

| Category | Path pattern |
|---|---|
| Most categories | `images/{category}/{series}/{name-slug}.webp` |
| BAM, Spongebob, Spiderman | `images/{category}/{name-slug}.webp` (flat) |
| BAM (duplicate slugs) | `images/bam/{name-slug}-{itemNum}.webp` |

Name slugs are lowercased with non-alphanumeric chars replaced by `-`. If a local image 404s, the card falls back to the Rebrickable CDN URL from `data.json`.

### UI (all in `index.html`)
- Sticky header with LEGO logo and search input
- Sticky filter bar with category pills (driven by `CATEGORY_ORDER` array in the JS)
- Main grid grouped by category → series, each section showing per-series progress
- Clicking a card toggles `collected` state; all instances of a fig across categories update together (via `figToCards` WeakMap)
- URL hash (`#CollectibleMinifigs`) persists the active category filter
- localStorage migration: v2 keys (with category prefix) are migrated to v3 on load

### Category ordering
`CATEGORY_ORDER` in `index.html` controls the display order. Categories present in `data.json` but not in this array appear at the end.

## Adding new minifigs

Edit `data.json` directly. Each entry needs:

```json
{
  "name": "Name Here",
  "itemNum": "71234",
  "series": "Series 28",
  "collected": false,
  "img": null,
  "categories": ["Collectible Minifigs"]
}
```

Then add the corresponding `.webp` image to the correct `images/` subdirectory.
