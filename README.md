# Regime-Aware Sector Rotation Model

This notebook builds a regime-aware sector rotation model using 25 years of S&P 500 data to answer a practical question: **which sectors should you rotate into given current market conditions?**

The core hypothesis is that sector performance is not random — it depends on the broader market environment. By classifying the market into distinct regimes and measuring historical sector returns within each, we can make informed, data-driven rotation decisions.

## What it does

1. **Data preparation** — loads S&P 500 stock price history alongside company metadata (sector, industry, exchange). Stocks are merged with their sector labels and sorted chronologically per symbol.

2. **Feature engineering** — computes a rich set of per-stock technical indicators:
   - Multi-horizon returns: 1-day, 5-day, 21-day, 63-day
   - Simple moving averages: SMA 10, 21, 50, 200
   - Price-to-SMA ratios (50-day and 200-day)
   - Momentum: 21-day and 63-day
   - Volatility: rolling 21-day and 63-day standard deviation of daily returns
   - Volume ratio: current volume vs. 21-day average
   - Forward returns: 5-day, 21-day, and 63-day (used as prediction targets)

3. **Market regime classification** — aggregates stock-level data into a daily market table, then labels each day with two independent regimes:
   - **Trend regime** (bull / sideways / bear): based on the 63-day average market return — above +8% is bull, below -8% is bear, otherwise sideways
   - **Volatility regime** (low / normal / high): based on 21-day average volatility relative to its historical 25th and 75th percentiles
   - These combine into **9 possible regimes** (e.g., `bull_low_vol`, `bear_high_vol`)

4. **Sector performance analysis** — for each of the 9 regimes, computes per-sector statistics across all historical observations: average and median 21-day and 63-day forward returns, hit rate (% of positive outcomes), standard deviation, and a Sharpe-like return-to-risk ratio.

5. **Current recommendation** — detects the most recent market regime and surfaces the top 3 historically strongest sectors for that regime, with expected 21-day return and hit rate.

## Data
Two CSV files: `sp500_stocks.csv` (daily OHLCV price data) and `sp500_companies.csv` (company metadata including sector and date added to the index).

## Stack
Python 3 · pandas · numpy · matplotlib · seaborn
