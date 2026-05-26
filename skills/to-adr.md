# Skill: to-adr

Convert a generic principle into a project-specific ADR.

## Inputs

- Generic principle path
- Project name
- Project context
- Relevant boundaries, modules, or workflows
- ADR number or naming convention
- ADR status

## Output

A project ADR that explains how the target project adopts the principle.

## Procedure

1. Read the generic principle and metadata.
2. Identify why the principle matters in this project.
3. Identify the concrete project boundaries in scope.
4. State the project decision.
5. Define implementation expectations.
6. List consequences.
7. Reference the generic principle.

## Constraints

- Do not restate the entire generic principle.
- Do not include generic examples unless they clarify the project decision.
- Make the ADR specific enough that future agents can follow it.
