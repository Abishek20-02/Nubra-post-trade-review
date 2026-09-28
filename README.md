# Post-Trade Review: turning a month of order history into habits worth changing

A tool that reads a retail trader's monthly order history and surfaces at most 3 specific, verified habits that are costing them money, each with one real number as evidence and one concrete rule to try next month.

**Live demo:** https://dulcet-selkie-54917a.netlify.app
Click **"a typical month"** or **"a clean month"**. No upload or signup needed.

Built as a take-home for a Product Management Intern role at Nubra (a trading platform). The full write-up is in [`Nubra_Submission.pdf`](Nubra_Submission.pdf): one-pager, prompts, evals and app notes.

<!-- Add a screenshot: ![Post-Trade Review screenshot](screenshot.png) -->

## The problem

Most retail traders never review their own trades. Brokers already show P&L per trade, so this is not a reporting problem. It is a **noticing** problem: someone needs to connect the dots across trades and say it back in a way that feels true, not generic. The brief also warned against building "another dashboard."

## What it detects

Three habits, each with an explicit numeric threshold so the tool does not invent findings:

| Habit | Flagged only if |
|---|---|
| Cutting winners early, holding losers | Losing trades held at least 2x longer than winners, with at least 3 trades of each type |
| Oversized trades right after a loss | A trade at least 1.5x average size within 60 minutes of a loss, at least 3 times in the month |
| Clusters of quick, near break-even trades | 4+ trades in a day averaging under 30 minutes with P&L within +/-1%, on at least 2 different days |

If nothing meets its threshold, the tool says so instead of forcing a finding.

**Success was defined before any prompt was written:** precision over recall. A false finding is worse than a missed one, because a wrong habit destroys a trader's trust.

## Results (tested against ground truth)

| Check | Result |
|---|---|
| Disposition pattern (August dataset) | 13.2x, avg 20.4h vs 1.5h. Exact match to ground truth |
| Revenge pattern | 5 instances. Exact match |
| Overtrading pattern | 15 trades across 3 days. Exact match |
| Clean control month (no injected patterns) | 0 findings, correctly |
| Borderline case (loser held 1.8x vs 2x threshold) | Not flagged, as intended |

## The bug that shaped the design

The first version asked the model to reconstruct trades, run the statistics and write the prose in one pass. Testing on a real 52-trade tradebook showed the model confidently reporting a **4.7x** ratio when the true figure was **13.2x**. It also sometimes broke JSON output with a preamble, even when told not to.

**Fix:** trade reconstruction and all threshold checks moved into deterministic JavaScript. The model now only writes plain-language sentences around numbers that are already computed and verified, and is told never to recompute or round them.

That rewrite then exposed **two bugs in my own code**, caught only by comparing output line by line against ground truth:

- Revenge trading showed 6 instead of 5, because one oversized trade was matched against two prior losses.
- Overtrading showed 17 trades instead of 15, because two revenge trades that were also quick got counted twice.

Lesson: moving logic out of the model only makes it trustworthy if the code is then checked just as rigorously.

## Reliability: it never shows an error screen

The sentence-writing step tries three paths in order:

1. Claude's built-in capability, when opened through its Claude artifact link.
2. A user-supplied Anthropic API key, kept in memory only for the session.
3. Hand-written template sentences, used by default when neither is available.

All three use the same verified numbers. Only the wording differs.

## How it's built

- Single self-contained HTML file with client-side JavaScript, hosted on Netlify
- No backend, no signup
- Trade parsing, pattern detection and thresholds all run in the browser
- AI is used only for the final wording

## Known limitations

- Trade matching assumes one BUY and one SELL per order ID, which fits this dataset. Real tradebooks with partial fills or multiple lots would need FIFO lot matching.
- Boundary conditions (a ratio of exactly 2x, a gap of exactly 60 minutes) are not yet explicitly tested.
- Evaluated on one engineered dataset and one clean control, so there is no precision/recall percentage yet.
- Equity only. F&O behavior, cross-month tracking and live broker integration are out of scope.

## Repository contents

- `index.html`: the app
- `Nubra_Submission.pdf`: one-pager, prompts, evals and app write-up

## Run locally

Open `index.html` in a browser. Use the two sample-month buttons to try it.

## Author

S. Abishek Karthikeyan
LinkedIn: https://www.linkedin.com/in/s-abishek-karthikeyan-8020b622b
