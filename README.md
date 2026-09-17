# USSY Governance

Central source of truth for **cross-repository engineering and execution governance** across USSY projects.

This repository defines **how technical work is executed**. Project-specific scientific methodology, frozen research semantics, production contracts, acceptance criteria, and workstream state remain owned by their respective repositories.

## Governance documents

- [`governance/GLOBAL_EXECUTION_RULES.md`](governance/GLOBAL_EXECUTION_RULES.md) — continuous execution, error handling, safe workarounds, stop boundaries, and governance preservation.
- [`governance/GITHUB_ACTIONS_EXECUTION.md`](governance/GITHUB_ACTIONS_EXECUTION.md) — commit/push-trigger/compute discipline and temporary trigger policy.
- [`governance/COMPUTE_HEAVY_WORKFLOWS.md`](governance/COMPUTE_HEAVY_WORKFLOWS.md) — checkpointed architecture, recovery boundaries, lineage, resume, and concurrency for expensive workflows.
- [`governance/CI_OBSERVABILITY.md`](governance/CI_OBSERVABILITY.md) — minimum logging and sparse progress/heartbeat requirements.

## Scope boundary

Global engineering rules belong here. Repository-local rules stay local.

Examples of rules that **do belong here**:

- autonomous execution discipline;
- GitHub Actions trigger/compute governance;
- CI observability;
- durable checkpoint/recovery architecture;
- generic fail-closed and verification rules.

Examples of rules that **do not belong here**:

- a specific CAN SLIM threshold;
- an O'Neil pattern frozen contract;
- a TrendFoll signal definition;
- a particular experiment verdict;
- repo-specific production schema semantics.

Those remain authoritative in the owning project repository.

## Precedence

1. Safety, credentials, permissions, and irreversible-action boundaries.
2. Explicit frozen/local contract in the owning repository.
3. This global governance.
4. Implementation convenience.

Global governance must never be used to silently change frozen project semantics.

## Core operating principles

> **Execute → monitor → verify → diagnose → patch → regression → rerun → continue until DONE or genuinely blocked.**

> **Prepare once → commit once → compute once → learn once.**

> **Compute once → persist evidence → validate → reuse safely.**

## Adoption by project repositories

A project README may reference this repository as its global engineering governance source, while keeping its local governance and frozen semantics in the project itself.

## Change discipline

Governance changes should be explicit, auditable, and should not retroactively reinterpret completed research evidence without an explicit project-level decision.
