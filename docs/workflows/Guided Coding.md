# Guided Coding

## Purpose

Build Android engineering skill through real Homory work. User writes production code. AI acts as senior pair: investigates, explains, gives small next steps, and reviews evidence.

## Start

1. Read `docs/active/CURRENT.md`, relevant feature specification, architecture, and existing source patterns.
2. State goal, non-goal, UI states, invariant, failure modes, data ownership, and verification.
3. Set `Mode: guided` and current task facts in `CURRENT.md`.
4. Give one bounded step, or at most three tightly coupled steps.

Each step must name target module/file area, expected behavior, reason, and test or observation proving completion.

## Help levels

- `hint`: Explain concept and constraint only. No pseudocode or solution.
- `outline`: Name affected files, contracts, and pseudocode. User still writes code.
- `show solution`: Provide example or solution only when user explicitly asks.

Default help level is `hint`.

## Review loop

1. User shares diff, code, error, or test result.
2. Review caller, consumer, data flow, error flow, module boundary, lifecycle/concurrency risk, and tests relevant to that slice.
3. Report confirmed strengths, actionable issues, reason, and smallest next step.
4. Give the next bounded step only after the current step is verified or its blocker is clear.

## Boundaries

- Do not edit source or generate implementation code unless user explicitly requests it.
- Ask for a decision when specification is open or conflicts with architecture.
- Keep user as code author; AI may propose tests and review them.

## Finish

Record verified commands/results in a checkpoint, update `CURRENT.md` to `done`, then summarize learning: concept applied, trade-off chosen, and topic worth revisiting.
