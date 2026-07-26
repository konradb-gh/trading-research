# Experiment 03 — published data

Everything behind the numbers in the paper, except raw vendor price history (see
[What isn't here](#what-isnt-here)).

---

## Files

### Phase A — trend following on index ETFs

| File | Contents |
|---|---|
| `phaseA_universe_coverage.csv` | The 8 ETFs used, with first/last bar, total bars and pre-2013 bars. **This is also the exact ticker list.** |
| `phaseA_results_corrected.csv` | The 6 distinct configurations × full window and both halves. **Authoritative.** Produced by the corrected engine. |
| `phaseA_results_SUPERSEDED_prefix_engine.csv` | The original 12-row run. **Superseded — do not use.** Published only so the correction is auditable. |
| `phaseA_window_split_SUPERSEDED_prefix_engine.csv` | The original window split. **Superseded — do not use.** |

The two `SUPERSEDED` files were produced before four defects in the calculation engine were
found and fixed (costs charged on the whole portfolio regardless of turnover; an unused
slippage parameter; a mismeasured metric; implicit free daily rebalancing) — the audit
described in §5 of the paper. Every number quoted in the paper itself, including §4's Phase A
table, now comes from `phaseA_results_corrected.csv`; nothing in the published prose depends
on the superseded file any more.

They are kept anyway, specifically so §5's claim can be checked rather than taken on trust: a
reader can diff `phaseA_results_SUPERSEDED_prefix_engine.csv` against
`phaseA_results_corrected.csv` and see the actual pre-fix and post-fix numbers side by side —
the best config's margin moving from +1.08pp to +2.24pp, and the first/second-half win counts
moving from 10-of-12 / 2-of-12 to 4-of-6 / 2-of-6 — rather than being asked to believe the audit
happened. **Do not use these two files for anything except that comparison.**

**`phaseA_results_corrected.csv` columns**

| Column | Meaning |
|---|---|
| `lookback` | Lookback window in months (3, 6, 12) |
| `skip` | `True` = signal excludes the most recent month; `False` = plain total return |
| `cagr_full`, `cagr_first_half`, `cagr_second_half` | Annualised return, % after costs |
| `margin_*` | Same, minus the equal-weight buy-and-hold benchmark for that window |
| `vol_*` | Annualised standard deviation of daily returns, % |
| `dd_*` | Worst peak-to-trough decline, % (negative) |

Window boundaries: full 1993-01-29 → 2012-12-31; split at 2003-01-14 (the median trading day).

### Phase B — ranking stocks, exploration (pre-2013)

| File | Contents |
|---|---|
| `phaseB_insample_grid.csv` | All 36 configurations × 3 windows. Columns below. |
| `phaseB_insample_benchmark.csv` | Equal-weight benchmark for each window |
| `phaseB_insample_window_split.csv` | Per-configuration margin and rank in each half — the persistence test behind "+0.39, p = 0.11" |
| `phaseB_excluded_symbols.csv` | **37 symbols dropped by the data-integrity screen, with the reason for each** |
| `phaseB_survivorship_by_era.csv` | Momentum vs matched-random by 4-year era, with membership coverage — the raw table behind §10's survivorship caveat |
| `phaseB_margin_vs_coverage_monthly.csv` | 140 filter-open months: momentum return, expected random return, margin, coverage that month |
| `phaseB_crashtest_monthly.csv` | 203 monthly returns with filter on and off, and whether the filter was active — the source for "145 of 203" and "ten specific episodes" |

**`phaseB_insample_grid.csv` columns**

| Column | Meaning |
|---|---|
| `window` | `full`, `first_half`, `second_half` |
| `lookback` | Months (3, 6, 12) |
| `hold` | `top10pct`, `top20pct`, or `top10n` (ten names) |
| `gate` | Market filter on/off |
| `quality` | Trend-Template quality screen on/off |
| `cagr_net`, `cagr_gross` | Annualised return after / before costs, % |
| `margin_net` | `cagr_net` minus the benchmark for that window |
| `vol`, `sharpe`, `max_dd` | Annualised vol %, Sharpe (0% risk-free), worst drawdown % |
| `in_market` | % of days holding a position |
| `turnover_pct` | Mean one-way monthly turnover, % of portfolio |
| `cost_drag_pp` | Annualised cost drag, percentage points |
| `avg_n_held`, `n_rebal` | Mean holdings per month; number of rebalances |

**`phaseB_excluded_symbols.csv`** — `symbol`, `reason`. Two reasons appear: a corrupt series
(a one-day move above +100% after zero-volume bars were removed) and insufficient history
after that removal. The extreme case is `TNB`, a former S&P industrial whose ticker now
resolves to a penny instrument printing `OHLC = 11000.00, Volume = 0` placeholder bars
interleaved with real ~$0.58 bars, producing a nominal +1,972,322% daily return.

### The sealed test (2013-01-01 → 2026-07-23)

| File | Contents |
|---|---|
| `PREREGISTRATION_sealed.md` | **The pre-registration, verbatim.** Committed before any post-2013 data was evaluated; never edited since. |
| `OOS_results.csv` | Both arms + benchmark × 3 windows. The paper's §7 table. |
| `OOS_random_control_gated.csv` | 200 random portfolios (seeds 0–199), criterion (c) as pre-registered |
| `OOS_random_control_ungated_posthoc.csv` | The same control on the unfiltered arm. **Post-hoc, not pre-registered.** |
| `OOS_turnover_diagnostic.csv` | Turnover and cost drag for both sides, plus before/after-cost percentiles — the source for all of §8 |
| `JOURNAL_closure_and_verdicts.md` | Closure entry, the cost-confound addendum, and the verdict-log rows, verbatim |

**`OOS_turnover_diagnostic.csv` columns**: `arm` (gated/ungated), `window`,
`mom_turnover` / `rand_turnover` (mean one-way monthly turnover %),
`mom_drag_pp` / `rand_drag_pp` (annualised cost drag, pp),
`mom_net` / `rand_net_mean` / `rand_net_p95` / `pctile_net` (after costs),
`mom_gross` / `rand_gross_mean` / `rand_gross_p95` / `pctile_gross` (before costs).

### Universe

`universe_sp500_membership_spans.csv` — every ticker that was ever an S&P 500 member in the
point-in-time dataset, with the dates it entered and left (`end_date` blank = still a member
at the dataset's end). This is the exact universe definition; combined with the source below
it reproduces the membership input without redistributing it.

---

## Data sources

| What | Source | Notes |
|---|---|---|
| Daily prices | Yahoo Finance via [`yfinance`](https://github.com/ranaroussi/yfinance) | Daily bars, split/dividend adjusted, `interval="1d"`, max available history |
| S&P 500 point-in-time membership | [`fja05680/sp500`](https://github.com/fja05680/sp500) | Daily membership snapshots, begins **1996-01-02** |
| S&P 500 current constituents | [Wikipedia: List of S&P 500 companies](https://en.wikipedia.org/wiki/List_of_S%26P_500_companies) | Used only by the live screener, not by this experiment's point-in-time universe |

**The 1996 start date matters.** The membership dataset does not exist before 1996-01-02. An
early version of Phase B was run from the 1960s, where the member set is empty and the
strategy holds nothing; that run was discarded and the window clamped to 1996 onwards. This is
described in §5 of the paper.

---

## Date ranges and screens

| | Phase A | Phase B exploration | Sealed test |
|---|---|---|---|
| Window | 1993-01-29 → 2012-12-31 | 1996-01-02 → 2012-12-31 | 2013-01-01 → 2026-07-23 |
| Rebalances | monthly, last trading day | 204 | 163 |
| Universe | 8 index ETFs | S&P 500 point-in-time | S&P 500 point-in-time |
| Membership coverage | n/a | ~49% | ~87% |

**Screens applied to Phase B (both windows, identically):**

1. Drop bars with zero volume (vendor placeholder rows, not tradeable).
2. Reject any symbol whose remaining series contains a one-day move above +100%.
3. Require at least 260 remaining bars.
4. Require a closing price of at least **$5** at selection.

Screens 1–3 are evaluated over each symbol's full available history, so a symbol corrupted in
2020 is excluded from 2013 as well. This is a technical look-ahead, accepted because the
excluded bars are untradeable vendor artifacts rather than economic information. It is
declared in the pre-registration.

Signals use a **warm-up** period before each window: the 12-month lookback and the 200-day
average at the first 2013 rebalance necessarily read 2012 prices. Every signal at date *D*
uses only data up to and including *D*.

---

## Cost model

Charged on every trade, in both windows, in every figure and table:

| Component | Value |
|---|---|
| Slippage | **10 basis points (0.10%) per side** |
| Commission | **0.20% round trip** |

A rebalance that trades nothing costs nothing. Cost on a rebalance is
`(bought_fraction + sold_fraction) × (0.10% + 0.10%)`, where the two fractions are the
proportions of the portfolio bought and sold. A full round trip therefore costs
0.20% commission + 0.20% slippage = 0.40%, matching the convention used in Experiments 01
and 02.

---

## What isn't here

**Raw Yahoo price history is not published.** It is large, and redistributing vendor price
data is a licensing grey area. Everything needed to re-download identical inputs is published
instead: the exact ticker lists, the date ranges, and the interval (`1d`, adjusted).

Note that Yahoo's adjusted history is itself revised over time, so a re-download will not
always be bit-identical to the snapshot used here.

---

## Reproducing

Code lives in the source project, not this repository.

```bash
# Phase A — corrected engine (the authoritative run)
python research_momentum_phase_a3.py

# Phase B exploration — 36 configurations, pre-2013 only
python research_momentum_phase_b2.py

# The sealed test — runs exactly the pre-registered cell, no tunable parameters
python research_momentum_oos.py

# Diagnostics
python research_momentum_crash_test.py       # the filter's episode concentration
python research_momentum_turnover_check.py   # the cost-matching problem in §8

# Figures in this paper
python research_momentum_figures.py
```

The engine is deterministic: repeated runs are bit-identical, and
`research_momentum_figures.py` asserts that its re-run reproduces `OOS_results.csv` exactly
before drawing anything.
