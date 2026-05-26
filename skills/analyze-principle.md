# Skill: analyze-principle

Review whether a `principle.md` is complete, project-agnostic, and adoptable.

## Inputs

- Path to `principle.md`
- Optional path to `metadata.yml`

## Checks

- Does the principle have a clear summary?
- Does it explain the core idea?
- Does it include first-principles reasoning?
- Does it define when it applies?
- Does it define when it does not apply?
- Does it include a decision rule?
- Does it include required checks?
- Does it include anti-patterns?
- Does it include agent guidance?
- Can it produce an ADR, rule, checklist, and implementation guide?

## Output

Return:

- missing sections;
- project-specific language that should be removed;
- unclear triggers;
- weak or missing agent guidance;
- suggested edits before adoption.
