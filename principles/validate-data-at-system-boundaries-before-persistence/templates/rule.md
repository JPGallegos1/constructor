# Agent Rule: Validate Data at System Boundaries Before Persistence

## Applies When

- You create or modify code that writes to persistent storage.
- You create or modify code that writes to external systems.
- You create or modify code that writes to search, retrieval, memory, event, analytics, or index stores.
- You change ownership, visibility, provenance, versioning, or derived payload fields used by long-lived workflows.

## Inspect

- The payload built before the write.
- The destination contract.
- Required identifiers and domain fields.
- Ownership, access scope, provenance, and version metadata.
- Failure handling for invalid payloads.
- Logging for domain validation failures.

## Required Behavior

- Validate payloads before the write boundary.
- Keep validation close to the adapter, service, or domain operation that performs the write.
- Decide whether invalid data is rejected, skipped, quarantined, retried, or marked as degraded.
- Preserve enough metadata for future consumers to trust and interpret the data.

## Avoid

- Treating serialization as validation.
- Assuming static types prove runtime payload validity.
- Persisting partial derived data without marking it as partial or degraded.
- Relying on downstream consumers to discover invalid records.

## Stop And Plan When

- The change writes long-lived data but no validation contract exists.
- The destination requires ownership, provenance, or visibility metadata and the payload does not include it.
- Invalid data would be silently persisted.
- The correct policy for partial data is unclear.

## Reference

- Generic principle: principles/validate-data-at-system-boundaries-before-persistence/principle.md
