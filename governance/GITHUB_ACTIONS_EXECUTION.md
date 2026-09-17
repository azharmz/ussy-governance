# GitHub Actions Execution Governance

## Core rule

When a temporary or automatic push trigger is active:

> **COMMIT = POTENTIAL COMPUTE EXECUTION**

Treat a commit as an intentional compute decision, not merely a save operation.

## Required execution discipline

For push-triggered research/compute workflows:

`INSPECT → PLAN COMPLETE PATCH → EDIT/PREPARE → VERIFY EXPERIMENT-READY → ONE MEANINGFUL COMMIT → ONE INTENTIONAL RUN → MONITOR → EVALUATE`

Target:

> **one meaningful experiment-ready commit ≈ one intentional Actions run**

Batch related cleanup, hardcode removal, configuration fixes, observability changes, and experiment preparation when they can safely be verified before compute.

Avoid avoidable sequences such as formatting commit → run, hardcode cleanup commit → run, logging commit → run, experiment commit → run.

## Before every compute-triggering commit

Check:

1. Will this commit trigger a workflow?
2. Is that compute actually needed now?
3. Is the patch ready to produce useful evidence?
4. Are known cleanup/hardcode/config issues still outstanding?
5. Can related changes safely be batched before the run?

If the run is not yet needed, do not intentionally spend compute merely to persist an intermediate edit.

## Hardcode audit

Before compute-heavy validation, audit accidental hardcodes such as:

- security IDs or ticker lists;
- arbitrary sample size;
- sample/experiment dates that should be discovered/configurable;
- stale R2/object keys or storage prefixes;
- legacy namespace;
- environment-specific paths;
- wrong experiment config;
- output collisions;
- leftover debug/sample mode.

Do **not** confuse these with intentional frozen constants such as frozen algorithm parameters, frozen SHAs, schema versions, or contract semantics.

## Temporary push-trigger policy

A temporary branch-specific push trigger is valid when autonomous iteration requires `patch → run → inspect → patch` and direct dispatch is unavailable.

It must be as narrow as practical:

- research branch only;
- relevant paths only;
- no broad production/main trigger without explicit architectural need.

After the workstream is DONE, TERMINAL, or FROZEN, restore manual-only / `workflow_dispatch` unless automatic CI is intentionally part of the repository design.

## User command interpretation

`perbaiki dulu`, `rapikan dulu`, `hardcoded dulu`, `jangan run dulu` means prepare the code first and do not intentionally trigger validation compute before readiness.

`lanjut`, `kerjakan`, `jalan terus` authorizes the autonomous loop but does not authorize wasteful runs.

`hijau` means inspect actual successful evidence before proceeding.

`merah` means inspect the failure, diagnose, patch in scope, and rerun only when the patch is ready.

## Unneeded runs

If a run starts and is later known to be unnecessary:

- do not treat it as evidence merely because it ran;
- cancel it through available tooling when possible;
- if cancellation is unavailable through tooling, request only the specific user cancellation action.

Never rerun a job as a way to cancel it.

## Core compute principle

> **prepare once → commit once → compute once → learn once**
