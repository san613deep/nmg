# DECISIONS.md - decision & learnings log

A short running note of the real choices you made: what you tried, what failed and why, what
you changed. This is your engineering judgement on the record - it is what separates a builder
from a button-presser, and it is graded (from git history + this file + PROMPTS.md, NOT from
an auto audit log, which may be empty on cloud models).

Append a 1-2 line entry whenever you make a real decision or hit/fix a wall. Add a timestamp.

Format:
`[HH:MM] <decision or problem> -> <what you did and why>`

---

## Example (replace with your own)
- `[10:20]` Used `Unique Inlinks` for orphan detection, not `Inlinks` -> `Inlinks` counts
  duplicate links (nav appearing twice), so it never hits 0; `Unique Inlinks` is the real
  orphan signal.
- `[11:05]` Path-segment clustering merged unrelated root pages -> kept it as the starter but
  added TF keywords so the topic-agent can split/name them properly.
- `[12:40]` Dashboard not updating live -> server tool wasn't emitting the SSE event; added
  `_emit(...)` in each li_* tool.

---

## My log
- `[13:05]` Performed baseline end-to-end run before changing any code -> verified the starter project executed successfully, generated report.json, and provided a known-good starting point for future development and debugging.

- `[13:10]` Added report.json verification immediately after baseline execution -> confirmed JSON validity, required schema keys, summary metrics, and run metadata before starting feature development.

- `[14:10]` Baseline run: 221 pages, 201 indexable, 0 orphans, 73 broken — starter works end-to-end, all 9 schema keys present -> confirmed starting point before any changes.

- `[14:10]` Performed full schema compliance audit against report.schema.json -> verified required fields, nested structures, and data types before extending analysis logic.

- `[14:21]` Replaced manual schema inspection with automated validation script -> provides repeatable PASS/FAIL checks and catches missing fields or type mismatches before submission.

- `[14:21]` Added report contract regression testing -> future code changes can be verified quickly without manually inspecting report.json after every run.

- `[14:52]` Refactored over-linked page detection -> replaced complex percentile logic with an explicit 95th-percentile calculation based on Unique Inlinks for easier verification and maintenance.

- `[14:52]` Matched implementation directly to the published rulebook definition -> reduces risk of hidden-test discrepancies caused by interpretation differences.

- `[14:52]` Added verification of threshold value and qualifying URLs -> confirms the calculated top 5% pages are being selected correctly after the refactor.


- `[13:05]` Over-linked count differed from expectation -> inspected the complete Unique Inlinks distribution instead of relying on the final count alone.

- `[13:05]` Verified 95th-percentile threshold calculation -> confirmed that the threshold value matched the published rulebook formula.

- `[13:05]` Identified percentile boundary behavior -> multiple pages shared the threshold value, causing slightly more than 5% of pages to qualify, which is consistent with the rulebook requirement to include all pages at or above the threshold.

- `[13:15]` Added independent CSV-based verification for over-linked pages -> validates analyzer results against the original Screaming Frog export rather than trusting derived output.

- `[13:15]` Cross-checked percentile threshold calculation using raw data -> confirmed analyzer logic and source-data calculations produced identical results.

- `[13:15]` Established source-of-truth validation workflow -> graph-analysis changes must be verified against raw CSV data before being considered complete.