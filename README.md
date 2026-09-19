# GreenCardPulse

**AI-Powered US Visa Bulletin & Priority Date Predictor**

GreenCardPulse is a static, client-side web app that helps employment-based (EB) and family-based (FB) green card applicants understand where they stand in the U.S. immigration queue. It compares a user's priority date against the official Department of State (DOS) Visa Bulletin cutoffs and generates a forecast of when their priority date is likely to become current, using a historical-trend simulation engine.

## Pages

| File | Purpose |
|---|---|
| `index.html` | The main calculator — profile inputs, results, projection chart, scenario comparison, and data inspector |
| `faq.html` | A static FAQ page explaining priority dates, per-country limits, cross-chargeability, and other Visa Bulletin concepts |

There is no build step, bundler, or backend — both pages are self-contained HTML files that can be opened directly in a browser or served from any static host.

## Features

- **Immigration profile form** — select visa stream (Employment or Family-based), specific category (EB-1 through EB-5 with sub-categories, FB-1 through FB-4), country of chargeability (India, China, Mexico, Philippines, Rest of World), priority date, and current processing stage (PERM pending, I-140 pending, I-485 filed, consular processing).
- **Spousal cross-chargeability toggle** — models the effect of claiming a spouse's (less-backlogged) country of birth under INA § 202(b).
- **Movement pace scenarios** — Conservative, Baseline (historical trend), and Optimistic projections.
- **Results dashboard** — hero card with estimated approval date and remaining wait, a 4-step immigration roadmap progress bar, and key metrics (active bulletin cutoff, priority date queue length, estimated I-485 filing date).
- **Priority Date Catch-up Projection chart** (Chart.js) — visualizes the simulated bulletin cutoff trend against the user's priority date over time.
- **Pace scenario comparison table** — side-by-side estimates across conservative/baseline/optimistic scenarios.
- **Data Inspector modal** — full table of current EB and FB final-action/filing cutoffs by category and country.
- **Presets** — one-click sample profiles (e.g., EB-2 India, EB-3 India, EB-2 China, EB-2 ROW, FB-4 India) for quickly exploring typical scenarios.
- **Bulletin verification / "Sync Latest Bulletin"** — attempts a live fetch of the official DOS Visa Bulletin page to confirm the embedded dataset is current, with a clear cached-data fallback if the network request fails or is blocked (no API key is used or required — this is intentionally not a live AI-grounded fetch, despite the "Sync" naming).
- **FAQ page** — plain-language explanations of priority dates, per-country backlogs, filing-chart types, cross-chargeability, and a disclaimer that the tool is not legal advice.

## Data Model

All data lives inline in `<script>` blocks in `index.html`:

- **`VISA_BULLETIN_DATA`** — official **Final Action Dates** by category and country (the September 2026 bulletin baseline).
- **`VISA_FILING_DATA`** — official **Dates for Filing** by category and country, from the same bulletin.
- **`FORECAST_DATA`** — a small set of separately labeled, non-official forward-looking estimates (e.g., for the following month), each with a source link and confidence note — kept explicitly distinct from the official DOS data.
- **`HISTORICAL_CUTOFFS`** / **`RECENT_ANCHORS`** — a seeded, hand-curated series of past monthly cutoffs per category/country combination, used to derive movement statistics (median/quartile monthly advancement, retrogression rate, hold rate).
- **`PLANNING_PRIORS`** — fallback low/base/high annual movement assumptions per category, used when historical data is thin.

### Forecast engine

The projection logic (`analyzeHistory`, `simulateOnePath`, `simulateForecast`) runs a **deterministic Monte Carlo simulation** (300 seeded paths per calculation) that:
1. Derives monthly movement statistics from the embedded historical series.
2. Applies a scenario multiplier (conservative / baseline / fast).
3. Simulates month-by-month cutoff advancement — including retrogression and "hold" months — using a deterministic pseudo-random noise function (so results are reproducible in a static file with no server).
4. Reports the 20th/50th/80th percentile outcome dates as the projected range.

The engine explicitly avoids fabricating data for category/country pairs it has no history or forecast for — it clearly states when no model output is available rather than guessing.

## Tech Stack

- Plain HTML/CSS/JavaScript (no framework, no build tooling)
- [Tailwind CSS](https://tailwindcss.com/) via CDN (`cdn.tailwindcss.com`)
- [Chart.js](https://www.chartjs.org/) via CDN for the projection chart
- [Font Awesome 6](https://fontawesome.com/) via CDN for icons
- Google Fonts (Inter)

## Running Locally

No installation required — simply open `index.html` in a browser, or serve the folder with any static file server, e.g.:

```bash
npx serve .
# or
python3 -m http.server 8000
```

Then visit `index.html` (calculator) and `faq.html` (FAQ) in the browser.

## Data Sources & Disclaimers

- Official cutoff data is sourced from the [U.S. Department of State Visa Bulletin](https://travel.state.gov/content/travel/en/legal/visa-law0/visa-bulletin.html).
- USCIS filing-chart guidance: [uscis.gov/visabulletininfo](https://www.uscis.gov/visabulletininfo).
- Forward-looking forecast figures are model estimates from a third-party source, kept visually and structurally separate from official data.
- The app and FAQ page both state clearly that **forecasts are not guarantees** and that users should verify against the latest official bulletin and consult an immigration attorney for case-specific guidance.

## Possible Next Steps

- Automate refreshing `VISA_BULLETIN_DATA` / `VISA_FILING_DATA` each month (currently a manual/hardcoded update per bulletin release).
- Replace the best-effort `fetch()`-based "Sync Latest Bulletin" check with a real backend/proxy if live verification or AI-grounded lookups are desired (a static file can't safely hold an API key).
- Expand historical series coverage for more category/country pairs to improve forecast confidence.
