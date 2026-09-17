# CI Observability Governance

Every workflow step should be observable from the beginning without flooding logs or materially slowing the workload.

## Minimum logging

Every step should emit at least a start/status indication.

For short steps, this may be only:

`START → DONE/PASS/FAIL`

For long-running operations, use sparse:

`START → input/scope → milestone/progress/heartbeat → completion/result`

## Long-running operations

Prefer coarse progress such as:

- phase changes;
- every meaningful batch;
- approximately 10–25% progress intervals;
- periodic heartbeat when total work is unknown.

Do not log every row, ticker, trade, or object unless needed for targeted diagnosis.

## Required useful context

Where applicable, logs should expose:

- current phase;
- input/checkpoint identity;
- work count/scope;
- sparse progress or heartbeat;
- output identity;
- validation result;
- terminal summary.

The purpose is to distinguish:

- `RUNNING NORMALLY`
- `HUNG / STALLED`
- `COMPUTE EXPLOSION`

## Python

Ensure long Python jobs do not hide progress due to output buffering. Use flushed output or unbuffered execution when needed, for example `print(..., flush=True)` or `python -u`.

## Performance constraint

Observability must remain lightweight. Logging itself must not become a meaningful source of runtime, storage, or Actions-log overhead.
