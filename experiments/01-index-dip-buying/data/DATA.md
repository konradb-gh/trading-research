# Experiment 01 — published data

Everything behind the numbers in the paper, except raw vendor price history (see
[What isn't here](#what-isnt-here)).

---

## Files

| File | Contents |
|---|---|
| `insample_grid_cells.csv` | Every parameter combination × every market, on pre-2013 data. 1,200 rows. This is the source for the heatmap figure. |
| `universe_indices.csv` | The exact ten instruments, with their role in the pass/fail test |
| `experiment_spec.yaml` | The machine-readable experiment definition: universe, grid, exit rules, stop rules |
| `PREREGISTRATION_and_verdicts.md` | The pre-registration verbatim, the power-check protocol amendment it produced, and the verdict-log rows |

**`insample_grid_cells.csv` columns**

| Column | Meaning |
|---|---|
| `instrument` | Yahoo ticker |
| `rsi_period` | RSI lookback in days (1–4) |
| `threshold` | Entry fires when RSI closes below this (5, 10, 15, 20, 30) |
| `exit` | `close>5SMA` (first bounce), `RSI>65` (hold until strength returns), or `time3d` (three days) |
| `stop` | `nostop` or `2xATR` (2 × 14-day average true range) |
| `n_trades` | Completed trades in the window |
| `win_rate` | % of trades profitable after costs |
| `expectancy_pct` | **Mean profit per trade, % after costs.** The headline metric |
| `profit_factor` | Gross profit ÷ gross loss |
| `max_dd_pct` | Worst peak-to-trough decline, % (negative) |
| `tim_pct` | % of days holding a position ("time in market") |
| `ann_ret_pct` | Annualised return, % |
| `ann_over_tim` | Annualised return divided by time in market — return per unit of exposure |

1,200 rows = 10 instruments × 4 RSI periods × 5 thresholds × 3 exits × 2 stops.

**`universe_indices.csv`** — `ticker`, `name`, `role`. Nine indices count toward pass/fail;
`EPOL` is reported but carries no weight, because its history begins in 2010 and leaves almost
no exploration-window data. That exclusion was declared in the pre-registration, not chosen
afterwards.

---

## Data sources

| What | Source | Notes |
|---|---|---|
| Daily prices | Yahoo Finance via [`yfinance`](https://github.com/ranaroussi/yfinance) | Daily bars, split/dividend adjusted, `interval="1d"`, max available history |

Index tickers carry Yahoo's `^` prefix (`^GSPC`, `^NDX`, …). `EPOL` is an ETF, used as a proxy
for the Warsaw market because no usable index history was available free — a limitation
described in the paper.

---

## Date ranges and screens

| | Exploration | Sealed |
|---|---|---|
| Window | earliest available → 2012-12-31 | 2013-01-01 → present |

Start dates vary by market (whatever Yahoo offers; `^GSPC` reaches back furthest, `EPOL` only
to 2010). No liquidity or price screens are applied — these are major indices, and the
tradeability question is handled by the cost model rather than by filtering.

Entries require the close to be above the 200-day simple moving average, so each series needs
a 200-bar warm-up before its first possible signal. All indicators are causal: the value at
date *D* uses only bars up to and including *D*.

---

## Cost model

Charged on every trade:

| Component | Value |
|---|---|
| Slippage | **10 basis points (0.10%) per side** |
| Commission | **0.20% round trip** |

A completed round trip costs 0.20% commission + 0.20% slippage = **0.40%**. Against a mean
profit per trade measured in tenths of a percent, this is the whole story of the experiment.

---

## What isn't here

**Raw Yahoo price history is not published** — it is large, and redistributing vendor price
data is a licensing grey area. The exact ticker list, date ranges and interval (`1d`, adjusted)
are published instead, which is enough to re-download identical inputs.

Yahoo revises its adjusted history over time, so a re-download will not always be bit-identical
to the snapshot used here.

**Out-of-sample per-cell results are not published as a CSV.** The sealed run reports through
the verdict log and the equity-curve figure rather than a results table; the pre-registered
variant's out-of-sample numbers are in `PREREGISTRATION_and_verdicts.md`.

---

## Reproducing

Code lives in the source project, not this repository.

```bash
# Exploration — the full parameter grid, pre-2013 only
python research.py --experiment connors_dip_v1 --phase insample

# The sealed test — refuses a grid; demands the single pre-registered variant
python research.py --experiment connors_dip_v1 --phase oos \
    --variant 'rsi_period=4;threshold=10;exit=RSI>65;stop=nostop'
```
