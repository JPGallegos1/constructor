# Contribution Guide

Principles should be useful across projects and should not depend on one framework, vendor, product, database, or language.

## Adding a Principle

Use `skills/init.md` to create the canonical structure:

```txt
principles/<principle-slug>/
  principle.md
  metadata.yml
  templates/
    adr.md
    rule.md
    checklist.md
    implementation-guide.md
    eval.md
```

Then add the principle to `registry.yml`.

## Writing Style

- Prefer durable engineering reasoning over tool-specific advice.
- Include examples when they clarify the principle, but do not let examples define the principle.
- Keep project-specific details out of the canonical principle.
- Use direct language for agent guidance.
- Mark experimental artifacts explicitly.

## Review Checklist

- Is the principle project-agnostic?
- Does it explain why the principle matters?
- Does it define when the principle applies and does not apply?
- Does it include a decision rule?
- Does it include agent guidance?
- Does metadata include useful triggers and applicable areas?
- Can the principle be adapted into an ADR, rule, checklist, and implementation guide?
