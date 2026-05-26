# Skill: adopt

Adopt a generic principle into a specific project by generating selected local artifacts.

## Inputs

- Principle path
- Target project name
- Target project context
- Relevant modules, boundaries, or workflows
- Desired artifacts
- Target directories
- Adoption mode: lightweight, formal, or complete

## Output

Depending on the adoption mode, generate project-specific artifacts such as:

```txt
docs/adr/000X-<principle-slug>.md
.agents/rules/<principle-slug>.md
.agents/checklists/<principle-slug>.md
docs/guides/<principle-slug>.md
.agents/evals/<principle-slug>.md
```

## Procedure

1. Read the generic principle.
2. Read the principle metadata.
3. Gather or infer project context.
4. Determine which artifacts are appropriate.
5. Use the specific `to-*` skills to generate each artifact.
6. Keep project-specific details inside the target project artifacts, not the generic principle.

## Constraints

- Do not install every artifact by default.
- Do not copy the entire principle into an ADR unless the target project explicitly asks for it.
- Prefer local, operational agent rules over generic copied text.
- Keep evals experimental.
