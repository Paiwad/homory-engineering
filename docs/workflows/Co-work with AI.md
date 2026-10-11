# Co-work with AI

## Purpose

Solve work jointly where requirement, architecture, migration, or risk needs explicit shared decisions.

## Start

1. Read `docs/active/CURRENT.md`, relevant specification, architecture, and source patterns.
2. State goal, non-goal, UI states, invariant, failure modes, data ownership, and verification.
3. Set `Mode: cowork` in `CURRENT.md`.
4. Present evidence-backed options whenever a product or technical choice materially changes scope, data, or dependency direction.

## Decision gates

Pause for user direction before:

- schema or migration design;
- public/domain contract change;
- dependency or module-boundary change;
- behavior not fixed by specification;
- destructive, privacy, security, or release-impacting action.

Record accepted durable decisions in `docs/decisions/` using `docs/templates/Decision Record.md`.

## Implementation loop

1. Agree a vertical slice and test plan.
2. Implement ownership split agreed by user.
3. Verify relevant tests/build/lint.
4. Report evidence, residual risk, and next decision or slice.

## Finish

Create checkpoint with actual verification results and update `CURRENT.md` to `done`.
