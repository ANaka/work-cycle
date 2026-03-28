---
name: clean-work-cycle
description: Use when the user gives any implementation task — bug fix, feature, refactor, or multi-phase project. Triages scope first — small tasks skip formal planning but still go through worktree, test, and checkpoint gates; larger tasks add planning and review phases.
---

# Clean Work Cycle

Plan-to-ship cycle with three explicit user checkpoints: plan review, PR strategy, and continuation.

## Terminology

**sync** — sync local repo `main` to remote `main` (`git checkout main && git pull` in the main worktree). If continuing in a worktree, also sync the worktree's local `main` reference (`git fetch origin && git merge origin/main`, or rebase if preferred).

**mergesync** — merge the PR (`gh pr merge --merge --delete-branch`), then sync. Always treated as one atomic operation. Note: `--delete-branch` deletes the remote branch; the local worktree and branch remain until explicitly cleaned up.

## Step 1 — Triage

Quick scope assessment (~30 seconds of exploration):

- Which packages/subdirs are touched?
- Single file, single package, or cross-package?
- Any CLI entry points, arguments, or data paths changing?

Classify and tell the user:

| Class | Criteria | Plan location |
|-------|----------|---------------|
| **Small** | 1–3 files, single package, no API changes | Skip Step 2 only — all other steps (worktree, test, checkpoints) still apply |
| **Single-session** | Contained feature/fix within one package | `.omc/plans/` |
| **Multi-session** | Cross-package, multi-phase, or architectural | `docs/plans/<date>-<name>.md` + `.omc/plans/` |

Present classification with reasoning. User can override.

## Step 2 — Plan

**Small tasks:** skip this step. Go to Step 3 and describe intended approach inline. Do NOT skip Steps 3–10 — worktree, test, commit, and checkpoints still apply even for small tasks.

**Single-session / multi-session:** run `/omc-plan --interactive`. For multi-session or high-complexity (critical paths, architectural changes), ask whether to escalate to `--consensus`.

**Multi-session only — produce both:**
1. **Roadmap** → `docs/plans/<YYYY-MM-DD>-<name>.md` — full scope, phases, dependencies, acceptance criteria per phase
2. **Session plan** → `.omc/plans/` — current phase only

## Step 3 — Checkpoint 1: Plan Review

Present the plan (or for small tasks, intended approach). User options:

1. **Approve** → Step 4
2. **Request changes** → revise plan, re-present
3. **Request further review** → solicit critique, re-present

## Step 4 — Execute

Signpost before proceeding:

> Creating isolated worktree on branch `<branch-name>`. Your main worktree stays untouched. All changes happen there.

Always use an isolated worktree unless the user explicitly says otherwise (e.g. "just do it here", "no worktree").

Then: `EnterWorktree` → execute (direct for small tasks, `/team` for single/multi-session).

## Step 5 — Test

Run the relevant test suite:

```bash
# Single package
cd <package> && uv run pytest tests/ -x

# Cross-package: run for each affected package
```

If tests fail: diagnose, fix, re-run. Do not proceed with failing tests.

## Step 6 — Doc Check

Check before shipping — update in the same branch if needed:

- CLI entry points, arguments, or flags changed? → update root `CLAUDE.md` quick reference table
- Pipeline invocations changed? → update package `CLAUDE.md`
- New vocabulary, reference proteins, or data paths? → update vocabulary translations
- New scripts or tools? → add to the appropriate table

Tell the user what you updated (or "no doc changes needed").

## Step 7 — Commit

Stage and commit all changes from execution, test fixes, and doc updates. All work should be committed before proceeding to PR strategy.

```bash
git add <changed-files>
git commit -m "<conventional commit message>"
```

If there are multiple logical changes, use multiple commits. The branch should be clean (no uncommitted changes) before Step 8.

## Step 8 — Checkpoint 2: PR Strategy

> Branch: `<branch-name>` | Worktree: `<path>` | Changes committed, ready to push.

Present options:

1. **`open-pr`** — push branch, open PR, enter check loop (Step 9) (default)
2. **`open-pr-review`** — push + PR + `/review-pr`, Claude reviews and fixes, then check loop (Step 9). Note: when invoked from here, `/review-pr` should end with `done` (not `mergesync`) — mergesync is handled by Step 9.
3. **`open-pr-peer-review`** — push + PR + `/peer-pr-review`, external model reviews (ask which: 1. gemini, 2. codex, 3. cursor, 4. claude), then await completion (see below) and enter check loop (Step 9)
4. **`push-mergesync`** — push + PR + mergesync immediately (no review, no check loop); then Step 10

**PR body must include the worktree path** so reviewers can check out directly:

> Worktree: `<absolute-worktree-path>`

**Awaiting peer review (option 3):** after spawning the worker, poll PR comments every 30s for a `## Review Summary` comment. When found (or worker signals done via `SendMessage`), run a **check** and present Step 9 options.

## Step 9 — PR Open: Check Loop

**"Check"** = fetch ALL of:

```bash
gh pr view <number>
gh api repos/{owner}/{repo}/pulls/{number}/reviews
gh api repos/{owner}/{repo}/pulls/{number}/comments
gh pr view <number> --json commits
gh pr checks <number>
```

Present options:

1. **`check`** — fetch PR state, report, re-present options
2. **`check-fix`** — fetch, fix in worktree, re-test, commit, push, re-present options
3. **`check-fix-mergesync`** — fetch, fix in worktree, re-test, commit, push, mergesync; then Step 10
4. **`mergesync`** — mergesync now (no check); then Step 10
5. **`abandon`** — close PR (`gh pr close <number>`), leave worktree intact; then Step 10

Repeat options 1 or 2 until user picks option 3, 4, or 5.

## Step 10 — Continuation

> Worktree: `<path>` | PR: #`<number>` (merged or closed)

Present options:

1. **Stop + cleanup** — `ExitWorktree`, sync, run `/clean_gone` to remove stale local branches. Done.
2. **Continue in worktree** — stay in the worktree for follow-up work. Push a new branch when ready (remote branch was deleted by mergesync; if abandoned, the remote branch still exists — reuse or rename). Sync main in background.
3. **New worktree** — `ExitWorktree`, sync, cleanup current worktree, create fresh one for next piece of work.

**Multi-session only — before stopping:**
1. Update roadmap in `docs/plans/` — mark phase complete, note scope changes
2. Write `## Session Notes` in the roadmap: decisions made, approaches tried and rejected, surprising discoveries
3. Generate next session plan from roadmap into `.omc/plans/`
4. Present next session plan → loop to Step 3
