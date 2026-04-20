# Automation

This directory contains two automation skills that power the eval pipeline for `Biostatistics-skills`.

## issue-to-eval

Converts GitHub issues (labeled `benchmark`) into test cases in `evals.json`.

**Trigger:** Automatically on issue open/edit (via GitHub Actions), or manually:
```bash
# Single issue
python3 issue-to-eval/scripts/import_issue_eval.py --issue 5

# All benchmark issues
python3 issue-to-eval/scripts/sync_benchmarks.py
```

**Output:** `plugins/{skill}/skills/{skill}/evals/evals.json`

## benchmark-runner

Runs each eval case with and without the skill (two parallel sub-agents), scores
outputs against assertions, and posts a verdict to the originating GitHub issue.

**Trigger:** Every Monday at 06:00 UTC, on push to SKILL.md or evals.json, or manually.

**Output:** GitHub issue comment with scorecard + assertion breakdown + verdict.

---

## Eval Pipeline Flow

```
GitHub Issue (label: benchmark)
        │
        ▼
sync-benchmarks.yml (GitHub Action)
        │
        ▼
import_issue_eval.py  ──►  plugins/{skill}/skills/{skill}/evals/evals.json
        │
        ▼
benchmark-schedule.yml (Monday / push / manual)
        │
        ▼
get_next_eval.py  ──►  selects next unrun eval case
        │
        ├── Agent A (WITH skill)  ──►  output_A/
        └── Agent B (WITHOUT skill) ──►  output_B/
                │
                ▼
        Score assertions → Markdown report
                │
                ▼
        gh issue comment #{N}  ──►  GitHub issue verdict
```
