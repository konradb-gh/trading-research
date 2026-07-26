<!-- Extracted verbatim from research/journal.md in the source project.
     Pre-registrations are written BEFORE their run and never edited afterwards. -->

# Experiment 03 — journal closure, addendum and verdicts (verbatim)

## Experiment 3 — momentum (`momentum_v1`), CLOSED  (2026-07-26)

First non-mean-reversion experiment. Two phases; monthly-rebalance harness added
(`src/research/portfolio.py`, share-based, bit-reproducible), reusing the data layer,
cost model and the hard 2013-01-01 wall verbatim.

**Phase A — time-series momentum on index ETFs (SPY/QQQ/EFA/EEM + 4 regional): NEGATIVE,
no OOS shot spent.** Lookback 3/6/12mo x skip-variant x gate A/B on 1993-01-29→2012-12-31.
Rank-persistence across the window halves: rho = -0.657 on n=6 effective configs
(exact permutation p = 0.175) — no evidence first-half success predicts second-half
success. 1/6 configs beat B&H in both halves against a chance expectation of 1.33.
Two confounds recorded rather than buried: the gate arm was an exact duplicate (no regime
series was wired in, so n=6 not 12), and the universe grows 1→8 instruments, leaving
31.5% of first-half days holding SPY alone. Closed as a documented negative; forfeits
its sealed shot.

**Phase B — cross-sectional momentum, PIT S&P 500: primary cell DIES.**
Pre-registered at commit `00b40c7` BEFORE any 2013+ evaluation; result at `d4d7933`.
Config chosen on THEORY (12mo = canonical Jegadeesh-Titman formation window, top-quintile
= conventional breadth), explicitly NOT the in-sample winner (6mo/top-10-names, 19.84%),
because in-sample config ranking was statistically indistinguishable from chance
(rho = +0.391, p = 0.11). 163 OOS rebalances, 86.9% PIT coverage, ~71-85 holdings/month.

  (a) CAGR   > benchmark :  7.46% vs 12.66%              FAIL
  (b) Sharpe > benchmark :  0.503 vs 0.729               FAIL
  (c) > 95th pctile of 200 matched-random : 100th pctile  PASS
  (d) both sealed halves positive                         PASS
  VERDICT: DIES (needs all four)

**The gate is what killed it: -5.75pp CAGR full-window** (-3.95pp 2013-19, -7.70pp
2020-26) versus the identical ungated arm. It did cut drawdown (-35.1% → -25.5%) but
Sharpe fell 0.72 → 0.50.

**The pre-registered shape prediction was FALSIFIED in its load-bearing half.** Recorded
before the run: the gate should hurt in a sharp V and help in a grind. It hurt in the 2020
V as predicted (-3.20pp) — but in the 2022 grind, the regime where in-sample it produced
its single strongest result (+22.93pp across 2000-2002), it instead cost **-8.95pp**, its
WORST episode of the sealed window, also losing to the plain index (-11.29% vs -7.52%).
The in-sample grind result was a one-episode artifact, consistent with the crash test
showing the whole gate effect concentrated in ~10 of 203 months with a median monthly
contribution of exactly 0.00pp.

**What survived: the ranking, not the gate.** Criterion (c) passed at the 100th percentile
(7.46% vs random mean 4.53%, best-of-200 5.98%). A POST-HOC, non-binding control on the
ungated arm confirms the information content is general rather than gate-specific:
100th percentile in all three windows (+4.74pp full, +2.31pp 2013-19, +7.00pp 2020-26
versus matched-random mean). 12-month cross-sectional momentum ranking carries real,
persistent out-of-sample information about relative forward returns.

**But costs consume it — the third consecutive experiment to die this way.** The ungated
arm turns over 25.2%/month, costing **1.21pp/yr** against the benchmark's 0.17pp. Net
result: CAGR 13.21% vs 12.66% (+0.55pp) but Sharpe **0.72 vs 0.73** — a dead heat with
simply owning the index, and behind it in 2013-2019 (11.84% vs 13.65%). The random control
holds concentration and turnover constant, which is why momentum wins there decisively and
only ties the index; the ~85-name random portfolios average 8.47% against the 427-name
benchmark's 12.66% purely from concentration vol-drag plus turnover.

**FOR THE RECORD — the production regime gate has never faced a pre-registered
out-of-sample test, and the closest thing to it has just failed one.** The backtest's
regime-gate A/B (`backtest.py`, the gate ON vs OFF headline) is an IN-SAMPLE finding,
not a validated one, and must be read that way. The SPY-200dma proxy tested here is not
the production O'Neil gate (distribution days, breadth, follow-through) — it is simpler —
but it is the nearest available stand-in, it was pre-registered, and it destroyed 5.75pp
of annual return out of sample while its in-sample rationale reversed sign in the exact
regime it was supposed to own. Any claim that the production gate adds value is currently
unsupported by out-of-sample evidence.

| 2026-07-26T08:29:46 | momentum_v1 | 00b40c7 | phaseA-insample | NEGATIVE | time-series index momentum: rank persistence rho=-0.657, p=0.175, n=6; 1/6 beat B&H both halves vs 1.33 chance; no OOS shot spent |
| 2026-07-26T08:29:46 | momentum_v1 | 00b40c7 | oos | DIES | primary (12mo/top20%/gate ON) DIES — (a) CAGR 7.46% vs bench 12.66% FAIL, (b) Sharpe 0.503 vs 0.729 FAIL, (c) 100th pctile of 200 matched-random PASS, (d) both halves positive PASS; gate -5.75pp; shape prediction falsified in the 2022 grind |


---

---

## ADDENDUM to Experiment 3 closure — criterion (c) was cost-confounded  (2026-07-26)

Post-hoc diagnostic, non-binding. **Does not alter the DIES verdict**, which rests on
criteria (a) and (b) and is unchanged. It does materially qualify the one criterion that
passed, and corrects two claims made in the closure entry above.

**The matched-random control was NOT matched on turnover.** It was matched on eligible
pool, hold-size and gate — as the pre-registration specified — but a re-randomised
k-of-n draw replaces ~80% of its names every month by construction
(E[overlap] = k·k/n = 85·85/427 ≈ 17 of 85 retained), whereas momentum rankings are
persistent and turn over ~25%/mo. Measured:

| arm | window | momentum turnover | random turnover | mismatch | drag mismatch |
|---|---|---|---|---|---|
| gated   | full      | 27.0%/mo | 69.3%/mo | -42.3pp | -1.99pp/yr |
| ungated | full      | 25.2%/mo | 80.3%/mo | -55.1pp | -2.59pp/yr |
| ungated | 2013-2019 | 25.7%/mo | 80.3%/mo | -54.6pp | -2.56pp/yr |
| ungated | 2020-2026 | 24.9%/mo | 79.9%/mo | -55.0pp | -2.59pp/yr |

The random arm therefore carried a **~2.0-2.6pp/yr structural cost handicap that has
nothing to do with ranking skill.** Re-running both arms GROSS (zero cost) isolates the
ranking:

| arm | window | mom NET | rand NET mean | pctile NET | mom GROSS | rand GROSS mean | **pctile GROSS** |
|---|---|---|---|---|---|---|---|
| **gated (criterion c)** | full | 7.46% | 4.53% | **100.0%** | 8.87% | 8.08% | **87.5%** |
| ungated | full      | 13.21% | 8.47% | 100.0% | 14.59% | 12.75% | 100.0% |
| ungated | 2013-2019 | 11.84% | 9.53% | 100.0% | 13.23% | 13.84% | **26.0%** |
| ungated | 2020-2026 | 14.68% | 7.68% | 100.0% | 16.06% | 11.92% | 100.0% |

**Criterion (c), evaluated on a cost-matched basis, would have FAILED.** On the
pre-registered gated arm the momentum portfolio reaches the **87.5th percentile gross**,
below its own 95th-percentile bar. 73% of its net edge over random was the cost mismatch,
not ranking. The pre-registration stated (c) was there to test "the *hypothesis* (that
momentum ranking carries information) rather than merely the performance" — by that stated
intent the gross comparison is the correct one, and it does not clear the bar. (c) as
literally written passed; (c) as intended did not.

**Two claims in the closure entry above are hereby corrected:**

1. *"The ranking's information content is general, not gate-specific"* — OVERSTATED. On a
   cost-matched basis it is **absent in 2013-2019** (26th percentile, momentum gross 13.23%
   vs random gross mean 13.84% — momentum is BELOW the random mean) and strong only in
   2020-2026 (100th pctile, +4.14pp). The net-basis "100th percentile in all three windows"
   was 127% cost-artifact in the 2013-2019 half. The honest statement is that the ranking
   signal is **concentrated in 2020-2026 and not demonstrable in 2013-2019** once the
   control is cost-matched.
2. *"the ~85-name random portfolios average 8.47% against the 427-name benchmark's 12.66%
   purely from concentration vol-drag plus turnover"* — WRONG on the mechanism. Random-85
   GROSS averages **12.75%**, essentially identical to the 427-name benchmark's 12.66%.
   Concentration vol-drag is negligible at these breadths; the entire 4.19pp gap was
   turnover cost.

**Design lesson for future experiments.** A random control must be matched on turnover, not
only on pool, hold-size and gate — otherwise it silently benchmarks persistence rather than
predictive skill, and flatters any low-turnover strategy. The corrected control is a
persistence-matched random draw (carry forward the same fraction of names the test arm
retains, randomise only the replaced slice). This is now a standing requirement.

What still stands after the correction: momentum ranking beats cost-matched random selection
over the full window on both arms (gated +0.79pp at the 87.5th pctile; ungated +1.84pp at
the 100th), and strongly in 2020-2026. It is a real but **much smaller and less consistent**
signal than the net figures suggested, and it remains fully consumed by costs in deployment
(the ungated arm ties the index on Sharpe, 0.72 vs 0.73).

| 2026-07-26T08:41:22 | momentum_v1 | 00b40c7 | oos-addendum | QUALIFIED | criterion (c) cost-confounded: random control unmatched on turnover (69-80%/mo vs momentum 25-27%/mo, ~2.0-2.6pp/yr handicap). Gross-basis pctile on the pre-registered gated arm falls 100.0% -> 87.5%, below its own 95% bar. Ranking signal absent in 2013-2019 gross (26th pctile), strong only in 2020-2026. DIES verdict unchanged. |

---

## Verdict log rows

| timestamp (UTC) | experiment | spec hash | phase | verdict | detail |
|---|---|---|---|---|---|
| 2026-07-26T08:29:46 | momentum_v1 | 00b40c7 | phaseA-insample | NEGATIVE | time-series index momentum: rank persistence rho=-0.657, p=0.175, n=6; 1/6 beat B&H both halves vs 1.33 chance; no OOS shot spent |
| 2026-07-26T08:29:46 | momentum_v1 | 00b40c7 | oos | DIES | primary (12mo/top20%/gate ON) DIES — (a) CAGR 7.46% vs bench 12.66% FAIL, (b) Sharpe 0.503 vs 0.729 FAIL, (c) 100th pctile of 200 matched-random PASS, (d) both halves positive PASS; gate -5.75pp; shape prediction falsified in the 2022 grind |
| 2026-07-26T08:41:22 | momentum_v1 | 00b40c7 | oos-addendum | QUALIFIED | criterion (c) cost-confounded: random control unmatched on turnover (69-80%/mo vs momentum 25-27%/mo, ~2.0-2.6pp/yr handicap). Gross-basis pctile on the pre-registered gated arm falls 100.0% -> 87.5%, below its own 95% bar. Ranking signal absent in 2013-2019 gross (26th pctile), strong only in 2020-2026. DIES verdict unchanged. |
