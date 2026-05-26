# Skill: to-checklist

Convert a generic principle into a project-specific planning or review checklist.

## Inputs

- Generic principle path
- Project context
- Relevant implementation area
- Target checklist location

## Output

A checklist for planning, review, or pre-merge verification.

## Procedure

1. Read the generic principle and metadata.
2. Identify the project-specific applicability questions.
3. Convert required checks into concrete checklist items.
4. Add implementation checks.
5. Add stop conditions.

## Constraints

- Checklist items should be verifiable.
- Avoid broad questions that cannot be answered during review.
- Include stop conditions when missing information creates architectural risk.
