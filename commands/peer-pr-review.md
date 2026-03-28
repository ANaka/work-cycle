---
description: Delegate PR review to an external model worker (Gemini, Codex, Cursor, or Claude) via omc-teams or Cursor agent CLI
---

Delegate the `pr-review-fix` skill to an external worker. The worker runs in a **separate tmux window** — never in the current pane.

## Parse Arguments

`$ARGUMENTS` should contain a PR number/URL. It may also contain a model preference.

## Choose Worker

Present numbered options:

1. **`gemini`** — spawn Gemini worker
2. **`codex`** — spawn Codex worker
3. **`cursor`** — spawn Cursor agent CLI
4. **`claude`** — spawn Claude worker (separate session)

If the user already specified a model in the arguments (e.g. `/peer-pr-review 1657 gemini`), skip the prompt and use that model.

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
tmux new-window -d -n "review-<number>" "cd <worktree-path> && agent -p --trust --force --workspace <worktree-path> \"$(cat /tmp/pr_<number>_review_prompt.txt)\" | tee /tmp/pr_<number>_review_output.txt; echo 'Review complete. Press enter to close.'; read"
```

**Fallback — direct tmux for any model:**
```bash
tmux new-window -d -n "review-<number>" "cd <worktree-path> && <model-cli> -p < /tmp/pr_<number>_review_prompt.txt | tee /tmp/pr_<number>_review_output.txt; echo 'Review complete. Press enter to close.'; read"
```

6. Tell the user: "Review spawned in tmux window `review-<number>`. I'll poll PR #<number> for a `## Review Summary` comment."
7. Poll PR comments every 30s for the review comment. When found, report back.
