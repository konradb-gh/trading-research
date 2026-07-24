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

More experiments will be added to this table as they're completed.

---

## What's here, and what isn't

This repo holds the **papers and figures** — the finished write-ups you can read on GitHub.

The **research code and data pipeline** that produce the numbers and charts live in a separate
project. This repository is publish-only: it's the reading room, not the lab.

---

## Disclaimer

This is a research project, not investment advice. All results are simulations using free
public data, net of modelled costs. No strategy described here is recommended for use with real
money, and the author holds no position based on any of it.
