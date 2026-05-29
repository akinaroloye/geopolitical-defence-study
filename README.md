# Geopolitical Events and Defence Equity Returns

An exploratory data study examining whether spikes in geopolitical conflict activity — measured via the GDELT 2.0 Events database — are associated with abnormal returns in defence sector equities.

## Motivation

Defence stocks are often cited as beneficiaries of geopolitical tension, but that relationship is rarely quantified systematically. This project attempts to do exactly that: ingest a publicly available conflict/cooperation signal, align it with daily market data, and test whether statistically meaningful price responses exist around high-tension periods.

## Data Sources

All data used in this project is freely available with no authentication required.

| Source | What it provides |
|--------|-----------------|
| [GDELT 2.0 Events](https://www.gdeltproject.org/) | Daily CSV exports of global events, coded by actor, event type, and Goldstein scale |
| [yfinance](https://github.com/ranaroussi/yfinance) | Adjusted daily OHLCV prices for ETFs and individual equities |

Primary instruments: `ITA` (iShares U.S. Aerospace & Defense ETF) as the sector proxy, `SPY` as the market benchmark. Peer stocks: LMT, RTX, NOC, GD, LHX, HWM.

## Methodology

1. **GDELT signal construction** — filter events where at least one actor is a country of interest (Russia, Ukraine, Israel, Iran, China, North Korea, Saudi Arabia, Syria). Compute daily conflict count, cooperation count, mean Goldstein scale, and a net-conflict ratio. Cache raw GDELT downloads locally to avoid re-fetching.

2. **Rolling Z-scores** — normalise the net-conflict signal over a 30-day rolling window. Flag spikes at 1.5σ and 2.0σ thresholds.

3. **Event study** — for each spike day, extract a [−5, +10] day window of ITA excess returns (ITA return minus SPY return). Average across all events to produce a cumulative abnormal return (CAR) path.

4. **Peer analysis** — repeat the event study for each of the six individual defence names to check whether sector-level effects are consistent across constituents.

5. **Market-model abnormal returns** — fit a rolling 120-day OLS market model (r_stock ~ α + β·r_SPY) on pre-event data, then compute out-of-sample abnormal returns during the event window.

6. **Bootstrap significance** — 5,000 random reshufflings of event dates to build a null distribution for CAR, yielding an empirical p-value.

7. **OLS robustness** — regress ITA excess return on the rolling Z-score and lagged values to check whether the relationship holds outside the event-study framework.

## Project Structure

```
geopolitical-defence-study/
├── geopolitical_defence_study.ipynb   # main analysis notebook
├── requirements.txt
├── .gitignore                         # excludes gdelt_events_cache.csv
└── README.md
```

The GDELT cache file (`gdelt_events_cache.csv`) is excluded from version control — it regenerates automatically on first run.

## Setup

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook geopolitical_defence_study.ipynb
```

Run all cells top-to-bottom. The first run fetches ~365 days of GDELT data (one HTTP request per day) and caches the result; subsequent runs load from the local cache instantly.

## Key Findings

Results vary with the lookback period and threshold chosen, as expected for a noisy signal. The event study consistently shows a short-lived positive impulse in ITA excess returns in the 1–3 days following a conflict spike, which fades within the 10-day window. Bootstrap p-values hover around 0.05–0.15 depending on the threshold, suggesting the effect is present but not overwhelmingly strong at daily frequency. The OLS regression confirms a small positive coefficient on the contemporaneous Z-score.

This is an exploratory study, not a trading signal. Sample size is limited to the available lookback, and GDELT coding is noisy by design (it processes news text automatically). The findings are consistent with the literature on geopolitical risk premia but should not be over-interpreted.

## What I Learned

- Working with the GDELT data format: fixed-column TSV layout, Goldstein scale interpretation, CAMEO event codes
- Designing an event study from scratch: choosing the estimation window, handling overlapping events, averaging abnormal returns
- Bootstrap hypothesis testing as an alternative to parametric t-tests when normality is uncertain
- How to structure a quantitative research notebook for reproducibility: config block at the top, cached I/O, clear section separation
