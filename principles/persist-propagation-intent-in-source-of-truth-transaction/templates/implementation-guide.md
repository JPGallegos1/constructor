# Implementation Guide: Persist Propagation Intent in the Source-of-Truth Transaction

## Goal

Guarantee that when a write must reach a second independent store, the intent to
propagate is durable — committed atomically with the primary write — and
delivered asynchronously with retries and idempotency, so a delivery failure is
recoverable instead of silently lost.

## Recommended Flow

```txt
BEGIN transaction (single RPC / DB function)
  insert primary entity + detail rows
  insert outbox row (status=pending, idempotency_key, payload)
COMMIT                      -- one atomic boundary

worker (async, separate process):
  SELECT ... WHERE status = pending FOR UPDATE SKIP LOCKED
  deliver to second store (idempotency_key)
    success -> UPDATE status = done
    failure -> attempts++, stays pending; attempts > max -> failed (alert / DLQ)
```

## Implementation Notes

- The outbox is only as strong as the transaction it rides on. If the primary
  writes are separate autocommitted statements with a compensating delete, make
  them a single transaction first — otherwise the outbox insert reintroduces the
  same window inside the database.
- The outbox row is not business data; it is a durable promise of delivery.
- Use `FOR UPDATE SKIP LOCKED` polling for a simple, robust worker. Full CDC
  (e.g. Debezium) is usually overkill.
- Reuse a deterministic idempotency key derived from source entity IDs so the
  second store collapses duplicates from at-least-once delivery.
- Define an archival/cleanup job for `done` rows.
- If any event needs causal order, order by `created_at` or a partition key.

## Example

Source-of-truth write (atomic, via a single RPC):

```sql
-- one transaction, all or nothing
create function create_entity_with_outbox(p_entity jsonb, p_records jsonb)
returns void language plpgsql as $$
declare v_id uuid;
begin
  insert into entities (...) values (...) returning id into v_id;
  insert into entity_records (...) select ... from jsonb_array_elements(p_records);
  insert into outbox (status, idempotency_key, payload)
    values ('pending', v_id::text, jsonb_build_object('entity_id', v_id));
end; $$;
```

Worker delivery (idempotent, reprocessable):

```ts
const batch = await db.query(
  "SELECT * FROM outbox WHERE status='pending' ORDER BY created_at " +
  "FOR UPDATE SKIP LOCKED LIMIT 100"
);

for (const row of batch) {
  try {
    await memoryStore.add(row.payload, { idempotencyKey: row.idempotency_key });
    await db.query("UPDATE outbox SET status='done' WHERE id=$1", [row.id]);
  } catch (err) {
    const attempts = row.attempts + 1;
    const status = attempts > MAX_ATTEMPTS ? "failed" : "pending";
    await db.query(
      "UPDATE outbox SET attempts=$1, status=$2, last_error=$3 WHERE id=$4",
      [attempts, status, String(err), row.id]
    );
    // status='failed' is alertable and reprocessable — the intent is NOT lost
  }
}
```

## Anti-Patterns

- `void sync().catch(log)` — the intent lives only in RAM.
- Inserting the outbox row in a third autocommitted call instead of the
  transaction.
- Treating the second store as a co-equal source of truth instead of a derived
  projection.
- Marking a failure with only a log line and no persisted, reprocessable state.

## Review Notes

- Check the write path, not only the sync module.
- Confirm the outbox insert shares the primary write's transaction.
- Confirm there is a worker with retries, terminal state, and idempotency.
- Confirm failures are observable as queryable state and can be reprocessed.
