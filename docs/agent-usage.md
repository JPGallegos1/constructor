# Agent Usage

Agents should not blindly apply every principle in this repository.

Agents should first determine whether the current project has adopted a principle and whether the current task touches the relevant boundary, risk, or implementation area.

## Recommended Hierarchy

```txt
1. Read the generic principle.
2. Check whether the target project has adopted it.
3. If adopted, prioritize the project ADR and local agent rule.
4. Use the checklist before implementation or review.
5. Use the implementation guide when writing code.
6. Use evals only when present and applicable.
```

## Planning Mode

An agent should stop and enter planning mode when:

- the principle applies but the project has no local adoption artifact;
- the implementation touches a high-risk boundary;
- required validation, ownership, provenance, or observability is missing;
- the correct local policy is unclear;
- an implementation would violate an accepted project ADR.

## Skills

The files in `skills/` are markdown-first transformation instructions. They are intended to be invoked by humans or agents, for example as `/to-adr`, `/to-rule`, or `/adopt`.

They are not executable commands in the MVP.
