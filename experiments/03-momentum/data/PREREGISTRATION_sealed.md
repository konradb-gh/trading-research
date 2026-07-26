# Experiment 3, Phase B — OOS PRE-REGISTRATION (SEALED)

**Committed before any 2013+ data was evaluated.** The commit containing this file is the
timestamp of record. Nothing below may be renegotiated after the numbers are seen.

This is the single sealed shot for Experiment 3. Phase A (time-series momentum on index
ETFs) was closed as a documented negative and forfeits its shot; it is not tested OOS.

---

## 1. The two declared arms

House format, as in Experiments 01 and 02: both arms are run and reported in full, and the
comparison between them is itself a declared output.

| | Primary | Secondary |
|---|---|---|
| Lookback | 12-month total return, no skip-month | same |
| Hold | top 20% of eligible, equal-weighted | same |
| Quality filter | OFF | OFF |
| **Regime gate** | **ON** — no position while SPY < 200-day SMA | **OFF** — always invested |
| Binding | **yes — kill criteria apply to this arm** | no — reported, not binding |

**Primary** is the configuration whose risk profile motivated the whole experiment: the
production-style regime gate is the headline output this project exists to measure.

**Secondary** isolates the momentum ranking alone — the one component that was independently
verified in-sample against matched-random selection (100th percentile, both halves).

**The primary-vs-secondary comparison is a declared output, not a footnote.** It directly
tests whether the gate's in-sample value was a drawdown-shape artifact. The sealed window is
unusually well suited to this: it contains both a sharp V-shaped crash and recovery (Feb–Aug
2020) and a slow grinding decline (2022). In-sample, the gate helped in the grind and *hurt*
in the V. If that pattern repeats OOS, the gate is shape-dependent and should not be trusted
in deployment regardless of whether the primary arm passes.

### Fixed parameters (both arms)

- **Universe**: S&P 500 point-in-time membership (`research/data/sp500_hist.csv`), reused
  from Experiment 02's survivorship machinery.
- **Data-integrity screen**: drop zero-volume bars; reject any symbol whose remaining series
  contains a >100% one-day move; require Close ≥ $5 at selection. Necessary — recycled
  tickers carry vendor placeholder bars (e.g. TNB printing OHLC=11000.00 on Volume=0) that
  corrupt an equal-weight mean. *Methodological note: the screen is evaluated over each
  symbol's full available history, so a symbol corrupted in 2020 is excluded from 2013 too.
  This is a technical look-ahead, accepted because the excluded bars are untradeable vendor
  artifacts rather than economic information.*
- **Costs**: 10 bps/side slippage + 0.20% round-trip commission, always on.
- **Rebalance**: monthly, last trading day.
- **Engine**: `src/research/portfolio.py` (share-based, bit-reproducible as of commit 566ddb3).
- **Signal warm-up**: the 12-month lookback and the 200-day SMA at the first 2013 rebalance
  necessarily read 2012 prices. This is causal warm-up, not peeking — every signal at date D
  uses only data up to and including D.
- **OOS window**: 2013-01-01 through the last available bar in the cached data at run time
  (data as-of 2026-07-23). Recorded in the results.

### Why gate ON is the primary despite the crash-test result

The in-sample crash test (recorded in §3) showed the gate is fragile. The gate remains the
primary arm anyway because the kill criteria in §2 are constructed so that a fragile gate
*fails* rather than passes unnoticed — in particular criterion (b), which a strategy that
wins rare-but-large and bleeds the rest of the time cannot pass. Pre-registering the gate-off
variant as primary would answer a narrower question and duck the one the project was scoped
to ask. We test the gate and let it fail if it fails.

---

## 2. Binding kill criteria

All four bind on the **primary arm**. All four must pass. No renegotiating.

- **(a) Net CAGR > benchmark CAGR.**
- **(b) Net Sharpe > benchmark Sharpe.** Strictly greater — not "within 20% of". The claim
  under test is that this beats indexing on a risk-adjusted basis; a criterion that can be
  satisfied while underperforming the benchmark cannot test that claim.
- **(c) Above the 95th percentile of 200 matched random portfolios** — random selection from
  the identical eligible pool each month, matched on hold-size (20%) and matched on gate
  (random also holds cash when the gate is RED). Seeds 0–199. This replicates the control
  that made the Phase B result credible in-sample, and tests the *hypothesis* (that momentum
  ranking carries information) rather than merely the performance.
- **(d) Positive expectancy in both halves** of the sealed window — first half 2013-01-01 to
  2019-12-31, second half 2020-01-01 to end. Defined, to remove all post-hoc latitude, as:
  **mean monthly net return > 0 AND net CAGR > 0, in each half independently.**

**Benchmark** for (a) and (b): equal-weight, monthly-rebalanced buy-and-hold of the same
point-in-time universe, same integrity screen, same costs, same engine, same window.

**Verdict**: SURVIVES only if (a) AND (b) AND (c) AND (d). Otherwise DIES.

---

## 3. Facts recorded before the run

### 3.1 The crash-test finding (in-sample, median config, 12mo/top-20%)

The regime gate's in-sample value is **concentrated and drawdown-shape-dependent**:

- Only 58 of 203 months (28.6%) are ever gated; on the other 145 the two arms hold identical
  positions by construction.
- Of the total compounded gate advantage (1.9568× over the full in-sample window), the
  **top 10 crash-avoidance months alone account for ≈2.42× in isolation (131.6% of the total
  log-effect)**; the remaining 193 months are collectively a **net drag (≈0.81×)**.
- **Median monthly gate contribution: exactly 0.00pp.** Noise-robust win/loss among the 58
  genuinely gated months is dead even at **29–29**.
- By crash shape: **+22.93pp** through the 2000–2002 grind, but **−7.71pp** over the 12 months
  following the March 2009 V-shaped bottom.
- A small, real, one-directional cost drag exists even on ungated months (0 wins, 7 losses,
  0.06–0.12pp each) from re-entering full positions out of cash after a gated episode.

**Prediction implied by this**: if the sealed window's 2020 V behaves like 2009, the gate
should cost the primary arm relative to the secondary across 2020; if 2022 behaves like
2000–2002, it should help there. Recording this now so the pattern cannot be rationalised
after the fact either way.

### 3.2 Config chosen on theory, not on in-sample ranking

12-month lookback is the canonical academic momentum formation window (Jegadeesh–Titman).
Top-quintile is conventional breadth. No-skip because Phase A found skipping the most recent
month is net-negative across every configuration tested — that question is closed, not
reopened. Quality filter off because Trend-Template shrank the in-sample eligible pool to
single digits in some months, producing undiversified artifact portfolios (−80% to −94%
drawdowns) that reflect filter-induced concentration rather than any tested quality effect.

**This is explicitly not the in-sample winner** (6-month lookback / top-10-names, 19.84%
CAGR). That configuration cannot be justified: the rank-persistence test across in-sample
halves returned rho = +0.391, p = 0.11 within the quality-off arm — config ranking is
statistically indistinguishable from chance, so nothing in-sample licenses picking the peak.

### 3.3 Power check (projection, computed before the run)

- OOS window ≈ 2013-01-01 → 2026-07-23 ≈ **13.6 years ≈ ~163 monthly rebalances**.
- Experiment 02 established sealed-window PIT coverage at **~81%**, versus **~49%** in-sample.
- At ~500 nominal constituents and 81% coverage, expect **~400+ eligible names/month**, so
  top-20% projects to **~80 holdings/month** — versus ~40–57 in-sample. The OOS portfolio
  should therefore be *structurally more diversified* than anything tested in exploration.
  This is a real regime difference, not a formality: in-sample drawdown behaviour should not
  be assumed to transfer.
- ~163 monthly observations is adequate for the four criteria as specified. The verdict is a
  single evaluation regardless.

### 3.4 Standing survivorship caveat

**Absolute return levels — for the strategy and the benchmark alike — remain
survivorship-inflated by an unquantified amount, and this run does not fix that.**

In-sample testing ruled out one specific mechanism: momentum's edge *over random selection
from the same incomplete universe* does not vary with coverage (r = −0.069, p = 0.43 across
140 gate-open months). It could not rule out, and this OOS run cannot rule out, that both
arms are elevated together — momentum and benchmark both average only over whichever members
survive into the dataset, so a constituent that listed, rocketed, and delisted without ever
entering our frames is invisible to both. Sealed-window coverage (~81%) is materially better
than in-sample (~49%), so the OOS number should sit closer to truth — but *closer is not
correct*, and no claim about the absolute magnitude of the OOS result may be read as
survivorship-clean. Criteria (a) and (b) are relative comparisons against a benchmark
computed on the identical universe, which is why they remain meaningful under this caveat;
the absolute CAGR figure does not.

---

## 4. What is not decided here

If the primary arm dies, what to do next is decided *after* the single evaluation, not now.
The secondary arm's result does not rescue a failed primary — it is diagnostic, not a
fallback cell.
