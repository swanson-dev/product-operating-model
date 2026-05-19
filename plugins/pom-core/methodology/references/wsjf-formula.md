# WSJF — Formula & Scoring Rules

The single prioritization mechanism for the Use Case Backlog.

## Formula

```
WSJF = Cost of Delay ÷ Job Size

Cost of Delay = User-Business Value
              + Time Criticality
              + Risk Reduction & Opportunity Enablement
```

Each dimension scored on **modified Fibonacci**: `1, 2, 3, 5, 8, 13, 21`.

Higher WSJF = higher priority. The absolute number has no meaning — only the **relative ordering** of items in the same backlog matters.

## Calibration principle

Every step on the Fibonacci scale is approximately **2× the previous step**. If a "5" example doesn't feel meaningfully larger than a "3" example and meaningfully smaller than an "8", the score is miscalibrated.

When in doubt: **score lower**. Rubrics inflate over time, not deflate.

## The 21-smell rule

A score of **21 on any dimension** is a flag:

- **Cost of Delay dimensions at 21** → ask "is this really one use case?" before continuing.
- **Job Size = 21** → **hard-prohibited.** Split the use case into vertical slices first. Each slice gets its own scoring.

## Mandatory rules

1. **Scoring is done by the Product Trio** of the most likely receiving product, with input from the Product Enablement Team. If routing is ambiguous, PMs of candidate products score together.
2. **Scores are relative**, not absolute. Always compare to use cases already in the backlog.
3. **Re-score on disposition.** A use case parked > 1 quarter is re-scored before re-entering the active backlog. Time Criticality especially decays in both directions.
4. **Never compute WSJF in isolation.** Always score at least 3 use cases together so anchors stay relative.
5. **Cost of Delay can exceed 21 per item** (sum of three dimensions, each up to 21 → max 63). This is intentional — gives meaningful spread when divided by Job Size.

## Industry-preset rubrics

See [`../rubrics/`](../rubrics/) for calibrated rubrics by industry (insurance, fintech, healthcare, retail, generic).

## Calibration triggers

Re-calibrate the rubric when:

- **After 10 use cases have been scored** — the Product Enablement Team reviews for clustering or drift.
- **On regulatory change** — new state rule, NAIC update, federal action that changes Time Criticality dynamics.
- **On first regulatory examination** touching the portfolio — review against examiner feedback.
