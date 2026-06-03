# Agent Rule: Persist Propagation Intent in the Source-of-Truth Transaction

## Applies When

- You create or modify a write path that must also reach a second, independent
  store (memory / vector, search, retrieval, analytics, event store, external
  system).
- You add or change replication or sync logic between two stores.
- You see a write to a second store dispatched after a primary write.
- You see primary writes composed of multiple autocommitted statements with a
  hand-rolled compensating delete.

## Inspect

- The two atomic domains the write touches.
- Whether the second store is treated as a derived projection or a co-equal
  source of truth.
- Where the intent to propagate lives if delivery fails (durable row vs. process
  memory).
- Whether a single real transaction wraps the primary write(s).
- Whether a worker delivers with retries, a terminal `failed` state, and
  idempotency.
- Whether failures are persisted and alertable, not only logged.

## Required Behavior

- Persist the propagation intent (outbox row) inside the same transaction as the
  primary write.
- Ensure that transaction is real before relying on it (fix autocommitted-
  statements + compensating-delete first).
- Deliver asynchronously with retries, an explicit terminal state, and a
  deterministic idempotency key.
- Treat the second store as a consumer of a reliable event log, not a co-equal
  record of truth.

## Avoid

- Fire-and-forget propagation: `void sync().catch(log)` in the write path.
- Writing to a second store/SDK/HTTP client immediately after a commit, outside
  any transaction.
- Inserting the outbox row in a separate autocommitted call ("same transaction"
  in aspiration only).
- Logging only transport failures while the propagation intent is lost.

## Stop And Plan When

- The propagation intent would live only in process memory.
- No real transaction wraps the primary write(s) that the outbox row depends on.
- The derived store is product-critical and there is no durable, reprocessable
  failure path.
- It is unclear whether loss is acceptable at this stage (resolve and document
  before implementing).

## Reference

- Generic principle: principles/persist-propagation-intent-in-source-of-truth-transaction/principle.md
