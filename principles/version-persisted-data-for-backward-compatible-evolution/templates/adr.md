# ADR-<number>: Version Persisted Data for Backward-Compatible Evolution

Status: <proposed | accepted | superseded>

## Context

<Describe the project-specific derived data that is persisted and later read by
evolving code — interpreter snapshots, normalized records, parsed projections,
model outputs.>

<Explain which of these snapshots are meant to be immutable history, and what
breaks today when the producing logic changes and old and new records coexist in
the same store.>

## Decision

This project will make persisted derived records self-describing across time.

Each such record will carry an explicit schema version on write, and every reader
will decode tolerantly: branching on version, applying agreed safe defaults for
fields added later, migrating fields whose shape changed, and never throwing on
decode.

Absence of a version marker will be treated as the first/implicit version, giving
backward compatibility for existing data with no migration.

## Scope

This decision applies to:

- <persisted derived type / store>;
- <persisted derived type / store>;
- <persisted derived type / store>.

This decision does not apply to:

- raw immutable input with a stable shape;
- short-lived UI or request-local state;
- fully rebuildable throwaway stores.

## Implementation Expectations

- Add an explicit `schemaVersion` to the persisted payload type (distinct from the
  in-memory model).
- Write the current version on all new records; treat absent/`null` as the first
  version.
- Branch the deserializer by version; default new fields safely; migrate changed
  fields explicitly.
- Guard every open-map / JSONB key access for `undefined`.
- Make decoding via an old or unexpected version observable, not silent.

## Consequences

- New code can read old records without crashing or silently misreading them.
- Schema changes no longer require a coordinated migration of historical data.
- The deserializer accumulates version branches and needs periodic pruning.
- Safe defaults become part of the data contract and must be agreed per field.

## References

- Generic principle: principles/version-persisted-data-for-backward-compatible-evolution/principle.md
- Complement: principles/validate-data-at-system-boundaries-before-persistence/principle.md
