# Re-evaluate an existing benchmark submission with a fresh judge

Re-score a previously-submitted benchmark JSON using the **current** Claude
session as the judge. The translations themselves are reused verbatim — no LLM
calls are made for translation, only for evaluation.

**Args:**
- `{{arg1}}` = path to the submission JSON to rescore (e.g.
  `benchmark/data/submissions/2026-05-09_hydropix_gemma3-27b.json`)

If `{{arg1}}` is missing, list available submissions and ask the user which to
rescore via `AskUserQuestion`:

```bash
ls benchmark/data/submissions/*.json
```

---

## Step 1 — Load the binding references

Read these two files in full:

- [docs/JUDGE_RUBRIC.md](../../docs/JUDGE_RUBRIC.md) — penalty table, scale anchors, hard ceilings, dispersion rule.
- [docs/BENCHMARK_WORKFLOW.md](../../docs/BENCHMARK_WORKFLOW.md) — the canonical procedure.

The rubric is binding.

---

## Step 2 — Reconstruct the run

Run via Bash:

```
python scripts/submission_to_run.py {{arg1}}
```

Capture the `RUN_ID=...` line from stdout. Save it as `<RUN_ID>` for the
remaining steps.

This script:
- Reads the submission JSON.
- Validates it against the schema.
- Builds `benchmark_results/<RUN_ID>.json` containing all the original
  outputs but with `scores: null`.
- Preserves `verified` and `contributors` so the new submission keeps
  attribution.

If the script fails (file not found, schema invalid), report the error and
**stop**.

---

## Step 3 — Dump the evaluation brief

```
python scripts/dump_for_evaluation.py <RUN_ID> --batch-size 15 --out plan/eval_<RUN_ID>.md
```

Read every `plan/eval_<RUN_ID>_batch*.md` produced.

---

## Step 4 — Score per the rubric

Same as `/benchmark-test-model` Step 5 (cf. its skill file). For each
translation:

1. Identify all errors per the penalty table (§3 of the rubric).
2. Compute `accuracy`, `fluency`, `style` starting from 10 minus penalties.
3. `overall` is holistic, constrained by §4 hard ceilings (cap 9.0 without
   human reference; if any dimension < 6.0, overall ≤ 6.0; overall ≤
   `min(acc, flu, sty) + 0.5`).
4. Don't be charitable. Most LLM scores fall in 6.5–8.5.
5. Write a 1–2 sentence `feedback` documenting the deductions with explicit
   penalty values.

If the original submission's `judge_id` shows a **different rubric version**
than the current one (e.g. the original was `gemini-3-flash-rubric-v1` and
you're applying `v2`), state this explicitly to the user before scoring.

---

## Step 5 — Write the JSON reply

Single file `plan/eval_<RUN_ID>_rubric_v1.json`:

```json
[
  {"eval_id": "<10-hex>", "scores": {"accuracy": 0.0, "fluency": 0.0, "style": 0.0, "overall": 0.0, "feedback": "..."}},
  ...
]
```

One object per `eval_id` from the brief. Don't omit any.

---

## Step 6 — Apply the new scores

Determine the judge model identifier:
- Opus 4.7 → `claude-opus-4-7-rubric-v1`
- Sonnet 4.6 → `claude-sonnet-4-6-rubric-v1`
- Haiku 4.5 → `claude-haiku-4-5-rubric-v1`

Then:

```
python scripts/apply_evaluations.py <RUN_ID> plan/eval_<RUN_ID>_rubric_v1.json --judge-id <judge-id>
```

Idempotent.

---

## Step 7 — Report comparison

In the final report, compare **side-by-side** with the original submission.
Read the original to surface its `environment.judge_id` and the per-pair
averages it carried, then present:

| Pair | N | Original judge avg | New judge avg | Delta |
|---|---|---|---|---|

Plus:
- The original `judge_id` and the new one.
- 2-3 translations whose score moved the most (up or down) and why.
- Whether the new judge agrees with the old on the worst translations.

---

## Step 8 — Suggest the new submission

The output of this skill is a re-scored run, not a submission. Suggest the
copy-pasteable next command:

```
python -m benchmark.cli submit benchmark_results/<RUN_ID>.json \
  --by github:<original-submitter> \
  --provider <original-provider> \
  --judge-id <new-judge-id>
```

Where `<original-submitter>` and `<original-provider>` come from the source
submission (you read those in Step 2). The new submission file lands next to
the old one in `benchmark/data/submissions/` with a fresh date stamp. The
aggregator will surface both observations on the wiki: same `(model, text,
target_lang)` triple, `n_obs=2`, median scores.

**Don't auto-submit.** The user reviews the report first.

---

## Important guardrails

- **Don't re-translate.** This skill never invokes a translation provider.
  The model outputs are read from the submission verbatim.
- **The judge_id format is `<judge-model>-rubric-v<n>`.** Always.
- **Use TodoWrite** to track the 8 steps.
- **The rubric version must match what the judge applied.** If you ever
  bump the rubric to v2, the new submissions will be `*-rubric-v2` and the
  aggregator will treat them as a separate series on the wiki.
