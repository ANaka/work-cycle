# Clean Work Cycle

A Claude Code plugin for structured **Plan-Do-Review-Renew** development workflows.

Every task follows the same cycle — triage, plan, execute in isolation, test, review, ship, and decide what's next. Explicit checkpoints keep you in control.

```mermaid
graph LR
    A[Triage] --> B[Plan]
    B --> C{Checkpoint 1\nApprove plan}
    C --> D[Execute\nin worktree]
    D --> E[Test]
    E --> F[Commit]
    F --> G{Checkpoint 2\nPR strategy}
    G --> H[Review &\nFix loop]
    H --> I{Checkpoint 3\nContinue?}
    I -->|New task| A
    I -->|Continue| D
    I -->|Done| J[Cleanup]
```

## What's in it

### Skills (auto-invoked by Claude)

- **clean-work-cycle** — Full Plan-Do-Review-Renew cycle with explicit checkpoints: triage, plan, execute (in isolated worktree), test, commit, PR, and merge. Adapts to task size — small tasks skip formal planning but still go through every gate.
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
