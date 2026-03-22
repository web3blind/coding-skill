# Implementation Review Contract

Review should operate on explicit artifacts, not vague prose.

## Required review inputs

- the original user request
- the brief, if used
- `plan.md`
- changed files or a code diff
- validation results
- the implementation report

## Implementation report schema

```json
{
  "completed_work": ["implemented items"],
  "changed_files": ["path/to/file"],
  "tests_or_checks": ["command or validation result"],
  "blockers": ["current blocker or empty"],
  "open_risks": ["remaining risk or empty"],
  "plan_deviations": ["where implementation diverged from plan"],
  "recommended_next_step": "what should happen next"
}
```

## Review delta schema

If review sends work back, it should produce a bounded delta:

```json
{
  "issue": "what is missing or wrong",
  "why_it_matters": "impact on plan or acceptance criteria",
  "required_change": "specific correction to make",
  "acceptance_check": "how to verify the fix"
}
```

## Review rules

- Prefer evidence from `plan.md`, code, and validation over style opinions.
- Reject only the delta that is actually wrong or incomplete.
- Do not widen scope unless the user explicitly changes scope.
- If the work is acceptable, the review should say so clearly and allow final reporting.
