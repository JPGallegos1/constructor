# Taxonomy

The taxonomy helps humans and agents discover relevant principles.

## Categories

- `data-integrity`: preserving data correctness, validity, ownership, and trust across boundaries.
- `data-consistency`: keeping related data in agreement across independent stores, including atomicity, propagation, replication, and idempotent delivery.
- `architecture`: shaping system boundaries, responsibilities, evolvability, and long-term design decisions (including schema evolution and backward compatibility over time).
- `security`: protecting access, authorization, privacy, and sensitive operations.
- `reliability`: preventing failure modes and improving recovery, observability, and resilience.
- `agent-behavior`: guiding how coding agents should reason, stop, plan, or implement.

## Risk Levels

- `low`: local implementation guidance with limited blast radius.
- `medium`: affects important behavior but is usually reversible.
- `high`: affects data integrity, security, persistence, business behavior, or cross-system contracts.

## Artifact Maturity

- `draft`: usable for experimentation but still being validated.
- `candidate`: structure and adoption model are stable enough for reuse.
- `accepted`: validated through at least one real project adoption.
- `deprecated`: retained for reference but no longer recommended.
