# PR Review Fix

The PR Review Fix workflow is for review-and-repair work on an open pull
request. The default posture is to fix actionable issues when a safe worktree is
available. Comment-only mode is explicit.

## Terms

`sync` means update the local base branch after remote state changes.

`mergesync` means merge the PR, delete or prune the remote branch when safe,
then sync the local base branch and decide whether to remove or keep the local
worktree.

## Inputs

- PR number or URL, or a branch from which the PR can be inferred
- repository and base branch
- worktree path when available
- user constraints such as "comment only" or "do not push"

## Required State Fetch

A reviewer should gather:

- PR title, body, base branch, head branch, and merge state
- review comments and inline comments
- check status and failing jobs
- commit list
- existing worktree for the PR branch

## Modes

| Mode | Use When | Outcome |
| --- | --- | --- |
| Fix | A safe PR worktree is available and the user did not forbid pushes. | Fix critical and important findings, verify, push, and comment. |
| Comment-only | No safe worktree exists or user asks not to push. | Post review findings without editing. |

## Review Lenses

- correctness and behavioral regressions
- security and data exposure
- test gaps
- performance risks
- project convention mismatches
- generated or unrelated files

## Severity

- Critical: broken logic, security issue, data loss risk. Must fix or block.
- Important: correctness issue, missed edge case, weak test. Fix by default.
- Minor: style or simplification. Fix only if trivial.
- Observation: useful context. Comment only.

## Completion Evidence

- original findings are fixed or intentionally deferred
- focused checks ran after fixes
- pushed commit is identified when pushing
- PR comment summarizes fixed and deferred findings
- final state tells the user whether the PR is ready, waiting, or blocked
