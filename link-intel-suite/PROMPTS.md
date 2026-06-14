# PROMPTS.md - my key prompts log

Keep the handful of prompts that actually moved the build. Not every message - the ones that
mattered: the system/sub-agent prompts, the ones you iterated on, the "this finally worked"
moment. Paste them here MANUALLY as you go.

Why manual? Some free Ollama cloud models do not save a local session log, so an auto audit
log may be empty. That is fine and expected (see the brief's Model Fairness section). What
guarantees your process is judged fairly is: the working plugin + reproducible report.json,
incremental git commits, this PROMPTS.md, and a short DECISIONS.md. Keep these up to date.

Format per entry:
- **Prompt** (paste it)
- **For:** what you were trying to do
- **Revised?** did you have to change it, and why

---

## Example (replace with your own)

- **Prompt:** "Extend linkintel/analyzer.py over_optimized_anchors: flag a destination where
  one non-generic anchor is >= 60% of all internal anchors pointing at it AND count >= 10.
  Run python linkintel/analyzer.py and show the counts."
- **For:** completing the over-optimized exact-match anchor rule
- **Revised?** Yes - first version flagged tiny destinations; added the count >= 10 floor.

---

## My prompts
1. - **Prompt:** "Read CLAUDE.md, rulebook.md, report.schema.json and the link-intel-suite README.md to understand the project. Then: (1) Run pip install mcp (2) Run cd link-intel-suite && python run.py ../sample-export/ (3) Show the full terminal output including the summary (4) Verify outputs/report.json exists and print its top-level keys (5) Do NOT change any code — this is a baseline check."
- **For:** Baseline project verification and environment validation before starting development.
- **Revised?** No. Used to confirm the starter bundle worked correctly and generated the expected outputs before making any code changes.

2. - **Prompt:** "Run `python run.py ../sample-export/` and then verify the output: (1) Is outputs/report.json valid JSON? Parse it with `python -c "import json; json.load(open('outputs/report.json'))"` (2) Does it have all 9 required top-level keys? Check: site, pages_crawled, summary, link_graph, anchor_text, topical_clusters, entity_graph, link_recommendations, run_meta (3) Print the summary object — are all 8 fields populated with integers? (4) Print run_meta — does it show model_calls and duration_sec? Report any schema violations."
- **For:** Output validation and schema compliance testing after the first successful end-to-end run.
- **Revised?** No. Used to verify that the generated report matched the required contract before extending the analysis logic.

3. - **Prompt:** "Load outputs/report.json and report.schema.json. For every required field in the schema: (1) Check if it exists in report.json (2) Check if the type matches (integer, string, array, object) (3) For nested required fields (summary, link_graph, anchor_text, etc.), check those too. Create a compliance table: | Field Path | Required | Present | Type OK | Issue |. Also print the baseline summary numbers so I can record them. Then append the first entry to DECISIONS.md: [HH:MM] Baseline run: X pages, Y indexable, Z orphans, W broken — starter works end-to-end, all 9 schema keys present -> confirmed starting point before any changes."
- **For:** Full schema compliance audit and baseline metrics recording before implementing new features.
- **Revised?** No. Used to establish a verified baseline and document the initial state of the starter project before development began.

4.- **Prompt:** "Write a quick Python script (run it inline, don't save) that validates report.json against report.schema.json by checking: (1) All required top-level keys exist (2) summary has all 8 required integer fields (3) link_graph has all required fields (4) anchor_text has all required fields (5) topical_clusters is a non-empty array where each item has key, size, pages, authority (6) entity_graph is an object (can be empty) (7) link_recommendations is an array where each item has source, target, suggested_anchor (8) run_meta has model_calls. Print PASS or FAIL with detailed validation results."
- **For:** Automated schema validation and regression testing of report.json before making further changes.
- **Revised?** Yes. Initially relied on manual inspection of report.json, then switched to an automated validation script to catch missing fields and type mismatches consistently.

5. - **Prompt:** "Read rulebook.md section A, rule over_linked_page: 'page in the top 5% by Unique Inlinks (sitewide nav/footer noise)'. Now read linkintel/analyzer.py lines 175-178. The current implementation is convoluted with a double list comprehension and unclear logic. Rewrite it clearly: (1) Collect all Unique Inlinks values for indexable 200 HTML pages (2) Sort them ascending (3) Calculate the 95th percentile index: idx = math.ceil(len(vals) * 0.95) - 1 (4) The threshold is vals[idx] (5) Return all pages whose Unique Inlinks >= that threshold (6) Handle edge case: if threshold is 0 or 1, still return the top 5%. Replace ONLY lines 175-178. Keep the rest of graph_stats() unchanged. Run `python linkintel/analyzer.py ../sample-export/` and show the over_linked count. Print the threshold value and the qualifying URLs for verification."
- **For:** Refactoring over-linked page detection and aligning the implementation with the published rulebook definition.
- **Revised?** Yes. The starter implementation used difficult-to-read percentile logic. Replaced it with an explicit percentile calculation and verification output to ensure correctness against the grading rules.


6.- **Prompt**: "After fixing over_linked_pages, the count is [N] but I expected something different. Debug: (1) Print ALL Unique Inlinks values sorted: vals = sorted([_int(p.get('Unique Inlinks')) for p in idx200]) (2) Print len(vals) and the 95th percentile index (3) Print the threshold value: vals[math.ceil(len(vals) * 0.95) - 1] (4) Print which pages qualify and their Unique Inlinks values (5) Is the issue that many pages share the threshold value? The rulebook says 'top 5%' so all pages AT OR ABOVE the 95th percentile threshold should be included, even if that means slightly more than 5% of pages."
- **For**: Investigating unexpected over-linked page counts after implementing percentile-based threshold logic.
- **Revised**? Yes. Initial validation focused only on the final count. Expanded the prompt to inspect the full Unique Inlinks distribution, percentile threshold, and qualifying pages to explain the result.

7. - **Prompt**: "Verify over_linked_pages by cross-checking against the raw CSV: (1) Read internal_html.csv, filter to Content Type contains text/html, Status Code 200, Indexability == Indexable (2) Get all Unique Inlinks values, sort them (3) Calculate the 95th percentile threshold (4) Count how many pages are at or above it (5) Compare with the analyzer's output. Run both and show the comparison. They must match exactly."
- **For**: Independent verification of over-linked page detection using raw crawl data rather than analyzer-generated results.
- **Revised**? Yes. After validating the analyzer logic internally, added a direct CSV cross-check to ensure the implementation matched the source data and rulebook calculation exactly.

8. - **Prompt:** "Verify orphan detection in linkintel/analyzer.py line 164: (1) The rulebook says: orphan_page = 'indexable 200 HTML page with Unique Inlinks == 0' (2) Read the code at line 164 — does it use 'Unique Inlinks' (correct) or 'Inlinks' (wrong)? (3) Read the CSV header of internal_html.csv — what is the exact column name? (case matters) (4) Cross-check: filter internal_html.csv manually for Content Type contains 'text/html', Status Code == 200, Indexability == 'Indexable', Unique Inlinks == 0 (5) Compare the count with analyzer output. If there's a mismatch, fix line 164. If it matches, confirm it's correct and move on. Also check: does the Unique Inlinks column ever have empty/null values that _int() would convert to 0 (false orphan)?"
- **For:** Validating orphan-page detection against the rulebook and raw crawl data while checking for edge cases caused by missing values.
- **Revised?** Yes. Initial verification only compared counts. Expanded the validation to inspect the exact CSV column name, implementation details, and potential false positives caused by null values.

9. - **Prompt**: "Write an inline cross-check that independently computes orphans from internal_html.csv: import csv with open('../sample-export/internal_html.csv', encoding='utf-8-sig') as f: rows = list(csv.DictReader(f)) orphans = [r['Address'] for r in rows if 'text/html' in (r.get('Content Type','') or '').lower() and r.get('Status Code','').strip() == '200' and (r.get('Indexability','') or '').strip().lower() == 'indexable' and int(float(r.get('Unique Inlinks','0') or '0')) == 0] print(f'Manual orphan count: {len(orphans)}') for u in sorted(orphans): print(f' {u}') Compare this with the analyzer's orphan list. They must match exactly."
- **For**: Independent verification of orphan-page detection using a raw CSV calculation separate from the analyzer implementation.
- **Revised**? Yes. After verifying the orphan rule implementation, added an independent calculation to ensure analyzer results matched the source crawl data exactly.

10. - **Prompt**: "Cross-check the broken/redirect/nofollow link detection in analyzer.py (lines 181-195) against the raw CSV: (1) Read all_inlinks.csv (2) Filter to Type == 'Hyperlink' only (3) Count rows where Status Code is 400-599 (broken) — compare with analyzer (4) Count rows where Status Code is 300-399 (redirect) — compare with analyzer (5) Count rows where Follow is 'false' (case-insensitive) — compare with analyzer. Pay attention to edge cases: Empty or missing Status Code values — what does _int() return? The Follow column: is it 'true'/'false' or 'True'/'False'? Is it ever empty? Are there duplicate rows? Should we deduplicate? Report mismatches. If all match, confirm and move on."
- **For**: Independent validation of broken, redirect, and nofollow internal-link detection against the raw all_inlinks.csv export.
- **Revised**? Yes. Initial verification compared only aggregate counts. Expanded the audit to inspect edge cases, CSV value formats, null handling, and duplicate-link behavior to ensure rulebook compliance.

11. - **Prompt:** "Write an inline cross-check for all three link issue types: import csv with open('../sample-export/all_inlinks.csv', encoding='utf-8-sig') as f: rows = list(csv.DictReader(f)) hyper = [r for r in rows if r.get('Type','').strip() == 'Hyperlink'] broken = [r for r in hyper if 400 <= int(float(r.get('Status Code','0') or '0')) <= 599] redir = [r for r in hyper if 300 <= int(float(r.get('Status Code','0') or '0')) <= 399] nofol = [r for r in hyper if (r.get('Follow','') or '').strip().lower() == 'false'] print(f'Hyperlink rows: {len(hyper)}') print(f'Broken (4xx-5xx): {len(broken)}') print(f'Redirect (3xx): {len(redir)}') print(f'Nofollow: {len(nofol)}') Compare with analyzer output. All must match."
- **For:** Independent verification of broken, redirect, and nofollow link metrics using direct calculations from all_inlinks.csv.
- **Revised?** Yes. After validating the detection logic conceptually, added a raw CSV cross-check to ensure analyzer counts matched the source export exactly.

12. - **Prompt:** "Review the _norm() function in linkintel/analyzer.py (lines 46-53). The rulebook says: 'Normalize URLs before matching: drop the fragment and a single trailing slash.' Current implementation strips fragment (#...) and trailing slash. Verify it handles: (1) URLs with query strings: https://example.com/page?foo=bar — should KEEP the query string (2) URLs with fragments: https://example.com/page#section — should strip #section (3) URLs with trailing slash: https://example.com/page/ → https://example.com/page (4) Root URL: https://example.com/ → should stay as https://example.com (not empty string) (5) URLs with encoded characters: https://example.com/page%20name (6) Empty strings and None values. Write a quick test of _norm() with these cases. If any fail, fix the function. Also: scan all unique URLs in internal_html.csv Address column and all_inlinks.csv Source/Destination columns. Do any URLs fail normalization?"
- **For:** Verifying URL normalization logic against rulebook requirements and testing edge cases that could affect graph matching accuracy.
- **Revised?** Yes. Initial review focused on code inspection. Expanded the validation to include explicit test cases and a full scan of real crawl URLs to detect normalization issues in production data.

13. - **Prompt:** "Test URL normalization exhaustively: from linkintel.analyzer import _norm tests = [('https://example.com/page#section', 'https://example.com/page'), ('https://example.com/page/', 'https://example.com/page'), ('https://example.com/', 'https://example.com'), ('https://example.com/page?q=1', 'https://example.com/page?q=1'), ('https://example.com/page?q=1#s', 'https://example.com/page?q=1'), ('', ''), ('https://example.com', 'https://example.com')] for inp, expected in tests: result = _norm(inp) status = '✅' if result == expected else '❌' print(f'{status} _norm({inp!r}) = {result!r} (expected {expected!r})')"
- **For:** Unit testing the URL normalization helper function against rulebook-defined behavior and common URL edge cases.
- **Revised?** Yes. After reviewing the implementation, added explicit pass/fail test cases to verify normalization behavior rather than relying on manual inspection.
