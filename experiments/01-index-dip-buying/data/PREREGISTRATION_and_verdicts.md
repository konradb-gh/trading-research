<!-- Extracted verbatim from research/journal.md in the source project.
     Pre-registrations are written BEFORE their run and never edited afterwards. -->

# Experiment 01 — pre-registration and verdicts (verbatim)

## Pre-registration — connors_dip_v1 OOS  (appended 2026-07-23T13:26:33, BEFORE the run)

**Hypothesis under OOS test:** "moderate multi-day pullbacks in uptrending indices, held until short-term strength returns" — an emergent effect from the in-sample search, NOT the original deep-dip thesis, and weighted accordingly.

**Variant:** RSI period 4, threshold 10, exit RSI>65, trend filter close>200d SMA — round numbers from the plateau's center. Both stop arms of this one cell are declared as the test (primary: no-stop, as discovered; secondary: 2×ATR(14), as deployment reality).

**Kill criteria, binding:** the variant dies unless, on OOS 2013→2026, (a) trade-count-weighted family expectancy is positive after costs, (b) profit factor ≥ 1.10, and (c) ^GSPC and ^NDX individually show positive expectancy. KS11 and EPOL results are reported but carry no pass/fail weight (sample size). No renegotiating after seeing results.
| 2026-07-23T13:27:44 | connors_dip_v1 | 7fc413ba24e9 | oos | SURVIVES | variant SURVIVES — family exp +0.3699%, PF 1.30, GSPC +0.112%, NDX +1.275% [rsi_period=4;threshold=10;exit=RSI>65;stop=nostop] |


---

---

## Protocol amendment — power check (effective 2026-07-23)

Every future pre-registration MUST include a power check: the projected OOS trade
count, derived from in-sample signal frequency (in-sample trades/year × OOS years).
If the projection is under ~100 family trades, the registration must state
explicitly that only FALSIFICATION is possible — a "survives" verdict on <100
trades is not evidence of an edge, only failure to reject. (Lesson from
connors_dip_v1: SURVIVED on ~81 OOS trades, recorded as survives-but-weak, shelved.)
| 2026-07-23T14:02:01 | stock_dip_v1 | c35fb2c7a8bb | insample | ok | pooled 96 cells; best exit/stop mean-exp +1.142% |


---

---

## Verdict log rows

| timestamp (UTC) | experiment | spec hash | phase | verdict | detail |
|---|---|---|---|---|---|
| 2026-07-23T12:47:26 | connors_dip_v1 | 7fc413ba24e9 | insample | ok | 10 markets; supports=0, rejects=6, mixed=4 |
| 2026-07-23T12:51:21 | connors_dip_v1 | 7fc413ba24e9 | insample | ok | 10 markets; supports=0, rejects=8, mixed=2 |
| 2026-07-23T13:27:44 | connors_dip_v1 | 7fc413ba24e9 | oos | SURVIVES | variant SURVIVES — family exp +0.3699%, PF 1.30, GSPC +0.112%, NDX +1.275% [rsi_period=4;threshold=10;exit=RSI>65;stop=nostop] |
| 2026-07-24T16:26:33 | connors_dip_v1 | 7fc413ba24e9 | insample | ok | 10 markets; supports=0, rejects=8, mixed=2 |
