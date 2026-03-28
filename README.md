# Clean Work Cycle

A Claude Code plugin for structured plan-to-ship development workflows.

## What's in it

### Skills (auto-invoked by Claude)

- **clean-work-cycle** — Full dev cycle with explicit checkpoints: triage, plan, execute (in isolated worktree), test, commit, PR, and merge. Adapts to task size.
- **pr-review-fix** — Review an open PR, fix issues directly in the worktree, push fixes, and comment with a structured summary.

### Commands (user-invoked)

| Command | Description |
|---------|-------------|
| `/clean-work-cycle:plan-execute-review-renew` | Invoke the clean-work-cycle skill |
| `/clean-work-cycle:review-pr` | Invoke pr-review-fix on a PR |
| `/clean-work-cycle:peer-pr-review` | Delegate PR review to an external model (Gemini, Codex, Cursor, or Claude) |

## Install

Add the marketplace and install the plugin from within Claude Code:

```
/plugin marketplace add ANaka/clean-work-cycle
/plugin install clean-work-cycle@clean-work-cycle
```

Or browse available plugins interactively with `/plugin` after adding the marketplace.

To pick up changes without restarting, run `/reload-plugins`.

## Dependencies

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- `gh` CLI (for PR operations)
- `git` with worktree support
- `tmux` (for `/peer-pr-review` worker spawning)
- [oh-my-claudecode](https://github.com/anthropics/oh-my-claudecode) (optional — for `/omc-plan` and `/team` referenced in the skills)
