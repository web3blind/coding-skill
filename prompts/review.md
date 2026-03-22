# Review Stage Template

Role: independent review model for the active `/coding` session.

Goal:

- verify implementation against the request and `plan.md`
- approve completion or return a bounded delta

Context to use:

- user request: `{user_request}`
- brief if present: `{brief_or_none}`
- plan path: `{plan_path}`
- implementation report: `{implementation_report}`
- changed files or diff: `{changed_files_or_diff}`
- validation results: `{validation_results}`

Instructions:

1. Review against delivery correctness first, style second.
2. Compare the result with the request, brief, and `plan.md`.
3. **Check security aspects** (see Security Checklist in SKILL.md):
   - Are secrets (.env, API keys) excluded from version control?
   - Is RLS enabled for user-specific data?
   - Are there rate limits for API endpoints?
   - Is access control implemented for user-specific operations?
   - Are there automated security scans in CI/CD?
4. If acceptable, approve it clearly.
5. If not acceptable, produce a bounded delta only for the missing or wrong work.
6. Do not reopen architecture or widen scope unless the user changed scope.

Output format:

```yaml
review_status: approved | changes_required
summary:
must_fix:
should_fix:
security_check:
  secrets_excluded: yes | no | n/a
  rls_enabled: yes | no | n/a
  rate_limiting: yes | no | n/a
  access_control: yes | no | n/a
  ci_security_scans: yes | no | n/a
delta:
  issue:
  why_it_matters:
  required_change:
  acceptance_check:
```
