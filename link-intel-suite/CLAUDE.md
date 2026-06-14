# CLAUDE.md - project memory for the Link Intel Suite build

This file is your **context / memory for the AI**. Claude Code loads it automatically every
session. Strong builders engineer this file instead of re-explaining everything in chat - it
is one of the clearest signals of good practice, and it is graded (see the build brief
section on process). Keep it short, specific, and update it as you learn.

## What we are building
A Claude Code plugin that ingests a Screaming Frog export (`internal_html.csv` +
`all_inlinks.csv` + `all_outlinks.csv` + `all_anchor_text.csv` + a `page text/` folder) and
produces an **internal-linking + topical-authority** analysis: the internal link graph,
anchor-text issues, topical clusters, an entity graph, and **contextual internal-link
recommendations**. It serves a live dashboard at localhost:7700 and outputs
`outputs/report.json` + `outputs/report.html`.

## Hard rules (the agent must follow these)
- Do the graph, orphan detection, anchor classification and relatedness math in **plain
  Python** (`linkintel/analyzer.py`). Use the model ONLY for: extracting entities per page,
  naming clusters, and writing the contextual link suggestions + anchors. Never feed raw
  crawl rows to the model.
- `outputs/report.json` MUST match `report.schema.json`. Validate before declaring done.
- Pre-filter to `text/html` + 200 + Indexable for page-level checks; use `Type == Hyperlink`
  rows for link-level checks (see `rulebook.md`).
- Do not hard-code anything to the sample export - it must work on an unseen export with the
  same column shape.
- Keep model calls small and few (free-tier / cloud quota). One page per entity/anchor call.

## Architecture (keep it real)
- `skills/link-intel/SKILL.md` orchestrates. Sub-agents: `graph-agent`, `anchor-agent`,
  `topic-agent`, `linker-agent`, `reporter`.
- `linkintel/analyzer.py` = deterministic analysis (extend it - biggest score).
- `mcp/server.py` = MCP tools + the live dashboard host.

## Conventions
- Commit after each working step with a real message.
- Run `python run.py sample-export/` to test end to end.

## Things I have learned during the build (update this as you go)
- (e.g. "page text filenames are URL-encoded with an `original_https_` prefix - decode before
  matching to Address")
- (e.g. "orphans = `Unique Inlinks` == 0, NOT `Inlinks` == 0 - Inlinks counts repeated links")
- Always run automated report.json validation after each end-to-end execution; manual inspection is insufficient for detecting schema regressions.

- Over-linked pages must be determined from the distribution of Unique Inlinks across indexable 200 HTML pages.
- Prefer explicit percentile calculations over compressed list comprehensions when implementing grading rules.
- After modifying graph-analysis rules, verify both the threshold value and the resulting URLs to ensure rulebook compliance.

- When validating percentile-based rules, inspect the full metric distribution rather than only the final count.
- Multiple pages can share the 95th-percentile threshold value; include all pages at or above the threshold even if the result exceeds exactly 5% of pages.
- Verification should include threshold value, qualifying URLs, and their metric values to explain unexpected counts.

- Validate graph-analysis rules against raw Screaming Frog exports whenever possible, not only against analyzer outputs.
- Treat internal_html.csv as the source of truth for page-level metrics such as Unique Inlinks, Crawl Depth, and Indexability.
- A graph-analysis implementation is considered correct only when analyzer results match an independent calculation from the raw CSV.

- Orphan pages must be calculated using the exact "Unique Inlinks" column, never "Inlinks".
- Verify CSV column names directly before implementing rulebook logic; similar column names can produce incorrect results.
- Check for empty or null metric values before relying on integer conversion, as they can create false positives in rule-based analysis.

- Validate orphan-page detection using an independent calculation from internal_html.csv whenever the logic changes.
- Compare both counts and URL lists; matching totals alone do not guarantee correctness.
- Treat raw CSV calculations as the final verification layer for graph-analysis rules.

- Broken, redirect, and nofollow link checks must operate on Hyperlink rows only from all_inlinks.csv.
- Verify raw CSV value formats before implementing rules (e.g., Follow may vary in capitalization or contain empty values).
- Always inspect null-handling behavior when using _int() on Status Code fields to avoid incorrect classifications.
- Validate link-analysis outputs against raw CSV counts before considering a rule complete.

- Link-issue metrics (broken, redirect, nofollow) should be validated with independent calculations from all_inlinks.csv.
- Use identical Hyperlink filtering in both analyzer logic and verification scripts to avoid count discrepancies.
- For rulebook-based metrics, analyzer outputs must match raw CSV counts exactly before the implementation is considered correct.

- URL normalization must remove fragments (#...) while preserving query strings.
- Remove a single trailing slash but preserve valid root URLs (e.g. https://example.com/ → https://example.com).
- Validate normalization using both synthetic edge cases and real URLs from crawl exports.
- Graph-analysis accuracy depends on consistent normalization across Address, Source, and Destination URL fields.

- Every URL normalization change should be validated with explicit input/output test cases.
- Query strings must be preserved while fragments are removed.
- Helper functions that affect graph matching should have regression tests before modification.

- Generic-anchor detection must include all mandatory rulebook phrases plus sensible non-descriptive extensions.
- Prefer precision over aggressive expansion; avoid classifying potentially meaningful anchors as generic.
- Validate classification changes by comparing generic-anchor counts before and after modifications.
- Anchor classification rules should be tested against real crawl exports, not only static term lists.