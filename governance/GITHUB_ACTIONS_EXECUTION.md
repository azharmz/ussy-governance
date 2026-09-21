# GitHub Actions Execution Governance

## Core rule

When a temporary or automatic push trigger is active:

> **COMMIT = POTENTIAL COMPUTE EXECUTION**

Treat a commit as an intentional compute decision, not merely a save operation.

## Required execution discipline

For push-triggered research/compute workflows:

`INSPECT → PLAN COMPLETE PATCH → EDIT/PREPARE → VERIFY EXPERIMENT-READY → AUDIT TRIGGER BLAST RADIUS → ONE MEANINGFUL COMMIT → ONE INTENTIONAL TARGET RUN → VERIFY NO UNINTENDED SIBLING RUNS → MONITOR → EVALUATE`

Target:

> **one meaningful experiment-ready commit ≈ one intentional Actions run**

Batch related cleanup, hardcode removal, configuration fixes, observability changes, and experiment preparation when they can safely be verified before compute.

Avoid avoidable sequences such as formatting commit → run, hardcode cleanup commit → run, logging commit → run, experiment commit → run.

## Before every compute-triggering commit

Check:

1. Will this commit trigger a workflow?
2. **Exactly which workflows can this commit trigger?**
3. Is the intended workflow the only workflow that should run?
4. Is that compute actually needed now?
5. Is the patch ready to produce useful evidence?
6. Are known cleanup/hardcode/config issues still outstanding?
7. Can related changes safely be batched before the run?

If the run is not yet needed, do not intentionally spend compute merely to persist an intermediate edit.

## Trigger blast-radius audit

Before using a push as a dispatch mechanism, inspect the repository's workflow trigger topology.

Determine all workflows that can react to the proposed commit, including:

- branch filters;
- path/path-ignore filters;
- workflow chaining such as `workflow_run`;
- reusable/called workflow relationships where relevant;
- production publishers or other heavy workflows that could be activated indirectly.

The execution plan must identify the **intended target workflow** and prove that the proposed trigger is isolated enough not to launch unrelated compute.

> **One intentional dispatch commit should produce one intended workflow run, unless multiple runs are explicitly part of the approved design.**

Do not knowingly use a commit as a dispatch mechanism when the same commit can unnecessarily trigger sibling production, backtest, publisher, or other compute-heavy workflows.

After the triggering commit, inspect Actions promptly and verify that no unintended sibling workflows started. If unexpected runs appear, treat that as a trigger-governance defect: cancel them when tooling permits, diagnose the overlap, narrow the trigger, and add regression/documentation protection as appropriate.

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

- dedicated/research branch where practical;
- relevant or dedicated dispatch-marker path only;
- no broad production/main trigger without explicit architectural need;
- isolated from sibling workflows as established by the trigger blast-radius audit.

A temporary push trigger is a **dispatch mechanism**, not a permanent development trigger. Prepare ordinary code/config changes before enabling or exercising it whenever practical.

If a dedicated dispatch-marker path is used, it must not be watched by unrelated workflows and must not alter production semantics merely to cause a run.

After the required run is triggered, remove/disable the temporary push path as soon as practical. After the workstream is DONE, TERMINAL, or FROZEN, restore manual-only / `workflow_dispatch` unless automatic CI is intentionally part of the repository design.

If no isolated safe push path exists, do not broaden the trigger merely to avoid manual action. At that point, manual dispatch can be a genuine execution boundary.

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

> **prepare once → audit blast radius → commit once → run only intended compute → verify → learn once**
