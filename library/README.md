# Agentic Skills Library

This folder is the portable reference model for Work Cycle workflows.

It captures workflow semantics without binding them to one runtime. The current
Claude plugin still lives at the repository root. Until a sync or generation
process exists, root `skills/` and `commands/` are the stable implementation;
the files here are the design contract adapters should follow.

## Boundaries

The library owns:

- workflow state names and transitions
- verification tier policy
- debugging and root-cause gates
- worktree isolation policy
- acceptance criteria ledger format
- PR review, check-loop, and merge/sync semantics
- handoff and continuation contracts

Adapters own:

- invocation syntax
- tool names
- planning and state surfaces
- subagent or peer-worker mechanics
- install and packaging shape
- artifact roots

## Workflows

- [Plan-Do-Review-Renew](workflows/plan-do-review-renew.md)
- [PR Review Fix](workflows/pr-review-fix.md)
- [Peer Review](workflows/peer-review.md)

## Concepts

- [Acceptance Ledger](concepts/acceptance-ledger.md)
- [Checkpoints](concepts/checkpoints.md)
- [Debugging Gate](concepts/debugging-gate.md)
- [Handoff](concepts/handoff.md)
- [Verification Tiers](concepts/verification-tiers.md)
- [Worktree Isolation](concepts/worktrees.md)

## Compatibility Rule

A workflow adapter may be richer than the library when its host supports richer
tools. It may not silently weaken these invariants:

- preserve user and unrelated worktree changes
- make assumptions explicit before implementation
- record acceptance criteria for contained or broad work
- run fresh verification before completion claims
- keep PR/check/merge state explicit
- offer a continuation or handoff decision before declaring the cycle closed
