# Agent Rule: Version Persisted Data for Backward-Compatible Evolution

## Applies When

- You create or modify code that serializes *derived* data (an interpreter,
  parser, normalization, scoring, or model output) to a long-lived store.
- You add, rename, split, or change the type of a field on a persisted type.
- You introduce a new required field on an existing persisted type.
- You create or modify code that deserializes records that older code may have
  written.
- You read open maps (`Record<string, …>`) or JSONB blobs.

## Inspect

- Whether the record is a raw fact or a derived interpretation.
- Whether the persisted payload carries an explicit schema version.
- The decoder: does it branch on version, default new fields, migrate changed
  fields?
- Every open-map key access for an `undefined` guard.
- Whether the snapshot is meant to be immutable history.
- Whether decoding via an old/unexpected version is logged.

## Required Behavior

- Stamp an explicit schema version on derived payloads at write time (on the
  persisted type, not the in-memory model).
- Treat an absent version as the first/implicit version.
- Decode tolerantly: read what you understand, default the rest, migrate changed
  shapes, never throw on decode.
- Guard every open-map key read for `undefined`.
- Agree the safe default per newly added field — it is part of the contract.

## Avoid

- Serializing derived data with no version marker.
- Assuming the reader is the same code as the writer.
- A decoder that assumes the newest shape and throws or yields `NaN` on old
  records.
- Re-running new logic over old inputs to rewrite snapshots promised as immutable.
- Inferring the producing version from other fields instead of an explicit version.
- Silent defaulting with no observability.

## Stop And Plan When

- Derived data is persisted but no schema version exists.
- A field's type/shape is changing and the reader has no migration for the old
  shape.
- A new required field would reach consumers as `undefined` for old records.
- The snapshot is immutable history but the only "fix" on offer is re-deriving it.
- The correct safe default for a new field is unclear.

## Reference

- Generic principle: principles/version-persisted-data-for-backward-compatible-evolution/principle.md
