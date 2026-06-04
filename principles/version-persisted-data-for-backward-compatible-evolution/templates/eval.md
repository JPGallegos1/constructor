# Eval: Version Persisted Data for Backward-Compatible Evolution

Status: experimental

## Purpose

Evaluate whether persisted derived data is self-describing across time and whether
readers decode it tolerantly as the schema evolves.

## Manual Checks

- [ ] Identify all persisted *derived* types (outputs of interpreters, parsers,
      normalizers, scorers, models).
- [ ] For each, confirm the persisted payload carries an explicit schema version.
- [ ] Confirm the deserializer branches on version and does not throw on decode.
- [ ] Confirm fields added in later versions have safe defaults for old records.
- [ ] Confirm changed fields have explicit old→new migrations.
- [ ] Confirm open-map / JSONB reads guard for `undefined`.

## Semi-Automated Checks

- Search for serialization/persistence of derived types and verify a version field
  is written.
- Search for deserializers and verify a version branch exists.
- Search for open-map key reads (`record[`, JSONB field access) and verify guards.
- Search for newly added required fields and verify a default path for old records.

## Automated Checks

- Add a backward-compatibility test: feed a stored payload from an earlier version
  (without fields the current version added) and assert it decodes to a safe
  default without throwing and without producing `NaN`/`undefined` downstream.
- Add a type-change migration test: feed the old scalar shape and assert it
  migrates to the new object shape.
- Add a test asserting decode of an unknown (newer) version does not throw.
- Add a regression check that new persisted derived types include a version field.

## Known Limits

- Static search cannot prove a default is semantically *correct*, only present.
- Tests may miss legacy shapes that exist only in production data.
- Tolerant decoding can mask malformed records; observability of old/unexpected
  versions requires human review of the logs.
