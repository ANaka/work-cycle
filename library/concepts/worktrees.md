# Worktree Isolation

Implementation work should happen in an isolated worktree unless the user opts
out or the task is a trivial read-only/one-line operation.

## Goals

- protect the user's main checkout
- keep branch, base, setup, PR, and verification state together
- reduce accidental staging of unrelated files
- allow cleanup or continuation decisions at the end of a cycle

## Selection Order

1. Reuse an existing safe worktree for the same branch or PR.
2. Use a project-local `.worktrees/` or `worktrees/` directory only when it is
   ignored.
3. Otherwise use an external agent worktree path.

## Required Checks

- repo root
- current branch
- dirty status
- base ref
- destination ignore safety
- existing worktrees
- setup requirements
- baseline verification when feasible

## Cleanup Decision

Before declaring the cycle closed, choose one:

- remove the worktree
- keep it for follow-up
- create a new worktree for the next task
- leave it intentionally with a handoff
