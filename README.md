# CalcMonte

**The Financial Calculator That Shows What Others Won't**

[calcmonte.com](https://calcmonte.com) — Free, no signup, 100% client-side.

## What It Does

Most financial calculators give you one number. CalcMonte shows you the full range of what could happen.

**Loan Calculator** — Compare up to 3 loan scenarios side by side. See how extra payments, different terms, and rates affect your total cost. Supports 6 payment frequencies with asset appreciation/depreciation overlay.

**Investment Simulator** — Monte Carlo simulation using Geometric Brownian Motion. See percentile bands from 5th to 95th instead of a single projected line. Understand the realistic spread of outcomes before making decisions.

## Architecture

Single self-contained HTML file (~130KB). No framework, no build step, no backend.

- **Charts**: Chart.js (CDN)
- **Math rendering**: KaTeX (lazy-loaded CDN)
- **Fonts**: DM Serif Display + IBM Plex Mono (Google Fonts)
- **Analytics**: GoatCounter (privacy-friendly, no cookies)
- **Hosting**: Cloudflare Workers (free tier)

## Run Locally

Just open `index.html` in a browser. Everything runs client-side.

For a local server (needed for some mobile testing):

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

## Deploy

Upload the contents of this repo to Cloudflare Workers & Pages:

1. Go to [dash.cloudflare.com](https://dash.cloudflare.com) → Workers & Pages → `calcmonte`
2. Create new deployment → upload project folder
3. Live in seconds

## Files

```
index.html      ← The entire application
og-image.png    ← Social sharing preview (1200×630)
robots.txt      ← Search engine directives
sitemap.xml     ← URL listing for Google
HANDOFF.md      ← Comprehensive technical documentation
```

## Math

The investment simulator uses GBM with Itô correction:

```
S(t+Δt) = S(t) · exp[(μ - σ²/2)Δt + σ√Δt · Z]
```

Where Z ~ N(0,1) via Box-Muller transform. The Methodology tab on the site documents all formulas.

## License

MIT
