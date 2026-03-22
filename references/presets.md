# coding Presets

Suggested routing presets for OpenClaw integrations.

These are conceptual presets for the skill contract. They do not imply a required script or CLI command.

## two-model-default

Recommended starting point with the user's current model set:

- `clarify` -> `Minimax`
- `brief` -> `Minimax`
- `plan` -> `Codex`
- `implement` -> `Codex`
- `review` -> `Minimax`
- `final_report` -> `Minimax`

Use this for most project work.

## single-model-codex

Fallback when one model must handle the full workflow:

- `clarify` -> `Codex`
- `brief` -> `Codex`
- `plan` -> `Codex`
- `implement` -> `Codex`
- `review` -> `Codex`
- `final_report` -> `Codex`

Use this when orchestration simplicity matters more than independent review.

## three-model-experimental

Use only when a third model has shown implementation gains:

- `clarify` -> `Minimax`
- `brief` -> `Minimax`
- `plan` -> `Codex`
- `implement` -> `Kimi`
- `review` -> `Minimax`
- `final_report` -> `Minimax`

Do not enable this preset by default.
Benchmark it first on real project tasks.

## Global safety rule

No preset should mutate OpenClaw's global default model or leak routing outside the active `/coding` session.
