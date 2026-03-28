---
description: Delegate PR review to an external model worker (Gemini, Codex, Cursor, or Claude) via omc-teams or Cursor agent CLI
---

Delegate the `pr-review-fix` skill to an external worker. The worker runs in a **separate tmux window** — never in the current pane.

## Parse Arguments

`$ARGUMENTS` should contain a PR number/URL. It may also contain:

- A **worker** preference: `gemini`, `codex`, `cursor`, or `claude`
- A **model** override (for cursor): e.g. `--model sonnet-4-thinking`, `--model gpt-5`

Examples:
- `/peer-pr-review 1657 cursor` — use cursor with its default model
- `/peer-pr-review 1657 cursor --model gpt-5` — use cursor with gpt-5
- `/peer-pr-review 1657 gemini` — use gemini worker

## Choose Worker

Present numbered options:

1. **`gemini`** — spawn Gemini worker
2. **`codex`** — spawn Codex worker
3. **`cursor`** — spawn Cursor agent CLI
4. **`claude`** — spawn Claude worker (separate session)

If the user already specified a worker in the arguments, skip the prompt.

If cursor is selected and no `--model` was provided, ask which model to use:

1. **`sonnet-4`** (default)
2. **`sonnet-4-thinking`**
3. **`gpt-5`**
4. **`o3`**
5. **custom** — enter a model name (anything `agent --list-models` supports)

If the user passed `--model <name>` in arguments, skip the model prompt too.

## Execute

1. Read the skill file at `~/.claude/skills/pr-review-fix/SKILL.md`
2. Locate the worktree: check `git worktree list` for the PR's head branch
3. Fetch the PR diff: `gh pr diff <number> > /tmp/pr_<number>_diff.txt`
4. Write the full prompt (skill instructions + diff + PR number + worktree path) to `/tmp/pr_<number>_review_prompt.txt`
5. Spawn the worker:

**CRITICAL: use `tmux new-window -d` (the `-d` flag) so the new window does NOT steal focus from the user's current pane.**

**Primary method — `omc team` with `run_in_background: true`:**
```bash
omc team 1:<model> "$(cat /tmp/pr_<number>_review_prompt.txt)"
```
Run this Bash call with `run_in_background: true` so it doesn't block the current session.

**For cursor:**
```bash
tmux new-window -d -n "review-<number>" "cd <worktree-path> && agent -p --trust --force --model <model> --workspace <worktree-path> \"$(cat /tmp/pr_<number>_review_prompt.txt)\" | tee /tmp/pr_<number>_review_output.txt; echo 'Review complete. Press enter to close.'; read"
```
If no model was selected, omit the `--model <model>` flag entirely (cursor uses its default).

**Fallback — direct tmux for any model:**
```bash
tmux new-window -d -n "review-<number>" "cd <worktree-path> && <model-cli> -p < /tmp/pr_<number>_review_prompt.txt | tee /tmp/pr_<number>_review_output.txt; echo 'Review complete. Press enter to close.'; read"
```

6. Tell the user: "Review spawned in tmux window `review-<number>` using `<model>`. I'll poll PR #<number> for a `## Review Summary` comment."
7. Poll PR comments every 30s for the review comment. When found, report back.
