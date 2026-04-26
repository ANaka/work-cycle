# Codex Adapter

This is a planned adapter, not an installable Codex plugin yet.

## Posture

Codex should get a native translation, not a literal copy of Claude commands.
Do not promise one-to-one `/work-cycle:*` slash command parity.

## Expected Mapping

| Library Concept | Codex Surface |
| --- | --- |
| visible workflow cursor | `update_plan` |
| worktree setup | explicit `git worktree` shell commands |
| planning gate | inline Work Cycle state plus optional plan mode when available |
| peer review | Codex subagents when authorized, or external review prompts |
| GitHub state | GitHub plugin skills and `gh` fallback |
| handoff | paste-ready prompt and local state summary |
| reusable behavior | Codex skills/plugins or shell aliases/functions |

## Non-Goals For This Slice

- no `.codex-plugin/plugin.json`
- no claim of installable Codex adapter parity
- no fake slash command support
- no dependency on OMC

## First Real Adapter Slice

Suggested future path:

```text
adapters/codex/work-cycle/
  .codex-plugin/plugin.json
  skills/work-cycle/SKILL.md
  skills/pr-review-fix/SKILL.md
  skills/peer-review/SKILL.md
```

Implementation order:

1. Translate `plan-do-review-renew` into `skills/work-cycle/SKILL.md`.
2. Translate `pr-review-fix` into `skills/pr-review-fix/SKILL.md`.
3. Validate each skill draft before adding plugin packaging:
   `python3 ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py <skill-dir>`.
4. Add `.codex-plugin/plugin.json` only after the skills validate locally.
5. Document install and update commands for the verified adapter.
6. Keep the library contract and Codex adapter differences explicit.

The first Codex adapter PR should include a manual smoke transcript showing the
skill trigger, visible `update_plan` cursor, worktree decision, and verification
before completion.
