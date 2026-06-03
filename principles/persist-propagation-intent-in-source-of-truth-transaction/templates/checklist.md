# Checklist: Persist Propagation Intent in the Source-of-Truth Transaction

Use this checklist before implementation, during review, or before merge.

## Applicability

- [ ] Does this change propagate a write to a second, independent store?
- [ ] Is that second store product-relevant (its silent loss would matter)?
- [ ] Is propagation done from application code after the primary write?

## Atomicity Checks

- [ ] Is there a single real transaction wrapping the primary write(s)?
- [ ] Are the primary writes free of hand-rolled compensating deletes standing in
      for a transaction?
- [ ] Is the propagation intent (outbox row) inserted inside that same
      transaction?

## Delivery Checks

- [ ] Is there an asynchronous worker that delivers the outbox events?
- [ ] Does delivery retry on transient failure?
- [ ] Is there an explicit terminal state (`failed` / DLQ) that is alertable?
- [ ] Are failed events reprocessable?
- [ ] Is delivery idempotent via a deterministic key?

## Observability Checks

- [ ] Are propagation failures persisted as state, not only logged?
- [ ] Is the lag (eventual consistency) a documented contract?
- [ ] Is there an archival policy for `done` rows?
- [ ] If events need causal order, does the worker honor it?

## Stop Conditions

- [ ] Propagation intent would live only in process memory (fire-and-forget).
- [ ] No real transaction backs the outbox row.
- [ ] The derived store is critical but failures have no durable, reprocessable
      path.
- [ ] It is unclear whether loss is acceptable at this stage.
