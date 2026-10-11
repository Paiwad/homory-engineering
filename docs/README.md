# Homory Engineering Docs

- `ProjectPaths.md` — source roots and workspace boundaries.
- `architecture/` — module boundaries and dependency direction.
- `guides/` — engineering quality guardrails.
- `roadmaps/` — platform migration plans.
- `features/` — product specifications, organized by feature ID.
- `checkpoints/` — implementation progress and verified results.
- `workflows/` — collaboration modes for source work.
- `templates/` — reusable task, decision, and checkpoint formats.
- `active/CURRENT.md` — current task pointer and selected mode.

## Collaboration modes

- `guided` — default. User writes code; AI gives bounded steps, reviews, and explains engineering decisions.
- `cowork` — AI and user decide at explicit decision gates; implementation may be shared.
- `vibe` — AI implements autonomously within confirmed scope and reports at checkpoints.

Set a mode with `[guided]`, `[cowork]`, or `[vibe]` in the task. The selected workflow defines its operating rules.

## Reading order before source changes

1. `ProjectPaths.md`
2. Relevant file in `architecture/` when changing module boundaries, data flow, DI, or UI architecture.
3. Relevant feature specification in `features/`.
4. `guides/Senior Android Handbook.md` as a risk-proportionate design guardrail.
5. Selected document in `workflows/`.
