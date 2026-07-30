# What Actually Governs Blow-Up Risk in Our Own Screener?

**Turning the method inward — a risk audit of the system we actually run, at the account we actually run it on**

*Trading Rules Research · Experiment 04 · July 2026*

---

## How this was built

*Pulled from the running system this session, not written from memory.*

| | |
|---|---|
| **Hardware** | MacBook Pro, Apple M3 Pro (11 cores: 5P+6E), 36GB RAM — `system_profiler SPHardwareDataType` |
| **OS** | macOS 26.5.2, build 25F84 — `sw_vers` |
| **Python** | 3.9.6, project venv |

**Libraries actually imported** by this study's own scripts (`grep`, then `pip freeze` for versions):

| Library | Ver. | Role |
|---|---|---|
| pandas | 2.3.3 | portfolio state, equity curves, every result table |
| numpy | 2.0.2 | vectorised P&L, `corrcoef` for the concurrency-correlation column, `percentile`, `median` |
| PyYAML | 6.0.3 | `config.yaml` (costs, risk budget, the new backtest-only controls) |
| yfinance | 1.2.0 | daily OHLCV for the one-time candidate generation (US + GPW) |
| pyarrow | 21.0.0 | parquet read/write, local price cache |
| requests | 2.32.5 | HTTP layer under the fetch helpers |

No plotting library: this paper has no figures, only tables, and every number in them comes from
a published CSV in `data/`.

**Real data, simulated trading — and one number that needs stating up front.** Prices are real
historical data. What's simulated is the trading. The simulation runs on a **100,000-unit**
starting equity because results are reported in percentages, but the live account this study is
*about* holds **10,000 PLN**. That gap is deliberate and it matters, so it gets its own section:
because position sizing here is entirely percentage-based, the percentage results transfer
directly to 10,000 PLN — and at 10,000 PLN the currency granularity only makes the constraint
this paper describes *tighter*, never looser. Synthetic price series appear only in unit tests,
never in a published result.

---

## What this paper is — and how it differs from the first three

The first three experiments in this series each took a **published idea from the literature** —
dip-buying, quality dip-buying, momentum — and put it through a sealed, pre-registered test to
see whether it survived realistic costs. Three for three, it didn't, or not usably.

This one is a different kind of piece. It takes the same instruments — the backtest engine, the
cost model, the habit of trusting the CSV over the write-up — and **points them at our own live
screener** instead of at a textbook. The question is not "does some famous anomaly work." It's:

> **In the system we actually run, on the account we actually run it on, what governs the risk of
> blowing up — and do any of the usual position-sizing rules change it?**

There's no sealed OOS test here, because there's no edge being claimed: a companion finding, from
the exploration work behind Experiments 01–03, is that this screener's rule-level edge is
statistically indistinguishable from zero. That's the premise, not the disappointment. When the
edge is noise, the only honest thing left to engineer is the **risk profile** — and the entire
study reports risk metrics first and never, anywhere, recommends a setting because it returned
more. Returns are shown, but only to prove they wandered around inside the noise while the risk
numbers moved for real reasons.

**Short version of the result:** the published drawdown was real, not an artifact of over-betting;
the one risk rule that turned out to matter was hiding inside an unrelated parameter by accident;
and at a 10,000 PLN account the binding risk constraint is the account itself — most of the
position-sizing machinery you might reach for never even fires.

---

## 1. Why audit our own system this way

Every backtest in this project reports a drawdown, and drawdown is the number that actually ends
accounts. But a drawdown figure is only trustworthy if the accounting underneath it is honest
about **what the account could afford to hold.**

The live screener sizes positions by risk: it risks a fixed fraction of equity per trade (1.8%),
places a stop, and buys however many shares make the distance-to-stop equal that risk. The
backtest engine did the same — and tracked *committed risk* against a 4% open-risk cap. What it
did **not** track was committed *capital*. Nothing stopped the simulator from holding positions
whose combined value exceeded the account, because in a risk-only ledger a tight-stopped position
looks cheap (little risk) while actually consuming a large slice of equity (a 1.8% risk on a 2%
stop is a 90%-of-equity position).

That's a real gap, and it cuts both ways. If the published backtests had been quietly holding
unaffordable, leveraged books, their drawdowns would be *understated* — the real account couldn't
have taken those positions, and would have behaved differently. Before trusting any of the risk
figures this project has published, that had to be checked. That check is Step 0, and it's the
credibility anchor for everything after it.

---

## 2. Step 0 — the published drawdown was real, not inflated by over-betting

The first thing built was a capital-tracking layer for the backtest engine: a no-leverage
constraint that the sum of open **position values** may not exceed equity, sitting alongside the
existing risk cap. Crucially, it can be switched fully **off**, and with it off the engine must
reproduce the old published numbers exactly — otherwise the new apparatus is measuring something
different and nothing downstream can be trusted.

It does. With capital tracking off, on the full US+GPW history, gate on:

| | Return | Worst drawdown | Trades |
|---|---|---|---|
| Published engine, reproduced to the decimal | **+17.73%** | **−43.84%** | **1200** |

That's the anchor: same inputs, same outputs, twelve significant figures agreeing
(`+17.734221…%`, `−43.844543…%`). The measuring instrument is faithful.

Now turn capital tracking **on** and ask the real question — was that −43.84% drawdown propped up
by holding positions the account couldn't afford?

| Arm | Return | Worst drawdown | Trades | Positions clamped by capital |
|---|---|---|---|---|
| Capital **off** (published) | +17.73% | −43.84% | 1200 | — |
| Capital **on**, same risk-cap rule | +19.46% | **−43.92%** | 1188 | 216 clamped, 12 skipped |

The drawdown barely moves: −43.84% → −43.92%. Enforcing that the account can actually afford every
position it holds changes almost nothing, because the 4% risk cap was **already** keeping the book
small — only 216 of ~1,200 positions ever needed clamping, and just 12 were skipped outright for
lack of cash. **The published −43.84% is a real property of the strategy, not an over-betting
artifact.** Every drawdown figure in Papers 01–03 survives this check.

### A correction, on the record

An early version of this Step 0, run on a **short window** (from 2015, 40 tickers), reported the
opposite of the eventual finding — that capital tracking *improved* both return and drawdown. On
the full history it reversed: capital tracking, when the risk cap is also relaxed, makes the
drawdown **worse** (see the next section). The short-window result was retracted rather than
quietly updated. It's noted here because the reversal is itself the point — a small window is not
a small version of the full test, it's a different and less reliable test, and this project's
standard is to say so out loud when one misleads.

---

## 3. Sweep C — the risk cap was an accidental concurrency limiter

Step 0 left a loose thread. The 4% open-risk cap "keeps the book small" — but *how*? It was never
designed as a limit on how many positions you hold at once. It's a limit on total open *risk*. Yet
it was behaving like a concurrency limit, and that raised a question worth a sweep: **is a risk
cap the right tool for that job, or is it doing it by accident, illegibly?**

The tell is what happens when you relax it. Charge the risk cap against *actual* (post-clamp) risk
instead of intended risk — a change that sounds like a rounding detail — and the cap loosens,
admitting far more simultaneous positions:

| | Return | Worst drawdown | avg concurrent | peak concurrent | Trades |
|---|---|---|---|---|---|
| Published (risk cap tight) | +17.73% | −43.84% | 0.89 | 4 | 1200 |
| Risk cap relaxed | +4.39% | **−48.45%** | 1.61 | **16** | 2221 |

Peak concurrency jumps from 4 to **16**, average from 0.89 to 1.61, the book nearly doubles, and
the drawdown deepens by four and a half points. All of that from a change to how a *risk* number
is charged. The concurrency control was a **side effect** — invisible, and hostage to an unrelated
accounting choice. That's exactly the kind of thing that should be made explicit.

So: replace the accident with an honest, legible rule — a hard cap on simultaneously-open
positions — and sweep it. Under production semantics (capital on, risk cap on actual risk), varying
only `max_concurrent_positions`:

| Cap | Return | Worst drawdown | avg conc | peak conc | Trades |
|---|---|---|---|---|---|
| none | +4.39% | −48.45% | 1.61 | 16 | 2221 |
| 3 | **+11.26%** | **−41.80%** | 0.96 | 3 | 1336 |

A cap of 3 pulls the drawdown back to −41.80% — better than the published −43.84% — with an average
concurrency (0.96) almost identical to the original engine's (0.89). It does the job the risk cap
had been doing by accident, except you can now say what it does in one sentence.

### The mechanism, shown clean

Return under the gate is noisy — it's a regime filter reacting to a handful of episodes, so any
single number is a small sample. To see the concurrency mechanism *without* that noise, run the
same sweep with the gate **off**, where the filter never sits out and the full history is always
in play. The drawdown then falls monotonically as you tighten the cap:

| Cap | none | 8 | 6 | 5 | 4 | 3 |
|---|---|---|---|---|---|---|
| Worst drawdown | −81.09% | −77.25% | −76.03% | −67.60% | **−65.70%** | −68.17% |

From −81% at no cap to −65.7% at a cap of 4 — a clean, monotonic 15-point improvement, exactly the
shape you'd expect if uncontrolled concurrency is what deepens drawdowns: more correlated longs
open at once, and a bad week hits all of them together. (The cap of 3 ticks *back up* to −68.17%,
breaking the monotonic run at the very end; it's reported as it landed, not smoothed. At a cap that
tight in the no-gate world you start starving the book of the few positions it needs to recover,
which is its own small lesson about over-constraining.)

One number that **doesn't** move across this whole sweep: the worst single trade stays pinned at
**−3.13%** of equity at every cap. That's the right behaviour — a concurrency limit governs how
many things can go wrong at once, not how much any one of them costs. Single-trade loss is a
different lever, and it belongs to the next sweep.

---

## 4. Sweep A — concentration caps, and where they quietly become "risk-per-trade"

The obvious tool for single-name blow-up risk is a **concentration cap**: never let one position
exceed X% of equity. Sweep A tests it, stacked on top of the adopted concurrency cap of 3, at the
live risk budget (1.8% per trade), varying the position cap from none down to 15%.

The first result is that at a real risk budget, the concentration cap has **almost nothing to do**,
because a constraint you didn't design as a diversification rule got there first: the no-leverage
capital limit.

| Position cap | worst trade | maxDD | median pos | p95 pos | bound by *concentration* | bound by *capital* |
|---|---|---|---|---|---|---|
| none | −3.13% | −41.80% | 29.5% | 62.9% | **0%** | **32.3%** |
| 50% | −3.07% | −33.31% | 29.8% | 50.0% | 15.1% | 27.5% |
| 33% | −2.77% | −38.03% | 33.0% | 33.0% | 58.3% | 1.7% |
| 25% | −2.65% | −35.19% | 25.0% | 25.0% | 77.8% | 0% |

Read the last two columns. With **no** concentration cap, a third of all positions (32.3%) are
already being clamped — by *capital*, the no-leverage wall — because at 1.8% risk on a tight stop
a single position wants 60–90% of equity and simply can't have it. The concentration cap, at that
point, is clamping **nothing** (0%).

As you tighten the cap it starts taking over — but look at *where* the handover happens. At a 50%
cap, concentration binds 15.1% of the time while capital still binds 27.5%: capital is still the
dominant constraint. Only at a **33%** cap does concentration (58.3%) finally overtake capital
(collapsed to 1.7%). So the honest statement — and here I'm correcting the compressed version I
started from, which said capital binds "before *any* concentration cap engages" — is narrower and
verifiable in the table: **a loose concentration cap (50%) does engage, for about 15% of trades,
but capital remains the dominant clamp until the cap is set to roughly 33% or tighter.** Above
that, the concentration machinery is mostly redundant with a limit that was already there.

### The lever that actually moves single-name loss — and the one that doesn't move cleanly

Does anything reliably shrink the worst single trade? Lowering **risk-per-trade** does, directly
and legibly (no position cap, so nothing else is interfering):

| risk-per-trade | worst trade (stop fills) | median pos | p95 pos |
|---|---|---|---|
| 1.8% | −3.13% | 29.5% | 62.9% |
| 1.25% | −2.17% | 23.6% | 50.2% |
| 1.0% | −1.74% | 19.4% | 44.2% |
| 0.75% | −1.30% | 14.6% | 34.7% |

Worst-trade loss falls almost in proportion to the risk budget, and the whole position-size
distribution slides down with it. That's the clean lever for the loss you take **when your stop
fills as intended.**

But there's a worse loss the worst-trade figure doesn't capture: the one you take when the market
**gaps through your stop overnight** and fills far below it. The study estimates it as
`gap_risk_10%` — what a position costs if it opens 10% below its stop. Here the picture is
different, and it's the second place I'm siding with the CSV over the tidy summary:

| | risk 1.8%, no cap | risk 0.75%, no cap | risk 1.8%, **25% cap** |
|---|---|---|---|
| gap-through-stop loss | **−11.62%** | −8.18% | **−4.12%** |

Cutting risk-per-trade all the way from 1.8% to 0.75% only drops the gap tail from −11.6% to −8.2%
— it **floors around −8%** and won't go lower, because whichever single position capital lets grow
largest still carries a fat overnight tail. A **25% concentration cap**, by contrast, cuts it to
−4.1%. So for the *gap* tail specifically, the concentration cap is the **stronger** lever — the
opposite of "inert."

The catch, and the reason this doesn't rescue the concentration cap as an independent tool: a cap
tight enough to bite the gap tail (≤25%) is one that binds 78% of all positions (from the table
above). At that point it isn't "diversification" — it's **a size reduction wearing a different
label.** You've capped every position at a quarter of equity, which is just a blunter, less
legible way of running less risk per name. The two levers converge: past the point where a
concentration cap does anything capital wasn't already doing, it *is* risk-per-trade, applied
crudely.

---

## 5. The conclusion: at 10,000 PLN, the account is the binding constraint

Put the three findings together and they point at one structural fact, and it's about the size of
the account, not the cleverness of the rules.

The screener runs a **real** risk budget — 1.8% of equity per trade — with ATR-based stops that
are tight relative to that budget. Position size as a fraction of equity is
`risk% ÷ stop-distance`, so those two facts *force* large individual positions: a median of
**29.5%** of equity and a 95th-percentile of **62.9%**. At that size, the no-leverage wall is hit
constantly — capital clamps a third of all entries before any diversification rule is consulted —
and you can only ever hold a **handful** of names at once (peak concurrency of 3 under the adopted
cap; left uncapped it wants 16, but capital and drawdown punish that).

You cannot diversify this away while keeping the risk budget real. To hold ten or fifteen names
simultaneously you'd have to cut risk-per-trade toward 0.5%, at which point each trade barely moves
the account and the fixed costs of trading start to dominate the returns — the same
edge-eaten-by-costs wall the first three experiments kept hitting. Every position-sizing rule in
this study either (a) never fires at this scale, or (b) fires only by becoming a disguised cut to
the risk budget. There is no setting that buys diversification for free.

At **10,000 PLN** this stops being abstract. A median 29.5% position is **2,950 PLN**; a
95th-percentile one is **6,290 PLN** — more than half the account in a single name. You do not have
the capital to run a diversified book *and* a meaningful per-trade risk at the same time. The math
is percentage-based, so it would read the same at 100,000 or 1,000,000 units — but the currency
granularity of a small account only makes it **worse**: fewer names are affordable at a sensible
share count, and the fixed per-trade cost is a bigger drag on every one of them.

This is the part worth stating plainly, because it's tempting to file it as a problem to grow out
of. **It isn't a stage.** For an account that is permanently small — by choice, or by circumstance
— structural concentration is not a bug in the rules to be patched; it's the **operating reality**,
and it's the thing that actually sets the risk of ruin. The one honest lever over it is
risk-per-trade, which the operator already controls, and which this study pointedly does **not**
tell them to change, because nothing here beats anything else on anything but noise.

---

## 6. What was changed in the live system: nothing

**No live-path code was modified by this study.** The live screener's scanning, its risk module,
and its regime model were not touched. Everything built here — the capital-tracking layer, the
concurrency cap, the concentration cap — lives entirely in the **backtest** engine and its config,
behind switches that the live scan never reads.

That's a deliberate outcome, not an unfinished one, for three reasons:

1. **The study's own finding argues against a change.** The position-sizing machinery you'd add is
   inert at this account's scale — it either never fires or silently duplicates the risk budget.
   There is nothing here that would measurably reduce risk of ruin that isn't already implied by
   the risk-per-trade knob that exists.
2. **The one lever that works is already live, and this study doesn't rank on the thing that would
   justify moving it.** Returns across every sweep wandered inside the noise; drawdown moved for
   real reasons but never in a way that recommends a *specific* risk-per-trade over the current
   1.8%. Changing a live risk setting on the basis of a return that's indistinguishable from
   chance is exactly the mistake this whole project exists to avoid.
3. **Simulate before you touch the live path.** The discipline the first three experiments applied
   to *ideas* applies here to *our own system*: model the change, measure it honestly, and only
   then — if the measurement warrants it — change the thing that trades real money. This
   measurement doesn't warrant it. The correct output of a risk audit is sometimes "leave it
   alone, and now you know why."

---

## 7. Honest limitations

- **This is not a sealed test.** There's no pre-registration and no out-of-sample wall, because no
  edge is being claimed — the study measures risk-profile mechanics, not a predictive result. The
  numbers are descriptive of one historical simulation, not a bet about the future.
- **`gap_risk_10%` is an estimate, not a measured loss.** It models what a position would cost if
  it opened 10% below its stop; real gaps vary and can be worse. It's used to *rank* the levers,
  which it does robustly, not to predict a specific loss.
- **Absolute returns carry the same survivorship inflation as every other paper here**, and are
  shown only to demonstrate they stayed inside the noise. Nothing in the verdict rests on a return
  figure.
- **The simulation equity is 100,000 units; the live account is 10,000 PLN.** The transfer is
  exact only because sizing is percentage-based. The claim that a small account is *worse* (cost
  drag, share-count granularity) is argued, not separately simulated at 10,000 PLN.
- **One market-regime mechanism is shown gate-off to remove filter noise.** That's a legitimate way
  to isolate the concurrency mechanism, but the gate-off world is not the live configuration; the
  live scan runs the gate on.
- **Costs are modelled, not measured** against a real broker's fills — 0.10% slippage per side plus
  0.20% round-trip, the same model as Experiments 01–03.
- **Nobody independent has reproduced this.** Verification was internal: every headline number in
  this paper was cross-checked against the published CSV before the prose was written, and two
  places where a working summary disagreed with the CSV were corrected *to* the CSV and flagged in
  the text (§3's cap-of-3 tick-up, §4's concentration-cap engagement and gap-tail lever). The data
  is published so someone else can check.

---

## 8. Reproducing this

```bash
# Step 0 — capital tracking, all four arms, full US+GPW history
python research_backtest_step0.py

# Sweep C — explicit concurrency control (gate on and off)
python research_backtest_sweep_c.py --gate on
python research_backtest_sweep_c.py --gate off

# Sweep A — concentration, on top of the adopted concurrency cap of 3
python research_backtest_sweep_a.py --gate on
```

Candidate generation is expensive (~38 min over full history) and model-agnostic — the risk
budget, concurrency cap and concentration cap don't affect *which* signals fire, only how they're
scheduled — so it's generated once and cached to a pickle; every sweep re-schedules from that cache
in seconds. Data files are in this folder; `DATA.md` documents every file and column. The
capital-tracking engine is covered by synthetic unit tests that pin the clamp arithmetic to a known
answer by construction.

---

## 9. Where this sits in the series

| # | Question | Instrument | Verdict |
|---|---|---|---|
| 01 | Does dip-buying work on indices? | Indices | Survives, but too rare to use |
| 02 | Does dip-buying work on quality stocks? | Individual stocks | Real behaviour, edge ≈ its costs once survivorship is removed |
| 03 | Does momentum work? | Indices and stocks | Complete negative — and a control that flattered it |
| **04** | **What governs blow-up risk in our own screener?** | **Our live system** | **Account size is the binding constraint; most position-sizing rules never fire** |

The first three turned this method on the market's textbook ideas. This one turned it on the tool
we built to trade them — and found that at a permanently small account, the risk that matters isn't
in the rules you can tune, it's in the size of the account you're tuning them for.

---

*This is a research project, not investment advice. All results are simulations using free public
data, net of modelled costs. No strategy or setting described here is recommended for use with real
money, and the author holds no position based on any of it.*
