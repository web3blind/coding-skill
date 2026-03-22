# ADR-001: Project Delivery Stage Flow

## Status

Accepted (v3)

## Context

The older Shaw workflow emphasized many fixed stages and optional local state, but the user now wants `/coding` to act as an OpenClaw project-delivery pipeline with explicit model handoffs.

The main failure modes to prevent are:

- planning before requirements are clear
- mandatory prompt artifacts that add cost without improving quality
- implementation starting before a concrete `plan.md` exists
- review relying on vague summaries instead of structured handoff data
- model overrides leaking outside the skill

## Decision

Adopt a stage flow centered on delivery rather than prompt ceremony:

1. Clarify
2. Brief if needed
3. Plan
4. Implement
5. Review loop
6. Final report

Key rules:

- The brief is optional.
- `plan.md` is the required durable artifact that unlocks implementation.
- Implementation and review exchange structured handoff packets.
- Review returns bounded delta work instead of forcing full replans.
- The loop continues until the plan and acceptance criteria are satisfied.

## Consequences

- Simple tasks can skip unnecessary prompt-building overhead.
- Complex tasks still get a normalizing brief when it materially improves planning quality.
- The project gains one durable source of truth: `plan.md`.
- Review becomes more reliable because it evaluates explicit implementation facts.
- The workflow maps cleanly onto multi-model routing inside the skill.
