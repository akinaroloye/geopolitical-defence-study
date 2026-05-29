# Geopolitical Events and Defence Equity Returns

A short data study looking at whether spikes in geopolitical conflict, measured via GDELT, line up with unusual returns in defence sector equities.

## Motivation

Defence stocks are often said to benefit from geopolitical tension, but it's rarely tested directly. This is an attempt to put numbers on it using only free public data.

## Data Sources

| Source | What it provides |
|--------|-----------------|
| [GDELT 2.0 Events](https://www.gdeltproject.org/) | Daily CSV exports of global events, coded by actor, event type, and Goldstein scale |
| [yfinance](https://github.com/ranaroussi/yfinance) | Adjusted daily OHLCV prices for ETFs and individual equities |

Primary instruments: `ITA` (iShares U.S. Aerospace & Defense ETF) as the sector proxy, `SPY` as market benchmark. Peer stocks: LMT, RTX, NOC, GD, LHX, HWM.

## Methodology

1. Pull GDELT event data for the past year and filter to countries where conflict is likely to move defence markets: Russia, Ukraine, Israel, Iran, China, North Korea, Saudi Arabia, Syria. Compute a daily net-conflict signal from this and cache the raw downloads locally.

2. Normalise the signal over a 30-day rolling window into Z-scores. Flag days above 1.5 and 2.0 as conflict spikes.

3. For each spike, extract ITA excess returns (ITA minus SPY) from 5 days before to 10 days after. Average across events to produce a cumulative abnormal return (CAR) path.

4. Repeat for each of the six individual defence names to check whether sector-level effects hold at the stock level.

5. Fit a rolling 120-day OLS market model before each event window and compute out-of-sample abnormal returns over the window.

6. Run 5,000 bootstrap reshufflings of event dates to build a null distribution for CAR and get an empirical p-value.

7. Regress ITA excess return directly on the rolling Z-score and lagged values as a robustness check outside the event-study framework.

## Project Structure

```
geopolitical-defence-study/
├── geopolitical_defence_study.ipynb   # main analysis notebook
├── requirements.txt
├── .gitignore                         # excludes gdelt_events_cache.csv
└── README.md
```

The GDELT cache file (`gdelt_events_cache.csv`) is excluded from version control. It regenerates automatically on first run.

## Setup

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook geopolitical_defence_study.ipynb
```

Run all cells top-to-bottom. The first run fetches roughly 365 days of GDELT data (one HTTP request per day) and caches it locally. Subsequent runs load from the cache instantly.

## Key Findings

The event study shows a small positive bump in ITA excess returns 1-3 days after a conflict spike, which fades within the 10-day window. Bootstrap p-values come out around 0.05-0.15 depending on the threshold used, so the effect shows up but isn't particularly strong. The OLS regression has a small positive coefficient on the contemporaneous Z-score.

This is exploratory. GDELT coding is automated from news text so the signal is noisy, and the sample only covers one year. I wouldn't read too much into the specific numbers.

## What I learned

GDELT is a useful free data source but requires some care since the event coding is automated and the signal is inherently noisy. Designing the event study from scratch was a good exercise in thinking through estimation windows and how to handle overlapping events without double-counting. Bootstrap testing made more sense here than parametric t-tests given how few events there are. Structuring the notebook with a single config block at the top and cached I/O made re-running with different parameters much less painful.
