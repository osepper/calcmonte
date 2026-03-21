# CalcMonte

A financial calculator combining loan amortization analytics with Monte Carlo investment simulation.

**Live:** [calcmonte.com](https://calcmonte.com)

## Features

### Loan Calculator
- Amortization with 6 payment frequencies
- Extra payment modeling with interest savings
- Side-by-side scenario comparison (up to 3)
- Stochastic home price simulation (GBM, 200 paths, underwater risk)
- Rent vs Buy comparison with paired Monte Carlo
- Full housing costs (property tax, insurance, maintenance, PMI, HOA)
- Refinance calculator with break-even analysis
- Decision summary cards on every tab

### Investment Simulator
- Monte Carlo simulation with GBM and Itô correction
- Percentile bands (5th–95th) fan charts
- Clickable assumption presets (S&P 500, Bonds, NASDAQ, Bitcoin, CDs)
- Goal-seeking mode: find required monthly contribution for any target
- One-time scheduled events (lump sum add/withdraw)
- Final value distribution histogram and probability analysis

### Design
- Single self-contained HTML file (~177KB)
- No framework, no build step, no backend
- Dark theme, responsive layout
- Chart.js for visualization, KaTeX for math rendering
- Full methodology page documenting all formulas
- Shareable URL state, localStorage persistence
- Privacy-friendly (GoatCounter analytics, no cookies, no tracking)

## Tech Stack

- HTML/CSS/JS (single file, no build)
- Chart.js 4.4.1 (CDN)
- KaTeX 0.16.9 (lazy-loaded CDN)
- Google Fonts (IBM Plex Mono, DM Serif Display)
- Cloudflare Workers (hosting, free tier)
- GoatCounter (analytics)

## Feedback

feedback@calcmonte.com

## License

MIT
