---
name: deslop-review
description: Use when a PR needs an AI-slop cleanup pass (including the shorthand "deslop-pr"). Locates the PR's worktree, removes slop from the changed-file diff with a regression-safe deletion-first workflow, fixes obvious bugs found along the way, pushes, comments a summary, then asks whether to mergesync.
---

# Deslop Review

Strip the AI slop out of a PR's diff, push the cleanup, comment a summary, then ask the user whether to mergesync. Default behavior is clean-and-document, not report-and-wait.

This is the **simplification** sibling of `pr-review-fix`. That skill hunts correctness and security bugs; this one removes bloat — duplication, dead code, needless abstraction, boundary leaks, weak tests. It is **not** a bug hunt, but obvious correctness bugs encountered while cleaning get fixed too (and called out). Anything that needs real investigation is noted, not chased — that's `pr-review-fix`'s job.

## Terminology

Uses the same definitions as `plan-do-review-renew` and `pr-review-fix`:

**sync** — sync local repo `main` to remote `main` (`git checkout main && git pull`). If continuing in a worktree, also sync the worktree's local `main` reference.

**mergesync** — merge the PR (`gh pr merge --merge --delete-branch`), then sync. One atomic operation.

**deslop-pr** — shorthand for this workflow. Treat user phrases like `deslop-pr`, `deslop this PR`, `clean the slop from PR #N`, "remove AI slop and push", or "tidy up this PR" as invocations of this skill. Default mode is **clean** (edit + push), not comment-only.

**check** — fetch ALL PR state:

```bash
gh pr view <number>
gh api repos/{owner}/{repo}/pulls/{number}/reviews
gh api repos/{owner}/{repo}/pulls/{number}/comments
gh pr view <number> --json commits
gh pr checks <number>
```

## Step 1 — Locate PR and Worktree

Accept PR number, URL, or infer from the current branch. Then:

1. Run **check** to fetch full PR state.
2. Extract the worktree path from the PR body (plan-do-review-renew puts `Worktree: <path>` in PR descriptions).
3. If no worktree path in the body, check `git worktree list` for a worktree on the PR's head branch.
4. If no worktree is found, **create one from the PR head branch**:
   - Choose a location following the **Worktree Pre-Creation Safety** conventions from `plan-do-review-renew` Step 5 (prefer existing `.worktrees/` or `worktrees/`; verify the directory is git-ignored; otherwise create outside the repo at `~/.claude/worktrees/<project>/<branch-slug>/`).
   - Materialize the branch and worktree:
     ```bash
     gh pr checkout <number>          # creates/updates the local head branch (handles forks)
     git worktree add <path> <head-branch>
     ```
   - If the head branch lives on a fork you cannot push to (no maintainer-edit permission), fall back to **comment-only mode**: produce the deslop suggestions as a PR comment in Step 6 and stop — do not push.

Confirm before proceeding:

> Deslopping PR #`<number>`: `<title>`
> Worktree: `<path>` | Branch: `<branch>` | Base: `<base>` | Mode: **clean** (or **comment-only** if fork/no-push)

User can also explicitly request comment-only mode (e.g. "just suggest", "don't push").

## Step 2 — Scope the Pass

Bound the cleanup to the PR's changed files only. Never expand beyond the diff.

```bash
cd <worktree-path>
git diff <base-branch>...HEAD --name-only   # the deslop surface
git diff <base-branch>...HEAD                # the actual changes to clean
```

Read the changed files in full (not just the hunks) so cleanup respects surrounding context, conventions, and package-level `CLAUDE.md`.

## Step 3 — Lock Behavior First

Before editing anything, protect current behavior:

- Identify what must stay the same (public APIs, sync/async boundaries, error propagation, side effects).
- Run the narrowest regression tests covering the touched area and confirm they pass — this **locks behavior** before cleanup.
- If tests cannot be run first, record an explicit verification plan (what you'll check, how) before touching code.

```bash
cd <worktree-path>
# focused tests for the touched package/area, e.g.
uv run pytest <touched-tests> -x
```

## Step 4 — Classify the Slop

Inspect the diff and name the concrete smells before editing:

| Smell | Examples |
|-------|----------|
| **Duplication** | repeated logic, copy-paste branches, redundant helpers |
| **Dead code** | unused code/exports, unreachable branches, stale flags, debug leftovers |
| **Needless abstraction** | pass-through wrappers, speculative indirection, single-use helper layers |
| **Boundary violations** | hidden coupling, misplaced responsibilities, wrong-layer imports or side effects |
| **Missing tests** | behavior not locked, weak regression coverage, edge-case gaps |
| **Obvious bugs** | clear off-by-one, wrong variable, inverted condition spotted while cleaning — fix and flag (do **not** open a broader bug hunt) |

## Step 5 — Run One Smell-Focused Pass at a Time

Work safest-deletion-first. Re-run the Step 3 tests after each pass. Do not bundle unrelated refactors into one edit set. Prefer deletion over addition; reuse existing utilities before introducing new ones; add no new dependencies.

1. **Pass 1 — Dead code deletion**
2. **Pass 2 — Duplicate removal**
3. **Pass 3 — Naming and error-handling cleanup**
4. **Pass 4 — Test reinforcement** (add the narrowest tests for any behavior left unlocked)

Fix obvious bugs from Step 4 in the pass where you encounter them, keeping the fix minimal.

If any pass changes behavior unintentionally, back out that edit (or fix it) before continuing — keep diffs small and reversible.

## Step 6 — Quality Gates, Commit, Push, Comment

Run the gates, then commit and push:

```bash
cd <worktree-path>
# regression tests still green + relevant lint/typecheck for the touched area
git add <changed-files>
git commit -m "deslop: <concise summary>"
git push
```

If a gate fails, fix the issue or back out the risky cleanup instead of forcing it through.

Then post a single evidence-dense comment:

```bash
gh pr comment <number> --body-file /tmp/deslop-comment.md
```

Comment structure:

```markdown
## Deslop Summary

### Simplifications
- [Dead code] <what was removed> — <file>
- [Duplication] <what was consolidated> — <file>
- [Abstraction] <wrapper/indirection removed> — <file>

### Bugs Fixed
- <obvious bug> — fixed in <commit-sha-short>

### Behavior Lock / Verification
- <test command> — <result>

### Remaining Risks
- <anything deferred, or "none">
```

Use `--body-file` to avoid shell quoting issues with markdown. **Comment-only mode ends here** — report what you'd change and stop.

## Step 7 — Ask About Mergesync

After pushing the cleanup, present options:

1. **`mergesync`** — merge the PR now, then sync (one atomic operation)
2. **`done`** — leave the PR open, report what was done
3. **`another-pass`** — run Steps 2–6 again (e.g. if the first pass surfaced more slop)

State a one-line recommendation before the options (e.g. tests green and diff is clean → recommend `mergesync`). The user always decides.
