# Implementation Guide: Version Persisted Data for Backward-Compatible Evolution

## Goal

Ensure derived data persisted to a long-lived store can be read by a future,
changed version of the code — by stamping an explicit schema version on write and
decoding tolerantly on read.

## Recommended Flow

```txt
write (version N):
  build derived payload
  stamp schemaVersion = N
  persist payload + version together

read (version M >= N):
  read raw payload
  detect version   (absent => first/implicit version)
  branch decoder by version
    new fields     -> safe default
    changed fields -> migrate old shape -> new shape
    unknown fields -> preserve or ignore
  return usable object   (never throw on decode)
```

## Implementation Notes

- Put the version on the *persisted* type, not the in-memory model.
- Treat absence/`null` as the first version — this gives backward compatibility
  for existing data with no back-fill. It is reliable only once; from then on,
  write the version explicitly.
- Agree the safe default for each newly added field; a wrong default is a silent
  error of its own.
- For open maps (`Record<string, …>` / JSONB), type every key access as
  `value | undefined` and guard it.
- Log when a record decodes via an old or unexpected version, so tolerance does
  not become invisibility.

## Example

```ts
type PersistedRecord = {
  schemaVersion: number;     // decoder selector for future code
  // ...derived fields...
};

function decode(raw: unknown): DomainModel {
  const version = (raw as any).schemaVersion ?? 1; // absent => implicit v1
  switch (version) {
    case 1:  return fromV1(raw);  // default fields added in v2
    case 2:  return fromV2(raw);
    default: return fromKnownFields(raw); // newer: decode what we know
  }
  // never throw here
}
```

## Migrating an Existing Unversioned Store

- Add a nullable version column (or JSONB default `null`).
- Treat `null` / absent as the first version.
- Start writing the current version on all new records.
- No historical back-fill is required for backward compatibility.

## Anti-Patterns

- Serializing derived data with no version marker.
- A decoder that assumes the newest shape and throws / yields `NaN` on old data.
- Reading open-map keys without an `undefined` guard.
- Re-deriving immutable-history snapshots with new logic.
- Defaulting silently with no log.

## Review Notes

- Check the read path, not only the write path.
- Confirm old records decode without throwing.
- Confirm new fields have agreed defaults and changed fields have migrations.
- Confirm decoding via an old/unexpected version is observable.
