# R2 Storage Encoding / Compression Governance

## Purpose and invariant

Optimize only the **physical storage representation** of an existing logical dataset.

> `logical dataset X → more efficient physical representation of X`

Encoding/compression work MUST NOT change schema, logical values, row count, null semantics, PIT/as-of semantics, methodology, business logic, universe, price basis, corporate-action treatment, signal/entry/exit semantics, or downstream logical data contracts. If logical content changes, it is a separate workstream, not an encoding optimization.

## Validated reference implementation

The validated reference pattern is `azharmz/ussy-fundamentals`, especially institutional sponsorship publishing:

- publisher: `src/ussy_fundamentals/sec_13f_publish.py`
- tests: `tests/test_sec_13f_publish.py`
- canonical workflow: `.github/workflows/sec-13f-publish-canonical.yml`
- validated production run: `35161251835`
- validated snapshot: `institutional_sponsorship/snapshots/2026-09-16/run-35161251835`
- reference status: `PRODUCTION / VERIFIED`
- production contract tests at validation: `9 passed`

This implementation is a **reference pattern**, not proof that the same codec/level is optimal for every dataset.

## Proven result

For `institutional_sponsorship`, snapshot physical size decreased from about 236.9 MiB (2026-09-14) to about 167.8 MiB (2026-09-16), saving about 69.1 MiB / ~29% without deleting logical data. Most saving came from `history/`, about 230.3 MiB → 164.2 MiB.

## Parquet encoding

For the validated large Parquet artifacts in the reference implementation, use internal Parquet compression:

`ZSTD level 9`

Reference production storage contract:

`lossless-gzip-v1-selective+parquet-zstd9-v1`

Validated targets:

- `history/sponsorship_state_events.parquet`
- `history/final_period_state.parquet`

Keep object keys as `.parquet`. **Do not outer-gzip Parquet files.**

Reference benchmark using PyArrow 25.0.1:

- `final_period_state.parquet`: 811,757 → 633,876 bytes; saved 177,881 bytes.
- `sponsorship_state_events.parquet`: 240,620,596 → 171,490,769 bytes; saved 69,129,827 bytes.
- combined saving: about 69.3 MB (~66.1 MiB) per snapshot.

Do not generalize ZSTD-9 blindly. Benchmark representative production-size data in each new dataset/repo and compare useful codec/level candidates against CPU/runtime cost.

## Non-Parquet objects

Use selective outer gzip where appropriate under the existing `lossless-gzip-v1-selective` pattern. Gzip only when both are true:

- original file size >= 1024 bytes;
- compressed/original ratio <= 0.90.

Do not gzip indiscriminately.

## Mandatory lossless semantic validation

A transcoded Parquet file is not valid merely because it can be read.

Required sequence:

`SOURCE PARQUET → read logical table → write encoded Parquet → read result → FULL SEMANTIC COMPARISON`

The validation MUST prove:

- schema identical;
- row count identical;
- column count identical;
- all logical values identical;
- null placement/count identical.

Reference production benchmark validated 14,529,166 rows × 9 columns with `schema_equal=true`, `values_equal=true`, zero mismatches in every column, and matching nulls.

IPC/binary fingerprints do not need to match because physical representation, chunking, encoding, and compression may legitimately change. Logical-table equality is authoritative.

**FAIL CLOSED** if full semantic equality cannot be established.

## Manifest and lineage

Every optimized artifact must record at least:

- source/logical SHA;
- source/logical bytes;
- stored SHA;
- stored bytes;
- compression codec;
- compression level;
- representation (for example `parquet-zstd`);
- compression ratio;
- semantic verification result;
- producer commit/run;
- snapshot identity.

Consumers continue using the normal logical object contract, including normal `.parquet` keys. Physical compression must not leak into or unnecessarily alter downstream contracts.

## Publish safety

Mandatory publication order:

`prepare → validate input → encode/compress → semantic verification → upload immutable snapshot → verify snapshot → write current.json LAST → guarded retention/cleanup`

Never update the current pointer before the new snapshot has been validated. Immutable snapshots must not be overwritten.

## Retention governance for snapshot datasets

Where the dataset follows the validated institutional-sponsorship snapshot model, enforce:

- maximum one retained run for the current `snapshot_date`;
- retain two distinct snapshot dates.

When republishing the same date:

1. upload and validate the new run;
2. point `current.json` to the new run;
3. reread and verify the current pointer;
4. prove the new run is protected;
5. delete the superseded run for that same date;
6. verify the superseded prefix is empty;
7. preserve the prior distinct snapshot date.

Deletion must be guarded and fail closed. Do not apply this exact retention count to unrelated datasets without confirming their local retention contract.

## Storage audit before migration

Do not immediately migrate a repository to ZSTD-9. Start with a storage audit:

1. inventory the largest objects/files;
2. identify format and current codec/encoding;
3. measure current physical size;
4. rank candidates by material saving potential;
5. benchmark representative production data;
6. compare reasonable codecs/levels and CPU/runtime cost;
7. calculate actual/projected byte savings;
8. perform full semantic equality validation;
9. select an encoding only if lossless and materially useful;
10. add regression protection;
11. promote conservatively;
12. perform production readback verification.

Prioritize large objects. Saving ~30% on a ~200 MiB object is generally more material than micro-optimizing hundreds of tiny JSON files.

## Governance boundary

Encoding work must not modify frozen methodology or logical dataset semantics to improve storage results. In particular, do not change signal semantics, fundamental interpretation, PIT/as-of rules, universe, price basis, corporate-action handling, entry/exit logic, methodology, schema, or logical values as part of this workstream.

Any such change requires its own explicitly governed workstream.

## Operating rule for receiving repositories

Start with **STORAGE AUDIT**, not migration:

`largest objects → current representation → benchmark → actual/projected saving → semantic proof → regression → controlled promotion → readback verification`

Use the `ussy-fundamentals` ZSTD-9 implementation as the canonical reference pattern for how to validate and publish a representation-only optimization, while independently determining whether ZSTD-9 is appropriate for the receiving dataset.