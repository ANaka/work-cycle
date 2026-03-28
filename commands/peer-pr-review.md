---
description: Delegate PR review to an external model worker (Gemini, Codex, Cursor, or Claude) via tmux
---

Delegate PR review to an external worker in a separate tmux window (never the current pane).

## Parse Arguments

`$ARGUMENTS` should contain a PR number/URL. It may also contain a model preference.

## Choose Worker

Present numbered options:

1. **`gemini`** — Gemini CLI
2. **`codex`** — Codex CLI
3. **`cursor`** — Cursor agent CLI (default model: codex-5.3-high)
4. **`claude`** — Claude CLI (separate session)

If the user already specified a model in the arguments (e.g. `/peer-pr-review 1657 gemini`), skip the prompt.

## Execute

Locate the worktree (`git worktree list | grep <PR branch>`) and spawn:

```bash
tmux new-window -d -n "review-<number>" "cd <worktree-path> && <cli> 'Review PR #<number>. Read ${CLAUDE_PLUGIN_ROOT}/skills/pr-review-fix/SKILL.md and follow it exactly.'"
```

Where `<cli>` is `gemini`, `codex`, `claude`, or `agent -p --trust --force --model codex-5.3-high --workspace <worktree-path>` for cursor.

Tell the user: "Review spawned in tmux window `review-<number>`. I'll poll for the `## Review Summary` comment on PR #<number>."

Poll PR comments every 30s. When found, report back.
