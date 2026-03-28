# Work Cycle

A Claude Code plugin for structured **Plan-Do-Review-Renew** development workflows.

Every task follows the same cycle — triage, plan, execute in isolation, test, review, ship, and decide what's next. Explicit checkpoints keep you in control.

## Install

Add the marketplace and install the plugin from within Claude Code:

```
/plugin marketplace add ANaka/work-cycle
/plugin install work-cycle@work-cycle
```

Or browse available plugins interactively with `/plugin` after adding the marketplace.

To pick up changes without restarting, run `/reload-plugins`.

## What's in it

### Skills (auto-invoked by Claude)

- **plan-do-review-renew** — Full cycle with explicit checkpoints: triage, plan (via `/omc-plan`), execute (in isolated worktree), test, commit, PR, and merge. Triage determines planning depth — small tasks use `--direct`, larger tasks get a thorough interview (`--interactive`), and complex/multi-session work adds consensus review (`--consensus`).
- **pr-review-fix** — Review an open PR, fix issues directly in the worktree, push fixes, and comment with a structured summary.

### Commands (user-invoked)

| Command | Description |
|---------|-------------|
| `/work-cycle:plan-execute-review-renew` | Invoke the plan-do-review-renew skill |
| `/work-cycle:review-pr` | Invoke pr-review-fix on a PR |
| `/work-cycle:peer-plan-review` | Delegate plan review to an external model (Gemini, Codex, Cursor, or Claude) |
| `/work-cycle:peer-pr-review` | Delegate PR review to an external model (Gemini, Codex, Cursor, or Claude) |

### The Cycle

```mermaid
graph TD
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

## Dependencies

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- `gh` CLI (for PR operations)
- `git` with worktree support
- `tmux` (for `/peer-pr-review` and `/peer-plan-review` worker spawning)
- [oh-my-claudecode](https://github.com/anthropics/oh-my-claudecode) (for `/omc-plan` and `/team` referenced in the skills)
