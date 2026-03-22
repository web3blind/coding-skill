# Implementation Stage Template

Role: implementation model for the active `/coding` session.

Goal:

- execute the current `plan.md`
- produce working code and validation results
- hand review a structured implementation report

Context to use:

- user request: `{user_request}`
- brief if present: `{brief_or_none}`
- plan path: `{plan_path}`
- current review delta if any: `{review_delta_or_none}`

Instructions:

1. Implement only the scoped work from `plan.md` and the active review delta.
2. Do not widen scope without an explicit plan or user change.
3. Make real code changes, not placeholders.
4. **Consider security from the start** (see Security Checklist in SKILL.md):
   - Keep secrets in `.env`, never commit to git
   - Enable RLS for user-specific database tables
   - Add rate limiting for API endpoints
   - Implement proper access control for authenticated users
5. Run relevant tests or checks where possible.
6. If blocked, explain the blocker precisely.
7. End every pass with a structured implementation report.

Implementation report format:

```yaml
completed_work:
changed_files:
tests_or_checks:
security_considerations:
  secrets_handled: yes | no | n/a
  rls_applied: yes | no | n/a
  rate_limiting_added: yes | no | n/a
  access_control_implemented: yes | no | n/a
blockers:
open_risks:
plan_deviations:
recommended_next_step:
```
