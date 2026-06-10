# Principle: Persist Propagation Intent in the Source-of-Truth Transaction

## Summary

When a write must also reach a second, independent store, the *intent* to
propagate that write must be persisted inside the **same transaction** as the
primary write, and delivered afterwards by an asynchronous worker with retries
and idempotency.

A system should not write the same event to two independent stores directly from
application code (a *dual write*), because there is no shared atomic boundary
between them.

A propagation that fails should leave a durable, queryable, reprocessable trace,
not a line in a log.

## Core Idea

Atomicity only exists *within a transactional domain*.

When application code writes to two independent domains — for example a
relational database and a separate memory, search, or analytics store — there are
two commit points and no protocol coordinating them. Between commit one and
commit two there is a window. Any failure in that window (a 5xx, a timeout, a
network partition, the process dying) leaves the first store written and the
second not.

Because nothing reconciles, the divergence is **permanent, not transient**. It is
not an occasional bug; it is the structural consequence of having no common
atomic boundary.

The first-principles question is:

> Where, durably, does the intent to propagate this event live?

If the answer is "in process memory" — a dispatched-but-not-awaited promise plus
a `console.error` on failure — then a critical state transition has been placed
in the most volatile tier of the system, and a failed propagation is lost with no
trace.

## Principle Statement

Reduce "two stores must agree" to "**one** store must commit."

Place the intent of the second write inside the same atomic domain as the first.
An outbox row is not business data: it is a **durable promise** that the event
will be delivered.

```txt
BEGIN
  insert primary entity + detail rows
  insert outbox row (pending, idempotency_key, payload)
COMMIT            -- one atomic boundary, all or nothing

worker (async):
  SELECT ... WHERE status = pending FOR UPDATE SKIP LOCKED
  deliver to second store (idempotency_key)
  on success -> status = done
  on failure -> attempts++, stays pending or -> failed (alert / DLQ)
```

After commit, two invariants hold:

1. If the entity exists in the source of truth, its propagation event exists —
   they are the same transaction.
2. A delivery failure no longer destroys information — the state lives in a row
   (`pending` / `failed`, `attempts`), which is queryable, alertable, and
   reprocessable.

## First-Principles Reasoning

A dual write has no shared atomic boundary, so it cannot stay consistent under
failure. *Fire-and-forget* dispatch (dispatched, not awaited, only logged on
failure) is the worst case: the intent to replicate lives only in RAM, so an
exhausted-retry failure is lost silently.

The outbox converts **temporal coupling** (the primary write succeeding only if
the second store is available *at that instant*) into coupling on a durable
buffer — which is what buffers are for. It reframes the second store as a
**consumer of a reliable event log**, not a co-equal system of record. The source
of truth keeps one authoritative write; everything else is a derived projection
that can be rebuilt by re-projection.

The deepest point: the danger was never the divergence itself — a derived store
can always be rebuilt. The danger is the **silence**: a failure mode with no
durable trace, no signal, and no path to recovery. Good architecture does not
promise nothing fails; it promises that when something fails, the system knows,
and the information needed to recover is still there.

## When This Applies

- A write must also be reflected in a second, independent store (memory / vector
  store, search index, analytics pipeline, cache with persistence semantics, or
  an external system) where divergence matters.
- That second store is a product-relevant asset whose silent loss is
  unacceptable.
- Propagation is currently done directly from application code after the primary
  write (dual write), especially fire-and-forget.
- The primary write itself is composed of multiple autocommitted statements with
  a hand-rolled compensating rollback instead of a real transaction (the same
  antipattern one layer down).

## When This Does Not Apply

- There is only one store; no cross-store propagation exists.
- The second store is a cheap, best-effort cache or fan-out that can be rebuilt
  trivially and whose loss is genuinely acceptable.
- The propagation already happens inside a single shared transaction (no second
  independent atomic domain is crossed).
- Early-stage / pre-PMF context where the derived store is fully reconstructable
  by re-projection, loss is tolerable, and the operational cost of a worker is
  not yet justified — *provided this is a conscious, documented trade, not an
  accident*. The moment the derived store becomes a differentiator, this
  exemption expires.

## Decision Rule

Before propagating a write to a second independent store, ask where the intent to
propagate lives durably. If the answer is "only in process memory," redesign:

1. Ensure a **real transaction** wraps the primary write(s). If the primary
   writes are separate autocommitted statements with a compensating delete, fix
   that first — otherwise inserting the outbox row in a third autocommitted call
   reintroduces the exact window you set out to eliminate.
2. Insert an outbox row (`pending`, idempotency key, payload) **inside that same
   transaction**.
3. Deliver from an asynchronous worker with retries and an explicit terminal
   state (`failed` / DLQ) that is alertable and reprocessable.
4. Make delivery idempotent via a deterministic key so at-least-once delivery
   becomes effectively-once.

## Why This Matters

Dual writes "work" in the demo and diverge silently under load — exactly when a
product is scaling. The failure surfaces far from its cause: stale or missing
memory, wrong search results, misleading automation, with no durable trace
pointing back to the lost propagation.

| | Fire-and-forget dual write | Outbox + worker |
|---|---|---|
| Delivery guarantee | at-most-once (0 or 1) | at-least-once (≥1) |
| Where intent lives | RAM + log line | Durable row in the source of truth |
| Failure after retries | Lost | Persisted as `failed`, reprocessable |
| Duplicate risk | N/A | Resolved by idempotency key |
| Net result | Possible silent divergence | Effectively-once |

## Honest Trade-Offs

- **Explicit eventual consistency.** The second store lags by the worker's
  polling interval. It lagged before too — but now it is a conscious contract,
  not an accident.
- **You need a worker process.** More operable infrastructure. Polling with
  `FOR UPDATE SKIP LOCKED` is simple and robust; full CDC (e.g. Debezium) is
  usually overkill.
- **Table maintenance.** `done` rows accumulate; periodic archival is needed.
- **Ordering.** If any event needs causal order, the worker must honor it (by
  `created_at` or a partition key).

The trade now points the right way: pay operational cost and latency to buy
durability of the derived asset.

## Required Checks

- Identify every place application code writes to a second independent store
  after a primary write.
- Confirm a single real transaction wraps the primary write(s) — not separate
  autocommitted statements plus a compensating delete.
- Confirm the propagation intent is persisted inside that transaction (outbox
  row), not dispatched from memory.
- Confirm an asynchronous worker delivers with retries, a terminal `failed`
  state, and idempotency.
- Confirm failures are persisted and alertable, not only logged.

## Implementation Guidance

The expected implementation shape is a transactional outbox backed by the same
source-of-truth store as the primary write.

The write path should:

- open one real transaction;
- perform the primary state change;
- insert a pending outbox record with a deterministic idempotency key and payload;
- commit once.

Delivery should happen outside the request path through a worker that claims
pending records, sends them to the derived store, retries transient failures, and
persists terminal failures as reprocessable state.

If the primary write is not already atomic, make that transaction real before
adding the outbox. Otherwise the outbox only moves the dual-write window to a
different place.

## Anti-Patterns

- `void sync().catch(log)` — fire-and-forget propagation from the write path.
- Writing to a second store/SDK/HTTP client immediately after a commit, outside
  any transaction.
- Treating two independent stores as co-equal systems of record instead of
  source-of-truth + derived projection.
- Simulating atomicity in application code with a hand-rolled compensating
  delete.
- Inserting the outbox row in a separate autocommitted call ("same transaction"
  in aspiration only).
- Logging only transport failures while the propagation intent is lost.

## Agent Guidance

When an agent creates or modifies code that propagates a write to a second,
independent, persistent store, it should check whether the propagation intent is
durable before the request returns.

The agent should ask:

- What two atomic domains does this write touch?
- Is the second store a derived projection or is it being treated as a co-equal
  source of truth?
- Where does the intent to propagate live if delivery fails — a durable row, or
  process memory?
- Does a single real transaction wrap the primary write(s)?
- Is the outbox row inserted inside that same transaction?
- Is there a worker with retries, a terminal `failed` state, and idempotency?
- Are failures persisted and alertable, not just logged?
- Is the derived store cheap to rebuild and is loss acceptable at this stage?
  (If yes, document the exemption; if no, the outbox is required.)

If propagation intent is not durable, the agent should stop and plan the outbox
strategy — including making the underlying transaction real — before
implementing the write.

## Derived Artifacts

This principle can be adopted into projects as:

- project ADRs;
- agent rules;
- review checklists;
- implementation guides;
- evals, currently experimental.

## Related Principles

- `validate-data-at-system-boundaries-before-persistence` — its complement.
  That principle governs *whether the data crossing a boundary is valid*; this
  one governs *whether the cross-store effect propagates durably*. The
  hand-rolled compensating delete is the same "atomicity simulated in
  application code" antipattern one layer down.
- `version-persisted-data-for-backward-compatible-evolution` — adjacent on the
  time axis. This principle keeps two stores consistent by making propagation
  durable; that one keeps persisted derived records readable as schemas and code
  evolve.

## Rule of Thumb

Never let the only record of an important intent be a line in a log.

Turn "two stores must agree" into "one store must commit." Keep one source of
truth; make everything else a derivable projection with a delivery guarantee.
