# CalcMonte — Project Handoff

## Executive Summary

**CalcMonte** (https://calcmonte.com) is a live, publicly deployed single-file financial calculator web application (~137KB, ~1,500 lines) that combines loan amortization analytics with a Monte Carlo investment simulator. It targets financially literate users who want deeper analysis than typical online calculators provide. The design philosophy is **"sophisticated simplicity"** — quantitative rigor behind an intuitive, clean interface.

The entire application is one self-contained HTML file (`index.html`) with inline CSS and JavaScript. No build system, no npm, no framework. External dependencies are loaded from CDN at runtime: Chart.js for charts, Google Fonts for typography, and KaTeX for math rendering (lazy-loaded).

**Current status: LIVE and fully operational.** Deployed March 1, 2026. Google-indexed with FAQ and Breadcrumb rich results active. Analytics running. Ready for distribution.

---

## Live Infrastructure

### Domain & Hosting
| Component | Provider | Details | Cost |
|-----------|----------|---------|------|
| Domain | Cloudflare Registrar | `calcmonte.com` + `www.calcmonte.com` | ~$10.44/year |
| Hosting | Cloudflare Workers (static assets) | Worker name: `calcmonte` | Free |
| CDN/Security | Cloudflare | HTTPS, DDoS protection, global CDN | Free |
| Analytics | GoatCounter | `calcmonte.goatcounter.com` | Free |
| Search | Google Search Console | Verified, indexed, sitemap submitted | Free |

**Total running cost: ~$10/year**

### Deployed Files
```
deploy/
├── index.html        ← The entire application (~137 KB)
├── og-image.png      ← Social sharing preview image (1200×630, 41 KB)
├── robots.txt        ← Search engine crawl directives (66 B)
└── sitemap.xml       ← URL listing for Google (231 B)
```

### Cloudflare Account
- Account: Osepper@gmail.com
- Dashboard: https://dash.cloudflare.com
- Worker: Workers & Pages → `calcmonte`
- Domain settings: Domains → `calcmonte.com`
- Preview URL: `calcmonte.osepper.workers.dev`

### How to Redeploy
1. Edit `index.html` locally (at `~/Desktop/LoanAnalytics/deploy/index.html`)
2. Go to Cloudflare → Workers & Pages → `calcmonte`
3. Create new deployment → upload the `deploy` folder
4. Live within seconds, zero downtime

### Local Project Files
```
~/Desktop/LoanAnalytics/
├── deploy/                          ← Upload this folder to Cloudflare
│   ├── index.html
│   ├── og-image.png
│   ├── robots.txt
│   └── sitemap.xml
├── loan-analytics-backup-v1.html    ← Pre-polish backup (all features, no v3 fixes)
├── financial-calculator.jsx         ← Earlier React prototype (historical, not used)
└── HANDOFF.md                       ← This document
```

---

## Architecture

### Internal Organization (index.html)
The file is structured in this order:
1. **`<head>`** — SEO meta tags, Open Graph, Twitter Cards, structured data (JSON-LD), favicon (inline SVG)
2. **`<style>`** — All CSS (~300 lines), including responsive breakpoints at 900px and 600px
3. **`<body>` HTML** — Landing page → Top nav → Loan page → Asset page → Methodology page → Footer → Modal
4. **`<script>`** — All JavaScript (~640 lines), organized as:
   - Shared utilities (formatting, chart helpers, debounce, page switching)
   - `L` namespace (Loan module, IIFE returning public API)
   - `A` namespace (Asset module, IIFE returning public API)
   - Boot sequence (KaTeX lazy-loader, URL state encoder/decoder, localStorage, initialization)
5. **GoatCounter** — Analytics script tag just before `</body>`

### Namespaces
- **`L`** — Loan Analytics module. Public API: `init, update, st, openModal, renderTable, toggleAsset, setAssetType, saveScenario, removeScenario, exportCSV, switchTab, rebuildTabs, loadPreset, saveState, loadSaved`
- **`A`** — Asset Simulator module. Public API: `init, run, addEvent, removeEvent, switchTab, exportCSV, loadPreset, saveState, loadSaved`

### External Dependencies (CDN)
| Library | Version | Size | Purpose | Load Strategy |
|---------|---------|------|---------|---------------|
| Chart.js | 4.4.1 | ~200KB | All charts | Synchronous in `<head>` |
| Google Fonts | — | ~100KB | DM Serif Display + IBM Plex Mono | Async preconnect |
| KaTeX | 0.16.9 | ~300KB | Math formulas on methodology page | **Lazy-loaded** on first methodology tab click |
| GoatCounter | — | ~3.5KB | Privacy-friendly analytics | Async before `</body>` |

### Design System
```
Brand: CalcMonte
Domain: calcmonte.com

Colors (CSS variables):
  --bg: #080c14          (background)
  --card: #0f1520        (card surfaces)
  --border: #1a2436      (borders)
  --text: #e1e7f0        (primary text)
  --text-muted: #7e92b0  (secondary text)
  --text-dim: #3f5170    (tertiary/hint text)
  --cyan: #06d6a0        (primary accent — positive values, CTAs)
  --blue: #58a6ff        (secondary accent — loan balance)
  --red: #ff6b6b         (negative values, interest)
  --amber: #fbbf24       (warnings, asset values)
  --purple: #c4b5fd      (tertiary accent — total paid, scenarios)

Fonts:
  --font-display: DM Serif Display  (headings)
  --font-mono: IBM Plex Mono        (everything else — monospace for numbers)
```

---

## Loan Analytics Module (L)

### Mathematical Model
Standard fixed-rate fully-amortizing loan with annuity formula:
```
PMT = P · r · (1+r)^n / ((1+r)^n - 1)
```
Where P = principal, r = periodic rate, n = total periods.

### Features
- **6 payment frequencies**: Monthly, Bi-Weekly, Weekly, Semi-Monthly, Quarterly, Annual
- **Extra payments**: Added to principal each period, reduces term
- **Down payment**: Percentage input; derives total property value as `tpv = principal / (1 - downPct/100)`
- **Asset simulation toggle**: Appreciating (home) or Depreciating (car) overlay
  - Asset value: `V(t) = V₀ · (1 ± rate)^(t/12)` — exponential growth/decay (deterministic mode, σ=0)
  - **Stochastic mode (σ>0, appreciating only)**: GBM Monte Carlo with 200 paths, showing 5th/25th/50th/75th/95th percentile bands. Uses same `S(t+Δt) = S(t)·exp[(μ-σ²/2)Δt + σ√Δt·Z]` model as Asset Simulator.
  - Underwater detection: deterministic warns at specific month; stochastic shows **P(Underwater)** — percentage of paths where home value drops below loan balance
  - Volatility slider: 0-30%, hidden for depreciating assets. Hint: Home ~8%, Car ~5%
- **Scenario comparison**: Save up to 3 configurations, compare side-by-side (balance curves, interest curves, table)
- **Tabs**: Overview, Breakdown, Schedule, Compare (if scenarios saved), Asset (if enabled)
- **Export**: CSV of full amortization schedule, browser print-to-PDF
- **Click-to-snapshot**: Click any chart point → modal with detailed period breakdown
- **Orientation hint**: "Choose a preset or adjust the parameters below" below preset buttons

### Key Implementation Details
- Amortization runs in `calcAmort()` — returns full schedule array
- Asset values computed in `calcAsset()` — monthly exponential
- Charts use shared `sample()` function to downsample large schedules to ~120 points
- Debounced input handlers (150ms) on number fields; sliders update instantly
- All inputs auto-save to localStorage on every update

### Presets
| Preset | Principal | Rate | Term | Down% | Asset | Vol |
|--------|-----------|------|------|-------|-------|-----|
| 30yr Mortgage | $350K | 6.5% | 30yr | 20% | Appreciating 3.5% | 8% |
| 15yr Mortgage | $350K | 5.75% | 15yr | 20% | Appreciating 3.5% | 8% |
| 5yr Auto | $35K | 5.9% | 5yr | 10% | Depreciating 15% | — |
| 3yr Auto | $25K | 4.5% | 3yr | 15% | Depreciating 20% | — |
| Student Loan | $45K | 5.5% | 10yr | 0% | Off | — |
| Personal | $15K | 10.5% | 5yr | 0% | Off | — |

---

## Asset Simulator Module (A)

### Mathematical Model
**Deterministic mode (σ=0):** Standard compound interest with configurable compounding frequency.
```
Continuous: FV = PV · e^(rt)
Discrete:   FV = PV · (1 + r/n)^(nt)
```

**Stochastic mode (σ>0):** Geometric Brownian Motion with Itô correction.
```
S(t+Δt) = S(t) · exp[(μ - σ²/2)Δt + σ√Δt · Z]
```
Where Z ~ N(0,1) via Box-Muller transform: `Z = √(-2·ln(U₁)) · cos(2π·U₂)`

The `-σ²/2` drift adjustment (Itô correction) ensures the expected value of the stochastic process equals the deterministic growth path.

### Features
- **Compounding modes**: Continuous, Daily (365), Monthly (12), Quarterly (4), Semi-Annual (2), Annual (1)
- **Volatility**: 0% = deterministic, ~15-20% = stock market, ~30%+ = crypto
- **Monte Carlo**: 50–5,000 configurable paths
- **Cash flows**: Regular contributions/withdrawals at 4 frequencies + one-time events at specific months
- **Percentile bands**: 5th, 10th, 25th, 50th (median), 75th, 90th, 95th
- **Historical reference card**: Static reference in sidebar showing typical APY and volatility for S&P 500 (~10% / ~18σ), Total Bond Mkt (~5% / ~6σ), NASDAQ (~12% / ~22σ), Bitcoin (~60% / ~65σ), CDs/HYSA (~4% / 0σ)
- **Tabs**:
  - Projection: Fan chart (stochastic) or single line (deterministic) + milestones
  - Breakdown: Value decomposition, cumulative cash flows, cost basis
  - Distribution: Histogram of final values (color-coded profit/loss), percentile table at multiple horizons, probability analysis
- **Export**: CSV of median path data
- **Orientation hint**: "Choose a preset or adjust below, then Run Simulation" below preset buttons

### Key Implementation Details
- `simPath(inp, stochastic)` — runs one GBM path with monthly steps and cash flow events
- `growthMultiplier(apy, comp, dt)` — returns exact multiplier for any compounding mode
- Percentiles computed by sorting all paths at each month and extracting quantiles
- Balance floored at 0 (prevents negative portfolio values)
- Median path for breakdown: finds path closest to median final value

### Presets
| Preset | Initial | APY | Vol | Horizon | Compounding | Contrib |
|--------|---------|-----|-----|---------|-------------|---------|
| CD/HYSA | $10K | 4.5% | 0% | 5yr | Daily | $500/mo |
| S&P 500 | $10K | 10% | 18% | 20yr | Continuous | $500/mo |
| Bond Fund | $50K | 5% | 6% | 10yr | Monthly | $200/mo |
| Crypto | $5K | 15% | 55% | 5yr | Continuous | $100/mo |
| Retirement | $100K | 7% | 15% | 30yr | Continuous | $1K/mo |

---

## UX Flow

### Landing Page
- Headline: "The Financial Calculator That Shows What Others Won't"
- Three feature cards (outcomes-focused copy, not technical jargon)
- SVG chart preview (CSS-only Monte Carlo fan chart visualization)
- Two CTAs: "Open Loan Calculator" (primary) / "Open Asset Simulator" (secondary)
- Trust line: "Free forever · No signup · No data collected · 100% client-side"
- No tool pages visible until user clicks a CTA — landing is the sole entry point

### Navigation
- **Primary tabs**: Loan Analytics, Asset Simulator (full-size nav buttons)
- **Secondary tabs**: Methodology, Share (visually demoted with `.secondary` class — 50% opacity, smaller font)
- Share button encodes current state into URL hash and copies to clipboard

---

## State Management

### URL Hash State
All parameters encode into the URL hash for sharing:
```
#p=loan&lp=350000&lr=6.5&lt=30&lu=years&lf=12&le=0&ld=20
#p=asset&ai=10000&aa=10&ac=continuous&ah=20&ahu=years&av=18&ap=1000&aca=500&acf=12&awa=0&awf=12
```
- `encodeURLState()` — writes current config to hash (called on every page switch)
- `decodeURLState()` — reads hash on page load, returns true if valid state found
- Share button copies full URL to clipboard

### localStorage Persistence
- Loan state saved under key `L_state` on every `update()` call
- Asset state saved under key `A_state` on every `run()` call
- Load priority: URL hash > localStorage > defaults

---

## Responsive Design

### Breakpoints
- **Desktop (>900px):** Two-column grid — 330px sidebar + flexible content
- **Tablet (≤900px):** Single column — sidebar on top, content below. `flex-direction:column`. All `max-height` and `overflow` constraints cleared.
- **Phone (≤600px):** Further compacted — smaller fonts, single-column stat cards, stacked field rows, larger touch targets

### iOS/Mobile-Specific Fixes
- Range inputs: 36px tall element with transparent background, separate `::webkit-slider-runnable-track` (4px), 20px thumb with `margin-top:-8px`. `touch-action:manipulation` prevents scroll-grab conflicts.
- Chart containers: Explicit `height` at all breakpoints (300/260/220px) — Chart.js requires height-constrained parent when `maintainAspectRatio:false`.
- Number inputs: `font-size:16px!important` prevents iOS auto-zoom on focus.
- Nav: `overflow-x:auto` with hidden scrollbar for horizontal scroll on small screens.
- Removed all collapsible sidebar patterns — mobile uses natural document scroll with sidebar on top, content below.

### Important: Local File Limitation
**Opening the HTML file directly on iOS (via Files app or download) opens it in Quick Look, not Safari.** Quick Look has severely limited JavaScript support — sliders, buttons, charts, and tab switching will not work. The file **must be served from a web server** to function on mobile. This is an iOS platform limitation, not a code bug. This is why the app is deployed on Cloudflare.

---

## SEO Implementation (LIVE)

### Meta Tags (Optimized)
- **Title**: "Free Loan Calculator & Monte Carlo Investment Simulator | CalcMonte" — hits three highest-volume keyword clusters
- **Description**: "Compare loan scenarios side by side, model extra payments, and simulate investment growth with real market volatility. See best-case to worst-case outcomes — not just one number. Free, private, no signup."
- **Keywords**: mortgage calculator, loan amortization, Monte Carlo simulation, investment simulator, financial calculator, compound interest calculator, loan comparison tool, amortization schedule, extra payment calculator, 15 vs 30 year mortgage, portfolio simulator, retirement calculator, CD calculator, investment volatility, GBM simulation, free financial tools
- **Open Graph + Twitter Card**: Full social sharing preview with 1200×630 OG image showing Monte Carlo fan chart
- **Canonical URL**: `https://calcmonte.com/`

### Structured Data (JSON-LD) — All Active
1. **WebApplication** — Declares the app with feature list, free pricing, any OS
2. **FAQPage** — 4 Q&A pairs targeting long-tail queries (triggers FAQ rich results in Google — **CONFIRMED ACTIVE**)
3. **BreadcrumbList** — CalcMonte → Asset Simulator (**CONFIRMED ACTIVE**)

### Google Search Console Status
- **Property verified**: calcmonte.com (DNS verification via Cloudflare)
- **Page indexed**: ✅ URL is on Google
- **Sitemap**: ✅ Processed successfully, 1 page discovered
- **Rich results**: ✅ FAQ detected, ✅ Breadcrumbs detected
- **HTTPS**: ✅ Valid

### Crawlable Content
- Landing page: ~200 words of keyword-rich descriptive text
- Footer: Tool links, feature keywords, disclaimer (distinct from landing — not duplicative)
- Methodology page: ~1,500 words of unique technical content (extremely SEO-valuable for long-tail finance/quant queries)
- `<noscript>` fallback with full feature description
- `robots.txt` and `sitemap.xml` deployed

---

## Analytics

### GoatCounter (LIVE)
- **Dashboard**: https://calcmonte.goatcounter.com
- **Script**: `<script data-goatcounter="https://calcmonte.goatcounter.com/count" async src="//gc.zgo.at/count.js"></script>`
- **Tracks**: Page views, unique visitors, referrers, browsers, OS, countries, screen sizes
- **No cookies**, GDPR-compliant, no consent banner needed
- **Referrer tracking**: Append `?ref=source` to shared URLs (e.g., `calcmonte.com/?ref=hackernews`)

### Upgrade Path (if needed later)
- **Plausible**: $9/mo, more features, Google Search Console integration
- **PostHog**: Free up to 1M events/month, includes product analytics, session replay

---

## Monetization Strategy

### Affiliate Integration (Infrastructure Ready)
The footer contains a hidden `#affiliate-row` div (`display:none`) that can be activated when affiliate partnerships are established. The design pattern:
- **Contextual placement in results area** (not sidebar, not between charts)
- Style as a muted card consistent with the design language
- Example: "Current 30-year average: 6.8% — Compare offers ›"
- Footer "Partners" row for logos

To activate: Set `#affiliate-row` to `display:flex`, populate with styled anchor tags matching the design system (use `--border` for borders, `--text-dim` for text, `--cyan` for hover accents).

### Revenue Channels (Priority Order)
1. **Affiliate links** — Mortgage lenders ($50-200/lead via Commission Junction/ShareASale), high-yield savings (contextual in asset simulator), brokerage referrals
2. **Sponsored partnerships** — "Powered by [Lender]" after traffic established
3. **Freemium** — Basic free, advanced features (Monte Carlo, scenario comparison, PDF reports) behind $5-10/mo paywall
4. **Lead generation** — Capture intent via forms, sell leads at $20-75+ each (requires financial advertising compliance)

### Distribution Plan (Ready to Execute)
Social sharing posts have been drafted for all platforms with `?ref=` tracking URLs:

1. **Hacker News** (Show HN) — Technical angle, "single HTML file" architecture → `?ref=hackernews`
2. **Reddit r/personalfinance** — Practical angle, "best-to-worst-case outcomes" → `?ref=reddit`
3. **Reddit r/dataisbeautiful** — Fan chart screenshot as image post → `?ref=dataisbeautiful`
4. **Twitter/X** — 3-tweet thread with screenshots → `?ref=twitter`
5. **LinkedIn** — Professional angle → `?ref=linkedin`
6. **Product Hunt** — Tuesday/Wednesday launch

**Recommended posting order**: HN first (Tue-Thu morning EST), Reddit same day, Twitter with screenshots, LinkedIn a day later.

---

## OG Image

### Current Image
- **File**: `og-image.png` (1200×630, 41KB)
- **Content**: CalcMonte branding on left (title, tagline, three feature bullets, URL), real Monte Carlo fan chart on right (S&P 500 simulation, 200 paths, percentile bands from 5th to 95th, median line showing $10K → $23K over 10 years)
- **Generated via**: Python script using Pillow (`create_og.py`) with actual GBM simulation data (μ=10%, σ=18%)
- **Verified**: `https://calcmonte.com/og-image.png` loads correctly, OG meta tags confirmed via `curl`

### Social Platform Cache Notes
Social platforms aggressively cache OG previews. If you update the image:
- **Facebook**: Use developers.facebook.com/tools/debug → "Scrape Again"
- **LinkedIn**: Use linkedin.com/post-inspector → "Refresh"
- **Twitter**: Cache clears automatically within ~24 hours
- **Slack/iMessage**: Append `?v=2` to force refetch

---

## Performance Optimizations

### Current
- **KaTeX lazy-loaded**: CSS + JS (~300KB) only downloaded when user clicks Methodology tab
- **Debounced inputs**: Loan number fields use 150ms debounce; sliders fire immediately for responsive feel
- **Single-file architecture**: One HTTP request for the entire app (no waterfall)
- **No cookies**: No consent banner, no tracking overhead
- **Chart.js only CDN dependency on initial load**: ~200KB, cached after first visit

### Future Optimizations
- **Web Worker for Monte Carlo**: Move simulation to a worker thread to prevent UI freeze on 5,000+ paths
- **Chart update vs. destroy**: Currently destroys and recreates charts on every update. Chart.js supports in-place data updates with animation, which would be smoother

---

## Known Issues & Technical Debt

### Bugs
- None known as of launch. All identified bugs have been fixed.

### Potential Improvements (Post-Launch)
- **Sensitivity analysis panel**: Show delta impact of parameter changes (e.g., "each extra $100/mo saves X months")
- **Light theme toggle**: Some users prefer light mode; also better for screenshots/printing
- **Animated number transitions**: CountUp-style effect when stat cards update
- **PWA manifest**: Add `manifest.json` for "Add to Home Screen" with proper icons
- **A/B test landing page**: Test different headlines, CTA copy, visual previews
- **Historical benchmark comparison**: Rolling-window historical S&P 500 percentile bands overlaid on Monte Carlo fan chart (discussed and deferred — adds academic rigor but minimal practical value for target users)
- **Live market data ticker**: Considered and rejected — doesn't fit the "planning tool" identity, adds maintenance burden, cheapens the aesthetic
- **Mobile app (iOS/Android)**: PWA is the recommended first step (30 minutes). Capacitor wrapper for App Store distribution is ~1-2 days. Full React Native rewrite unnecessary for a financial calculator.

### Feature Decisions & Rationale
| Feature | Decision | Rationale |
|---------|----------|-----------|
| Historical benchmark overlay | Deferred | More academic than practical; rolling-window approach is statistically sound but adds complexity without helping users make decisions |
| Live price ticker | Rejected | Doesn't fit planning-tool identity; creates API dependency; every fintech dashboard already has this |
| Reference card for volatility | Implemented | Solves actual user friction ("what number do I put for volatility?") with zero infrastructure cost |
| Collapsible mobile sidebar | Removed | Caused iOS touch failures; replaced with simple stacked layout |
| KaTeX eager loading | Replaced with lazy | Saved ~300KB on initial page load for users who never visit methodology page |
| Stochastic home prices | Implemented (v4) | Opt-in σ slider for appreciating assets; 200 GBM paths; P(Underwater) is a killer differentiator vs every other mortgage calculator |
| Nav readability fix | Implemented (v4) | Users couldn't see inactive tabs; bumped color from text-dim to text-muted, added subtle border, raised secondary opacity |
| Built-in contact form | Rejected | Requires backend, breaks single-file architecture; mailto + GitHub Issues achieves same goal with zero infrastructure |

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| v1 (backup) | Feb 28, 2026 | Full working app: amortization, Monte Carlo, presets, methodology with KaTeX, URL sharing, localStorage, mobile responsive |
| v2 | Mar 1, 2026 | Mobile fixes: removed broken collapsible sidebar, fixed range slider touch targets (36px element/4px track/20px thumb), explicit chart container heights, cleared nested scroll containers, iOS-specific input fixes |
| v3 | Mar 1, 2026 | Launch polish: landing copy rewritten (outcomes over methods), SVG chart preview added, KaTeX lazy-loaded, debounce on inputs, footer restructured (affiliate-ready, non-duplicative), nav hierarchy (methodology/share demoted), orientation hints, historical reference card in asset simulator, brand renamed to CalcMonte, domain references updated, GoatCounter analytics added, SEO meta optimized, OG image created and deployed |
| v4 (current, live) | Mar 4, 2026 | Nav readability: inactive buttons brightened (#3f5170→#7e92b0), subtle border on all tabs, secondary nav opacity 0.5→0.8. Stochastic home price simulation: GBM Monte Carlo (200 paths) on loan asset overlay with σ slider, fan chart percentile bands, P(Underwater) probability stat. Footer: added Feedback column with mailto + GitHub Issues links. |

### Reverting
`loan-analytics-backup-v1.html` in `~/Desktop/LoanAnalytics/` is the v1 backup with all core features but without mobile fixes, polish, or deployment configuration. Rename to `index.html` to use.

---

## Quick Reference: How to Modify

### Change a preset
Search for `LOAN_PRESETS` or `ASSET_PRESETS` in the JS section. Each is a plain object with field names matching the input IDs.

### Add a new chart
1. Add `<canvas id="X_chartName" height="NNN"></canvas>` inside a `<div class="chart-container">`
2. In the module's `renderTab()` function, create a new `Chart(ctx, config)` — follow existing chart patterns
3. Store the chart instance in `ch.name` to destroy on re-render

### Add an affiliate banner
Show the hidden `#affiliate-row` by setting `display:flex`, then populate with styled anchor tags matching the design system (use `--border` for borders, `--text-dim` for text, `--cyan` for hover accents).

### Change domain references
Search and replace `calcmonte.com` — there are exactly 8 occurrences (canonical URL, OG tags, Twitter cards, and all three JSON-LD blocks).

### Change analytics
The GoatCounter script is just before `</body>`. Replace with any other analytics provider (Plausible, PostHog, Google Analytics) by swapping the script tag.

### Add a new page
1. Add `<div class="page" id="page-name">...</div>` in the HTML
2. Add nav button: `<button class="nav-btn" data-page="name" onclick="switchPage('name')">...</button>`
3. `switchPage()` handles the rest automatically (hides landing, toggles active states)

### Update and redeploy
1. Edit `~/Desktop/LoanAnalytics/deploy/index.html`
2. Cloudflare → Workers & Pages → `calcmonte` → new deployment → upload `deploy` folder
3. Live in seconds
