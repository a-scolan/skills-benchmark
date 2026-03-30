# Troubleshooting

Use this page to diagnose common failures when reusing the benchmark harness.

## 1) `pre-aggregate-check` fails with missing blind comparisons

### Symptom

- `pre-aggregate-check` reports missing blind comparisons for prepared runs.

### Cause

- Blind artifacts exist for one or more runs, but `blind-comparisons.json` does not cover all `(eval_id, run_number)` pairs.

### Fix

1. Re-materialise from raw payloads in `test/<iteration>/_meta/raw-comparison-*.json`.
2. Run:
   - `python test/scripts/skill_suite_tools.py resume-finalize --iteration <iteration> --workspace-root .`
3. Re-run:
   - `python test/scripts/skill_suite_tools.py pre-aggregate-check --iteration <iteration> --workspace-root .`

---

## 2) Hook denies reads in baseline mode

### Symptom

- Baseline worker is denied when reading project files or broad paths.

### Cause

- Baseline modes are intentionally restricted to iteration-scoped prompt inputs and narrow approved grounding only.

### Fix

- Read prompts from:
  - `test/<iteration>/<skill>/eval-<id>/input/prompt.md`
  - or `test/<iteration>/<skill>/eval-<id>/input/prompt.json`
- Avoid direct reads from general project folders during baseline runs.

---

## 3) Cross-iteration write denial in worker sessions

### Symptom

- A worker that wrote to iteration A is denied writes to iteration B.

### Cause

- Worker sessions are iteration-locked after first allowed write.

### Fix

1. Start a fresh worker session.
2. Run preflight before the new iteration:
   - `python test/scripts/skill_suite_tools.py protocol-preflight --iteration <iteration> --workspace-root .`

---

## 4) Corrupted metrics JSON (`Extra data` parse errors)

### Symptom

- Summarisation fails with JSON parse errors, often mentioning `Extra data`.

### Cause

- Multiple JSON objects were appended into a single metrics file.

### Fix

- Regenerate per-run metrics files using manager commands (`write-run-metrics` or `write-run-metrics-auto`) and ensure one JSON object per file.
- Then run summarisation again.

---

## 5) Blind comparator cannot persist output

### Symptom

- Comparator output is lost in chat or materialisation cannot find payload.

### Cause

- `raw_output_path` was not provided or not used.

### Fix

1. Build task bundle with `blind-compare-bundle`.
2. Pass `raw_output_path` to comparator.
3. Have comparator write wrapped payload to that path.
4. Materialise immediately after comparator acknowledgement.

---

## 6) Setup succeeds but command fails with missing module `yaml`

### Symptom

- Python reports missing `yaml`.

### Cause

- Dependencies were not installed in the active virtual environment.

### Fix

1. Activate `.venv`.
2. Run:
   - `python -m pip install -r requirements.txt`

---

## 7) Stale or mixed generated artefacts in an iteration

### Symptom

- Unexpected summaries or stale review exports.

### Cause

- Generated review artefacts from old runs were kept.

### Fix

- Prune disposable generated artefacts:
  - `python test/scripts/skill_suite_tools.py prune-generated-artifacts --iteration <iteration> --workspace-root .`

## When to escalate

If issues persist after the steps above:

1. Capture the exact command and full error output.
2. Attach relevant files from `test/<iteration>/_meta/` and `test/_agent-hooks/`.
3. Re-run with trace level `audit` or `debug` for additional diagnostics.
