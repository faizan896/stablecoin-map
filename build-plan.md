# CoinMap — Stablecoin Acceptance Map: Build Plan

Prototype: `stablecoin-map.html` (open in any browser, no server needed).

---

## 1. Kya cheez already ban chuki hai (prototype)

| Feature | Status |
|---|---|
| Real map (Leaflet + OpenStreetMap dark tiles) | ✅ |
| Search: category/name + city | ✅ |
| Coin filter chips (USDC, USDT, DAI, PYUSD, BTC, Lightning) | ✅ |
| Result list ↔ map sync (click → fly to pin) | ✅ |
| Colour-coded pins by primary coin, emoji by category | ✅ |
| Self-submission form + click-to-drop pin | ✅ |
| PENDING badge (moderation queue simulation) | ✅ |
| 25 sample businesses across 6 cities | ✅ |

Limitation: submissions browser ke `localStorage` mein save hote hain — sirf aapko dikhenge. Real product ke liye backend chahiye.

---

## 2. Recommended stack (fastest path to live)

| Layer | Choice | Kyun |
|---|---|---|
| Frontend | Next.js + MapLibre GL (ya Leaflet) | SEO zaroori hai — "USDC accepted Karachi" jaise queries se organic traffic aayega. SSR chahiye. |
| Database | Supabase (Postgres + PostGIS) | Geo queries built-in (`ST_DWithin`), free tier, auth included. |
| Auth | Supabase Auth (email + Google) | Business owners ko apni listing edit karne ke liye. |
| Geocoding | Nominatim (free) → Mapbox Geocoding agar volume badhe | Address → lat/lng. |
| Map tiles | OpenStreetMap / CARTO free → MapTiler agar traffic badhe | Google Maps tiles mehngi aur license restricted hain. |
| Images | Supabase Storage | Shop photo, logo. |
| Hosting | Vercel | Free tier kaafi hai shuru mein. |

Google Maps API intentionally avoid ki hai — cost per load aur ToS ki wajah se. OSM se free chalta rahega.

---

## 3. Data model

```sql
create extension if not exists postgis;

create table businesses (
  id            uuid primary key default gen_random_uuid(),
  name          text not null,
  slug          text unique not null,
  category      text not null,
  description   text,
  address       text,
  city          text not null,
  country       text not null,
  location      geography(point, 4326) not null,   -- PostGIS
  coins         text[] not null,                   -- ['USDC','USDT']
  chains        text[],                            -- ['Base','Tron','Solana']
  payment_method text,                             -- 'wallet_qr' | 'pos_terminal' | 'invoice'
  phone         text,
  website       text,
  photo_url     text,
  owner_id      uuid references auth.users(id),
  status        text not null default 'pending',   -- pending|approved|rejected|flagged
  verified_at   timestamptz,
  last_confirmed_at timestamptz,                   -- "still accepting?" freshness
  created_at    timestamptz default now()
);

create index on businesses using gist(location);
create index on businesses (city, status);
create index on businesses using gin(coins);

-- community trust signals
create table confirmations (
  id uuid primary key default gen_random_uuid(),
  business_id uuid references businesses(id) on delete cascade,
  user_id uuid references auth.users(id),
  worked boolean not null,       -- "maine try kiya, chala / nahi chala"
  coin text,
  note text,
  created_at timestamptz default now(),
  unique (business_id, user_id, created_at)
);

create table reports (
  id uuid primary key default gen_random_uuid(),
  business_id uuid references businesses(id) on delete cascade,
  reason text not null,          -- closed | no_longer_accepts | spam | wrong_info
  detail text,
  created_at timestamptz default now()
);
```

**Nearby search query:**
```sql
select *, st_distance(location, st_point($lng,$lat)::geography) as dist
from businesses
where status = 'approved'
  and st_dwithin(location, st_point($lng,$lat)::geography, $radius_m)
  and ($coins is null or coins && $coins)
  and ($category is null or category = $category)
order by dist limit 50;
```

---

## 4. Submission + moderation flow

```
Owner form bhare
   │
   ├─ Auto-checks: duplicate detect (name + 100m radius), address geocode success,
   │  disposable email block, rate-limit per IP (3/day), Turnstile captcha
   │
   ├─ status = 'pending'  →  admin queue mein
   │
   ├─ Admin review (Supabase dashboard ya simple /admin page): approve / reject / request-info
   │
   └─ approved → live on map, owner ko email
```

**Data rot hi asli problem hai.** Crypto maps mostly is liye mar jaate hain ke listings purani ho jaati hain (dukaan band, ya ab accept nahi karti). Isay ese handle karein:

- Har listing par **"Still accepting?" 👍 / 👎** button. 3 din mein 3 👎 → auto-flag, map par "unverified" dikhaye.
- 6 mahine se koi confirmation nahi → **"Last confirmed 6 months ago"** warning badge.
- Owner ko har 90 din email: "confirm karein ke aap ab bhi accept karte hain" — ek click.
- Har listing par **Report** link.

Ye teen cheezein hi long-term me product ki credibility banati hain.

---

## 5. Roadmap

**Phase 1 — MVP (2–3 hafte)**
Next.js + Supabase, map + search + filters, submission form, admin approve page, 50–100 seed listings ek city ke liye (manually verify karein, hand-collected).

**Phase 2 — Trust layer (2 hafte)**
Auth, owner claim/edit, confirmations, reports, freshness badges, business detail page (`/karachi/kaffee-beans` — SEO ke liye).

**Phase 3 — Growth (ongoing)**
City landing pages, "Accepted here" QR sticker + window decal owners ke liye, public API, Telegram/WhatsApp bot ("near me" query), CSV import from BTCMap/OSM tags.

---

## 6. Cost estimate (monthly)

| Item | 0–5k users | 50k users |
|---|---|---|
| Vercel | $0 | $20 |
| Supabase | $0 | $25 |
| Map tiles (MapTiler/CARTO) | $0 | $0–50 |
| Geocoding (Nominatim → Mapbox) | $0 | ~$10 |
| Domain | ~$1 | ~$1 |
| **Total** | **~$1** | **~$60–105** |

Effectively pehle saal mein infra cost near-zero hai. Asli cost aapka waqt hai — listings verify karna.

---

## 7. Sabse bara risk (seedhi baat)

Ye **cold-start / chicken-and-egg** problem hai. Khaali map ki koi value nahi; users tab aayenge jab listings hongi, listings tab aayengi jab traffic hoga.

Rasta: **ek shehar, ek category se shuru karein** — misal Karachi ke coffee shops + restaurants. Khud 50–80 jaghen zameen par verify karein (ya WhatsApp/call se). Us ek shehar mein map genuinely useful ho jaye, phir agla shehar. Din 1 se global launch karne ki koshish sabse aam wajah hai jis se ye projects fail hote hain.

Competitors dekhne layak: BTCMap (Bitcoin-only, OSM-based, open data — aap unka data import kar sakte hain), Coinmap.org (largely stale). **Stablecoin-first** angle abhi khaali hai — yahi aapka differentiator hai, kyunke merchants volatility ki wajah se BTC se zyada USDC/USDT accept karte hain.

---

## 8. Legal note

Merchant listing directory chalana zyadatar jurisdictions mein money-transmission nahi hai — aap payments process nahi kar rahe. Lekin agar aap kabhi in-app payments, escrow, ya wallet add karein, to regulatory picture poori tarah badal jaata hai. Us step se pehle local counsel se baat karein. Main lawyer nahi hoon — ye general information hai, legal advice nahi.
