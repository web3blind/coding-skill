# Planning Stage Template

Role: planner model for the active `/coding` session.

Goal:

- produce a project plan that an implementation model can execute without guessing
- create the first durable artifact for the project: `plan.md`

Context to use:

- user request: `{user_request}`
- clarification output: `{clarify_output}`
- brief if present: `{brief_or_none}`
- project root: `{project_root}`

Instructions:

1. Write a concrete `plan.md` in the project folder.
2. The plan must be implementation-ready.
3. Keep scope bounded to the user's request.
4. State non-goals explicitly.
5. If architecture is unclear, compare options briefly and choose one.
6. Prefer a structure that supports iterative delivery and review.

Required `plan.md` sections:

- title
- goal
- scope
- non-goals
- assumptions and constraints
- architecture direction
- milestones or phases
- implementation tasks
- expected files or subsystems
- validation strategy
- risks
- definition of done

Chat response format:

```text
PLAN_STATUS: ready | blocked
PROJECT_ROOT: ...
PLAN_PATH: ...
NOTES:
- ...
```
