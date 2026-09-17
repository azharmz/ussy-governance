# Compute-Heavy Workflow Governance

Applies to workflows that are long-running, expensive, network-heavy, or process large datasets.

## Design question

Before implementing a heavy workflow, ask:

> **If this fails around 90%, from where can we safely resume without repeating the valid 90%?**

That boundary is a candidate checkpoint / workflow separation boundary.

## Recovery-boundary architecture

Do not split workflows merely because YAML is long. Split around meaningful resume points.

Typical architecture:

`PREPARE → FROZEN INPUT CHECKPOINT → HEAVY COMPUTE → RAW RESULT CHECKPOINT → VALIDATE → ROBUSTNESS/ANALYSIS → REPORT/PUBLISH`

Examples:

- data: `preflight → OHLCV → READY checkpoint → EMA → downstream`;
- backtest: `prepare universe/PIT/config → frozen input → heavy backtest → raw trades/results → integrity validation → robustness → report`.

If a downstream report fails, a valid heavy backtest should not be recomputed merely to regenerate the report.

## Durable checkpoints

Cross-job/workflow recovery must never assume a previous runner filesystem still exists.

Persist checkpoints durably using an appropriate project contract, such as:

- Cloudflare R2;
- GitHub artifacts where appropriate;
- database/object storage.

Checkpoint lineage should include at minimum:

- source/input identity;
- SHA/hash;
- `created_at`;
- schema/version;
- experiment/config identity;
- producer commit/run.

> **Resume from evidence, not from assumption.**

## Fail closed

Resume only when the checkpoint is proven compatible.

Fail closed when, for example:

- expected input SHA differs from checkpoint source SHA;
- experiment config differs;
- schema is incompatible;
- checkpoint is incomplete;
- required lineage is missing.

Never silently consume a stale checkpoint just to save compute.

## Heavy outputs are reusable assets

Persist expensive outputs such as raw trades/events/equity and a manifest. Downstream metrics, robustness, concentration analysis, charts, and reports should read those outputs when semantics allow.

Do not recompute millions of records merely because report formatting or a downstream metric changed.

## When heavy compute must be rerun

Recompute when a change affects core results, including as applicable:

- signal definition;
- entry/exit rule;
- PIT/as-of logic;
- universe selection;
- price basis;
- corporate-action handling;
- execution timing;
- fee/slippage model when part of the engine;
- source dataset;
- frozen pattern/qualification algorithm.

Changes such as report formatting, additional downstream summary metrics, charts, documentation, or non-semantic logging should not automatically invalidate a compatible heavy checkpoint.

## Audit before compute

Before launching a heavy stage, audit at least:

- accidental hardcoded ticker/security IDs;
- arbitrary sample sizes/dates;
- stale object paths/prefixes/namespaces;
- environment-specific paths;
- wrong experiment configuration;
- output path collision;
- missing lineage;
- leftover debug/sample mode;
- overly broad triggers.

Preserve intentional frozen constants/contracts.

## Concurrency

Main and resume workflows that can mutate the same experiment/production state need compatible concurrency governance. When they share mutable state, prefer a shared concurrency group and serialization such as `cancel-in-progress: false` where appropriate.

Concurrency does not replace lineage validation; use both.

## Trigger governance

Heavy compute should normally be intentional/manual (`workflow_dispatch`) rather than run on every commit.

Temporary narrow push triggers are allowed for autonomous iteration when direct dispatch is unavailable, subject to the GitHub Actions execution governance. Lightweight tests may remain automatic.

## Observability

Heavy workflows must follow `CI_OBSERVABILITY.md`: start log, input/checkpoint identity, scope, sparse progress/heartbeat, output identity, validation result, and terminal summary.

## Failure handling

When a workflow fails, do not automatically rerun everything. Determine:

1. Which stage failed?
2. Did the preceding stage create a valid durable checkpoint?
3. Does the patch change upstream/heavy semantics or only downstream logic?

If only downstream logic changed, resume from the valid checkpoint. If heavy computation semantics changed, recompute the affected heavy stage.

## Standard audit report

When auditing an existing heavy workflow, report:

`CURRENT TOPOLOGY`
→ `HEAVY STAGES`
→ `EXISTING DURABLE CHECKPOINTS`
→ `FAILURE/RESTART WASTE`
→ `PROPOSED CHECKPOINT BOUNDARIES`
→ `RESUME WORKFLOWS`
→ `CONCURRENCY`
→ `TRIGGER GOVERNANCE`

Then implement conservatively without changing methodology or experiment semantics merely for modularization.

## Core principle

> **compute once → persist evidence → validate → reuse safely**
