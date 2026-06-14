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

4.

5.