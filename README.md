# CoinMap — Stablecoin Acceptance Map

A Google-Maps-style directory of businesses that accept stablecoins and Bitcoin. Search by category and city, filter by accepted coin, and let business owners submit themselves.

**Live demo:** https://stablecoin-map-pink.vercel.app

![status](https://img.shields.io/badge/stage-prototype-orange)

## What it does

- **Live data — ~29,000 real merchants.** Pulled from the [BTC Map](https://btcmap.org) public API on load and cached for 24h. Not sample data.
- **3D world globe** — interactive, auto-rotating globe (globe.gl + Three.js) plotting every node worldwide.
- **Street map view** — toggle to a 2D Leaflet map; the directory then lists whatever is in view.
- **Search + network filters** — free text across name/category/address, plus BTC, LN, USDC, USDT, DAI, PYUSD.
- **Freshness badges** — nodes not verified in over a year are flagged `UNVERIFIED`, so stale listings are visible rather than hidden.
- **Register your business** — prominent form with category, networks, and click-to-drop pin.

Two data layers: real Bitcoin/Lightning merchants from BTC Map, plus a curated stablecoin layer (rendered as hollow markers) that the submission form feeds into.

## Design

Editorial / Swiss layout: pale steel-blue viewport, `#f7f7f5` paper directory panel, Helvetica uppercase headings, monospace micro-labels, outlined network tags, and a single serif accent on *Submit Business*.

## Run locally

It's a single static file — no build step, no dependencies to install.

```bash
open index.html
```

Or serve it:

```bash
python3 -m http.server 8000
# http://localhost:8000
```

## Deploy

Static site, zero config:

```bash
npx vercel --prod
```

Or connect the GitHub repo at [vercel.com/new](https://vercel.com/new) — Framework Preset: **Other**, no build command needed.

## Prototype limitations

Submissions are stored in the browser's `localStorage`, so they're only visible to the person who submitted them. There is no backend, no auth, and no real moderation queue yet.

See [`build-plan.md`](./build-plan.md) for the production architecture: Next.js + Supabase/PostGIS data model, moderation and anti-staleness flow, cost estimates, and launch strategy.

## Roadmap

- [ ] Supabase backend (Postgres + PostGIS) so submissions are shared
- [ ] Auth + owner claim/edit of listings
- [ ] "Still accepting?" confirmations and freshness badges
- [ ] Per-business SEO pages (`/karachi/kaffee-beans`)
- [ ] City landing pages
- [ ] Public API

## Tech

Vanilla HTML/CSS/JS · [globe.gl](https://globe.gl) (Three.js) · [Leaflet 1.9](https://leafletjs.com) · OpenStreetMap + CARTO tiles · [BTC Map API](https://api.btcmap.org)

## Data note

BTC Map is OpenStreetMap-derived, so it covers **Bitcoin and Lightning** merchants. There is no equivalent open bulk source for USDC/USDT acceptance — that gap is the reason this project exists, and why stablecoin listings have to be community-submitted and verified.

## License

MIT
