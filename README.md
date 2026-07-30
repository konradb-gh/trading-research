# Trading Rules Research

Testing published trading ideas honestly — one experiment at a time, with trading costs
charged, and results published whether they work or not.

Plenty of trading strategies look good on paper. A lot of that shine comes from things that
quietly flatter a backtest: costs left out, results reported before the fees that would have
eaten them, strategies tuned until they fit the past, and the failures never mentioned. This
project takes well-known, published ideas and puts each one through a test designed to strip
those flattering effects away — then writes up what's left, good or bad.

Every number in every paper here is **after realistic trading costs**. A strategy of many
small, fast wins can look like an edge gross and be underwater net, and that distinction is
usually the whole story.

---

## How each experiment is run

The method is borrowed from clinical research, and every rule in it exists to stop us fooling
ourselves:

- **Split the history in two.** Everything up to a fixed cutoff is the *exploration* half; the
  rest is *sealed*. You need one body of data to form ideas on and a separate one to test them,
  or you're just grading your own homework.

- **Explore freely — but only on the older half.** Search as many variations as you like, look
  at everything, follow hunches. That's what exploration data is for. The sealed half stays
  untouched so it can still surprise you.

- **Before touching the sealed data, write the test down in advance** — the exact rule *and*
  the exact pass/fail criteria. Committing to what would count as success *before* you see the
  outcome is the only thing that separates a real result from a story told after the fact.

- **The criteria are binding.** No renegotiating a threshold because the result landed just
  short, no adding a caveat that happens to rescue it. If it fails the bar it set for itself,
  it failed — moving the bar afterwards is exactly the mistake this whole method exists to
  prevent.

- **Failures get published too.** A strategy that doesn't clear its costs is a genuine finding,
  and burying it is how the field ends up full of ideas that never worked. The write-up is the
  same either way.

---

## The experiments

| # | Question | Instrument | Verdict | |
|---|---|---|---|---|
| 01 | Does buying the dip work on stock indices? | Indices | Survives, but fires too rarely to be usable | [read →](experiments/01-index-dip-buying/) |
| 02 | Does dip-buying work on quality stocks? | Individual stocks | Real behaviour, but the edge ≈ its costs once survivorship is removed | [read →](experiments/02-quality-stock-dip-buying/) |
| 03 | Does momentum work? | Indices and stocks | Complete negative — and a control that flattered it | [read →](experiments/03-momentum/) |
| 04 | What governs blow-up risk in our own screener? | Our live system | Account size is the binding constraint; most position-sizing rules never fire | [read →](experiments/04-system-risk/) |

More experiments will be added to this table as they're completed.

Across the first three, the pattern is the same: the behaviour is real enough, and the cost of
capturing it is about the same size as the edge. The fourth is a different kind of piece — it turns
the same method inward, auditing the risk profile of the screener this project built rather than a
textbook idea, and finds that at a permanently small account the risk that matters lives in the
size of the account, not in the rules you can tune.

---

## How to read the numbers

Three caveats apply to every paper here. None of them are footnotes — they change what the
figures mean.

**Absolute returns are upper bounds, not estimates.** Free data can only see part of any
index's true historical membership, and the part it can't see skews toward companies that
failed and were delisted. Coverage in these experiments runs from roughly 40% in the late
1990s to about 90% in recent years. Every absolute return figure — for the strategies *and* for
the benchmarks they're measured against — is therefore inflated by an unknown amount.

**Relative comparisons are the trustworthy output.** Each strategy is compared against a
benchmark built from the *identical* universe with the *identical* screens, so both sides carry
the same distortion and it largely cancels. When a paper concludes something, the conclusion
rests on a comparison, never on an absolute number.

**Costs are modelled, not measured.** Every result is net of 0.10% slippage per side plus 0.20%
round-trip commission, applied uniformly. No real broker's fills were used, no market-impact
model is included, and financing costs are discussed rather than charged in most tables. Real
execution would differ.

---

## What's here, and what isn't

This repo holds the **papers, figures and the data behind them** — the finished write-ups you
can read on GitHub, plus a `data/` folder in each experiment containing the results tables,
coverage tables, universe lists, exclusion lists and verbatim pre-registrations. Each has a
`DATA.md` describing every file, the column definitions, the sources with URLs, the screens
applied and the cost model.

Raw vendor price history is deliberately **not** published: it's large, and redistributing it
is a licensing grey area. The exact ticker lists, date ranges and interval are published
instead, so identical inputs can be re-downloaded.

The **research code and data pipeline** that produce the numbers and charts live in a separate
project. This repository is publish-only: it's the reading room, not the lab.

---

## Disclaimer

This is a research project, not investment advice. All results are simulations using free
public data, net of modelled costs. No strategy described here is recommended for use with real
money, and the author holds no position based on any of it.
