# Cursor Adapter

**Status: Planned. No live Cursor rule files exist yet.**

This is a planned adapter, not an installed Cursor rule set yet.

## Posture

Cursor rules are context injection and guidance. They are not the same as Claude
plugin commands or Codex skills. The Cursor adapter should provide rules and
paste-ready prompts, not pretend to expose the exact same command runtime.

## Expected Mapping

| Library Concept | Cursor Surface |
| --- | --- |
| workflow policy | `.cursor/rules/*.mdc` project rules |
| invocation | Agent Requested or Manual rules |
| planning gate | prompt/rule instructions |
| worktree policy | shell instructions in rule text |
| peer review | separate agent chat or manual prompt handoff |
| handoff | paste-ready prompts |

## Non-Goals For This Slice

- no live `.cursor/` project rules
- no generated `.mdc` files
- no claim that Cursor can run Claude plugin commands
- no dependency on OMC

## Example Future Rule Shape

Suggested future path:

```text
adapters/cursor/rules/
  work-cycle.mdc
  pr-review-fix.mdc
  peer-review.mdc
adapters/cursor/prompts/
  work-cycle.md
  pr-review-fix.md
```

Cursor rule files should be copied or generated into a consuming repo's
`.cursor/rules/` directory only after their trigger behavior has been checked in
Cursor.

```text
description: Use Work Cycle for implementation tasks
alwaysApply: false

Follow the portable Work Cycle state machine. Keep a visible state cursor, use
isolated worktrees for implementation work, record acceptance criteria, verify
before completion claims, and close with a continuation or handoff decision.
```

The first Cursor adapter PR should include a manual smoke transcript showing the
rule attachment mode, the prompt used, and whether the agent followed the
worktree, acceptance ledger, verification, and continuation rules.
