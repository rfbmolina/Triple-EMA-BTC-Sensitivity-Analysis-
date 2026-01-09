# Triple EMA Crossover (BTC-USD) — Backtest and Sensitivity Analysis

A signal-based backtest is implemented for BTC-USD using a triple exponential moving average (EMA) crossover rule. The repository has been structured to emphasize diagnostic analysis and parameter sensitivity rather than a single optimized backtest. The primary objective has been to evaluate whether a simple trend signal can improve drawdown behavior and risk-adjusted performance relative to buy-and-hold, and whether any observed performance is stable across reasonable parameter neighborhoods.

## Strategy overview

A long-only exposure regime is inferred from three EMAs computed on the daily close. The strategy is traded as a binary allocation between BTC exposure and cash.

- A long position is entered when a bullish crossover is detected under the specified crossover logic.
- A long position is exited when a bearish crossover is detected under the specified crossover logic.
- Signals are shifted forward by one trading day so that crossovers observed at time *t* are assumed to be executed at time *t+1*. This is intended to avoid same-bar execution and implicit look-ahead.

## Data

Daily BTC-USD price data are downloaded using `yfinance`. The analysis is performed on the close series by default. The start date and the evaluation window are parameterized in the notebook.

## Indicator construction

Three EMAs are computed using exponential weighting on the close series. The EMA window lengths are treated as hyperparameters and are varied during the grid search and local sensitivity analysis.

## Backtest design

Backtests are executed with a signal-to-portfolio workflow in which entry and exit boolean arrays are converted into a trades and equity curve.

- The initial portfolio value is set to $100,000.
- Transaction costs are modeled as constant proportional fee and slippage rates.
- Positions are assumed to be fully invested when long and fully in cash when flat.
- Shorting, leverage, margin, funding rates, and interest on cash are not modeled unless explicitly extended.

## Parameter search and selection

A grid search is performed to evaluate combinations of the three EMA lengths on a training subset of the data. Invalid or redundant configurations are excluded by enforcing a consistent ordering constraint between the EMA windows.

A performance metric (for example Sharpe) is computed for each candidate configuration on the training sample. Top-ranked configurations are then re-evaluated on a held-out validation sample to quantify out-of-sample stability and degradation. The selected configuration is treated as a baseline for subsequent diagnostics rather than as a definitive optimum.

## Sensitivity analysis and diagnostics

A local sensitivity analysis is performed around the selected baseline. Each EMA length is perturbed in a neighborhood while the other lengths are held fixed, and performance metrics are recomputed. Heatmaps and summary tables are produced to identify whether performance is concentrated in narrow optima or persists across a broader region.

Additional diagnostics are produced to characterize what is driving the backtest results:

- Equity curve and drawdown profile.
- Rolling risk and rolling performance statistics.
- Trade-level outcomes and return distribution summaries.

The analysis is intended to separate signal strength from parameter luck and to highlight conditions under which the strategy fails.

## Outputs

The notebook produces:

- Grid-search tables ranking candidate EMA triples on the training sample.
- Validation summaries for top-ranked candidates.
- Full-sample backtest results for the selected configuration alongside a buy-and-hold benchmark.
- Sensitivity heatmaps and stability diagnostics.
- Standard performance and drawdown plots.

## Key assumptions and limitations

- No investment advice is provided, and no claims of live tradability are made.
- BTC-USD daily bars from `yfinance` are used as a price proxy; exchange-level microstructure, 24/7 execution constraints, and time-of-day effects are not captured.
- Transaction costs are simplified to constant proportional fee and slippage; market impact and liquidity constraints are not modeled.
- Cash is modeled with zero return unless a cash yield model is explicitly added.
- Results are sensitive to the chosen sample window, data source, and the specific crossover definition. Robustness across regimes is treated as a required interpretation step.

## How to run

1. Install dependencies listed in the notebook (commonly `numpy`, `pandas`, `matplotlib`, `yfinance`, and the backtesting library used for signal evaluation).
2. Open the notebook and set the ticker (`BTC-USD`), start date, and cost assumptions.
3. Run all cells to download data, generate signals, run the grid search and validation, and produce sensitivity plots.
