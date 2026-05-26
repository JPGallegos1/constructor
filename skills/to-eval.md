# Skill: to-eval

Convert a generic principle into an experimental project-specific eval.

Status: experimental

## Inputs

- Generic principle path
- Project context
- Relevant code paths or artifacts
- Verification target

## Output

An experimental eval describing how compliance could be checked manually, semi-automatically, or automatically.

## Procedure

1. Read the generic principle and metadata.
2. Define what compliance would mean in the target project.
3. Add manual checks.
4. Add possible semi-automated searches or commands.
5. Add future automated checks.
6. Document known limits.

## Constraints

- Do not present evals as authoritative in the MVP.
- Be explicit about what the eval cannot prove.
- Prefer practical verification ideas over theoretical completeness.
