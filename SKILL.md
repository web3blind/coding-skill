---
name: coding
description: OpenClaw project-delivery workflow for /coding. Uses focused clarifications, an optional brief, planner-produced plan.md, implementation-review loops, and skill-local model routing without changing global model defaults.
metadata:
  clawdbot:
    triggers:
      - /coding
      - программировать
---

# coding

Project-oriented delivery workflow for OpenClaw.

This skill is for `/coding` sessions where the user describes a project or feature and expects a high-quality handoff from clarification to plan to implementation.

## Hard Contract

These rules are the controlling contract for this skill.

1. This skill owns only the current `/coding` session.
2. Model switching must stay local to this skill session.
3. Do not change OpenClaw's global default model, global routing, or unrelated skills.
4. Clarification comes before planning when requirements are incomplete.
5. A separate "super-prompt" is optional, not mandatory.
6. If a brief is useful, keep it short and structured.
7. The planning stage must end with a project folder and a `plan.md` inside it.
8. After `plan.md` exists, implementation proceeds against that plan.
9. Implementation must report back in a structured form before review.
10. Review may send a delta back to implementation and repeat until the work is complete.
11. Do not create extra markdown artifacts such as `brief.md`, `implementation-report.md`, or `review-report.md` unless the user explicitly asks.
12. **Architectural unity, code transparency, and code hygiene are non-negotiable.** Every piece of code must be clear, consistent, and maintainable.

If anything in `references/`, `decisions/`, `prompts/`, or `openclaw-routing.json` conflicts with this contract, ignore the conflicting guidance.

## Priority Order

1. Direct user request and explicit task constraints
2. This `SKILL.md`
3. `prompts/` as reusable stage templates
4. `openclaw-routing.json` as machine-readable routing metadata
5. `references/` and `decisions/` as secondary support material

## Default Model Topology

Use the smallest model setup that preserves quality.

### Recommended current default

With the models currently available to the user, prefer:

- `Minimax` for intake clarifications
- `Minimax` for the optional brief
- `Codex` for planning and writing `plan.md`
- `Codex` for implementation
- `Minimax` for independent review and the final user-facing report

### Allowed simplifications

- Skip the brief for simple tasks.
- Let one model produce both the brief and the plan when that reduces handoff loss.
- If only one suitable model is available, `Codex` may handle the whole flow.

### Future third-model option

A third model such as `Kimi` is optional, not assumed.

Use it only if it is explicitly configured and has shown better implementation quality for the user's tasks.
If enabled, the safest experimental split is:

- `Minimax` -> clarify and final review
- `Codex` -> planning and orchestration
- `Kimi` -> implementation only

Do not introduce a third model just to increase process complexity.

## Stage Selection

Classify each `/coding` request before acting:

- `simple`: clear request, narrow scope, low ambiguity
- `standard`: meaningful project work, but manageable with one planning pass
- `complex`: many unknowns, architecture trade-offs, or likely multi-pass review

Routing guidance:

- `simple` -> clarify only what is missing, then go straight to plan
- `standard` -> clarify, optional brief, plan, implement, review
- `complex` -> clarify, brief, plan, implementation/review loop, final report

## Workflow

Run the session in this order.

### 1. Intake and Clarification

Collect only the information needed for a good plan.

Target questions include:

- project goal
- target users
- stack or platform constraints
- must-have features
- non-goals
- delivery constraints
- acceptance criteria
- unknowns or risky assumptions

Stop asking once the planner has enough signal to produce a defensible `plan.md`.

### 2. Optional Brief

Use a brief only when it improves planning quality.

The brief should stay short and structured:

- goal
- constraints
- stack
- acceptance criteria
- open questions
- risks

Do not force a brief for every task.
Do not persist the brief to disk unless the user explicitly asks.

### 3. Planning

The planning stage produces the first durable project artifact.

Required output:

- project folder
- `plan.md` inside that folder

`plan.md` should be concrete enough for an implementation model to execute without guessing.

At minimum, the plan should cover:

- scope
- non-goals
- milestones or phases
- expected files or subsystems
- implementation tasks
- validation or test strategy
- risks or assumptions
- definition of done

If architecture is unclear, the planner may compare options before choosing a direction.

### 4. Implementation

After `plan.md` exists, implementation may start.

The implementation model should:

- follow the plan instead of widening scope
- make concrete code changes
- run relevant validation where possible
- note blockers early
- keep a structured implementation report for handoff

Implementation does not finish with "code written". It finishes with a handoff packet suitable for review.

### 5. Review Loop

The review model checks the result against:

- the user's request
- the brief, if one exists
- `plan.md`
- test or validation results
- the implementation report

If the review finds gaps, it should return a bounded delta task to implementation instead of restarting the project from scratch.

Repeat the loop until:

- planned work is complete
- acceptance criteria are satisfied
- remaining gaps are explicitly accepted by the user

### 6. Final Report

The final user-facing report comes after review passes.

It should summarize:

- what was delivered
- what was validated
- any remaining limitations
- any explicit next steps

## Prompt Templates

Reusable stage templates live in `prompts/`.

Use them as execution scaffolding, not as mandatory saved artifacts.

Available templates:

- `prompts/clarify.md`
- `prompts/brief.md`
- `prompts/plan.md`
- `prompts/implement.md`
- `prompts/review.md`
- `prompts/final-report.md`

Machine-readable routing metadata:

- `openclaw-routing.json`

Rules:

- Fill placeholders with the current session context.
- Skip `brief.md` when the brief stage is unnecessary.
- Use `plan.md` after the project folder is known.
- Do not dump raw templates to the user unless that is the task.

## Handoff Contracts

Use structured handoffs between stages.

### Brief contract

When a brief is used, it should contain:

- `goal`
- `constraints`
- `stack`
- `acceptance_criteria`
- `unknowns`
- `risks`

### Implementation report contract

Every implementation pass should hand review a structured packet containing:

- `completed_work`
- `changed_files`
- `tests_or_checks`
- `blockers`
- `open_risks`
- `plan_deviations`
- `recommended_next_step`

### Review delta contract

If review sends work back, it should produce a bounded delta containing:

- `issue`
- `why_it_matters`
- `required_change`
- `acceptance_check`

## Routing Isolation

All routing and stage state must remain local to this skill.

Use skill-local state such as:

- `coding.session_id`
- `coding.stage`
- `coding.models`
- `coding.project_root`
- `coding.plan_path`
- `coding.review_round`
- `coding.acceptance_status`

Rules:

- Apply model overrides only inside the active `/coding` session.
- Clear or discard those overrides when the skill exits.
- Do not mutate the main OpenClaw conversation defaults.
- Do not let `/coding` routing leak into unrelated chats or skills.

## Artifact Rules

Persist only what is necessary.

Required durable artifact:

- `plan.md`

Optional durable artifacts only when explicitly requested by the user:

- a saved brief
- a saved implementation report
- a saved review report

Suggested filenames:

- `brief.md`
- `implementation-report.md`
- `review-report.md`

Prefer chat-local or runtime-local handoffs for everything except `plan.md`.

## STOP And Ask

Stop and ask before proceeding if any of the following is true:

- The requested project behavior is ambiguous.
- Core requirements or acceptance criteria are missing.
- The target stack or runtime constraints are unknown and matter to the plan.
- The project folder path is unclear.
- The plan would require risky assumptions the user has not accepted.
- The implementation needs dependencies, credentials, auth changes, or external services that were not agreed.
- A reviewer requests changes that widen scope beyond the plan.
- You are unsure which stage should own the next action.
- You are unsure whether a model switch would escape the current skill session.

## References And Decisions

Use `references/` and `decisions/` as read-only support material.

- Read them only when needed.
- Treat them as secondary to this file.
- If they appear stale, follow this `SKILL.md`.

Use `openclaw-routing.json` when OpenClaw needs a machine-readable routing table for stages, model presets, local state fields, and handoff contracts.

## Security Checklist

When implementing and reviewing code, always verify these security aspects:

### 1. Code Reviewers

Integrate automated security scanning into the workflow:

- **Semgrep** — static analysis for OWASP Top 10
- **Snyk** — vulnerability scanning
- **CodeRabbit** — AI-powered code review
- **Sourcery** — automated refactoring and checks

Add these to CI/CD pipeline to prevent vulnerable code from reaching `main`.

Example (GitHub Actions):
```yaml
- name: Run Semgrep
  uses: returntocorp/semgrep-action@v1
  with:
    config: p/owasp-top-ten
```

### 2. Access Control

For projects with authenticated users:

- Use centralized authorization (Auth0, Supabase Auth, or similar)
- Follow the principle: "if a rule doesn't explicitly allow an action, it denies it"
- Test admin actions from regular user accounts
- Verify permissions on the backend for every request, not just at login

### 3. Rate Limiting

Protect API endpoints from abuse:

- Implement rate limits per user/IP/API key
- Use Upstash (for Vercel/Next.js), Cloudflare Rate Limiting, or similar
- Set stricter limits for AI endpoints (e.g., 10 req/min vs 100 req/min)
- Identify users by multiple factors: user ID, device fingerprint, API key

### 4. Row-Level Security (RLS)

For database-driven projects (Supabase, PostgreSQL):

- Enable RLS on every table that stores user data
- Create policies for SELECT, INSERT, UPDATE, DELETE
- Always set context on each query (which user is querying?)
- RLS protects against data leaks even if code has bugs

Example (PostgreSQL):
```sql
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users see own documents" ON documents
 FOR SELECT USING (auth.uid() = user_id);

CREATE POLICY "Users insert own documents" ON documents
 FOR INSERT WITH CHECK (auth.uid() = user_id);
```

### 5. API Keys and Secrets

- **Never commit secrets to Git** — use `.env` files
- Store production secrets encrypted or hashed (SHA-256 for signatures, AES-256 for data at rest)
- Use JWT with RS256/PS256 for authentication
- Use environment-specific configuration (dev vs prod)

## Git Safety Rules

**NEVER roll back to an older version.** The current state is the working state — modify it instead of reverting.

### Before Work
1. Run `git status` and `git rev-parse HEAD` to know the current state
2. Note the current commit hash
3. If working on a significant change, consider creating a feature branch:
   ```bash
   git checkout -b feature/your-feature-name
   ```

### During Work
**FORBIDDEN commands:**
- `git checkout` (any form)
- `git switch` (to different branch/commit)
- `git reset` (any form)
- `git restore`
- `git pull` (can cause unwanted merges)
- `git rebase`
- `git merge`

**ALLOWED commands:**
- `git status` — check state
- `git diff` — see changes
- `git log` — view history
- `git add` — stage files
- `git commit` — save changes
- `git branch` — list/create branches (no switching)

### After Work
1. Run `git diff` to show all changes
2. Ask user to review before applying
3. If user wants to revert — they will do it manually

### Why This Matters
An agent can accidentally run `git checkout` on an old commit, resetting the project state. This has happened — the fix is to prohibit these commands entirely.

## Debugging Errors

When analyzing errors (especially Telegram API parse errors, Markdown errors, etc.):

1. **Always look at the FULL error message** - check the exact byte offset to find WHERE in the text the problem is
2. **Find ALL special characters** - don't just focus on one part (e.g., URL). Look for: `_` (underscore), `*` (asterisk), `\` (backslash), `@` (at sign), etc.
3. **Check the ENTIRE variable** - don't assume, print/log the full value
4. **Cross-reference with error log** - if error says "byte offset 306", the problem is FAR into the message, not at the beginning

Example: Telegram error "can't parse entities at byte offset 306" means the problem is 306 characters into the message, not in the first few words.

## Security Notes

- No network is required by this skill itself.
- The skill should avoid creating extra files beyond the project folder and `plan.md` unless requested.
- Keep model routing local to the skill so the broader OpenClaw environment stays stable.
