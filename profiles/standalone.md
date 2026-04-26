# Standalone Profile

Use this profile when no external workflow framework is assumed.

## Assumptions

- `git` is available.
- `gh` is optional and used only for GitHub PR/issue workflows.
- `tmux` is optional and used only for peer worker commands.
- The host agent can read files, run shell commands, and edit the workspace.

## Behavior

- Implement the Work Cycle state machine directly.
- Use plain markdown artifacts when durable specs or plans are needed.
- Use explicit shell commands for worktree creation and verification.
- Use natural-language handoff prompts for continuation.

## When To Use

- portable prompts
- agents without plugin systems
- repos where installing Superpowers or OMC is not desirable
- fallback behavior when an adapter-specific primitive is missing
