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

- `[13:53]` Audited orphan-page detection against the published rulebook -> verified that orphan status is based on Unique Inlinks rather than Inlinks.

- `[13:53]` Cross-checked analyzer output against raw CSV filtering -> ensured orphan counts were derived from the same conditions used by the grader.

- `[13:53]` Investigated null-value handling for Unique Inlinks -> checked whether empty values could be converted to zero and incorrectly classified as orphan pages.

- `[13:40]` Added independent orphan-page calculation from internal_html.csv -> verifies analyzer output against the raw crawl export rather than relying on internal implementation alone.

- `[13:40]` Compared orphan URLs, not just orphan counts -> ensures the analyzer identifies the exact same pages as the source-data calculation.

- `[13:40]` Established URL-level validation for graph metrics -> matching counts alone are insufficient; page lists must also match exactly.

- `[13:44]` Audited broken, redirect, and nofollow detection against raw all_inlinks.csv data -> verified analyzer results independently from source exports.

- `[13:44]` Confirmed graph-link analysis uses Hyperlink rows only -> excluded Canonical, CSS, JavaScript, Redirect, and other non-content link types from validation.

- `[13:44]` Investigated Status Code and Follow edge cases -> verified handling of empty values, case sensitivity, and integer conversion behavior before accepting counts as correct.

- `[13:44]` Checked duplicate link rows during validation -> confirmed whether analyzer logic should count rows exactly as exported or apply deduplication.

- `[13:51]` Added direct CSV validation for broken, redirect, and nofollow metrics -> verifies analyzer counts against the original Screaming Frog export.

- `[13:51]` Used Hyperlink-only filtering for validation -> matches the rulebook definition and avoids counting non-link asset rows.

- `[13:51]` Established independent count verification for link issues -> analyzer metrics must match raw CSV calculations exactly before considering the implementation complete.

- `[13:51]` Validated all three link-issue categories together -> reduced the risk of inconsistent filtering logic across broken, redirect, and nofollow checks.

- `[13:54]` Audited URL normalization logic against rulebook requirements -> verified fragment removal and trailing-slash handling before relying on URL matching throughout the graph analysis.

- `[13:54]` Added explicit edge-case tests for _norm() -> covered query strings, fragments, root URLs, encoded characters, empty values, and None inputs.

- `[13:54]` Validated normalization against real crawl data -> scanned Address, Source, and Destination URLs from exports to identify potential matching failures not covered by synthetic tests.

- `[13:54]` Treated URL normalization as a foundational dependency -> graph analysis, orphan detection, and link recommendations all depend on consistent URL matching.

- `[13:59]` Added unit-test style validation for _norm() -> verifies normalization behavior automatically instead of relying on visual code review.

- `[13:59]` Tested fragment removal, trailing-slash handling, root URLs, and query-string preservation -> confirms compliance with rulebook URL normalization requirements.

- `[13:59]` Used expected-output assertions for URL normalization -> makes future regressions immediately visible when modifying helper functions.

- `[14:09]` Audited GENERIC_ANCHORS against the published rulebook -> verified all mandatory anchor phrases were present before extending the set.

- `[14:09]` Expanded generic-anchor coverage with additional non-descriptive phrases -> improves detection of weak internal anchor text without relying solely on the starter list.

- `[14:09]` Excluded borderline descriptive anchors from the generic set -> avoided increasing false positives by classifying potentially meaningful anchors as generic.

- `[14:09]` Compared generic-anchor counts before and after the update -> verified the impact of the expanded classification rules on real crawl data.

- `[14:20]` Performed full topical-cluster audit -> evaluated cluster size distribution, authority classification, and page coverage rather than relying on cluster counts alone.

- `[14:20]` Added authority verification against member-page inlink distributions -> ensured 'hub' vs 'scattered' labels matched the published rulebook definition.

- `[14:20]` Flagged oversized and undersized clusters -> identified candidates for splitting overly broad topics and merging weak clusters.

- `[14:20]` Verified cluster coverage against indexable pages -> ensured every eligible page belonged to exactly one cluster and no pages were silently excluded.

- `[14:20]` Compared clustered-page totals against indexable-page totals -> used coverage reconciliation as a quality gate before improving topic classification.

- `[14:25]` Identified limitation in path-segment clustering -> large sections such as blog categories grouped unrelated content into a single cluster.

- `[14:25]` Added keyword-overlap refinement after initial URL-based grouping -> combines structural signals with content similarity for more meaningful topical clusters.

- `[14:25]` Implemented greedy sub-clustering for oversized groups -> reduces broad clusters without introducing expensive clustering algorithms.

- `[14:25]` Added singleton-cluster merging using keyword similarity -> prevents isolated pages from forming weak topical groups when a clear related cluster exists.

- `[14:25]` Preserved cluster coverage invariant -> ensured every indexable page remains assigned to exactly one cluster after refinement.

- `[14:25]` Recomputed hub and authority metrics after re-clustering -> authority signals must reflect the final cluster structure, not the original grouping.

- `[16:30]` Improved keyword extraction weighting in `page_keywords()` -> shifted from equal weighting of all sources to weighted multipliers: Title (3x), H1 (3x), H2s (2x), and Body (1x). This ensures that high-signal areas (titles/headers) have a stronger influence on topical clustering and entity relatedness. Verified shift in top-12 keywords across 5 sample pages and confirmed no pipeline regressions.