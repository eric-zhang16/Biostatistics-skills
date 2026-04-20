---
name: benchmark-runner
description: Auto-discover all skills with evals in eric-zhang16/Biostatistics-skills, benchmark each with vs. without the skill using parallel sub-agents, score against assertions, and post results to the originating GitHub issue. Use whenever someone says "run benchmarks", "eval the skills", "compare skill performance", or wants to measure whether a skill improves output quality.
---

# Skill Benchmark Runner

Benchmark every skill in `eric-zhang16/Biostatistics-skills` that has an
`evals/evals.json` file. For each eval case, run two Claude sub-agents in
parallel — one using the skill, one without — then post a scored comparison
as a comment on the originating GitHub issue.

Repository: `eric-zhang16/Biostatistics-skills`

---

## Step 1 — Get the Next Evaluation Case

```bash
python3 _automation/benchmark-runner/scripts/get_next_eval.py --model {CURRENT_MODEL_NAME}
```

Optional flags:
- `--priority-skill km-digitizer` — focus on one skill
- `--priority-issue github-issue-3` — run a specific eval case

Output:
- `STATUS: UP_TO_DATE` → all benchmarks are current; stop.
- JSON object → parse it. Fields: `id`, `prompt`, `files`, `assertions`,
  `_skill_name`, `_skill_sha`, `_skill_content`, `_bundled_resources`.

---

## Step 2 — Run Two Sub-Agents in Parallel

Launch both agents simultaneously. Record start time for each.

**Agent A — WITH the skill:**
- Provide the full `_skill_content` (SKILL.md).
- Provide all files from `_bundled_resources` (`.md`, `.py`, `.R` files).
- Provide any URLs listed in `files`.
- Give the `prompt`.
- Instruct: *"Follow the skill workflow to complete this task. Save all outputs to `output_A/`."*

**Agent B — WITHOUT the skill:**
- Give the same `prompt` and `files` only.
- Instruct: *"Complete this task using only your base knowledge. Do NOT use any SKILL.md. Save all outputs to `output_B/`."*

After both return:
- Record end time → compute duration in minutes (1 decimal place).
- Note any errors, tool failures, or retries.

---

## Step 3 — Score Each Output Against Assertions

**If `assertions` is present and non-empty:** evaluate each assertion against
both `output_A/` and `output_B/`:

| Rating | Meaning |
|---|---|
| Pass | Assertion clearly met |
| Partial | Partially met |
| Fail | Not met |

```
Score = (passes + 0.5 × partials) / total_assertions
```

**If `assertions` is empty or absent:** use `expected_output` as a holistic
rubric. Score each agent 0–100% based on how completely they met the
expected output description. Report as a single "Overall Quality" metric.

Identify 2–3 Key Metrics from the assertions or expected_output for the
scorecard (e.g., "Median Survival Arm A", "Rows in CSV", "Boundary values").

---

## Step 4 — Archive Outputs (Optional)

For deeper inspection, create a gist with key output files:

```bash
gh gist create output_A/*.py output_A/*.R output_A/*.json output_A/*.csv \
               output_B/*.py output_B/*.R output_B/*.json output_B/*.csv \
               --public --desc "Benchmark: {_skill_name} - {id}"
```

Capture the gist URL for the report.

---

## Step 5 — Write the Benchmark Report

Write to `/tmp/benchmark_comment_{_skill_name}_{id}.md`:

```markdown
## Automated Benchmark Results — `{_skill_name}`

### Run Metadata

| Field | Value |
|---|---|
| **Eval ID** | `{id}` |
| **Run date** | {YYYY-MM-DD HH:MM UTC} |
| **Model** | `{model name}` |
| **Skill version** | `{_skill_sha}` |
| **Triggered by** | Scheduled / Manual |

### Scorecard

| Metric | With Skill | Without Skill |
|---|---|---|
| **Score** | {score_A} ({pct_A}%) | {score_B} ({pct_B}%) |
| **Assertions** | {pass_A}P {partial_A}Pt {fail_A}F | {pass_B}P {partial_B}Pt {fail_B}F |
| **Execution time** | {time_A}m | {time_B}m |
| **{Key Metric 1}** | {value_A1} | {value_B1} |
| **{Key Metric 2}** | {value_A2} | {value_B2} |

### Key Observations

- {2-4 bullets comparing both agents}

### Verdict

{1-2 sentence overall verdict}

---

<details>
<summary>Assertion Breakdown & Artifacts</summary>

### Assertion Breakdown

| Assertion | With Skill | Without Skill |
|---|---|---|
| {assertion_1} | Pass/Partial/Fail | Pass/Partial/Fail |

### Artifacts

**Outputs:** [View Gist]({gist_url})

#### Agent A (With Skill) — Key Files
```{lang}
{file_content}
```

#### Agent B (Without Skill) — Key Files
```{lang}
{file_content}
```

</details>

---
*Posted automatically by `benchmark-runner` · Repo: https://github.com/eric-zhang16/Biostatistics-skills*
```

---

## Step 6 — Post to GitHub Issue

Extract issue number from `id` (e.g., `github-issue-5` → `#5`):

```bash
gh issue comment {issue_number} \
  --repo eric-zhang16/Biostatistics-skills \
  --body-file /tmp/benchmark_comment_{_skill_name}_{id}.md
```
