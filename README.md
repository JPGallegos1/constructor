# Constructor

Constructor is a project-agnostic toolkit for turning software construction principles into adoptable project artifacts: ADRs, agent rules, implementation guides, checklists, and evals.

## Purpose

Constructor stores software construction principles in a project-agnostic form and provides markdown-first skills for adapting those principles into project-specific artifacts.

The goal is not to install every principle into every project. The goal is to help humans and agents decide which principles apply, then generate the right local artifacts for a specific project context.

## Adoption Flow

```txt
Generic Principle
  -> Project ADR
  -> Agent Rule
  -> Checklist
  -> Implementation Guide
  -> Eval (experimental)
```

The generic principle remains in this repository. Project-specific ADRs, rules, checklists, and guides should be created inside the adopting project.

## Repository Structure

```txt
principles/
  <principle-slug>/
    principle.md
    metadata.yml
    templates/
      adr.md
      rule.md
      checklist.md
      implementation-guide.md
      eval.md

skills/
  init.md
  analyze-principle.md
  adopt.md
  to-adr.md
  to-rule.md
  to-checklist.md
  to-implementation-guide.md
  to-eval.md

templates/
  principle.md
  metadata.yml
  adr.md
  rule.md
  checklist.md
  implementation-guide.md
  eval.md

docs/
  artifact-model.md
  agent-usage.md
  contribution-guide.md
  installation-model.md
  taxonomy.md

registry.yml
```

## Principles

Current principles are listed in `registry.yml`.

- [Validate Data at System Boundaries Before Persistence](principles/validate-data-at-system-boundaries-before-persistence/principle.md): validate data crossing system boundaries before writing it to persistent, external, shared, or long-lived storage.
- [Persist Propagation Intent in the Source-of-Truth Transaction](principles/persist-propagation-intent-in-source-of-truth-transaction/principle.md): when a write must reach a second independent store, persist the propagation intent in the same transaction as the primary write and deliver it asynchronously with retries and idempotency.
- [Version Persisted Data for Backward-Compatible Evolution](principles/version-persisted-data-for-backward-compatible-evolution/principle.md): persisted derived data carries an explicit schema version and is decoded tolerantly so new code can read old records as schemas evolve.

These principles validate whether the repository format can produce useful ADRs, agent rules, checklists, implementation guides, and experimental evals for real projects.

## Status

This repository is markdown-first. Skills are currently instructions for agents and humans, not executable commands.

Evals are experimental and should be treated as future verification artifacts until the adoption model is proven.
