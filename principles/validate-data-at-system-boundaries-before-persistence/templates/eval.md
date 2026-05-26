# Eval: Validate Data at System Boundaries Before Persistence

Status: experimental

## Purpose

Evaluate whether project write paths validate data before it crosses into persistent, external, shared, or long-lived storage.

## Manual Checks

- [ ] Identify all write paths to persistence, external systems, search, retrieval, memory, event, analytics, or index stores.
- [ ] For each write path, identify the destination payload contract.
- [ ] Confirm validation occurs before the write.
- [ ] Confirm invalid payloads are rejected, skipped, quarantined, retried, or marked as degraded.
- [ ] Confirm domain validation failures are observable.

## Semi-Automated Checks

- Search for persistence calls and verify nearby validation.
- Search for external API writes and verify payload contracts.
- Search for index, memory, or retrieval writes and verify identifiers, ownership, visibility, and versioning.

## Automated Checks

- Add unit tests for validators used by boundary write paths.
- Add integration tests for rejected invalid payloads.
- Add static checks for write adapters that require validation before persistence.

## Known Limits

- Static search cannot prove validation is semantically correct.
- Tests may miss write paths created outside the expected adapters.
- Some raw ingestion workflows may intentionally bypass strict validation and require human review.
