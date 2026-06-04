# Principle: Version Persisted Data for Backward-Compatible Evolution

## Summary

Any data that is persisted and later read by a *different version of the code*
must carry an explicit schema version, and every reader must decode it
tolerantly — branching on the declared version, defaulting unknown or missing
fields safely, and never crashing on a shape it did not expect.

A system should not serialize derived data to a long-lived store as if the code
that reads it will be the same code that wrote it. Write and read are separated
in time, and the code changes in between.

A persisted record without a version marker is an undecidable message: the reader
cannot tell which rules produced it, so it cannot safely interpret it.

## Core Idea

Writing and reading are separated in time. The code changes between those two
moments. When you serialize something to a durable store you are not "saving an
object" — you are sending a message to a future version of your own program that
does not exist yet.

That message must be self-describing. The reader on the other side is newer code,
running over a table where records written months apart, by different versions of
the producing logic, coexist with incompatible shapes. If the message does not
say *which rules it was born under*, the reader has only two options, both bad:
assume the newest shape and crash on old records, or guess — and silently produce
a wrong result.

The first-principles question is:

> When this byte string is read back by code that has since changed, can the
> reader tell which contract produced it, and read it without breaking?

If the answer depends on "the reader happens to still match the writer," the
contract is implicit and temporary. It holds until the first deploy that changes
the shape, and then it fails — far from the change that caused it.

This is sharpest for **derived data** — the stored output of some logic
`f(input)` (an interpreter result, a normalized projection, a parsed snapshot, a
model output). A raw fact ages well because it is immutable by nature. A derived
interpretation is coupled to the version of `f` that produced it: the day `f`
improves, every record written by the old `f` is now a different shape than the
new code expects.

## Principle Statement

Make every persisted derived record self-describing across time, and make every
reader tolerant of versions it has not seen.

Two halves, write and read:

**On write** — stamp an explicit schema version on the persisted payload. The
data must *tell the reader* which rules it was born under. A version field is not
business data; it is the decoder selector for future code.

**On read** — the deserializer is always a *Tolerant Reader*. It reads what it
understands, defaults the rest, preserves or ignores what it does not recognize,
and never throws during decoding.

```txt
write path (version N):
  build derived payload
  stamp schemaVersion = N
  persist

read path (version M, M >= N):
  read raw payload
  detect schemaVersion   (absent => treat as the first/implicit version)
  branch decoder by version
  map to the in-memory model:
    new fields        -> safe default
    changed fields    -> migrate old shape -> new shape
    unknown fields    -> preserve or ignore, never crash
  return a usable object   (never throw on decode)
```

The obligatory direction here is **backward compatibility**: new code must read
old data. Records written before the change do not disappear when the new version
deploys. **Forward compatibility** — old code, still live on another replica
during a rolling deploy, reading data written by the new code — is the secondary
but real case: tolerant reading and additive change protect it too.

## First-Principles Reasoning

A serialized object outlives the code that produced it. The bytes are durable;
the producing function is not. So the bytes must encode enough to be decoded by a
function that has since changed — otherwise decoding depends on an invariant
("reader == writer") that persistence specifically breaks.

Adding the version marker reframes the stored record from an *implicit* contract
(its meaning lives in whichever code is currently deployed) into an
*explicit, self-describing* one (its meaning travels with the data). The cost of
this is one field at write time. The cost of *not* doing it compounds: each schema
change adds another undistinguishable shape to the same table, until reading
becomes data archaeology — inferring the producing version from the presence or
type of other fields.

The deepest point mirrors the read side: a Tolerant Reader is the dual of a
strict writer. The system can be strict about *what it emits* (a single, current
shape) while being liberal about *what it accepts* (every shape it has ever
emitted). That asymmetry — strict out, tolerant in — is what lets the schema move
without a coordinated, all-at-once migration of historical data.

## Raw Fact vs Derived Interpretation

|                        | Raw fact                       | Derived interpretation                 |
|------------------------|--------------------------------|----------------------------------------|
| Example                | the original input text        | the parsed/normalized result of it     |
| Source of truth        | the datum itself               | the **logic** that produced it         |
| Ages well              | yes — immutable by nature      | no — coupled to a version of `f`        |
| If the code improves   | still valid                    | now stale / a different shape           |
| Versioning need        | low (it does not change)       | high (it changes every time `f` does)  |

The discipline is to know which one you are persisting. Raw facts can be re-derived
from at any time and rarely need a version. Derived interpretations are frozen
snapshots of a computation; if the domain promises they are immutable history
("this is the interpretation used *at that moment*"), you cannot re-run the new
logic over old inputs without rewriting history — so backward-compatible reads are
not optional, they are the only correct strategy.

## When This Applies

- A change persists *derived* data — the output of interpreting, parsing,
  normalizing, scoring, or otherwise transforming an input — that future code
  will read back.
- The persisted snapshot is meant to be immutable history (it must reflect the
  logic used at the time it was written, not the current logic).
- The shape of a persisted field is about to change: a new required field, a type
  change (e.g. a scalar becoming an object), a renamed or split field.
- A store holds open maps / loosely-typed bags (`Record<string, …>`, JSONB) whose
  keys may be present or absent across records.
- Records written by different versions of the producing logic will coexist in the
  same table or collection.

## When This Does Not Apply

- The data is purely raw, immutable input that is never re-derived and whose shape
  is stable by definition.
- The data is short-lived UI or request-local state that is never read back by a
  different version of the code.
- The store is genuinely throwaway and fully rebuildable on every deploy, so old
  shapes never survive long enough to be read by new code.
- The change is strictly additive *and* every reader already tolerates missing
  fields — in which case a version bump may be unnecessary, but the tolerant-read
  discipline still applies.

## Decision Rule

Before persisting derived data that future code will read, decide how the reader
will know what it is reading.

1. Add an explicit `schemaVersion` (or equivalent) to the persisted payload —
   distinct from the in-memory model, attached to the stored record.
2. Adopt the rule **"absence of a version ⇒ the first/implicit version"**. This
   buys backward compatibility for existing data with no migration — but it works
   *once*: from the moment you start stamping versions, the version must be
   explicit, because absence can no longer be assumed to mean "v1" reliably for
   data written afterward.
3. Make the deserializer branch on version and apply **safe defaults** for fields
   introduced later and **explicit migrations** for fields whose type/shape
   changed. It must never throw on decode and never produce `NaN`/`undefined`
   reaching code that assumes presence.
4. For open maps, treat every key access as `value | undefined` — never read a key
   without a guard.
5. Bump the version (and add a decoder branch) whenever a change is *not* a
   purely additive, already-tolerated field.

## Why This Matters

Missing schema versioning does not fail in the demo. It fails later, under
accumulation — exactly when a product has months of data and a maturing
interpreter. The first schema change ships, old and new records sit in the same
table, and a reader that assumes the new shape breaks on the old ones, or worse,
reads `undefined.level` as a silent wrong value.

| | Unversioned serialize | Versioned + tolerant read |
|---|---|---|
| Reader/writer coupling | Reader must match writer | Reader decodes any version it knows |
| Old data after a schema change | Crashes or silently misreads | Decoded via its declared version |
| New required field | `undefined` reaches consumers | Safe default for old records |
| Type change (scalar → object) | `.field` throws / `NaN` | Old shape migrated on read |
| Cost of the next schema change | Data archaeology | One decoder branch |
| Net result | Latent corruption surfaced late | Evolvable schema, no data migration |

## Honest Trade-Offs

- **A version field on every derived record.** Trivial to add early; the whole
  point is that it is cheap now and expensive once three shapes already coexist.
- **Decoder branches accumulate.** Each surviving version needs a read path. This
  is real maintenance — periodically retiring versions (by back-filling/migrating
  the oldest records) keeps the decoder bounded.
- **Tolerant reading can mask bugs.** Defaulting silently can hide a genuinely
  malformed record. Pair tolerant decode with observability: log when a record
  decodes via an old or unexpected version, so "tolerated" never means "invisible".
- **Defaults are a domain decision.** "Safe default" must be agreed per field — a
  wrong default is its own silent error. The default is part of the contract, not
  an implementation detail.

## Required Checks

- Identify which persisted records are *derived* (output of logic) versus raw
  immutable input.
- Confirm derived records carry an explicit schema version on write.
- Confirm the decoder branches on version and never throws on decode.
- Confirm fields added in later versions have agreed safe defaults for older
  records.
- Confirm type/shape changes have an explicit old→new migration in the reader.
- Confirm open-map (`Record<…>` / JSONB) reads guard for `undefined` on every key.
- Confirm decoding via an old or unexpected version is observable, not silent.

## Implementation Guidance

The expected shape is a self-describing persisted payload plus a version-branching
Tolerant Reader at the decode boundary.

The write path should build the derived payload, stamp the current schema version,
and persist the two together — the version belongs to the *persisted* type, not
the in-memory model.

The read path should detect the version (treating absence as the first/implicit
version for legacy data), select the matching decoder branch, map to the in-memory
model with safe defaults for newly added fields and explicit migrations for
changed fields, and return a usable object without ever throwing on decode.

```ts
// Persisted type carries the version; the in-memory model does not.
type PersistedRecord = {
  schemaVersion: number;        // decoder selector for future code
  // ...derived fields...
};

function decode(raw: unknown): DomainModel {
  const version = (raw as any).schemaVersion ?? 1; // absent => implicit v1, once
  switch (version) {
    case 1:
      return fromV1(raw); // map old shape; default fields v2 later added
    case 2:
      return fromV2(raw);
    default:
      // unknown (newer) version: decode the fields we understand, ignore the rest
      return fromKnownFields(raw);
  }
  // never throw here
}
```

The migration to introduce versioning into an existing, unversioned store is
itself trivial: add a nullable version column / default `null`, treat `null` /
absent as the first version, and start writing the current version on all new
records. No back-fill of historical data is required.

## Anti-Patterns

- Serializing a derived object to a long-lived store with no version marker.
- Assuming the reader is the same code as the writer ("it round-trips in our
  tests, so it is fine").
- A decoder that assumes the newest shape and `throw`s or yields `NaN` on older
  records.
- Reading open-map keys (`record[k]`) without a `undefined` guard.
- Re-running new logic over old inputs to "fix" stored snapshots that the domain
  promised were immutable history.
- Inferring the producing version from the presence or type of *other* fields
  (data archaeology) instead of an explicit version.
- Defaulting silently with no log, so a malformed or unexpectedly-old record is
  invisible.

## Agent Guidance

When an agent creates or modifies code that persists derived data to a long-lived
store, or that decodes such data, it should check whether the record is
self-describing and whether the reader tolerates versions it has not seen.

The agent should ask:

- Is this data a raw fact or a derived interpretation? (Derived data is the case
  that needs versioning.)
- Will a *different version* of the code read this back later?
- Does the persisted payload carry an explicit schema version?
- Does the decoder branch on version, default new fields safely, and migrate
  changed fields — without ever throwing on decode?
- For open maps, is every key read guarded for `undefined`?
- Is the snapshot meant to be immutable history? (If so, re-deriving over old
  inputs is forbidden; backward-compatible reads are required.)
- Is decoding via an old/unexpected version observable, not silent?

If derived data is persisted without a version, or the reader assumes a single
shape, the agent should stop and plan the versioning and tolerant-read strategy
before implementing the write or the read.

## Derived Artifacts

This principle can be adopted into projects as:

- project ADRs;
- agent rules;
- review checklists;
- implementation guides;
- evals, currently experimental.

## Related Principles

- `validate-data-at-system-boundaries-before-persistence` — its complement across
  the read/write axis. That principle governs *whether the data is valid as it is
  written* (strict, schema-on-write, at one moment). This one governs *whether the
  data stays readable as the code evolves* (tolerant, schema-on-read-back, across
  time). Together they are Postel's law for persistence: **write strict, read
  tolerant.** That principle treats "version of the payload or transformation
  pipeline" as a field to preserve on write; this one defines how the reader
  *uses* that version to survive change.
- `persist-propagation-intent-in-source-of-truth-transaction` — adjacent on a
  different axis. That principle keeps two stores consistent at one instant
  (atomicity across stores); this one keeps one store readable across versions
  (compatibility across time). A derived projection that is rebuilt by
  re-projection (as that principle allows) still needs *this* principle so the
  rebuilt and historical records can be read by the same evolving code.

## Rule of Thumb

A persisted derived record is a message to a future version of your own code.
Stamp it with the rules it was born under, and read every record as if it might
be older than the code reading it.

Write strict. Read tolerant. Version early — one field now, or data archaeology
later.
