# Clean Work Cycle

Claude Code skills for structured plan-to-ship development workflows.

## Skills

### clean-work-cycle

Full development cycle with explicit user checkpoints: triage, plan, execute (in isolated worktree), test, commit, PR, and merge. Adapts to task size — small fixes skip formal planning but still go through worktree/test/checkpoint gates.

### pr-review-fix

Review an open PR, fix issues directly in the worktree, push fixes, and comment on the PR with a structured summary. Defaults to fix-and-document, not report-and-wait.

## Commands

| Command | Description |
|---------|-------------|
| `/plan-execute-review-renew` | Invoke the clean-work-cycle skill |
| `/review-pr` | Invoke pr-review-fix on a PR |
| `/peer-pr-review` | Delegate PR review to an external model (Gemini, Codex, Cursor, or Claude) |

## Install

Symlink or copy into your Claude Code config:

```bash
# Skills
ln -s "$(pwd)/skills/clean-work-cycle" ~/.claude/skills/clean-work-cycle
ln -s "$(pwd)/skills/pr-review-fix" ~/.claude/skills/pr-review-fix

# Commands
ln -s "$(pwd)/commands/peer-pr-review.md" ~/.claude/commands/peer-pr-review.md
ln -s "$(pwd)/commands/plan-execute-review-renew.md" ~/.claude/commands/plan-execute-review-renew.md
ln -s "$(pwd)/commands/review-pr.md" ~/.claude/commands/review-pr.md
```

Or copy them directly:

```bash
cp -r skills/* ~/.claude/skills/
cp commands/* ~/.claude/commands/
```

## Dependencies

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- `gh` CLI (for PR operations)
- `git` with worktree support
- `tmux` (for peer-pr-review worker spawning)
- [oh-my-claudecode](https://github.com/anthropics/oh-my-claudecode) (optional, for `/omc-plan` and `/team` commands referenced in the skills)
