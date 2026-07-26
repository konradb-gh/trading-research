<!-- Extracted verbatim from research/journal.md in the source project.
     Pre-registrations are written BEFORE their run and never edited afterwards. -->

# Experiment 02 — pre-registration and verdicts (verbatim)

## Pre-registration — stock_dip_v1 OOS test  (appended 2026-07-23T14:29:55, BEFORE the run)

**Hypothesis:** short-term oversold entries on Trend-Template-eligible S&P 500 stocks, held until short-term strength returns, carry positive expectancy after costs (pre-declared from Exp 1's emergent finding + this in-sample surface).

**Variant:** RSI period 2, threshold 10, exit RSI>65, market gate ^GSPC>200SMA, template eligibility as-of D — round numbers, plateau center. Primary arm: no-stop (as the surface favors). Secondary arm: 2×ATR(14) (deployment reality).

**Power check:** projected ~2,000 OOS trades — confirmation-grade, passing the amendment.

**Binding kill criteria on the primary arm, OOS 2013→2026:** (a) pooled expectancy > 0 after costs; (b) pooled PF ≥ 1.10; (c) expectancy > 0 in both halves of the OOS window (2013–2019 and 2020–2026) — the edge must exist across eras, not ride one regime.

**Standing caveat, declared now:** even a full pass remains survivorship-inflated by an unknowable amount; a pass validates the phenomenon's direction and persistence, not the magnitude.
| 2026-07-23T14:44:49 | stock_dip_v1 | c35fb2c7a8bb | oos | SURVIVES | variant SURVIVES — pooled exp +0.2757%, PF 1.26, halves +0.263%/+0.290% [rsi_period=2;threshold=10;exit=RSI>65;stop=nostop] |


---

---

## Survivorship correction — stock_dip_v1 (point-in-time universe)  (2026-07-23)

Robustness re-test of the UNCHANGED pre-registered variant on point-in-time S&P 500
membership (fja05680/sp500, 1996-2026). Coverage: in-sample 52% (LOW-CONFIDENCE),
OOS 81%. Artifact guard on; trades restricted to member-days.

nostop OOS expectancy: current-constituents +0.276% -> PIT-corrected **+0.185%** (-0.091, ~1/3 removed).
2xATR OOS expectancy:  +0.107% -> **+0.045%**.  PIT OOS PF 1.18 (nostop) / 1.04 (2xATR).
Per-year + both halves stay positive (2013-19 ~+0.15%, 2020-26 ~+0.18%): direction/persistence SURVIVE.

BRACKET: true OOS edge < +0.185% (19% of members — the worst dip-buys — still missing).
Net of ~0.10%/trade financing: no-stop ~break-even-to-marginal (~+0.05-0.09%); the deployable
2xATR arm is ~break-even-to-negative. The +0.28% was substantially survivorship makeup.
VERDICT: phenomenon REAL + PERSISTENT in direction, but the TRADEABLE magnitude does not clear
cost+financing with room to spare. Not standalone-deployable. Recorded honestly.
| 2026-07-24T16:26:33 | connors_dip_v1 | 7fc413ba24e9 | insample | ok | 10 markets; supports=0, rejects=8, mixed=2 |


---

---

## Verdict log rows

| timestamp (UTC) | experiment | spec hash | phase | verdict | detail |
|---|---|---|---|---|---|
| 2026-07-23T14:02:01 | stock_dip_v1 | c35fb2c7a8bb | insample | ok | pooled 96 cells; best exit/stop mean-exp +1.142% |
| 2026-07-23T14:44:49 | stock_dip_v1 | c35fb2c7a8bb | oos | SURVIVES | variant SURVIVES — pooled exp +0.2757%, PF 1.26, halves +0.263%/+0.290% [rsi_period=2;threshold=10;exit=RSI>65;stop=nostop] |
