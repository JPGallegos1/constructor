# Checklist: Validate Data at System Boundaries Before Persistence

Use this checklist before implementation, during review, or before merge.

## Applicability

- [ ] Does this change write data to persistent, external, shared, or long-lived storage?
- [ ] Does this change create or update search, retrieval, memory, event, analytics, or index data?
- [ ] Will future workflows trust or reuse this data?

## Contract Checks

- [ ] Is the destination boundary identified?
- [ ] Is the minimum valid payload defined?
- [ ] Are required identifiers present?
- [ ] Are required domain fields present?
- [ ] Are ownership, access scope, provenance, or version fields preserved when relevant?
- [ ] Is partial or degraded data explicitly allowed or rejected?

## Implementation Checks

- [ ] Is validation performed before the write?
- [ ] Is validation close to the write boundary?
- [ ] Is invalid data rejected, skipped, quarantined, retried, or marked as degraded?
- [ ] Are domain validation failures logged separately from transport failures?
- [ ] Are raw ingestion paths clearly marked as raw?

## Stop Conditions

- [ ] No validation contract exists for a long-lived write.
- [ ] Invalid data would be silently persisted.
- [ ] Ownership, provenance, visibility, or version metadata is required but missing.
- [ ] The policy for partial data is unclear.
