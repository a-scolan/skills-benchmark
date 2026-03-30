# Setup and Reuse Guide

This guide shows how to set up the repository in a reproducible way and run a first benchmark iteration.

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

## Validation checklist before publishing results

- `protocol-preflight` succeeded
- Hook state was reset for worker modes
- `pre-aggregate-check` status is `ok`
- `suite-summary.json` and `suite-summary.md` are refreshed

## Related documents

- Workflow contract: `test/benchmark-agent-workflow.md`
- Protocol manifest: `test/benchmark-protocol.json`
- Troubleshooting: `docs/TROUBLESHOOTING.md`
