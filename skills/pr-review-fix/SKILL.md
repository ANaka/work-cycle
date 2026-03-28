---
name: pr-review-fix
description: Use when a PR is open and needs review. Fetches PR state, reviews the diff, fixes issues directly in the worktree, pushes, and comments on the PR about what was fixed. Can loop until clean.
---

# PR Review Fix

Review an open PR, fix issues directly, push, and comment. Default behavior is fix-and-document, not report-and-wait.

## Terminology

Uses the same definitions as `plan-do-review-renew`:

**sync** — sync local repo `main` to remote `main` (`git checkout main && git pull`). If continuing in a worktree, also sync the worktree's local `main` reference.

**mergesync** — merge the PR (`gh pr merge --merge --delete-branch`), then sync. One atomic operation.

**check** — fetch ALL PR state:

```bash
gh pr view <number>
gh api repos/{owner}/{repo}/pulls/{number}/reviews
gh api repos/{owner}/{repo}/pulls/{number}/comments
gh pr view <number> --json commits
gh pr checks <number>
```

## Step 1 — Locate PR and Worktree

Accept PR number, URL, or infer from current branch. Then:

1. Run **check** to fetch full PR state
2. Extract the worktree path from the PR body (plan-do-review-renew puts `Worktree: <path>` in PR descriptions)
3. If no worktree path in PR body, check `git worktree list` for a worktree on the PR's head branch
4. If no worktree found, fall back to **comment-only mode** (skip Step 3, go straight to Step 4 after review)

Confirm before proceeding:

> Reviewing PR #`<number>`: `<title>`
> Worktree: `<path>` | Branch: `<branch>` | Mode: **fix** (or **comment-only** if no worktree)

User can also explicitly request comment-only mode (e.g. "just review", "comment only", "don't fix").

## Step 2 — Review

Read the full diff against the PR's base branch:

**Fix mode:**
```bash
cd <worktree-path>
git diff <base-branch>...HEAD
```

**Comment-only mode (no worktree):**
```bash
gh pr diff <number>
```

Review for:

- Logic errors and bugs
- Security issues (injection, auth, data exposure)
- Test gaps — new code paths without test coverage
- Project conventions (check package-level `CLAUDE.md` for patterns)
- Performance issues (unnecessary allocations, N+1 queries, missing indexes)

Classify each finding by severity:

| Severity | Meaning | Action |
|----------|---------|--------|
| **Critical** | Broken logic, security vulnerability, data loss risk | Must fix |
| **Important** | Correctness issue, missing edge case, test gap | Fix by default |
| **Minor** | Style, naming, minor simplification | Fix if trivial, otherwise note |
| **Observation** | Not wrong, but worth knowing | Comment only, do not fix |

If in **comment-only mode**, skip to Step 4. After posting the comment, the review is done — do not proceed to Step 5.

Otherwise, proceed to Step 3 (fix is the default when a worktree is available).

## Step 3 — Fix

For each Critical and Important finding (and trivial Minor findings):

1. Make the fix in the worktree
2. Run relevant tests to confirm the fix doesn't break anything
3. Track what was fixed (for the PR comment in Step 4)

If a fix is non-trivial or ambiguous, ask the user before applying it.

After all fixes:

```bash
cd <worktree-path>
git add <fixed-files>
git commit -m "review: fix <concise summary>"
git push
```

## Step 4 — Comment on PR

Post a single review comment summarizing all findings and fixes:

```bash
gh pr comment <number> --body-file /tmp/review-comment.md
```

Comment structure:

```markdown
## Review Summary

### Fixed
- [Critical] <description> — fixed in <commit-sha-short>
- [Important] <description> — fixed in <commit-sha-short>
- [Minor] <description> — fixed in <commit-sha-short>

### Not Fixed (needs discussion)
- [Minor] <description> — <reason skipped>

### Observations
- <observation>
```

Use `--body-file` to avoid shell quoting issues with markdown.

**Comment-only mode ends here.** Do not proceed to Step 5.

## Step 5 — Re-review (fix mode only)

After pushing fixes, re-read the diff to check:

- Did any fix introduce a new issue?
- Are there remaining findings that were deferred?

If clean (no new issues, no deferred Critical/Important findings), present options:

1. **`mergesync`** — merge the PR now, sync (only offer when invoked standalone, not from within plan-do-review-renew)
2. **`done`** — leave PR open, report what was done
3. **`another-pass`** — run Steps 2–5 again (e.g. if fixes were complex)

If new issues found, loop back to Step 3 automatically (max 3 cycles, then present findings to user).
