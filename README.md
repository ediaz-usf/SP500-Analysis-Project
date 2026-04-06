## Regime-Aware Sector Rotation Model

This notebook builds a regime-aware sector rotation model using 25 years of S&P 500 data to answer a practical question: **which sectors should you rotate into given current market conditions?**

The core hypothesis is that sector performance is not random — it depends on the broader market environment. By classifying the market into distinct regimes and measuring historical sector returns within each, we can make informed, data-driven rotation decisions.

### What it does

1. **Setup** — imports all required libraries and initializes the environment.

2. **Data loading & preparation** — loads S&P 500 stock price history alongside company metadata (sector, industry, exchange). Stocks are merged with their sector labels and sorted chronologically per symbol.

3. **Feature engineering** — computes a rich set of per-stock technical indicators:
   - Multi-horizon returns: 1-day, 5-day, 21-day, 63-day
   - Simple moving averages: SMA-10, SMA-21, SMA-50, SMA-200
   - Price-to-SMA ratios (50-day and 200-day)
   - Momentum: 21-day and 63-day
   - Volatility: rolling 21-day and 63-day standard deviation of daily returns
   - Volume ratio: current volume vs. 21-day average
   - Forward returns: 5-day, 21-day, and 63-day (used as prediction targets)

4. **Market-level aggregation** — collapses stock-level data into a single daily market table by averaging key metrics across all constituents, including breadth indicators that measure how broadly a market move is distributed across stocks rather than concentrated in a handful of large-caps.

5. **Regime classification** — labels each trading day with two independent regimes:
   - **Trend regime** (bull / sideways / bear): based on the 63-day average market return — above +8% is bull, below -8% is bear, otherwise sideways
   - **Volatility regime** (low / normal / high): based on 21-day average volatility relative to its historical 25th and 75th percentiles
   - These combine into **9 possible regimes** (e.g., `bull_low_vol`, `bear_high_vol`)

6. **Regime validation** — validates the volatility regime labels against the CBOE VIX, the market's independently established fear index. Average VIX is computed per regime alongside mean absolute deviation and percentage deviation, confirming that days labeled `high_vol` correspond to meaningfully higher VIX readings than `normal_vol` or `low_vol` days.

7. **Regime timeline visualization** — overlays classified regimes as colored bands on a cumulative market index chart, with a zoom into the 2008 financial crisis as a representative example of the classifier capturing real market conditions.

8. **Sector performance by regime** — for each of the 9 regimes, computes per-sector statistics across all historical observations: average and median 21-day and 63-day forward returns, hit rate (% of positive outcomes), standard deviation, and a Sharpe-like return-to-risk ratio. Results are visualized as a heatmap.

9. **Current regime & recommendation** — detects the most recent market regime and surfaces the top historically strongest sectors for that regime, with expected 21-day return, hit rate, and Sharpe-like ratio.

10. **Backtesting** — simulates a monthly rebalancing strategy that rotates into the top N historically strongest sectors for the current regime, using only data available at each point in time to prevent look-ahead bias. Key features:
    - **Configurable sector count** — `get_backtest_df_per_n_top_sectors(n)` lets you test how performance changes as you vary the number of sectors held, exploring the concentration vs. diversification tradeoff
    - **Windowed analysis** — `plot_backtest_window(results_df, start, end)` lets you zoom into any date range with automatically rebased growth curves and recomputed metrics, making it easy to isolate performance during specific market events like the 2008 financial crisis or COVID-19
    - **Performance metrics** — cumulative return, annualized Sharpe ratio, and maximum drawdown reported for both strategy and equal-weight benchmark

### Data
Two CSV files: `sp500_stocks.csv` (daily OHLCV price data) and `sp500_companies.csv` (company metadata including sector and date added to the index).

### Stack
Python 3 · pandas · NumPy · Matplotlib · Seaborn · yfinance