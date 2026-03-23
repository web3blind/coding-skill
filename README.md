# coding

`/coding` is an OpenClaw skill for project-oriented delivery.

It is designed for sessions where the user describes a project, feature, or implementation goal and expects a structured handoff from clarification to planning to coding to review.

## What This Skill Does

The skill runs a bounded workflow:

1. Clarify what matters.
2. Create a short brief only if needed.
3. Produce a project folder with `plan.md`.
4. Implement against that plan.
5. Run a review loop until the work is acceptable.
6. Return a final report to the user.

The skill is optimized for:

- reducing ambiguity before planning
- keeping implementation scoped to a concrete plan
- using structured handoffs between models
- preventing model-routing changes from leaking into the rest of OpenClaw

It also ships with a machine-readable routing file for OpenClaw:

- [openclaw-routing.json](openclaw-routing.json)
- [openclaw-routing.schema.json](openclaw-routing.schema.json)

## Default Model Split

Recommended default with the current model set:

- `Minimax` -> clarification
- `Minimax` -> optional brief
- `Codex` -> planning and `plan.md`
- `Codex` -> implementation
- `Minimax` -> independent review
- `Minimax` -> final report

Allowed simplifications:

- skip the brief for simple tasks
- let one model own both brief and planning if handoff loss would be worse
- fall back to `Codex` for the full flow if only one model is available

## Core Rules

- The brief is optional.
- `plan.md` is the only required durable artifact.
- Implementation starts only after `plan.md` exists.
- Review works on structured inputs, not vague summaries.
- Review sends back a bounded delta instead of restarting the whole task.
- Model switching must stay local to the active `/coding` session.
- The skill must not change OpenClaw's global default model.

## Skill Structure

- [SKILL.md](SKILL.md) - controlling contract
- [openclaw-routing.json](openclaw-routing.json) - machine-readable stage routing and local-state metadata
- [openclaw-routing.schema.json](openclaw-routing.schema.json) - validation schema for the routing config
- [prompts/clarify.md](prompts/clarify.md) - clarification template
- [prompts/brief.md](prompts/brief.md) - optional brief template
- [prompts/plan.md](prompts/plan.md) - planning template
- [prompts/implement.md](prompts/implement.md) - implementation template
- [prompts/review.md](prompts/review.md) - review template
- [prompts/final-report.md](prompts/final-report.md) - user-facing closeout template
- [references/review-contract.md](references/review-contract.md) - handoff contract
- [references/presets.md](references/presets.md) - routing presets
- [references/adaptive.md](references/adaptive.md) - when to skip or use the brief

## Workflow Detail

### 1. Clarify

The skill asks only the questions that materially affect planning:

- goal
- target users
- stack constraints
- must-have features
- non-goals
- acceptance criteria
- risky unknowns

If the request is already concrete, clarification should stop early.

### 2. Brief

The brief is optional.

Use it when:

- the task is large
- architecture is still fuzzy
- acceptance criteria need normalization
- planning would otherwise depend on risky assumptions

Do not force a brief for small or already clear tasks.

### 3. Plan

Planning creates the first required artifact:

- project folder
- `plan.md`

`plan.md` should give implementation enough detail to execute without guessing.

### 4. Implement

Implementation follows `plan.md` and produces:

- code changes
- validation results
- a structured implementation report

#### TDD (Test-Driven Development)

For each implementation task, follow the RED → GREEN → REFACTOR cycle:

1. **RED** — Write a failing test that describes expected behavior
2. **GREEN** — Write minimal code to make the test pass
3. **REFACTOR** — Improve code without changing behavior

This ensures test coverage from the first lines, fast feedback, and confidence during refactoring.

#### Subagent-Driven Development (optional)

For large tasks requiring >30 minutes of work:

1. Break the plan into atomic tasks (2-5 minutes each)
2. Subagent completes task → Review → Next task
3. Micro-review after each subtask

Use for: long-running tasks, many independent subtasks.
Skip for: simple tasks (<15 min), tasks requiring deep context.

### 5. Review Loop

Review compares:

- the original request
- the brief if one exists
- `plan.md`
- changed files or diff
- validation results
- implementation report

If something is missing, review returns a bounded delta. Implementation handles that delta and hands back a new report.

### 6. Final Report

After review passes, the skill returns a concise user-facing summary of:

- what was delivered
- what was validated
- what remains limited or deferred

### 4-Phase Debugging

For complex bugs, use the formal debugging approach:

1. **Observe** — Gather facts: full error message, stack trace, context. Reproduce the issue.
2. **Hypothesize** — Form possible causes. Identify the most likely hypothesis.
3. **Test** — Write a test that confirms or refutes the hypothesis.
4. **Fix** — Make minimal change to solve the problem. Verify tests pass.

Repeat if the bug isn't fixed on the first attempt.

## Files Created During Use

By default, the skill should keep persistent artifacts minimal.

Required:

- `plan.md`

Only create additional saved artifacts if the user explicitly asks:

- `brief.md`
- `implementation-report.md`
- `review-report.md`

## Example Session

User:

```text
/coding
I need a Telegram bot for personal expense tracking. Python, SQLite, no external admin panel. MVP only.
```

Skill flow:

1. `Minimax` asks for the needed clarifications:
   - categories fixed or editable
   - monthly reports needed or not
   - export needed or not
   - deployment target
2. `Minimax` may produce a short brief if the task is still broad.
3. `Codex` creates the project folder and writes `plan.md`.
4. `Codex` implements the MVP from the plan.
5. `Minimax` reviews the result and either approves it or returns a bounded delta.
6. `Minimax` writes the final report for the user.

## Integration Notes For OpenClaw

To keep this skill isolated:

- store stage state under a local namespace such as `coding.*`
- apply model overrides only inside the active `/coding` session
- clear those overrides when the session ends
- do not mutate the global default conversation model

Suggested local state fields:

- `coding.session_id`
- `coding.stage`
- `coding.models`
- `coding.project_root`
- `coding.plan_path`
- `coding.review_round`
- `coding.acceptance_status`

The same state fields and stage routing are mirrored in `openclaw-routing.json` so the integration layer can consume them directly instead of parsing prose from `SKILL.md`.

Use `openclaw-routing.schema.json` to validate the routing config before loading it into OpenClaw.

## When Not To Use This Skill

This skill is not ideal for:

- casual one-off code questions with no project lifecycle
- tiny fixes that do not need planning
- requests where the user explicitly wants direct coding with no staged flow

For those cases, a simpler direct coding flow is usually better.

---

## Changelog

### 2026-03-23

Added from Superpowers methodology:

- **TDD (Test-Driven Development)** — RED → GREEN → REFACTOR cycle in Implementation
- **Subagent-Driven Development** — atomic tasks (2-5 min), micro-reviews for large tasks
- **4-Phase Debugging** — Observe → Hypothesize → Test → Fix
