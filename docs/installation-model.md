# Installation Model

Projects should adopt only the principles and artifacts they need.

This repository is the source of generic principles and transformation skills. Adopting projects should receive local artifacts that fit their own structure, naming conventions, and governance model.

## Lightweight Adoption

A lightweight adoption may include:

```txt
.agents/rules/<principle-slug>.md
.agents/checklists/<principle-slug>.md
```

Use this when a project needs immediate agent behavior and review guidance, but does not yet need a formal ADR.

## Formal Adoption

A formal adoption may include:

```txt
docs/adr/000X-<principle-slug>.md
.agents/rules/<principle-slug>.md
.agents/checklists/<principle-slug>.md
docs/guides/<principle-slug>.md
```

Use this when the principle affects architecture, governance, security, data integrity, or long-term project behavior.

## Complete Adoption

A complete adoption may add an eval:

```txt
.agents/evals/<principle-slug>.md
```

Evals are experimental in the MVP.
