# AGENTS.md

## Purpose

This repository is the global engineering-governance source for USSY projects. For project work, preserve the owning repository's frozen methodology and local contracts. Do not use global governance to change research semantics.

## Codex operating mode

Optimize for a **verified result with minimal context, token use, compute, commits, and user intervention**.

Use one bounded cycle:

`inspect narrowly → plan briefly → implement completely → run smallest relevant verification → diagnose/fix failures → rerun → report result`

### Context discipline

- Read only files needed for the current task. Start from named files, failing tests/logs, changed paths, and direct dependencies.
- Do not recursively inspect or summarize the whole repository unless the task requires it.
- Do not repeatedly reread unchanged files or restate repository history.
- Prefer targeted search and narrow diffs over broad repository scans.
- Treat existing governance as constraints; reference it rather than copying it into every task.
- Keep plans short. Spend tokens on implementation and verification, not narration.

### Execution discipline

- Once the task and constraints are clear, do not ask for approval between ordinary inspect/implement/test/fix steps.
- Make the smallest complete change that satisfies the task.
- Avoid speculative refactors, unrelated cleanup, and formatting churn.
- Prefer one meaningful patch/commit over many small commits.
- Run the smallest relevant tests first; broaden verification only when risk or failures justify it.
- If verification fails for an in-scope repairable reason, diagnose, patch, and rerun without stopping for confirmation.
- Report progress only when useful; progress is not an approval checkpoint.

### Production and compute discipline

Before any compute-heavy, storage-heavy, database-heavy, or production mutation:

1. identify expected cardinality, write amplification, storage/compute cost, and blast radius;
2. identify the recovery/checkpoint boundary;
3. reuse valid durable checkpoints and existing evidence;
4. verify lineage and inputs fail-closed;
5. validate with fixtures/local tests before production when practical.

Do not use production runs to discover design errors that can be caught by inspection, cardinality analysis, fixtures, or tests.

For long or expensive workflows, follow:
`prepare → frozen input checkpoint → heavy compute → durable result checkpoint → validate → publish`.

Never recompute a valid expensive stage merely because downstream reporting, metrics, logging, or presentation changed.

### Output discipline

Final report should normally contain only:

- what changed;
- verification/result;
- remaining blocker or risk, if any.

Do not produce long repository summaries, repeated plans, or step-by-step narration unless explicitly requested.

## Authoritative governance

Follow these documents when relevant:

- `governance/GLOBAL_EXECUTION_RULES.md`
- `governance/GITHUB_ACTIONS_EXECUTION.md`
- `governance/COMPUTE_HEAVY_WORKFLOWS.md`
- `governance/CI_OBSERVABILITY.md`
- `governance/R2_STORAGE_ENCODING_COMPRESSION.md`

If a project-local frozen contract conflicts with implementation convenience, the frozen contract wins.
