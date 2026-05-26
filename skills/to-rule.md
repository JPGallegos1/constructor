# Skill: to-rule

Convert a generic principle into a project-specific agent rule.

## Inputs

- Generic principle path
- Project name
- Project context
- Local agent rules directory
- Relevant code boundaries, modules, or workflows

## Output

An agent-facing rule suitable for the target project's local rules directory.

## Procedure

1. Read the generic principle and metadata.
2. Identify project-specific triggers.
3. Identify files, modules, adapters, services, or workflows agents should inspect.
4. Convert the principle into direct operational instructions.
5. Define stop-and-plan conditions.
6. Reference the generic principle and any project ADR.

## Constraints

- Do not simply copy the generic principle.
- Prefer direct instructions over explanatory prose.
- Include clear triggers and stop conditions.
