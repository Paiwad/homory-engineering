# Vibe Coding

## Purpose

Implement a confirmed, bounded task autonomously and quickly while preserving Homory architecture and verification standards.

## Start

1. Read `docs/active/CURRENT.md`, relevant specification, architecture, and source patterns.
2. Confirm goal, non-goal, invariant, failure modes, data ownership, and verification.
3. Set `Mode: vibe` in `CURRENT.md`.
4. Implement smallest safe vertical slice using existing patterns.

## Checkpoints

Stop and ask user before a schema/migration, public/domain contract, dependency-boundary, open-specification, security/privacy, or destructive decision.

Otherwise continue through implementation and testing. Record a checkpoint when a meaningful slice completes, verification is blocked, or work finishes.

## Finish

Create checkpoint with changed files, verified commands/results, and remaining limitations. Update `CURRENT.md` to `done`.
