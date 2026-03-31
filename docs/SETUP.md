# Setup and Reuse Guide

This guide shows how to set up the repository in a reproducible way and run a first benchmark iteration.

For trust rules and phase invariants, use `test/benchmark-agent-workflow.md` as the source of truth. This page stays task-oriented and non-redundant.

## Who this guide is for

Use this if you are reusing this repository in a new workspace and want a reliable first run.

## Prerequisites

- Python 3.8+
- Git
- VS Code with GitHub Copilot
- A workspace containing this repository
- At least one benchmarkable skill under `.github/skills/`

## Environment setup

### Windows (PowerShell)

1. Create a virtual environment:
   - `py -3 -m venv .venv`
2. Activate it:
   - `.\.venv\Scripts\Activate.ps1`
3. Install dependencies:
   - `python -m pip install --upgrade pip`
   - `python -m pip install -r requirements.txt`

### macOS/Linux (bash/zsh)

1. Create a virtual environment:
   - `python3 -m venv .venv`
2. Activate it:
   - `source .venv/bin/activate`
3. Install dependencies:
   - `python -m pip install --upgrade pip`
   - `python -m pip install -r requirements.txt`

## Minimal first run (CLI)

Use a dedicated iteration directory:

1. Create iteration folder:
   - `mkdir -p test/benchmark-run-001`
2. Write protocol lock:
   - `python test/scripts/skill_suite_tools.py write-protocol-manifest --iteration test/benchmark-run-001 --workspace-root .`
3. Run mandatory preflight:
   - `python test/scripts/skill_suite_tools.py protocol-preflight --iteration test/benchmark-run-001 --workspace-root .`
4. Run harness checks:
   - `python test/scripts/skill_suite_tools.py self-test --iteration test/benchmark-run-001 --workspace-root .`
5. Finalize and aggregate:
   - `python test/scripts/skill_suite_tools.py resume-finalize --iteration test/benchmark-run-001 --workspace-root .`

Expected outputs:

- `test/benchmark-run-001/suite-summary.json`
- `test/benchmark-run-001/suite-summary.md`
- Per-skill directories under `test/benchmark-run-001/<skill>/`

## New operational capabilities (`benchmark-v3`)

- `resume-finalize` now supports interruption-safe recovery by auto-materialising missing blind payloads from `test/<iteration>/_meta/`.
- `protocol-preflight` performs mandatory hook-state hygiene for worker modes before execution.
- `snapshot-public-evals` creates iteration-local prompt inputs under `test/<iteration>/<skill>/eval-<id>/input/`.
- Blind comparison materialisation supports stdin or file payloads with compatibility `--workspace-root` flags.
- Run metrics can be derived automatically from response artifacts with `write-run-metrics-auto`.

These commands improve repeatability without changing the benchmark output contract.

## Recommended CLI lifecycle for one iteration

1. Initialise protocol lock and safeguards:
   - `python test/scripts/skill_suite_tools.py write-protocol-manifest --iteration test/benchmark-run-001 --workspace-root .`
   - `python test/scripts/skill_suite_tools.py protocol-preflight --iteration test/benchmark-run-001 --workspace-root .`
2. Snapshot public prompts for worker-local reads:
   - `python test/scripts/skill_suite_tools.py snapshot-public-evals --iteration test/benchmark-run-001 --workspace-root .`
3. Run your worker phases (manager-driven or manual orchestration).
4. Validate and finalise:
   - `python test/scripts/skill_suite_tools.py pre-aggregate-check --iteration test/benchmark-run-001 --workspace-root .`
   - `python test/scripts/skill_suite_tools.py resume-finalize --iteration test/benchmark-run-001 --workspace-root .`

If execution is interrupted, rerun `resume-finalize` directly.

## Minimal first run (Manager agent)

1. Open Copilot Chat in VS Code.
2. Select agent: `Skill Benchmark Manager`.
3. Ask for a benchmark over a new iteration path.
4. Confirm outputs under `test/<iteration>/`.

## Re-run hygiene

Before rerunning a previously used iteration:

- Reset blind comparisons per skill:
  - `python test/scripts/skill_suite_tools.py reset-blind-comparisons --iteration test/benchmark-run-001 --workspace-root . --skill <skill-name>`
- Re-run preflight:
  - `python test/scripts/skill_suite_tools.py protocol-preflight --iteration test/benchmark-run-001 --workspace-root .`

Optional before reruns:

- Remove disposable generated exports only:
   - `python test/scripts/skill_suite_tools.py prune-generated-artifacts --iteration test/benchmark-run-001 --workspace-root .`
- Check metrics shape and aliases:
   - `python test/scripts/skill_suite_tools.py validate-metrics --iteration test/benchmark-run-001 --workspace-root .`
   - `python test/scripts/skill_suite_tools.py normalize-metrics --iteration test/benchmark-run-001 --workspace-root .`

## Validation checklist before publishing results

- `protocol-preflight` succeeded
- Hook state was reset for worker modes
- `pre-aggregate-check` status is `ok`
- `suite-summary.json` and `suite-summary.md` are refreshed

## Related documents

- Workflow contract: `test/benchmark-agent-workflow.md`
- Protocol manifest: `test/benchmark-protocol.json`
- Troubleshooting: `docs/TROUBLESHOOTING.md`
