---
description: Delegate plan review to an external model worker (Gemini, Codex, Cursor, or Claude) via tmux
---

Delegate plan review to an external worker in a separate tmux window (never the current pane).

## Parse Arguments

`$ARGUMENTS` may contain a path to the plan file and/or a model preference.

If no plan path is given, find the most recent plan:
1. Check `.omc/plans/` for the newest file
2. Check for a GitHub tracking issue (`gh issue list --label "tracking" --state open --limit 1`)
3. If neither exists, ask the user for the path

## Choose Worker

Present numbered options:

1. **`gemini`** — Gemini CLI
2. **`codex`** — Codex CLI
3. **`cursor`** — Cursor agent CLI (default model: codex-5.3-high)
4. **`claude`** — Claude CLI (separate session)

If the user already specified a model in the arguments (e.g. `/peer-plan-review gemini`), skip the prompt.

## Execute

Spawn in a new tmux window:

Generate a short unique suffix: `SUFFIX=$(date +%s | tail -c 5)`.

The window name is `plan-review-<worker>-<suffix>` (e.g. `plan-review-gemini-7321`).

```bash
tmux new-window -d -n "plan-review-<worker>-<suffix>" "<cli> 'Review the plan at <plan-path>. Evaluate it as a senior engineer would:

1. Are the requirements clear and complete?
2. Are there missing edge cases or failure modes?
3. Is the approach the simplest that could work?
4. Are there better alternatives not considered?
5. Are the acceptance criteria testable and sufficient?
6. Any risks or dependencies not accounted for?

Output your review as a structured markdown summary with sections: Strengths, Concerns, Suggestions, and a Verdict (approve / request-changes).'"
```

Where `<cli>` is `gemini`, `codex`, `claude`, or `agent -p --trust --force --model codex-5.3-high` for cursor.

Tell the user: "Plan review spawned in tmux window `plan-review-<worker>-<suffix>`. I'll poll for completion."

Poll the tmux window every 15s (`tmux capture-pane -t plan-review-<worker>-<suffix> -p | tail -5`). When the process exits or output contains `## Verdict`, capture the full output, present it to the user, and return to Checkpoint 1 (Step 3) options.
