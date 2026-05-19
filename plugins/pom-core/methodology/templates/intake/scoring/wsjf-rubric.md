# WSJF Scoring Rubric

> **SCAFFOLD — anchors to be defined.** Use `pom-bootstrap --rubric=interactive` to calibrate, or copy an industry preset from `the pom-core plugin's methodology/rubrics/`.

This file is the scoring contract for every use case in [`../use-case-backlog/`](../use-case-backlog/). All use cases are scored using this rubric — no exceptions, no parallel scoring systems.

## The formula

```
WSJF = Cost of Delay ÷ Job Size

Cost of Delay = User-Business Value
              + Time Criticality
              + Risk Reduction & Opportunity Enablement
```

Each component is scored on a **modified Fibonacci scale**: `1, 2, 3, 5, 8, 13, 21`.

Higher WSJF = higher priority. The number itself has no absolute meaning — only relative ordering matters.

## Calibration principle

Every step on the Fibonacci scale is approximately **2× the previous step**. If a "5" example doesn't feel meaningfully larger than a "3" example and meaningfully smaller than an "8", the score is miscalibrated. When in doubt, score lower.

## Rubric (TO BE DEFINED)

### User-Business Value (1–21)

| Score | Definition |
|---|---|
| 1 | _TBD_ |
| 2 | _TBD_ |
| 3 | _TBD_ |
| 5 | _TBD_ |
| 8 | _TBD_ |
| 13 | _TBD_ |
| 21 | _TBD_ |

### Time Criticality (1–21)

| Score | Definition |
|---|---|
| 1 | _TBD_ |
| 2 | _TBD_ |
| 3 | _TBD_ |
| 5 | _TBD_ |
| 8 | _TBD_ |
| 13 | _TBD_ |
| 21 | _TBD_ |

### Risk Reduction & Opportunity Enablement (1–21)

| Score | Definition |
|---|---|
| 1 | _TBD_ |
| 2 | _TBD_ |
| 3 | _TBD_ |
| 5 | _TBD_ |
| 8 | _TBD_ |
| 13 | _TBD_ |
| 21 | _TBD_ |

### Job Size (1–21)

Effort to deliver end-to-end through a pod, *including* discovery and runway work. Not story points.

| Score | Definition |
|---|---|
| 1 | _TBD_ |
| 2 | _TBD_ |
| 3 | _TBD_ |
| 5 | _TBD_ |
| 8 | _TBD_ |
| 13 | _TBD_ |
| 21 | _TBD_ |

## Rules

1. **Scoring is done by the Product Trio** of the most likely receiving product, with input from the Product Enablement Team.
2. **Scores are relative**, not absolute. Calibrate by comparing to existing use cases.
3. **Re-score on disposition.** A use case parked > 1 quarter is re-scored before re-entering the active backlog.
4. **A score of 21 is a smell.** Job Size = 21 is hard-prohibited — split first.
5. **Cost of Delay can exceed 21 per item** (sum of three dimensions, each up to 21 → max 63).
6. **Never compute WSJF in isolation.** Always score at least 3 use cases together.
