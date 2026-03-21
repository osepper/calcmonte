# CalcMonte — Project Handoff

## Executive Summary

**CalcMonte** (https://calcmonte.com) is a live, single-file financial calculator (~177KB, ~1,800 lines) combining loan amortization analytics with Monte Carlo investment simulation. Dark theme, responsive, no framework, no backend. Everything runs client-side.

**Current status: LIVE.** V2 features deployed March 21, 2026. Feedback via feedback@calcmonte.com (Cloudflare Email Routing → Gmail). GitHub repo is private.

## Infrastructure

| Layer | Service | Cost |
|-------|---------|------|
| Hosting | Cloudflare Workers (Pages) | Free |
| Domain | calcmonte.com via Cloudflare Registrar (expires Mar 2027) | ~$10/yr |
| Analytics | GoatCounter (calcmonte.goatcounter.com) | Free |
| Search | Google Search Console (indexed, FAQ + Breadcrumb rich results) | Free |
| Email | Cloudflare Email Routing: feedback@calcmonte.com → osepper@gmail.com | Free |

## Deployment

```bash
cd ~/Projects/calcmonte
mkdir -p deploy
cp index.html og-image.png robots.txt sitemap.xml deploy/
# Upload deploy/ to Cloudflare → Workers & Pages → calcmonte
rm -rf deploy
git add . && git commit -m "description" && git push
```

## Git

- Repo: ~/Projects/calcmonte, branch main
- Remote: github.com/osepper/calcmonte (PRIVATE)
- Cloudflare account: osepper@gmail.com
- Tag: v1-live-ready (V1 credibility patch checkpoint)

## Feature Summary

### Loan Calculator
- Amortization with 6 payment frequencies, extra payments, down payment tracking
- Scenario comparison (save up to 3, side-by-side table + charts)
- Stochastic home price simulation (GBM, 200 paths, fan chart, Underwater Risk %)
- Home appreciation presets: Pessimistic (1%/12σ), US Average (3.5%/8σ), Optimistic (6%/10σ)
- Rent vs Buy comparison: paired Monte Carlo (buy equity fan vs rent portfolio fan), Buy Wins %, Crossover Year, monthly cost chart, verdict summary
- Full housing costs: itemized Property Tax, Insurance, Maintenance (%/yr) + PMI, HOA ($/mo). True Monthly stat card.
- Refinance calculator: new tab, break-even analysis, cumulative savings chart, comparison table
- Decision summary cards on Overview, Breakdown, and Asset tabs (try-catch wrapped)
- 6 loan presets (30yr/15yr mortgage, 5yr/3yr auto, student, personal)

### Investment Simulator
- Deterministic compounding (6 modes) or stochastic GBM with Itô correction
- Configurable paths (50–1000), contributions/withdrawals, one-time events
- Fan chart (5th–95th percentile), histogram, percentile table, probability analysis
- Clickable reference card: S&P 500 (10%/18σ), Bond (5%/6σ), NASDAQ (12%/22σ), Bitcoin (60%/65σ), CD (4.5%/0σ)
- Goal-seeking mode: bisection search to find required monthly contribution for target amount at given confidence level
- V1 fix: compounding selector disabled when vol > 0 (stochastic uses monthly-step GBM)
- 6 investment presets

### V1 Credibility Patch (Mar 5, 2026)
- APY → "Expected Annual Return" (user-facing only, internal vars still use apy)
- Stochastic/deterministic mode clarified with UI hints
- Cash flow timing approximation disclosed
- Loan rounded-month display caveat added
- Summary language softened ("approximately")
- Methodology page strengthened

## Technical Debt

- **Internal `apy` variable names**: User-facing changed, internal deferred to modularization
- **Loan time axis rounded months**: Non-monthly frequencies use Math.round(); disclosed, architectural fix deferred
- **File size ~177KB**: Modularization recommended before adding more features (target: 4 JS files)
- **No automated tests**: Add validation fixtures during modularization

## Version History

| Version | Date | Key Changes |
|---------|------|-------------|
| v1-v3 | Feb 28–Mar 1 | Core app, mobile fixes, launch polish, SEO, GoatCounter |
| v4 | Mar 4 | Stochastic home prices, nav readability, footer feedback |
| V1 patch | Mar 5 | Credibility: APY→Expected Annual Return, disclosures, methodology |
| V2 | Mar 21 | Rent vs Buy, full housing costs, decision cards, clickable presets, refinance calculator, goal-seeking mode |

## V2 Roadmap (Remaining Priorities)

| Priority | Feature | Status |
|----------|---------|--------|
| 1 | Rent vs Buy | ✅ Done |
| 2 | Full Housing Costs | ✅ Done |
| 3 | Decision Summary Cards | ✅ Done |
| 4 | Clickable Assumption Presets | ✅ Done |
| 5 | Refinance Calculator | ✅ Done |
| 6 | Goal-Seeking Mode | ✅ Done |
| 7 | Modularize Codebase | Next — split into 4 JS files |
| 8 | Retirement Drawdown Mode | After modularization |
| 9 | Real vs Nominal Toggle | After modularization |
| 10 | Monetization Placements | After sustained traffic |

See CALCMONTE_V2_ROADMAP.md for full details on each priority.

## Key File Locations

| File | Purpose |
|------|---------|
| index.html | The entire application |
| og-image.png | Social sharing preview (1200×630) |
| robots.txt, sitemap.xml | SEO |
| HANDOFF.md | This document |
| CALCMONTE_V2_ROADMAP.md | Detailed V2 roadmap |
| README.md | GitHub readme |
| LICENSE | MIT |
