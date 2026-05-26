# Skill: init

Create a new generic principle in this repository.

## Inputs

- Principle name
- Principle slug
- Category
- Risk level
- One-sentence summary

## Output

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

## Procedure

1. Create the principle directory using kebab-case.
2. Copy `templates/principle.md` into `principles/<slug>/principle.md`.
3. Copy `templates/metadata.yml` into `principles/<slug>/metadata.yml`.
4. Copy global artifact templates into `principles/<slug>/templates/`.
5. Fill the metadata fields that are known.
6. Add the principle to `registry.yml`.

## Constraints

- Do not include project-specific details in the generic principle.
- Do not generate project ADRs or project rules from this skill.
- Keep evals marked as experimental.
