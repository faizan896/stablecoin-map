# CoinMap — Stablecoin Acceptance Map

A Google-Maps-style directory of businesses that accept stablecoins and Bitcoin. Search by category and city, filter by accepted coin, and let business owners submit themselves.

**Live demo:** _(add your Vercel URL here after deploy)_

![status](https://img.shields.io/badge/stage-prototype-orange)

## What it does

- **Real map** — Leaflet + OpenStreetMap (dark tiles via CARTO). No Google Maps API key, no per-load cost.
- **Search** — free-text (name / category / address) + city, with live filtering as you type.
- **Coin filters** — USDC, USDT, DAI, PYUSD, BTC, Lightning. Multi-select.
- **List ↔ map sync** — click a result to fly to its pin and open the popup.
- **Self-submission** — businesses add themselves via a form, and can click the map to drop an exact pin.
- **Moderation simulation** — new submissions show a `PENDING` badge until reviewed.

25 sample listings across Karachi, Lahore, Islamabad, Dubai, Lisbon, and Buenos Aires.

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

Vanilla HTML/CSS/JS · [Leaflet 1.9](https://leafletjs.com) · OpenStreetMap + CARTO tiles

## License

MIT
