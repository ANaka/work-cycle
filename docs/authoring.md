# Authoring Guide

Use this guide when adding or changing Work Cycle skills, workflows, profiles,
or adapters.

## Change Types

| Change | Update |
| --- | --- |
| portable workflow semantics | `library/` |
| runtime dependency posture | `profiles/` |
| Claude behavior | root `skills/`, root `commands/`, and `adapters/claude-code/` |
| Codex translation | `adapters/codex/` |
| Cursor translation | `adapters/cursor/` |
| migration or compatibility notes | `docs/migration.md` |

## Rules

- Keep root Claude plugin paths stable unless making a major compatibility
  change.
- For any concrete implementation, name both the adapter and profile being used.
- Do not call Codex or Cursor adapters installable until their files exist and
  have been validated.
- Treat adapter READMEs as snapshots unless they explicitly say they are
  generated from the portable library.
- Keep Claude-only tool names out of portable library docs.
- Add third-party attribution when copying substantial upstream text or code.
- Prefer small adjacent skills over one larger workflow when the task is
  independently useful.

## Adapter Checklist

- What host capability implements the visible state cursor?
- How does the adapter create or enter worktrees?
- How does it delegate peer review?
- How does it inspect PR checks and comments?
- Where do durable artifacts live?
- What is the fallback when a host capability is missing?
- What verification proves the adapter works?
