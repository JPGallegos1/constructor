# ADR-<number>: Persist Propagation Intent in the Source-of-Truth Transaction

Status: <proposed | accepted | superseded>

## Context

<Describe the project-specific writes that must also reach a second, independent
store — for example a memory / vector store, search index, analytics pipeline, or
external system — and how that propagation happens today (synchronous,
fire-and-forget, queued, etc.).>

<Explain why silent divergence between the source of truth and the derived store
would create risk in this project. Name the derived store's role: is it a
product differentiator whose loss is unacceptable, or a rebuildable best-effort
cache?>

## Decision

This project will not dual-write the same event to two independent stores from
application code.

For each write that must propagate to a second independent store, the project
will persist the propagation intent (an outbox row) inside the same transaction
as the primary write, and deliver it asynchronously with retries and idempotency.

## Scope

This decision applies to:

- <write path / module that propagates to a second store>;
- <write path / module>;
- <write path / module>.

This decision does not apply to:

- writes to a single store with no cross-store propagation;
- best-effort, trivially rebuildable caches or fan-outs where loss is acceptable;
- propagation that already happens inside a single shared transaction;
- early-stage paths where loss is a documented, conscious trade (the exemption
  expires once the derived store becomes a differentiator).

## Implementation Expectations

- Wrap the primary write(s) in a single real transaction (e.g. a database
  function / RPC), not separate autocommitted statements with a compensating
  delete.
- Insert the outbox row (`pending`, idempotency key, payload) inside that same
  transaction.
- Deliver from an asynchronous worker with retries and an explicit terminal
  state (`failed` / DLQ) that is alertable and reprocessable.
- Make delivery idempotent via a deterministic key (at-least-once becomes
  effectively-once).
- Define an archival policy for `done` rows and an ordering policy if events need
  causal order.

## Consequences

- The derived store lags by the worker's polling interval — explicit, documented
  eventual consistency instead of accidental divergence.
- A failed propagation is persisted, queryable, alertable, and reprocessable
  rather than lost.
- The project must operate a worker process and maintain the outbox table.
- Hand-rolled compensating-rollback code can be removed once the real
  transaction exists.

## References

- Generic principle: principles/persist-propagation-intent-in-source-of-truth-transaction/principle.md
- Related: principles/validate-data-at-system-boundaries-before-persistence/principle.md
