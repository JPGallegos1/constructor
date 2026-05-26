# Implementation Guide: Validate Data at System Boundaries Before Persistence

## Goal

Ensure that data crossing into persistent, external, shared, or long-lived storage satisfies the minimum contract required by that destination before it is written.

## Recommended Flow

```txt
domain data
  -> build destination payload
  -> validate payload contract
  -> handle invalid payloads explicitly
  -> persist valid payload
```

## Implementation Notes

- Define the contract at the boundary where the data leaves the controlled domain layer.
- Validate runtime payloads, not only static types.
- Include identifiers, ownership, provenance, versioning, and timestamps when future consumers need them.
- Treat raw ingestion as a separate policy and mark it clearly.
- Log validation failures with enough context to debug the source.

## Example

```ts
const payload = buildPayload(domainData);

const validation = validatePayload(payload);

if (!validation.success) {
  logger.warn("Persistence skipped due to invalid payload", {
    reason: validation.error,
    entityId: domainData.id,
  });

  return;
}

await persist(payload);
```

## Anti-Patterns

- Serializing JSON and assuming the payload is valid.
- Writing partial data to long-lived stores without marking it as partial.
- Adding validation only after consumers fail.
- Logging network failures but ignoring domain validation failures.

## Review Notes

- Check the write path, not only the data builder.
- Confirm invalid payloads cannot be silently persisted.
- Confirm downstream consumers know whether data is trusted, raw, partial, or degraded.
