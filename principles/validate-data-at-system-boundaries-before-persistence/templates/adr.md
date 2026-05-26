# ADR-<number>: Validate Data at System Boundaries Before Persistence

Status: <proposed | accepted | superseded>

## Context

<Describe the project-specific persistence, external, memory, search, analytics, or retrieval boundaries affected by this decision.>

<Explain why invalid or incomplete data at these boundaries would create risk in this project.>

## Decision

This project will validate data before writing it to persistent, external, shared, or long-lived storage boundaries.

Each affected boundary must define the minimum valid payload required by that destination before data is written.

## Scope

This decision applies to:

- <boundary or module>;
- <boundary or module>;
- <boundary or module>.

This decision does not apply to:

- temporary request-local data;
- short-lived UI state;
- explicitly raw ingestion paths where downstream consumers know the data is raw.

## Implementation Expectations

- Validate required identifiers before writes.
- Preserve ownership, access scope, provenance, or versioning when relevant.
- Reject, skip, quarantine, retry, or explicitly mark invalid data as degraded.
- Log validation failures separately from transport failures.
- Keep validation close to the write boundary.

## Consequences

- Invalid data fails closer to the source.
- Downstream consumers can trust persisted records more safely.
- Write paths may need explicit validation and failure handling.
- Raw ingestion paths must be clearly marked and documented.

## References

- Generic principle: principles/validate-data-at-system-boundaries-before-persistence/principle.md
