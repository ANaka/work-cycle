# Checkpoints

Checkpoints are explicit decision points. They prevent an agent from silently
changing scope, shipping strategy, or continuation mode.

## Standard Checkpoints

| Checkpoint | Timing | Decision |
| --- | --- | --- |
| Plan | Before execution. | Approve, revise, seek review, or stop. |
| PR Strategy | After local verification and review. | PR, draft PR, local-only, merge/sync, or wait. |
| Continue | Before closing the cycle. | Done, continue in same worktree, create new worktree, cleanup, or handoff. |

## Autonomous Handling

If the user has given explicit implementation authority, the agent may proceed
through a checkpoint after stating the decision and preserving a way back. It
should still record the decision in the visible workflow cursor or final
handoff.

## Non-Negotiable Checkpoints

- destructive operations
- production writes
- secrets or credential changes
- migrations and data-loss risk
- merge/sync after uncertain checks
