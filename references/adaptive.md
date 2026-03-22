# Adaptive Guidance

This file explains when `/coding` should collapse or expand its workflow.

## When to skip the brief

Skip the brief when all of the following are true:

- the request is already concrete
- the stack is known
- acceptance criteria are clear enough for planning
- there is little architectural ambiguity

In that case, go from clarification straight to `plan.md`.

## When to require the brief

Use the brief when any of the following is true:

- the user gives a large project idea rather than a scoped task
- there are multiple architecture paths
- acceptance criteria need normalization
- the planner would otherwise make risky assumptions
- the work is likely to require several implementation-review loops

## When one model may own brief and plan

Collapsing `brief` and `plan` into one model is allowed when:

- the same model already has the freshest context
- handoff loss is a bigger risk than model specialization
- the task is medium complexity rather than highly exploratory

## When to prefer two models

The two-model split is the default because it balances cost and quality:

- one model gathers and later reviews
- one model plans and implements

This keeps independent review without creating excessive orchestration overhead.

## When to add a third model

Add a third model only when:

- the implementation stage is the bottleneck
- the alternate model is measurably better for code generation in the user's stack
- the routing can stay local to the skill

If those conditions are not met, stay on the two-model default.

## Review loop heuristics

Send work back for another implementation pass when:

- `plan.md` tasks are incomplete
- acceptance criteria are not met
- validation failed
- implementation deviated from plan without justification

Do not reopen the loop for cosmetic or speculative issues that do not affect delivery.
