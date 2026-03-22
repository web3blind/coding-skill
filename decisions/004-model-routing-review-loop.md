# ADR-004: Skill-Local Model Routing and Review Loop

## Status

Accepted (v3)

## Context

`/coding` needs different model strengths across clarification, planning, implementation, and final review. At the same time, the user does not want these switches to interfere with the rest of OpenClaw.

The previous design focused on feature gates and tri-review mechanics. The current requirement is narrower and more practical:

- route models per stage
- keep routing isolated to the active skill session
- support a two-model default now
- leave room for an optional third implementation model later

## Decision

Use skill-local stage routing.

Default routing with the current model set:

- `clarify` -> `Minimax`
- `brief` -> `Minimax`
- `plan` -> `Codex`
- `implement` -> `Codex`
- `review` -> `Minimax`
- `final_report` -> `Minimax`

Allowed adjustments:

- Skip `brief` for simple work.
- Let one model handle both `brief` and `plan` when handoff loss would hurt quality.
- Fall back to `Codex` for all stages if a second model is unavailable.
- Add a third model only for implementation and only when explicitly configured.

Routing state must live inside the skill namespace and must be discarded on exit.

Review is loop-based:

- implementation hands over a structured implementation report
- review compares it against the request, brief, and `plan.md`
- review emits a bounded delta if fixes are needed
- implementation executes only that delta

## Consequences

- The user can start with two models without losing structure.
- Optional third-model experiments stay bounded to one stage.
- Global OpenClaw defaults remain untouched.
- Review becomes a controllable iterative loop rather than an unstructured veto.
