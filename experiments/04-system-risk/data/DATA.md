# Experiment 04 — published data

Everything behind the numbers in the paper. Unlike Experiments 01–03 this study has no figures,
so every headline number in the paper traces to one of the four CSVs below. Raw vendor price
history is not published (see [What isn't here](#what-isnt-here)).

All four files are the **canonical result CSVs** from the source project's risk-profile
rule-design study — the same files that project commits under `data/backtest/`, copied here
verbatim.

---

## Files

| File | Contents |
|---|---|
| `step0_us-gpw_full.csv` | Step 0 — the capital-tracking test. Four arms × two gate states, full US+GPW history. The paper's §2 tables. |
| `sweep_c_gate_on.csv` | Sweep C — concurrency cap {none, 8, 6, 5, 4, 3}, gate **on**, plus the published-engine reference row. The paper's §3 gate-on tables. |
| `sweep_c_gate_off.csv` | Sweep C — the same sweep, gate **off**. The monotonic-drawdown mechanism table in §3. |
| `sweep_a_gate_on.csv` | Sweep A — `risk_per_trade_pct` × `max_position_pct_of_equity` grid on top of concurrency cap = 3, gate on. The paper's §4 tables. |

### `step0_us-gpw_full.csv`

One row per (gate, arm). The four arms isolate what capital tracking does:

| Column | Meaning |
|---|---|
| (col 1) | Gate state: `on` (regime filter active) or `off` |
| (col 2) | Arm — see below |
| `total_return_%` | Total return over the full history, % after costs |
| `CAGR_%` | Annualised return, % |
| `maxDD_%` | Worst peak-to-trough decline, % (negative) |
| `worst_trade_%eq` | Worst single trade as a % of equity at entry (assumes the stop fills) |
| `worst_year` | Calendar year with the worst return, and that return |
| `n_taken` | Number of trades taken |
| `avg_concurrent` | Mean number of simultaneously-open positions |
| `pos_med_%eq`, `pos_p95_%eq` | Median and 95th-percentile position value, % of equity |
| `clamp_capital` | Positions whose size was clamped down by the no-leverage capital limit |
| `skip_capital` | Signals skipped outright for lack of free capital |
| `skip_risk` | Signals skipped by the open-risk cap |

**The four arms:**

| Arm | Capital tracking | Risk-cap charged on | Purpose |
|---|---|---|---|
| `OFF` | off | (original) | Reproduces the published engine. **The credibility anchor.** |
| `ON_frozen` | on | intended risk (unchanged) | Adds the no-leverage constraint *without* changing anything else — the apples-to-apples "was it over-betting?" test. |
| `ON_capfix` | on | intended risk | Capital on with the risk cap charged on desired (pre-clamp) risk. |
| `ON_full` | on | actual (post-clamp) risk | Capital on, risk cap relaxed — the production semantics the sweeps run on. Its gate-on row is the `cap=none` baseline in Sweep C. |

Paper §2 quotes the gate-on rows: `OFF` (+17.73% / −43.84% / 1200) and `ON_frozen`
(+19.46% / −43.92% / 1188; 216 capital-clamped, 12 skipped). The `OFF` row reproduces the
published backtest to twelve significant figures — that exact-reproduction claim is checkable by
comparing this row against the source project's committed backtest report.

### `sweep_c_gate_on.csv` and `sweep_c_gate_off.csv`

One row per arm. `OFF(pub)` is the published risk-only engine (capital off, no concurrency cap);
`cap=none … cap=3` are production semantics (capital on, risk cap on actual risk) with only
`max_concurrent_positions` varying.

| Column | Meaning |
|---|---|
| `total_return_%` | Total return, % after costs |
| `maxDD_%` | Worst drawdown, % (negative) |
| `worst_trade_%eq` | Worst single trade, % of entry equity (stop fills) |
| `avg_conc` | Mean simultaneously-open positions |
| `peak_conc` | Maximum simultaneously-open positions |
| `avg_corr` | Mean pairwise Pearson correlation of daily mark-to-market changes among concurrently-held positions (`numpy.corrcoef`, min 5 overlapping days) |
| `corr_pairs` | Number of position-pairs that entered that correlation average |
| `n_taken` | Trades taken |
| `skip_conc` | Signals skipped because the concurrency cap was full |

Paper §3 gate-on quotes: `OFF(pub)` (+17.73% / −43.84% / peak 4), `cap=none` (+4.39% / −48.45% /
peak 16), `cap=3` (+11.26% / −41.80% / peak 3). Gate-off quotes the `maxDD_%` column across caps:
−81.09 → −77.25 → −76.03 → −67.60 → −65.70 → −68.17. The worst-trade column is −3.13% at every cap
in both files (a concurrency limit doesn't touch single-trade loss); `OFF(pub)`'s −3.63% differs
only because capital-off sizing differs.

### `sweep_a_gate_on.csv`

One row per (`risk%`, `pos_cap`) cell — 4 risk budgets × 6 position caps = 24 rows. Built on top of
`max_concurrent_positions = 3`, gate on, capital on, risk cap on actual risk.

| Column | Meaning |
|---|---|
| `risk%` | `risk_per_trade_pct` for the cell (1.8, 1.25, 1.0, 0.75) |
| `pos_cap` | `max_position_pct_of_equity` (`none`, 50, 33, 25, 20, 15) |
| `worst_trade_%eq` | Worst single trade, % of entry equity, **assuming the stop fills** |
| `gap_risk_10pct` | Modelled loss (% of equity, negative) if the worst-case position **gapped 10% below its stop** instead of filling: `−max over positions of pos_frac × (0.10 + 0.90 × stop_frac)`. An estimate of the overnight-gap tail the worst-trade figure understates. |
| `maxDD_%` | Worst drawdown, % (negative) |
| `pos_med_%eq`, `pos_p95_%eq` | Median / 95th-percentile position value, % of equity |
| `conc_bound_%` | % of taken positions whose size was set by the **concentration** cap |
| `capital_bound_%` | % of taken positions whose size was set by the **capital** (no-leverage) limit |
| `below_budget_%` | % of taken positions forced below the intended risk budget by any clamp |
| `n_taken` | Trades taken |
| `total_return_%` | Total return, % after costs (shown to demonstrate it wanders inside the noise; never used to rank) |

Paper §4 quotes the `risk% = 1.8` rows (the live risk budget) and, for the risk-per-trade lever,
the `pos_cap = none` rows across all four risk budgets.

**Cross-check built into these files:** the `(1.8, none)` row of `sweep_a_gate_on.csv` is the same
configuration as the `cap=3` row of `sweep_c_gate_on.csv` — risk 1.8%, concurrency cap 3, no
position cap, gate on. Both report +11.2558% / −41.7958% / 1336 trades / worst −3.1293%,
bit-identical, from two independent scripts.

---

## What the numbers describe

| | Value |
|---|---|
| Simulation starting equity | **100,000 units** (results reported in %) |
| Live account this study is about | **10,000 PLN** |
| Live risk budget | 1.8% of equity per trade |
| Live open-risk cap | 4.0% total open risk |
| Markets | US (S&P 500 universe) + GPW (Warsaw), the live screener's two markets |
| History | Full available daily history per instrument |
| Gate | Regime filter (O'Neil-style market model); shown both on (live config) and off (to isolate mechanisms) |

The transfer from a 100,000-unit simulation to a 10,000 PLN account is exact because all sizing is
percentage-based. The paper argues (§5) that at 10,000 PLN the constraint is if anything *tighter*,
never looser — but that tightening (cost drag, share-count granularity) is argued, not separately
simulated.

---

## Cost model

Identical to Experiments 01–03. Charged on every trade:

| Component | Value |
|---|---|
| Slippage | **10 basis points (0.10%) per side** |
| Commission | **0.20% round trip** |

A full round trip costs 0.20% commission + 0.20% slippage = 0.40%.

---

## What isn't here

**Raw price history is not published** (large; redistributing vendor data is a licensing grey
area), and neither is the **candidate pickle** — the cached, model-agnostic set of signals the
sweeps schedule from. It's a ~38-minute regenerable intermediate, not a result; the four CSVs here
are the canonical outputs. Everything needed to regenerate the inputs is specified: the two live
markets (US S&P 500 universe + GPW), daily bars, full available history, and the cost model above.

Note that Yahoo's adjusted history is revised over time, so a re-download will not always be
bit-identical to the snapshot used here.

---

## Reproducing

Code lives in the source project (`swing-screener`), not this repository.

```bash
# Step 0 — capital-tracking test, four arms, full history
python research_backtest_step0.py

# Sweep C — explicit concurrency control
python research_backtest_sweep_c.py --gate on
python research_backtest_sweep_c.py --gate off

# Sweep A — concentration, on top of concurrency cap = 3
python research_backtest_sweep_a.py --gate on
```

Candidate generation is model-agnostic (the risk budget, concurrency cap and concentration cap
change only how signals are *scheduled*, not which fire), so it's generated once and cached; every
sweep re-schedules from that cache in seconds. The capital-tracking clamp arithmetic is pinned by
synthetic unit tests (`tests/test_backtest_capital.py`) whose correct answers are known by
construction.
