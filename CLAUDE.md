# Dual Use Tracker — Project Context

## What this is
A self-contained single-file React app tracking venture capital investment in dual-use defense tech startups. No build tools — React 18 + Babel loaded via unpkg CDN, all in one HTML file.

**Main file:** `index.html` (~2200 lines) — also symlinked as `dual-use-tracker.html` for local reference
**Preview server:** `dual-use` (port 8091, defined in `../.claude/launch.json`)
**Live URL:** https://dualuse.vc (GitHub Pages — repo: https://github.com/arepb/dualuse-vc)
**Deploy:** `git add index.html && git commit -m "..." && git push origin main`

## Design System

**Aesthetic:** Space Shuttle / NASA Helvetica — monumental, high-contrast, generous whitespace.

- Font: `"Helvetica Neue", Helvetica, Arial, sans-serif` — hardcoded in every component
- Background: `#ffffff` (pure white)
- Text: `#000000` (pure black)
- Muted text: `#888`
- Header background: `#d8d8d8`
- Borders: `1px solid #000` (cards), `3px/4px solid #000` (structural dividers)
- Hover: `rgba(0,0,0,0.03)` — barely perceptible
- Accent / citations: `#e00000` (bright red)
- No other accent colors — strict monochrome except red

**Header:** `#d8d8d8` background, `#e00000` "DUAL USE" wordmark at 96px/900 weight with `WebkitTextStroke: "3px #e00000"`. **Wordmark is clickable** — `onClick: () => setActiveTab("overview")` returns to Overview from any tab. Rotating halftone globe canvas is in the top-right of the header.

**Typography scale:**
- Wordmark: 96px / 900 weight / `-0.03em` tracking
- H2 subtitle: 22px / **IBM Plex Mono italic 400** (Google Fonts) / `#666` / NOT caps — "Tracking venture capital investment in defense tech"
- Stat numbers: 96px / 900 weight / `-0.04em` tracking
- Section headers (`h3`): 32px / 900 weight / `-0.02em` tracking — used consistently for Open Roles, Signals, Biggest Rounds, Jobs, etc.
- Card names: 18–20px / 700 weight
- Labels/caps: 10–11px / 700 weight / `0.2em` tracking / uppercase
- Body: 14px / 400 weight / 1.8 line-height

**Logos:** Google Favicons API — `https://www.google.com/s2/favicons?domain={domain}&sz=128`, size 36px in cards. Letter-initial fallback on black square.

## File Structure (line reference)

| Section | Approx. lines |
|---|---|
| `Logo` component | 31–54 |
| Data: `VC_FUNDS` (25 funds) | 57–300 |
| Data: `STARTUPS` (100 startups) | ~301–1300 |
| Data: `FUNDING_ROUNDS` (20 rounds) | ~1300–1370 |
| Data: `DEFENSE_PRIMES` (8 primes) | ~1370–1420 |
| `SECTORS`, `FUND_TYPES` constants | ~1420–1430 |
| `Cite`, `Badge`, `BadgeOutline`, `BadgeGray`, `StatCard`, `SearchBar`, `Select` components | ~1430–1540 |
| `RSS_FEEDS` constant + `NewsSection` component | ~1540–1660 |
| `FeaturedJobs` component | ~1720–1800 |
| `OverviewTab` | ~1800–1830 |
| `FundsTab` | ~1830–1900 |
| `StartupsTab` | ~1900–1980 |
| `JobsTab` | ~1980–2100 |
| `FundingTab` | ~2100–2160 |
| `PrimesTab` | ~2160–2200 |
| `AboutTab` | ~2200–2300 |
| `TABS` array + `App` component | ~2300–2420 |

## Data Schema

**VC_FUNDS:** `{ id, name, type, hq, aum, defenseThesis, domain, description, notableInvestments[], sectors[], stage }`

**STARTUPS:** `{ id, name, founded, hq, valuation, totalRaised, domain, description, sectors[], investors[], primePartners[], dualUse, commercialUse }`

**FUNDING_ROUNDS:** `{ company, round, amount, leadInvestor, valuation, date }`

**DEFENSE_PRIMES:** `{ name, ventureArm, domain, startupPartners[], relationship }`

## Tabs & Routing
6 tabs + About page with hash routing:
- Overview (`/`)
- VC Funds (`/#funds`)
- Startups (`/#startups`)
- Jobs (`/#jobs`) ← left of Funding Rounds
- Funding Rounds (`/#funding`)
- Defense Primes (`/#primes`)
- About (`/#about`) — footer-only, not in nav bar

`window.location.hash` is read on load and updated on tab change. `hashchange` event listener syncs browser back/forward. `VALID_TABS` set includes `"about"`.

## Signals News Feed (Overview tab)
- **Renamed from "Defense Tech News" to "Signals"** — heading is `h3` at 32px/900 matching Open Roles and Biggest Rounds
- **Layout:** Signals and Open Roles sit side by side in a 2-column grid on desktop (`.du-grid-2col`), stack on mobile
- **RSS sources (7):** Defense One, Breaking Defense, Defense News, C4ISRNET, The War Zone (twz.com), SOF Week, SBIR/STTR
- **Proxy:** `https://api.allorigins.win/get?url=ENCODED_URL` — handles CORS, returns base64-encoded XML; decode with `new TextDecoder('utf-8').decode(Uint8Array.from(atob(b64), c => c.charCodeAt(0)))` — **must use TextDecoder not atob directly** to avoid UTF-8/Latin-1 mangling of curly quotes etc.
- **Cache:** `localStorage` key `dualuse_news_cache_v3`, TTL 15 minutes. Stale-while-revalidate pattern.
- **Cold load:** section collapses to header-only until content arrives (no skeleton cards)
- **Recency filter:** articles older than 60 days are excluded
- **Source cap:** max 3 articles per source (`MAX_PER_SOURCE = 3`) — prevents any single feed from dominating
- **Interleave:** round-robin across sources after capping, so no publication runs consecutively
- **Header:** shows only timestamp ("UPDATED X AGO") — source list removed for cleanliness
- **Per-feed timeout:** 8 seconds via `AbortController`
- **Mobile:** source list (`.du-news-sources`) hidden; articles render as 1-column list (`compact` prop)
- **`compact` prop:** when `true` (Overview sidebar), news renders 1-col; when `false`/absent (full-width), renders 4-col grid

## Open Roles (Overview tab)
- **`FeaturedJobs` component** — fetches live from all ATS sources, picks interleaved sample
- **Header row** uses `flexWrap: "wrap"` so "View all X roles →" link wraps below heading on mobile instead of clipping
- **Preview list** on Overview shows interleaved jobs above Signals; "VIEW ALL" link navigates to Jobs tab

## Jobs Tab
- Fetches live from Greenhouse + Ashby APIs (20 + 5 companies confirmed)
- **Cache:** `localStorage` key `dualuse_jobs_cache`, TTL 4 hours
- **Round-robin interleave** across companies — no single employer dominates
- **Prime careers links** shown below the live job list
- **Heading:** 32px/900 with `borderBottom: "4px solid #000"` (full-width underline — not the old 48px fixed bar)

## About Page (`/#about`)
- Footer-only link — does not appear in main nav
- ~300 words across sections: What is Dual Use?, What We Track, Why Dual Use Matters, Data & Methodology, Contact
- **No em dashes** in copy — house style uses commas instead
- **Contact email:** `covington@dualuse.vc` — **JS-obfuscated**: assembled from `u = "covington"` + `d = "dualuse.vc"` in a `useEffect`, so static HTML never contains the full address

## Footer
- Left: data attribution text
- Right: "About" link + "© [current year] DualUse.vc" (year via `new Date().getFullYear()`)

## Globe
- Canvas-based rotating halftone sphere in header top-right
- Algorithm: lat/lon dot grid, dot size = f(lighting + edge), 23.5° axial tilt, clockwise rotation
- **Favicon:** `favicon.gif` — animated GIF, 64×64px, white background, 60 frames at 120ms/frame (~7s revolution), same algorithm
- **og:image:** Generated entirely in Python+Pillow (`og-image-generator.py` or inline script). Uses `/System/Library/Fonts/HelveticaNeue.ttc` index=1 (Bold, NOT italic) for wordmark, `/Library/Fonts/IBMPlexMono-Italic.otf` for subtitle. **Never use html2canvas for og:image** — LinkedIn/Facebook scrapers run no JS, they fetch the PNG directly.
- To regenerate og-image.png: run the Python script in the Bash tool, then `git add og-image.png && git push`

## SEO & Distribution
- **Sitemap:** `https://dualuse.vc/sitemap.xml` — 6 URLs (root + hash routes), `changefreq: weekly`
- **robots.txt:** allows all, points to sitemap
- **llms.txt:** at `https://dualuse.vc/llms.txt` — structured markdown for LLM crawlers
- **og:image:** `https://dualuse.vc/og-image.png` — 1200×630 PNG, baked in Python
- **JSON-LD:** WebSite schema in `<head>`
- **Google Search Console:** submit sitemap at https://search.google.com/search-console

## Weekly Automated Refresh
- **Scheduled task:** `dualuse-weekly-refresh` — runs every Monday at 9:00 AM local time
- **What it does:** Searches web for new defense tech funding rounds (≥$25M), checks for major startup status changes (IPOs, acquisitions, new valuations), updates `index.html` and `sitemap.xml`, commits and pushes to GitHub
- **Threshold:** Only adds rounds confirmed by at least one reliable source. Does not delete existing data.
- To run manually: trigger from Claude scheduled tasks sidebar

## Mobile Responsive
CSS classes with `@media (max-width: 768px)` override inline styles via `!important`:
- `.du-header` — header padding (48px 64px → 28px 20px)
- `.du-wordmark` — H1 font-size (96px → 52px)
- `.du-subtitle` — H2 font-size (22px → 17px)
- `.du-nav-inner` — nav padding (0 64px → 0 4px)
- `.du-tab` — tab padding/font-size (tightened)
- `.du-main` — main padding (64px → 24px 16px)
- `.du-footer` — footer padding
- `.du-grid-cards` — fund/startup/prime grids (`minmax(480px, 1fr)` → `1fr`)
- `.du-grid-2col` — overview 2-col layout (`1fr 1fr` → `1fr`) — used for Open Roles + Signals side-by-side
- `.du-grid-detail` — expanded card detail (`1fr 1fr` → `1fr`)
- `.du-grid-news` — news grid → `1fr`
- `.du-news-label` — Signals `h3` heading (letter-spacing only; font-size NOT overridden — keeps at 32px to match Open Roles)
- `.du-news-sources` — source list hidden on mobile
- `.du-globe` — globe shown on mobile (positioned below subtitle)
- `.du-job-meta` — job type/location/apply columns hidden on mobile

## Key Decisions (don't undo these)
- **No Clearbit** — was down. Use Google Favicons API instead.
- **No warm colors** — previous FT-inspired salmon palette was replaced with strict monochrome + red accent.
- **Connections tab removed** — was the 6th tab, deliberately cut.
- **Card borders are `1px solid #000`** (not `#e0e0e0`) — part of the monochrome aesthetic.
- **Links on logo+name:** Fund, Startup, and Prime name/logo are `<a>` tags opening `https://{domain}` in a new tab. `stopPropagation` prevents triggering card expand. The type badge stays outside the link.
- **Citations:** `<Cite>` component renders red superscript links.
- **Trucks Venture Capital** is 4th in the `VC_FUNDS` array (id: 4). Notable investments: JetZero, Joby Aviation, Skyryse, Auriga Space, Havoc.
- **JetZero** is startup id 23. Series B ($175M, Jan 2026) lead investors: **B Capital, Trucks Venture Capital**.
- **Biggest Rounds list** shows top 15 (`.slice(0, 15)`), border condition `i < 14`.
- **IBM Plex Mono** loaded via Google Fonts: `family=IBM+Plex+Mono:ital,wght@1,400` — only italic 400 weight needed.
- **og:image must be raster (PNG/JPG)** — iOS/iMessage and social platforms don't support SVG for previews.
- **Section heading underlines** use `borderBottom: "4px solid #000"` on the `h3` itself — never a separate fixed-width `div` (that was the old Jobs tab bug, now fixed).
- **Header flex rows** (Open Roles, Signals) use `flexWrap: "wrap"` so secondary links don't clip on mobile.
- **Email obfuscation:** contact address assembled via `useEffect` from split parts — never appears as plain text in static HTML.

## Content data notes
- Stats on Overview are hardcoded (not computed from data): $49.1B, 41% growth, 825 startups, 10 unicorns
- All dollar amounts in FUNDING_ROUNDS are strings (e.g. `"$2.5B"`, `"~$694M"`)
- Amount sort parses B/M suffix: `s.includes("B") ? n * 1000 : n`
- **100 startups** (ids 1–100). Key Trucks VC portfolio: Joby (24), Skyryse (25), Auriga Space (26), JetZero (23), Havoc.
- Several startups are public companies — valuation field uses market cap format e.g. `"$40.5B (NASDAQ: RKLB)"`
- Capella Space (id 35) acquired by IonQ May 2025 — noted in valuation field
- **Investor data audited March 2026** — 11 corrections applied. Key fixes: Epirus (removed Lux Capital), Saronic (removed Kleiner Perkins — not yet closed), ABL Space (removed Founders Fund), Gecko Robotics (removed Tiger Global + Bessemer), Firestorm Labs (corrected domain + description), Skydweller (Leonardo S.p.A. not Leonardo DRS), Joby (removed SoftBank)
- **Archer Aviation** removed from job sources — slug `archer` pulls Archer Veterinary, not Archer Aviation. No discoverable public board found.
- **House style:** no em dashes in copy. Use commas or restructure sentences instead.
