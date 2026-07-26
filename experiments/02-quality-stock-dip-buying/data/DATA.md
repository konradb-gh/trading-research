# Experiment 02 — published data

Everything behind the numbers in the paper, except raw vendor price history (see
[What isn't here](#what-isnt-here)).

---

## Note on `stock_dip_v1_pooled_expectancy.png`

This figure was republished after source commit `7ac7eb6` ("viz: use RdYlGn for research
heatmaps — red = loss, green = profit"), which changed the heatmap colour scale project-wide
from the matplotlib default (`RdBu_r`, where profit renders red) to the finance-convention
scale used throughout this series (`RdYlGn`, where profit renders green). The figure originally
published here predated that commit.

**Only the colour mapping changed.** Every annotated cell value in the corrected figure is
pixel-identical to the original — verified by decoding both PNGs and diffing the underlying
grid of numbers (0.62, 1.53, 2.04, … all 64 cells, exact) — so no result in this experiment was
ever affected. The figure is now consistent with the colour convention used in Papers 01 and 03.

---

## Files

| File | Contents |
|---|---|
| `insample_grid_cells.csv` | Every parameter combination, pooled across all stocks, pre-2013. 96 rows. Source for the heatmap figure. |
| `insample_per_year.csv` | Trade counts per configuration per calendar year — the signal-frequency data behind the power check |
| `survivorship_pit_comparison.csv` | **The survivorship correction.** Today's-constituents results vs point-in-time results, side by side. |
| `pit_coverage_by_era.csv` | How much of the index's true historical membership free data can actually see, by era |
| `universe_pit_availability.csv` | Every ever-member ticker with how many bars Yahoo returns, and the first/last date |
| `universe_sp500_membership_spans.csv` | Every ever-member with the dates it entered and left the index |
| `experiment_spec.yaml` | Machine-readable experiment definition: universe, eligibility, grid |
| `PREREGISTRATION_and_verdicts.md` | Pre-registration verbatim, the survivorship-correction write-up, and verdict-log rows |

**`insample_grid_cells.csv` columns**

| Column | Meaning |
|---|---|
| `rsi_period` | RSI lookback in days (2–4) |
| `threshold` | Entry fires when RSI closes below this (5, 10, 15, 20) |
| `exit` | Exit rule (`close>5SMA`, `RSI>65`, `time3d`, …) |
| `stop` | `nostop` or `2xATR` |
| `n_trades` | Completed trades, pooled across all eligible stocks |
| `win_rate` | % profitable after costs |
| `expectancy_pct` | **Mean profit per trade, % after costs.** The headline metric |
| `profit_factor` | Gross profit ÷ gross loss |
| `avg_hold_days` | Mean holding period |
| `projected_oos_n` | Projected sealed-window trade count, from in-sample frequency — the power check |

**`survivorship_pit_comparison.csv` columns**: `arm` (`nostop` / `2xATR`), `window`
(`insample` / `oos`), `metric`, `current` (today's constituents), `pit` (point-in-time
membership), `delta`. The paper's central number — out-of-sample expectancy falling from
+0.276% to **+0.185%** per trade once survivorship is removed — is the `nostop` / `oos` /
`expectancy%` row.

**`pit_coverage_by_era.csv` columns**: `era`, `members` (distinct tickers in the index during
that era), `with_data` (how many Yahoo returns usable history for), `coverage_pct`, `missing`.
Coverage rises from **40.5%** in 1996–2000 to **91.0%** in 2021–2026. This gradient is why
absolute figures are upper bounds and only relative comparisons are trustworthy — the missing
names skew toward companies that failed and were delisted.

---

## Data sources

| What | Source | Notes |
|---|---|---|
| Daily prices | Yahoo Finance via [`yfinance`](https://github.com/ranaroussi/yfinance) | Daily bars, split/dividend adjusted, `interval="1d"`, max available history |
| S&P 500 point-in-time membership | [`fja05680/sp500`](https://github.com/fja05680/sp500) | Daily membership snapshots, begins 1996-01-02 |
| S&P 500 current constituents | [Wikipedia: List of S&P 500 companies](https://en.wikipedia.org/wiki/List_of_S%26P_500_companies) | Used for the uncorrected "today's constituents" arm, which the correction then replaces |

The gap between those last two sources *is* the survivorship correction: the Wikipedia list is
who is in the index **now**, the point-in-time dataset is who was in it **then**.

---

## Date ranges and screens

| | Exploration | Sealed |
|---|---|---|
| Window | earliest available → 2012-12-31 | 2013-01-01 → present |
| Membership coverage (point-in-time arm) | ~52% | ~81% |

**Eligibility, applied as of each signal date:**

1. The stock passes the production Trend Template (a multi-condition trend-quality screen).
2. `^GSPC` closes above its 200-day simple moving average — the market filter.
3. Liquidity and minimum-price screens from the production configuration.
4. In the point-in-time arm only: the stock was an actual index member that day.

A history-splice guard also applies: a gap of more than 90 days between consecutive bars marks
a discontinuity (a recycled ticker, or a delist/relist), and trades touching data before the
final continuous segment are flagged and excluded.

All indicators are causal — the value at date *D* uses only bars up to and including *D*.

---

## Cost model

Charged on every trade:

| Component | Value |
|---|---|
| Slippage | **10 basis points (0.10%) per side** |
| Commission | **0.20% round trip** |

A completed round trip costs 0.20% commission + 0.20% slippage = **0.40%**. The paper also
discusses roughly **0.10% per trade** of financing cost on top; that is applied in the
discussion rather than baked into these CSVs, and the distinction matters for the verdict.

---

## What isn't here

**Raw Yahoo price history is not published** — it is large, and redistributing vendor price
data is a licensing grey area. The exact ticker lists (`universe_pit_availability.csv`,
`universe_sp500_membership_spans.csv`), date ranges and interval (`1d`, adjusted) are published
instead, which is enough to re-download identical inputs.

The upstream membership dataset (~5.5 MB of daily snapshots) is not mirrored here either; the
derived per-ticker spans are published and the source is linked above.

Yahoo revises its adjusted history over time, so a re-download will not always be bit-identical
to the snapshot used here — and for this experiment that matters more than most, because which
delisted tickers Yahoo still serves is exactly what drives the coverage figures.

**Out-of-sample per-cell results are not published as a CSV.** The sealed run reports through
the verdict log, the equity-curve figure and `survivorship_pit_comparison.csv`.

---

## Reproducing

Code lives in the source project, not this repository.

```bash
# Exploration — the full parameter grid, pre-2013 only
python research.py --experiment stock_dip_v1 --phase insample

# The sealed test — refuses a grid; demands the single pre-registered variant
python research.py --experiment stock_dip_v1 --phase oos \
    --variant 'rsi_period=2;threshold=10;exit=RSI>65;stop=nostop'

# The survivorship correction — same variant, point-in-time membership
python research/pit_rerun.py

# Membership coverage by era
python research/pit_coverage.py
```
