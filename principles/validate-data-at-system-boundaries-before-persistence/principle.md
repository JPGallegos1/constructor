# Principle: Validate Data at System Boundaries Before Persistence

## Summary

Any data that crosses a system boundary into a persistent, external, shared, or long-lived storage layer must be validated before it is written.

A system should not rely on downstream consumers to discover whether persisted data has the correct shape, meaning, ownership, or minimum required fields.

Invalid data should fail close to the source, not later when another part of the system attempts to read, interpret, retrieve, or reason over it.

## Core Idea

Data is not only a structure. Data is a promise.

When a system persists a record, event, document, memory, index entry, or derived artifact, it is implicitly declaring:

> This data is valid enough to be reused later.

If that promise is false, the system may continue operating temporarily, but future behavior becomes less reliable.

The problem may not appear at write time. It may appear later during retrieval, search, reporting, automation, reasoning, or user-facing workflows.

This is the danger of treating persistence as a schema-on-read process by accident.

## Principle Statement

Systems should apply schema-on-write validation at critical boundaries.

Before writing data into a persistent or external destination, the system must validate that the payload satisfies the minimum contract required by that destination.

This applies especially when data crosses from a controlled domain layer into:

- external systems;
- long-lived storage;
- shared databases;
- memory layers;
- search or retrieval systems;
- indexes;
- event stores;
- analytics pipelines;
- caches with persistence semantics;
- automation or agent-facing storage.

## First-Principles Reasoning

Persistence changes the cost of being wrong.

Before data is persisted, invalid state can often be rejected, rebuilt, recalculated, or kept local to a request. After data is persisted, it can be reused by other workflows, indexed by search systems, read by agents, interpreted by analytics, or trusted by future product behavior.

Validation before persistence protects the contract between producers and consumers. It makes the write boundary the place where data quality is enforced instead of pushing ambiguity downstream.

## When This Applies

- A change writes data to a persistent store.
- A change sends data to an external system.
- A change creates or updates a search, retrieval, memory, analytics, or event index.
- A change persists derived data that future workflows will trust.
- A change writes data that contains ownership, visibility, provenance, or versioning requirements.

## When This Does Not Apply

- The data is temporary, local, and not reused outside the current execution path.
- The destination is explicitly raw ingestion storage and downstream consumers know it is raw.
- The data is only used for short-lived UI presentation and is not persisted.
- The project has intentionally accepted schema-on-read for an exploratory or analytical workflow.

## Decision Rule

Before persisting data across a system boundary, define the minimum valid shape required by that destination.

At minimum, consider validating:

- required identifiers;
- ownership or access scope;
- required domain fields;
- normalized or canonical fields;
- source or provenance metadata;
- version of the payload or transformation pipeline;
- timestamps when relevant;
- consistency between related fields;
- whether partial data is acceptable.

Invalid records should not be silently persisted.

Depending on the use case, invalid data may be:

- rejected;
- skipped;
- quarantined;
- retried;
- logged with a domain-level warning;
- persisted only if explicitly marked as partial, degraded, or raw.

## Why This Matters

Invalid data is cheaper to prevent than to clean.

If incomplete, ambiguous, or partially transformed data is persisted, the system may silently accumulate corruption.

The failure may only become visible much later, when another workflow depends on that data.

Common symptoms include:

- incorrect search results;
- missing records;
- misleading automation;
- broken reports;
- inconsistent user experiences;
- hard-to-debug downstream errors;
- duplicated cleanup work;
- loss of trust in the system.

The later the error is discovered, the harder it is to understand where it came from.

## Schema-on-Write vs Schema-on-Read

Schema-on-write means the system validates data before accepting it into a persistent boundary.

Schema-on-read means the system accepts flexible data first and interprets it later.

Schema-on-read can be useful in exploratory, analytical, or raw ingestion contexts.

However, systems should avoid accidentally applying schema-on-read to data that will later drive product behavior, automation, retrieval, or decision-making.

The important question is not:

> Can this payload be serialized?

The important question is:

> Is this payload valid enough to be trusted later?

## Required Checks

- Identify the boundary being crossed.
- Define the minimum valid payload for the destination.
- Validate required identifiers and domain fields before writing.
- Preserve ownership, access scope, provenance, or versioning when relevant.
- Decide whether invalid data is rejected, skipped, quarantined, retried, or marked as degraded.
- Log domain validation failures separately from transport failures.

## Boundary-Specific Policies

Not every boundary needs the same validation policy.

Some destinations should reject incomplete data strictly.

Others may accept partial data if it is clearly marked as raw, degraded, or not suitable for trusted workflows.

Each boundary should define its own policy.

### Trusted Product Data

Should usually require strict validation.

The system should reject records that do not satisfy the minimum contract.

### Search or Retrieval Indexes

Should validate identifiers, searchable content, visibility rules, ownership metadata, and index version.

Bad indexing can cause records to disappear, leak, duplicate, or rank incorrectly.

### Memory or Reasoning Layers

Should validate that persisted data is complete, meaningful, and safe to reuse.

Incomplete memory can lead to misleading future behavior.

### Event or Analytics Stores

May allow partial records, but only when explicitly marked as partial or degraded.

Raw ingestion is acceptable only if downstream systems know they are reading raw data.

### External Integrations

Should validate outbound payloads before sending them outside the system.

The external provider should not be responsible for discovering whether the internal domain data is valid.

## Implementation Guidance

A typical boundary flow should look like this:

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

The important sequence is:

```txt
domain data
  -> build payload
  -> validate payload
  -> persist externally
```

Not:

```txt
domain data
  -> serialize JSON
  -> persist externally
  -> hope consumers handle bad data later
```

## Anti-Patterns

- Treating serialization as validation.
- Assuming static types are enough at runtime.
- Sending partially transformed domain data into long-lived systems.
- Relying on downstream consumers to discover invalid records.
- Allowing external systems to become dumping grounds for ambiguous data.
- Logging only transport errors while ignoring domain validation errors.
- Persisting data without ownership, provenance, or versioning when those fields matter for future interpretation.

## Agent Guidance

When an agent creates or modifies code that writes data to a persistent, shared, external, or long-lived boundary, it should check whether runtime validation exists before the write operation.

The agent should ask:

- What boundary is this data crossing?
- Will this data be reused later?
- What is the minimum valid shape?
- Are required identifiers present?
- Is ownership or access scope preserved?
- Are derived or normalized fields complete?
- What happens if the payload is partial?
- Is invalid data rejected, skipped, quarantined, or marked as degraded?
- Are domain validation failures logged separately from transport failures?

If validation is missing, the agent should stop and plan the validation strategy before implementing the write.

## Derived Artifacts

This principle can be adopted into projects as:

- project ADRs;
- agent rules;
- review checklists;
- implementation guides;
- evals, currently experimental.

## Related Principles

- `persist-propagation-intent-in-source-of-truth-transaction` — adjacent on the
  propagation axis. This principle governs *whether data is valid before it is
  written across a boundary*; that one governs *whether the intent to propagate a
  committed write is durable and reprocessable*.
- `version-persisted-data-for-backward-compatible-evolution` — complement across
  time. This principle governs *whether persisted data is valid when written*;
  that one governs *whether persisted derived data remains readable as code and
  schemas evolve*.

## Rule of Thumb

Do not let external or persistent systems become the first place where data quality problems are discovered.

Validate before persistence.

Treat every long-lived write as a contract.
