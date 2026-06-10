# Checklist: Version Persisted Data for Backward-Compatible Evolution

Use this checklist before implementation, during review, or before merge.

## Applicability

- [ ] Does this change persist *derived* data (output of some logic), not just raw
      immutable input?
- [ ] Will a different version of the code read this data back later?
- [ ] Is the persisted snapshot meant to be immutable history?

## Contract Checks

- [ ] Does the persisted payload carry an explicit schema version?
- [ ] Is the version on the persisted type, distinct from the in-memory model?
- [ ] Is "absent version ⇒ first/implicit version" handled?
- [ ] Do fields added in later versions have an agreed safe default for old records?
- [ ] Do fields whose type/shape changed have an explicit old→new migration?

## Implementation Checks

- [ ] Does the deserializer branch on version?
- [ ] Does decoding never throw on an unexpected or older shape?
- [ ] Is every open-map / JSONB key read guarded for `undefined`?
- [ ] Are new records written with the current version?
- [ ] Is decoding via an old/unexpected version logged (not silent)?

## Stop Conditions

- [ ] Derived data is persisted with no version marker.
- [ ] The reader assumes a single shape and would crash or misread old records.
- [ ] A new required field would surface as `undefined`/`NaN` to consumers.
- [ ] An immutable-history snapshot would be "fixed" by re-deriving over old inputs.
- [ ] The safe default for a new field is undecided.
