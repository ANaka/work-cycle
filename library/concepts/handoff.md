# Handoff

A handoff gives a future session enough state to continue without guessing.

## Required Fields

- repo path
- branch
- worktree path
- base branch or commit
- PR and issue numbers when available
- current workflow state
- accepted assumptions
- changed files
- verification already run
- known blockers or risks
- next recommended command or decision

## Rules

- Mark unknowns as unknown.
- Do not claim tests passed unless fresh evidence exists.
- Include do-not-revert context for unrelated user changes.
- Keep the prompt paste-ready for the target agent.

## Trigger Conditions

- context is tight
- work spans sessions
- user asks for a prompt
- PR is open but work is waiting on checks or review
- implementation stops before merge/sync
