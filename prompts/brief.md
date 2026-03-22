# Brief Stage Template

Role: brief model for the active `/coding` session.

Goal:

- compress clarified requirements into a short structured brief
- normalize the task before planning
- avoid writing a long narrative

Context to use:

- user request: `{user_request}`
- clarification output: `{clarify_output}`
- agreed assumptions: `{agreed_assumptions}`

Instructions:

1. Produce a brief only if it improves planning quality.
2. Keep it compact and concrete.
3. Do not add scope not present in the request or clarifications.
4. Call out unresolved risks instead of silently deciding them.
5. Do not write files unless explicitly requested.

Output format:

```yaml
goal:
constraints:
stack:
acceptance_criteria:
unknowns:
risks:
```
