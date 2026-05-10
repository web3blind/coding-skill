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

`/coding` is also the owner-mode for substantial inspection of the local technical workspace state: code, config, skills, scripts, cron, workflow, routing, architecture, and substantial edits to those artifacts.

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
13. Any `/coding` execution that performs planning, implementation, or review via a spawned subagent must explicitly disable fast mode in that child session before substantive work starts.
14. Fast-mode inheritance from the parent session is forbidden for `/coding` subagents.
15. After a user confirmation like `да`, `делай`, `давай`, `давай попробуем`, `ок`, `go`, or `поехали`, do not spend the turn on a standalone acknowledgement. Start execution in that same turn.
16. After such a start signal, silent execution in the parent session is forbidden. The parent must either return the final result in the same assistant message, if the work is truly immediate, or send a short status/progress message and in that same turn hand execution off to a subagent.
17. If the work should continue without waiting for another user message, `/coding` should prefer a subagent, send standalone `/fast off` to that child, and then `sessions_yield` instead of ending with a passive acknowledgement.
18. If subagent handoff is unavailable, thread binding is unsupported, or `/fast off` cannot be applied in the child, immediately fall back to executing in the current session. Do not surface that internal limitation to the user as a blocker unless it truly prevents the requested work.
19. Do not send a standalone status update as a substitute for execution. A status/progress message is valid only when it is immediately followed in the same turn by the final result, by the actual execution handoff, or by visible continued execution in the current session under the fallback rule.
20. For ordinary `/coding`, subagent-driven execution is the default. Treat work as substantive by default and do not continue it in the parent session unless the task is limited to a necessary clarification, a truly immediate final answer, or the fallback rule is active because child execution is not operable in the current environment.
21. `/coding цикл` means autonomous execution loop: after confirmation, immediately start an implementation/review cycle and keep working until a concrete result or blocker is ready; use a subagent when available, otherwise continue in the current session under the fallback rule.
22. Do not make adjacent cleanup changes outside the requested scope. Do not rewrite nearby comments, formatting, naming, or unrelated code unless the user asked for that cleanup.
23. Prefer the simplest implementation that satisfies the request. Do not add speculative abstractions, configurability, extensibility, or future-proofing unless the user explicitly asks for them or the accepted plan requires them.
24. When ambiguity is minor and a reasonable default is available, choose it and record the assumption in the plan or final report. Ask the user only when the ambiguity would materially affect architecture, API, data model, UX, validation strategy, or acceptance criteria.

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

### Fast-mode isolation

`/coding` must not inherit fast-mode behavior from the parent chat when it spawns a child session for planning, implementation, or review.

Required rule:

- if a `/coding` child session is spawned, explicitly send a standalone `/fast off` to that child session before the actual planning, implementation, or review prompt
- do not rely on parent-session defaults, inheritance, or implied non-fast behavior
- if fast mode cannot be explicitly disabled in the child session, do not use that child for substantive work; immediately fall back to the current session unless the task is truly blocked for another reason

This is a mandatory execution rule, not an optional hint.

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

- `simple` -> clarify only what is missing, then go straight to plan, usually through a subagent unless the final answer is truly immediate
- `standard` -> clarify, optional brief, plan, implement, review, using a subagent by default
- `complex` -> clarify, brief, plan, implementation/review loop, final report, using a subagent-driven path

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

### Immediate execution after confirmation

When the user has already confirmed execution, `/coding` must treat that message as the start signal, not as a request for another acknowledgement turn.

Required behavior:

- If the task can be completed immediately, return the final result in the same assistant message.
- Otherwise, send a short status/progress message and, in the same turn, immediately start the execution path.
- Prefer spawning a subagent.
- Use the parent session for a necessary clarification only when that clarification materially blocks planning or execution.
- After spawning a child, explicitly disable fast mode there with a standalone `/fast off` message before the real task prompt.
- If child execution is not operable in the current environment, continue immediately in the current session instead of surfacing the handoff problem to the user.
- When a child is used, the parent session must then `sessions_yield` and wait for the child result instead of idling or continuing invisible execution in the parent.
- A status/progress message without immediate execution handoff, visible current-session execution, or final result is not sufficient.

### Forbidden silent execution

The following pattern is forbidden in `/coding`:

- the user gives a clear start signal
- the assistant appears to begin work
- no final result is returned in the same message
- no subagent handoff occurs
- no visible current-session execution begins
- no yielded execution state is established

This pattern creates "I started but nothing came back" failures and must be avoided.

### `/coding цикл`

`/coding цикл` is the explicit autonomous mode for `/coding`.

Semantics:

- confirmation from the user is enough to start; do not ask for a second go-ahead
- run as a subagent-driven loop, not as a single acknowledgement reply
- preferred loop: implement -> validate -> review -> either continue or report blocker
- return to the user only with one of three states:
  - concrete completed progress
  - a precise blocker that needs human input
  - a short checkpoint if the user explicitly asked for checkpoints

**TDD (Test-Driven Development)** — применяй для каждой задачи в implementation:

1. **RED** — Напиши падающий тест, который описывает ожидаемое поведение. Тест должен упасть, потому что функционала ещё нет.
2. **GREEN** — Напиши минимальный код, чтобы тест прошёл. Не пытайся сделать идеально — только то, что нужно для прохождения теста.
3. **REFACTOR** — Улучши код без изменения поведения. Тесты должны оставаться зелёными.

Повторяй этот цикл для каждой подзадачи. Это обеспечивает:
- Тестовое покрытие с первых строк кода
- Быструю обратную связь
- Уверенность при рефакторинге

---

### 4.0.1 Импорты и зависимости (Node.js/JavaScript)

**Все `require` / `import` должны быть в начале файла**, перед любым другим кодом (кроме комментариев).

Правила:

1. **require в начале файла:**
   ```javascript
   // ✅ Правильно
   const fs = require('fs');
   const path = require('path');

   function myFunc() { ... }

   // ❌ Неправильно (require внутри функции)
   function myFunc() {
     const fs = require('fs'); // НЕ делай так
   }
   ```

2. **Проверяй дубли:**
   - Перед добавлением нового `require` — проверь, не импортируется ли уже этот модуль в файле
   - Используй существующий импорт вместо создания нового
   ```javascript
   // ❌ Дублирование
   const fs = require('fs');
   // ...много кода...
   function later() {
     const fs = require('fs'); // Лишнее!
   }
   ```

3. **Порядок импортов:**
   - Встроенные модули Node.js (`fs`, `path`, `os` и т.д.)
   - Внешние npm-пакеты
   - Локальные модули проекта (с относительными путями `./` или `../`)

4. **Для ESM `import`:**
   ```javascript
   // ✅ Правильно
   import { readFile } from 'fs/promises';
   import express from 'express';
   import { helper } from './utils/helper.js';
   ```

---

### 4.1 Subagent-Driven Development (опционально)

Для больших задач, требующих много времени, используй subagent-driven подход.

Для `/coding цикл` subagent-driven execution предпочителен, но при технической недоступности child execution нужно сразу продолжать в текущей сессии, а не останавливать работу сообщением о внутреннем ограничении.

Для обычного `/coding` subagent-driven execution является режимом по умолчанию и должен пропускаться только для необходимого уточнения, когда финальный ответ реально можно вернуть сразу в текущем сообщении, или когда child execution недоступен в текущей среде.


0. **Перед реальной работой subagent-а принудительно отключи fast mode** — отправь в child session отдельное сообщение `/fast off`, дождись применения и только потом передавай задачу на планирование, implementation или review. Если это невозможно, не используй child и сразу работай в текущей сессии.
1. **Разбей план на атомарные задачи** (каждая 2-5 минут работы)
2. **Subagent выполняет задачу → Review → Следующая**
3. После каждой подзадачи — микро-ревью перед переходом к следующей

Когда использовать:
- Задача требует >30 минут непрерывной работы
- Много независимых подзадач
- Риск потери контекста при длинной реализации

Когда НЕ использовать:
- Простые задачи (<15 минут)
- Задачи требуют глубокого контекста, который сложно передать subagent-у

---

The implementation model should:

- follow the plan instead of widening scope
- make concrete code changes
- prefer the simplest implementation that meets the accepted plan and request
- avoid speculative abstractions or future-proofing that were not requested
- avoid adjacent cleanup outside the requested scope
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

**Проверка тестов и TDD:**

- `tests_written`: yes | no | n/a
- `tdd_cycles_completed`: yes | no | n/a

Если тесты не написаны или TDD циклы не завершены — запросить объяснение или исправление.

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
- `tdd_cycles` — количество пройденных циклов TDD (RED → GREEN → REFACTOR)
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
- For spawned `/coding` child sessions, explicitly apply `/fast off` inside the child before the first substantive stage prompt.
- Treat a missing explicit child-session fast-off step as a contract violation.

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

### Hard Stop (всегда спрашивать)

Остановиться и спросить, если:

- Стек неизвестен и влияет на план
- Credentials или external services не согласованы с пользователем
- Review запрашивает выход за рамки плана

### Soft Check (спрашивать только если нельзя вывести из контекста)

Остановиться и уточнить, если:

- Path (путь до проекта) непятен
- Criteria (критерии приёмки) отсутствуют и не выводятся из контекста

### Автономные решения

Всё остальное — агент решает сам и фиксирует в `plan.md`:

- Как разбить задачу на этапы
- Какую архитектуру выбрать
- Какие файлы создать
- Как реализовать валидацию

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

---

## 4-Phase Debugging

Формализованный подход к отладке — используй для сложных багов:

### Phase 1: Observe (Наблюдение)

- Собери факты об ошибке: полное сообщение, стек-trace, контекст выполнения
- Воспроизведи проблему — найди минимальные шаги для повторения
- Зафиксируй: что ожидалось vs что получилось

### Phase 2: Hypothesize (Гипотеза)

- Сформулируй возможные причины на основе наблюдений
- Определи наиболее вероятную гипотезу
- Исключи невозможные варианты

### Phase 3: Test (Проверка)

- Напиши тест или проверку, которая подтвердит/опровергнет гипотезу
- Проверь одну переменную за раз
- Зафиксируй результат

### Phase 4: Fix (Исправление)

- Внеси минимальное изменение, решающее проблему
- Убедись, что тесты проходят
- Проверь, что не сломалось ничего другого

Повторяй цикл если баг не решён с первого раза.

## Security Notes

- No network is required by this skill itself.
- The skill should avoid creating extra files beyond the project folder and `plan.md` unless requested.
- Keep model routing local to the skill so the broader OpenClaw environment stays stable.
