# Algorithmic Trading with Machine Learning

A collection of quantitative trading strategy examples implemented as Jupyter
notebooks in Python. Each notebook is a self-contained walk-through of a
different strategy: feature engineering, signal generation, portfolio
construction, and performance comparison against a benchmark.

The examples are adapted from the tutorial notebook
[`Luchkata/Algorithmic_Trading_Machine_Learning`](https://github.com/Luchkata/Algorithmic_Trading_Machine_Learning),
split into focused notebooks and updated to run on current library versions
(pandas 3.x, yfinance 1.x, etc.).

## Notebooks

| Notebook | Strategy | Benchmark | Data source |
| --- | --- | --- | --- |
| [`01_unsupervised_learning_trading_strategy.ipynb`](01_unsupervised_learning_trading_strategy.ipynb) | Unsupervised learning: cluster S&P 500 stocks monthly by technical/factor features, pick a cluster, and optimise a max-Sharpe portfolio with PyPortfolioOpt | SPY buy & hold | Wikipedia S&P 500 list, `yfinance`, Fama-French 5 factors |
| [`02_twitter_sentiment_investing_strategy.ipynb`](02_twitter_sentiment_investing_strategy.ipynb) | Twitter engagement ratio: rank stocks monthly by social engagement, hold the top 5 with equal weights | QQQ Nasdaq-100 | `sentiment_data.csv` (bundled), `yfinance` |
| [`03_intraday_strategy_garch_model.ipynb`](03_intraday_strategy_garch_model.ipynb) | Intraday GARCH: forecast next-day variance with a rolling GARCH(1,3), combine a daily volatility-premium signal with an intraday RSI/Bollinger signal | — (strategy cumulative return) | bundled simulated daily and 5-minute data |

### 01 — Unsupervised Learning Trading Strategy

- Download S&P 500 prices and compute features and technical indicators
  (Garman-Klass volatility, RSI, Bollinger Bands, ATR, MACD, dollar volume).
- Aggregate to monthly level and filter the 150 most liquid stocks per month.
- Calculate monthly returns over multiple horizons (1/2/3/6/9/12 months).
- Download Fama-French factors and estimate rolling factor betas
  (`RollingOLS`).
- Fit a K-Means model each month to group similar assets, using pre-defined
  centroids, and select the momentum cluster.
- Form an efficient-frontier max-Sharpe portfolio, falling back to
  equal weights when optimisation fails.
- Visualise strategy returns vs. SPY.

### 02 — Twitter Sentiment Investing Strategy

- Load the Twitter sentiment dataset and compute an engagement ratio
  (`twitterComments / twitterLikes`), filtering out low-activity stocks.
- Aggregate monthly and rank stocks by engagement cross-sectionally.
- Select the top 5 stocks each month and form an equal-weighted,
  monthly-rebalanced portfolio.
- Compare strategy returns against QQQ.

### 03 — Intraday Strategy Using GARCH Model

- Load simulated daily and 5-minute data.
- Fit a rolling GARCH(1,3) model over a 6-month window to forecast
  next-day variance.
- Calculate a prediction premium and derive a daily signal from it.
- Merge with intraday data, compute RSI and Bollinger Band indicators,
  and generate an intraday signal.
- Enter positions on the signal and hold until the end of the day.
- Aggregate to daily strategy returns and plot cumulative performance.

## Datasets

| File | Size | Rows | Description |
| --- | --- | --- | --- |
| `data/sentiment_data.csv` | 1.6 MB | 27,235 | Daily Twitter metrics per stock symbol (2021-11 → 2023-01) |
| `data/simulated_daily_data.csv` | 270 KB | 3,289 | Simulated daily OHLCV bars |
| `data/simulated_5min_data.csv` | 10 MB | 177,877 | Simulated 5-minute OHLCV bars |

The bundled datasets come from the original repository and are committed under
`data/` so the notebooks run out of the box. Notebooks 01 and 02 additionally
download price and factor data live at run time.

## Requirements

- Python `>=3.12,<3.14` (the upper bound is required because `pandas-ta`
  depends on `numba`, which pins `numpy<2.3` and does not yet support Python
  3.14).
- Dependencies are declared in [`pyproject.toml`](pyproject.toml) and managed
  with [`uv`](https://docs.astral.sh/uv/). Key libraries include `pandas`,
  `numpy`, `scikit-learn`, `statsmodels`, `arch`, `pandas-ta`,
  `PyPortfolioOpt`, `pandas-datareader`, `yfinance`, and `matplotlib`.

## Setup

```sh
# Install dependencies into a local virtual environment (.venv)
uv sync
```

## Running the notebooks

Open a notebook in Jupyter or an editor that supports `.ipynb` and run the
cells top to bottom:

```sh
uv run jupyter lab
```

Notebooks 01 and 02 require network access to download prices and factors.

## Notes on library-version adaptations

Because the original examples were written against older releases, the
notebooks include a few compatibility adaptations:

- **pandas 3.x** — monthly resampling uses the `'ME'` alias (`'M'` was
  removed), and `pd.read_html` is fed via `requests` + `io.StringIO`.
- **yfinance 1.x** — `auto_adjust=True` is now the default, so `Close` is the
  adjusted price and the separate `Adj Close` column is generally unavailable.
  The notebooks select the price column that actually contains data and handle
  the MultiIndex columns that `yfinance` returns (including for single tickers).
- **Portfolio loop** — because `yfinance` labels the ticker column level
  (`Ticker`), the returns frame and weight frame are given explicit axis names
  so the `stack` / `reset_index` / `set_index` reshaping is stable.

### A note on reproducible outputs

Notebooks 01 and 02 depend on live data, so their results are **not**
byte-reproducible. The S&P 500 constituent list, `yfinance` price history, and
Fama-French factor data are all updated over time, and the example outputs
shown in the original tutorial are snapshots from ~2023. Results will differ;
the notebooks demonstrate the *method*, not a fixed set of numbers.

## Project structure

```
.
├── 01_unsupervised_learning_trading_strategy.ipynb
├── 02_twitter_sentiment_investing_strategy.ipynb
├── 03_intraday_strategy_garch_model.ipynb
├── data/
│   ├── sentiment_data.csv
│   ├── simulated_daily_data.csv
│   └── simulated_5min_data.csv
├── pyproject.toml
├── uv.lock
└── README.md
```

## Disclaimer

This project is for educational purposes only. It is not financial advice, and
the strategies are simplified illustrations that are not intended for live
trading.
