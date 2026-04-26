# Superpowers Integrated Profile

Use this profile when Superpowers is installed and the host agent can invoke its
skills.

## Dependency Posture

Superpowers is safe to lean on for generic software-engineering discipline when
available. It should not replace the Work Cycle state machine.

Work Cycle decides the current state and transition. Superpowers can execute
phase-level practices such as brainstorming, writing plans, worktrees, TDD,
debugging, verification, review, and branch finishing.

## Useful Superpowers Skills

- `brainstorming`
- `writing-plans`
- `using-git-worktrees`
- `test-driven-development`
- `systematic-debugging`
- `verification-before-completion`
- `requesting-code-review`
- `receiving-code-review`
- `finishing-a-development-branch`
- `dispatching-parallel-agents`

## Guardrails

- Do not let a Superpowers hard gate erase an already-approved Work Cycle
  checkpoint.
- Do not duplicate artifacts unless the user asked for a durable spec.
- Preserve Work Cycle terminology for `fixpr`, `mergesync`, handoff, and
  continuation.
- Keep adapter-specific invocation details out of the portable library.
