---
name: issue-to-eval
description: Convert a GitHub issue (labeled 'benchmark') into a test case in the skill's evals.json. Use when someone says "add this issue to evals", "import benchmark issue", "sync benchmarks", or wants to convert a GitHub issue into a test case.
---

# Issue-to-Eval Converter

Convert GitHub issues labeled `benchmark` into structured test cases stored in
`plugins/{skill}/skills/{skill}/evals/evals.json`.

Repository: `eric-zhang16/Biostatistics-skills`

---

## Step 1 — Identify the issue(s) to import

- **Single issue:** User provides an issue number or URL (e.g., `#5` or `.../issues/5`)
- **Bulk sync:** User says "sync all benchmarks" or "sync all issues"

---

## Step 2 — Import

**Single issue:**
```bash
python3 _automation/issue-to-eval/scripts/import_issue_eval.py --issue {ISSUE_NUMBER}
```

**Bulk sync (all issues labeled `benchmark`):**
```bash
python3 _automation/issue-to-eval/scripts/sync_benchmarks.py
```

The script will:
- Fetch the issue body from GitHub
- Parse the 5 required sections: Skills, Query, Expected Output, Attached Files, Rubric Criteria
- Upsert into `plugins/{skill_name}/skills/{skill_name}/evals/evals.json`
- Print: `Success: Added github-issue-N to ...` or `Updated:` or `Skipped: (up to date)`

---

## Step 3 — Commit the updated evals.json

After a successful import, stage and commit:

```bash
git add plugins/*/skills/*/evals/evals.json
git commit -m "eval: sync benchmark from issue #N"
git push
```

---

## Issue Template Requirements

Issues must follow `.github/ISSUE_TEMPLATE/benchmark.md` with these exact headers:

| Section | Purpose |
|---|---|
| `## Skills` | Skill name (e.g., `km-digitizer`) |
| `## Query` | Exact user prompt to test |
| `## Expected Output` | Human-readable success description |
| `## Attached Files / Input Context (Optional)` | Input file URLs or inline content |
| `## Rubric Criteria (Assertions)` | One objective, checkable assertion per line |

Missing or empty sections will produce a WARNING but will not block the import.

---

## Output Format

`plugins/{skill}/skills/{skill}/evals/evals.json`:

```json
{
  "skill_name": "km-digitizer",
  "evals": [
    {
      "id": "github-issue-5",
      "prompt": "Digitize the KM curve from...",
      "expected_output": "A CSV with time/survival columns...",
      "files": ["https://..."],
      "assertions": [
        "digitized_data.csv exists and has at least 20 rows",
        "median survival for Arm A is between 10 and 14 months"
      ]
    }
  ]
}
```
