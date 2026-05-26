# Artifact Model

This repository separates generic principles from project-specific adoption artifacts.

## Canonical Principle

`principle.md` is the source document. It should explain the principle independently of any single project, framework, vendor, or product.

It should answer:

- What is the principle?
- Why does it matter?
- When does it apply?
- When does it not apply?
- What should humans and agents check before implementation?

## Metadata

`metadata.yml` makes a principle discoverable by humans and agents.

It should include identifiers, category, risk level, applicable areas, recommended artifacts, and agent triggers.

## Project ADR

An ADR is a project-specific decision record derived from a generic principle.

It should explain how a specific project adopts the principle. It should not copy the full principle.

## Agent Rule

An agent rule is a project-specific operational instruction.

It should tell an agent when the principle applies, what to inspect, what to avoid, when to stop, and what implementation pattern is expected.

## Checklist

A checklist helps humans and agents evaluate whether the principle is being respected before implementation, during review, or before merge.

## Implementation Guide

An implementation guide translates the principle into practical flows, examples, edge cases, and anti-patterns for an adopting project.

## Eval

An eval describes how compliance could be tested manually, semi-automatically, or automatically.

Evals are experimental in the MVP.
