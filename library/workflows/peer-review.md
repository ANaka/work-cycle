# Peer Review

Peer review is a bounded second-opinion workflow. It is most useful for plans,
large diffs, risky changes, and merge readiness.

## Principles

- Split review by concern, not by vague seniority.
- Give each reviewer concrete artifacts and a narrow lens.
- Treat feedback as evidence, not instructions.
- Fix valid critical and important findings.
- Record rejected feedback with a short rationale.

## Common Lanes

- architecture and migration risk
- correctness and regressions
- tests and verification
- security and data exposure
- docs and user-facing behavior
- performance and operational risk

## Prompt Contract

A useful peer review prompt names:

- artifact under review
- repository and relevant branch or PR
- review lens
- constraints such as read-only or no pushes
- output format and verdict options

## Verdicts

- approve
- approve with nits
- request changes
- needs human decision

Adapters may implement peer review with native subagents, tmux workers, separate
CLI sessions, or manual prompt handoff. The library only requires bounded scope
and explicit synthesis.
