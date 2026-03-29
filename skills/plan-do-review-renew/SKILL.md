---
name: plan-do-review-renew
description: Use when the user gives any implementation task — bug fix, feature, refactor, or multi-phase project. Triages scope first, then brainstorms design before planning. Even small tasks get an assumption check. Larger tasks add full design exploration, planning, and review phases.
---

# Clean Work Cycle

Brainstorm-plan-ship cycle with three explicit user checkpoints: plan review, PR strategy, and continuation.

## Terminology

**sync** — sync local repo `main` to remote `main` (`git checkout main && git pull` in the main worktree). If continuing in a worktree, also sync the worktree's local `main` reference (`git fetch origin && git merge origin/main`, or rebase if preferred).

**mergesync** — merge the PR (`gh pr merge --merge --delete-branch`), then sync. Always treated as one atomic operation. Note: `--delete-branch` deletes the remote branch; the local worktree and branch remain until explicitly cleaned up.

## Workflow Tracking

**Rule: update the workflow cursor before the first action of every new step.** This keeps your position visible even through deep execution phases and context compression.

Use `notepad_write_priority` to maintain a live checklist. Format — use `[x]` done, `[>]` active, `[ ]` pending, `[-]` skipped:

```
WCYCLE: <task-summary> (<triage-class>)
[x] 1-Triage  [x] 2-Brainstorm  [x] 3-Plan
[x] 4-CP:Plan [>] 5-Execute     [ ] 6-Test
[ ] 7-Docs    [ ] 8-Commit      [ ] 9-CP:PR
[ ] 10-Check  [ ] 11-Continue
br:<branch> wt:<worktree-path>
```

Add `pr:#<number>` and `issue:#<number>` to the last line as they become available. Mark skipped steps `[-]` (e.g. small tasks may skip the plan file). For loops, append `(Rev N)` to the active step.

**Multi-session only:** also use `state_write` at durable milestones (triage done, plan approved, worktree created, PR opened, PR merged, continuation chosen) to persist across sessions:

```
state_write mode:"ralph" state:{
  "workflow": "work_cycle",
  "step": 5, "step_name": "Execute",
  "triage_class": "multi-session",
  "branch": "<branch>",
  "worktree_path": "<path>",
  "tracking_issue": <number>,
  "pr_number": null,
  "updated_at": "<iso-timestamp>"
}
```

## Step 1 — Triage

Quick scope assessment (~30 seconds of exploration):

- Which packages/subdirs are touched?
- Single file, single package, or cross-package?
- Any CLI entry points, arguments, or data paths changing?
- How ambiguous is the request? Could reasonable engineers disagree on approach?

Classify and tell the user:

| Class | Criteria | Planning approach | Plan location |
|-------|----------|-------------------|---------------|
| **Small** | 1–3 files, single package, no API changes | `/omc-plan --direct` (omc-team claude) — skip interview, describe approach inline | Skip plan file — approach stated in Checkpoint 1 |
| **Single-session** | Contained feature/fix within one package | `/omc-plan --interactive` (omc-team claude) — thorough interview before planning | `.omc/plans/` |
| **Multi-session** | Cross-package, multi-phase, or architectural | `/omc-plan --interactive --consensus` (omc-team claude) — interview + planner/architect/critic loop | GitHub tracking issue + `.omc/plans/` |

Present classification with reasoning. User can override class or planning approach.

## Step 2 — Brainstorm

Examine assumptions and explore the design space *before* planning implementation. Depth scales with the triage class. Even simple tasks benefit — "simple" is where unexamined assumptions waste the most work.

### Small: Quick Assumption Check (~30 seconds)

State three things and present them to the user for confirmation or correction:

1. **What I think the task is** — one sentence restating the goal
2. **How I'd approach it** — the intended change (files, logic)
3. **What I'm assuming** — constraints, scope boundaries, things I'm *not* changing

> **Assumption check:**
> - Task: [restate]
> - Approach: [describe]
> - Assuming: [list]
>
> Does this match your intent, or should I adjust?

User confirms → proceed to Step 3. User corrects → update and re-present.

If the assumption check reveals ambiguity, multiple viable approaches, or more scope than expected, **re-triage upward** to Single-session or Multi-session and run the full design exploration instead.

### Single-session: Full Design Exploration

Explore the design space before committing to an approach:

1. **Clarifying questions** — ask one at a time. Gather codebase facts (read code, check patterns) *before* asking the user about them. Only ask the user for preferences, scope decisions, and constraints.
2. **Propose 2–3 approaches** — for each, describe the approach in 1–2 sentences, list pros and cons, and note trade-offs. Present one at a time, get the user's reaction before presenting the next.
3. **Recommend** — after discussing approaches, state your recommendation with rationale.
4. **Design brief** — produce an inline summary:
   - **Goal:** one sentence
   - **Chosen approach:** what and why
   - **Alternatives considered:** what was rejected and why
   - **Assumptions:** what we're taking as given
   - **Open questions:** anything unresolved (ideally none)
   - **Verification:** how we'll know it works — specific tests, checks, or expected outcomes

### Multi-session: Deep Design Exploration

Same as single-session, plus:

- Explore **architecture implications** — how the change fits into the broader system, interface boundaries, dependency impacts
- Consider **phase decomposition** — which parts can ship independently, what order reduces risk
- Identify **cross-cutting concerns** — shared types, migration paths, backwards compatibility

### Self-Review Checklist (all classes)

Before proceeding to planning, run this checklist inline:

- [ ] **Unexamined assumptions?** — anything taken for granted that could be wrong
- [ ] **Scope creep?** — anything beyond what was actually requested
- [ ] **Ambiguity?** — requirements that could reasonably be interpreted two ways
- [ ] **Contradictions?** — tension between stated goals and chosen approach
- [ ] **Missing constraints?** — performance, compatibility, security, or other concerns not yet addressed
- [ ] **Verification gap?** — is there a concrete way to confirm the change works (named tests, commands, expected output)?

Flag any issues found. Resolve with the user before proceeding.

## Step 3 — Plan

**IMPORTANT: Always use `/omc-plan` for planning. Never use Claude Code's built-in plan mode (`EnterPlanMode`).**

Pass the design brief from Step 2 as context to `/omc-plan` so the planner doesn't re-ask resolved questions.

Run `/omc-plan` with flags determined by triage:

- **Small:** `/omc-plan --direct` (omc-team claude) with the task description and assumption check. Go to Step 4 with the generated approach. Do NOT skip Steps 4–11 — worktree, test, commit, and checkpoints still apply.
- **Single-session:** `/omc-plan --interactive` (omc-team claude) with the design brief as context — the planner should build on the design decisions already made, not restart the interview from scratch.
- **Multi-session:** `/omc-plan --interactive --consensus` (omc-team claude) with the design brief as context — interview focuses on implementation details, then planner/architect/critic deliberation loop until agreement.

**Offering consensus escalation:** For any class, if the task touches critical paths, has multiple viable approaches, or the user seems uncertain, ask:

> This could benefit from consensus planning (planner + architect + critic review loop). Want to upgrade to `--consensus`?

**No-placeholders rule:** Plans must be concrete and actionable. Flag any of these before proceeding to Checkpoint 1:

- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above" without specifying what to test
- "Similar to Task N" instead of repeating the actual steps
- Steps that describe *what* to do without showing *how*
- References to undefined types, functions, or methods

If the plan contains placeholders, send it back to `/omc-plan` with specific feedback on what needs to be concrete.

**Multi-session only — produce both:**
1. **Tracking issue** → `gh issue create --title "<name>" --label "tracking" --body "<roadmap markdown>"` — full scope with task-list checkboxes (`- [ ]`) for each phase, dependencies, and acceptance criteria per phase
2. **Session plan** → `.omc/plans/` — current phase only

## Step 4 — Checkpoint 1: Plan Review

Present the Step 2 output (assumption check for small tasks, or design brief for larger ones) and the plan together. User options:

1. **Approve** → Step 5
2. **Request changes** → revise plan, re-present
3. **Request further review** → solicit critique, re-present
4. **Peer review** → `/peer-plan-review` — delegate review to an external model (Gemini, Codex, Cursor, or Claude), then re-present options with their feedback

## Step 5 — Execute

**Update workflow cursor now** — `notepad_write_priority` with `[>] 5-Execute`. This is the deepest phase; the cursor keeps remaining steps visible.

Signpost before proceeding:

> Creating isolated worktree on branch `<branch-name>`. Your main worktree stays untouched. All changes happen there.

Always use an isolated worktree unless the user explicitly says otherwise (e.g. "just do it here", "no worktree").

Then: `EnterWorktree` → execute (direct for small tasks, `/team claude` for single/multi-session).

## Step 6 — Test

Run the relevant test suite:

```bash
# Single package
cd <package> && uv run pytest tests/ -x

# Cross-package: run for each affected package
```

If tests fail: diagnose, fix, re-run. Do not proceed with failing tests.

## Step 7 — Doc Check

Check before shipping — update in the same branch if needed:

- CLI entry points, arguments, or flags changed? → update root `CLAUDE.md` quick reference table
- Pipeline invocations changed? → update package `CLAUDE.md`
- New vocabulary, reference proteins, or data paths? → update vocabulary translations
- New scripts or tools? → add to the appropriate table

Tell the user what you updated (or "no doc changes needed").

## Step 8 — Commit

Stage and commit all changes from execution, test fixes, and doc updates. All work should be committed before proceeding to PR strategy.

```bash
git add <changed-files>
git commit -m "<conventional commit message>"
```

If there are multiple logical changes, use multiple commits. The branch should be clean (no uncommitted changes) before Step 9.

## Step 9 — Checkpoint 2: PR Strategy

> Branch: `<branch-name>` | Worktree: `<path>` | Changes committed, ready to push.

**Recommend an option** based on task context:

- Small task, tests pass, low risk → recommend **`push-mergesync`**
- Medium complexity or touching shared code → recommend **`open-pr-review`** or **`open-pr-peer-review`**
- Standard case → recommend **`open-pr`**

Always state your recommendation with a one-line rationale before presenting options. User can always override.

Present options:

1. **`open-pr`** — push branch, open PR, enter check loop (Step 10) (default)
2. **`open-pr-review`** — push + PR + `/review-pr`, Claude reviews and fixes, then check loop (Step 10). Note: when invoked from here, `/review-pr` should end with `done` (not `mergesync`) — mergesync is handled by Step 10.
3. **`open-pr-peer-review`** — push + PR + `/peer-pr-review`, external model reviews (ask which: 1. gemini, 2. codex, 3. cursor, 4. claude), then await completion (see below) and enter check loop (Step 10)
4. **`push-mergesync`** — push + PR + mergesync immediately (no review, no check loop); then Step 11

**PR body must include the worktree path** so reviewers can check out directly:

> Worktree: `<absolute-worktree-path>`

**Awaiting peer review (option 3):** after spawning the worker, poll PR comments every 30s for a `## Review Summary` comment. When found (or worker signals done via `SendMessage`), run a **check** and present Step 10 options.

## Step 10 — PR Open: Check Loop

**Update workflow cursor now** — `notepad_write_priority` with `[>] 10-Check`. Update again on each loop iteration with current PR state (checks passing/failing, open comments).

**"Check"** = fetch ALL of:

```bash
gh pr view <number>
gh api repos/{owner}/{repo}/pulls/{number}/reviews
gh api repos/{owner}/{repo}/pulls/{number}/comments
gh pr view <number> --json commits
gh pr checks <number>
```

**Recommend an option** based on current PR state:

- Peer review ran, issues were fixed, checks pass → recommend **`mergesync`**
- Open review comments or failing checks → recommend **`check-fix`** with summary of what needs attention
- No reviews yet, checks still running → recommend **`check`** and wait
- Review approved, no open threads → recommend **`mergesync`**

Always state your recommendation with a one-line rationale before presenting options. User can always override.

Present options:

1. **`check`** — fetch PR state, report, re-present options
2. **`check-fix`** — fetch, fix in worktree, re-test, commit, push, re-present options
3. **`check-fix-mergesync`** — fetch, fix in worktree, re-test, commit, push, mergesync; then Step 11
4. **`mergesync`** — mergesync now (no check); then Step 11
5. **`abandon`** — close PR (`gh pr close <number>`), leave worktree intact; then Step 11

Repeat options 1 or 2 until user picks option 3, 4, or 5.

## Step 11 — Continuation

> Worktree: `<path>` | PR: #`<number>` (merged or closed)

**Proactive suggestion:** Before presenting options, assess context and recommend a next step with rationale:

- **Multi-session with remaining phases** → recommend **New worktree**, present the next phase from the tracking issue, and offer to generate the session plan
- **Multi-session, final phase just completed** → recommend **Stop + cleanup**, note that the project is complete, and suggest closing the tracking issue
- **Standalone task, no follow-up** → recommend **Stop + cleanup**
- **Visible follow-up work** (e.g. user mentioned it, or the change surfaced something) → recommend **Continue in worktree** or **New worktree** with rationale

Present options:

1. **Stop + cleanup** — `ExitWorktree`, sync, run `/clean_gone` to remove stale local branches. Done.
2. **Continue in worktree** — stay in the worktree for follow-up work. Push a new branch when ready (remote branch was deleted by mergesync; if abandoned, the remote branch still exists — reuse or rename). Sync main in background.
3. **New worktree** — `ExitWorktree`, sync, cleanup current worktree, create fresh one for next piece of work.

**Multi-session only — before stopping:**
1. Update tracking issue: check off the completed phase checkbox (`gh issue edit`), add a session-notes comment (`gh issue comment`) covering decisions made, approaches tried and rejected, and surprising discoveries
2. Generate next session plan from the tracking issue into `.omc/plans/`
3. Present next session plan → loop to Step 4
