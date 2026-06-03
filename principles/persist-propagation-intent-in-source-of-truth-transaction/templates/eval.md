# Eval: Persist Propagation Intent in the Source-of-Truth Transaction

Status: experimental

## Purpose

Evaluate whether a project's write paths propagate to second, independent stores
durably — committing the propagation intent atomically with the primary write and
delivering it asynchronously with retries and idempotency — instead of
dual-writing from application code.

A passing verdict means: if the entity exists in the source of truth, its
propagation event also exists and is recoverable; a delivery failure is persisted
state, not a lost log line.

## Verdict Output (required for audit use)

The eval must return, per finding, evidence that points back to the code:

- **file + line range** of the offending write path.
- **the antipattern detected** (e.g. fire-and-forget dispatch, post-commit second
  write, compensating-delete instead of transaction).
- **the why** — which invariant is broken and the failure it produces at scale.
- **severity, stage-adjusted** — critical if the second store is a product
  differentiator; lower if it is a rebuildable best-effort cache.
- **recommended fix** — the outbox move, including making the transaction real
  first if needed.

A verdict without traceable evidence (files, lines, why) does not satisfy this
eval.

## Manual Checks

- [ ] Enumerate every write that propagates to a second independent store
      (memory / vector, search, retrieval, analytics, event store, external API).
- [ ] For each, classify the second store: product-critical vs. rebuildable
      best-effort.
- [ ] Confirm a single real transaction wraps the primary write(s).
- [ ] Confirm the propagation intent (outbox row) is inserted inside that
      transaction.
- [ ] Confirm a worker delivers with retries, a terminal `failed` state, and a
      deterministic idempotency key.
- [ ] Confirm failures are persisted as queryable state and are reprocessable.
- [ ] Confirm any accepted loss is a documented, conscious trade for the stage.

## Semi-Automated Checks (code signatures)

These searches surface candidates; each hit needs human confirmation (not every
match is a violation).

- Fire-and-forget dispatch in a write path:
  - `void\s+\w+\([^)]*\)\s*\.catch` (un-awaited promise with a catch-and-log)
  - `\.catch\(\s*\([^)]*\)\s*=>\s*(console\.(error|warn)|logger\.)`
- Second-store write after a primary commit (outside a transaction):
  - calls to memory / vector / search / analytics SDKs or `fetch`/`axios` to an
    external endpoint located after a DB write, not inside a transaction block.
- Atomicity simulated in application code:
  - search for "compensat", "rollback", "manual delete", or a `delete` issued in
    a `catch` after a prior `insert`.
- Absence of the outbox machinery:
  - no `outbox` table / migration; no worker/poller; no
    `FOR UPDATE SKIP LOCKED`.
- Idempotency present but durability absent (the common half-fix):
  - an `idempotencyKey` / `idempotency_key` is computed but the dispatch is still
    fire-and-forget.

## Automated Checks

- Add a test that asserts: after a primary write, a corresponding `pending`
  outbox row exists in the same transaction (rollback leaves neither).
- Add a test that a worker delivery failure transitions the row to `failed` and
  leaves it reprocessable (no information loss).
- Add a test that re-delivering an already-applied event is a no-op (idempotency
  key collapses duplicates).
- Add a static/lint check that flags un-awaited propagation calls in designated
  write-path modules.

## Known Limits

- Static search cannot prove an outbox row truly shares the primary write's
  transaction — confirm the transaction boundary by reading the code.
- A second-store write is not always a violation: rebuildable caches and
  best-effort fan-outs may legitimately skip the outbox.
- The eval cannot decide on its own whether stage-based loss is acceptable; that
  is a documented business trade requiring human judgment.
- Dynamically dispatched or reflection-based write paths may evade signature
  search.
