# Claude Code Adapter

The Claude Code adapter is the current stable implementation.

## Live Paths

The live Claude plugin remains at the repository root:

- `.claude-plugin/`
- `skills/`
- `commands/`

Do not move these paths in a compatibility-preserving change. The marketplace
manifest points at the repository root, so root discovery is part of the
published contract.

## Current Dependencies

- Claude Code plugin system
- `git`
- `gh`
- `tmux` for peer worker commands
- oh-my-claudecode surfaces referenced by the current skills
- Superpowers skills where the current workflow invokes them

## Mapping

| Library Concept | Claude Adapter Surface |
| --- | --- |
| visible workflow cursor | OMC notepad/state primitives in current skills |
| plan quality gate | `/omc-plan` plus inline gate language |
| peer review | root peer-review commands with tmux workers |
| worktree isolation | shell/git instructions inside the skill |
| final verification | skill text and command execution |
| continuation | final checkpoint and handoff prompt |

## Migration Notes

Future work may make the Claude adapter a generated or copied output from the
portable library. Until then, update root `skills/` and `commands/` directly
when changing actual Claude behavior, and update `library/` when changing the
portable contract.
