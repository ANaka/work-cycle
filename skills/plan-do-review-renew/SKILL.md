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
WCYCLE: <task-summary> (<triage-class>, <tier>)
[>] 1-Triage    [ ] 2-Brainstorm  [ ] 3-Plan
[ ] 4-CP:Plan   [ ] 5-Execute     [ ] 6-Test
[ ] 7-Simplify  [ ] 8-Docs        [ ] 9-FinalVerify
[ ] 10-Commit   [ ] 11-CP:PR      [ ] 12-Check
[ ] 13-Continue
```

Add `br:<branch>`, `wt:<worktree-path>`, `pr:#<number>`, and `issue:#<number>` to the last line as they become available. Mark skipped steps `[-]` (e.g. small tasks may skip the plan file or simplify pass). For loops, append `(Rev N)` to the active step.

**Multi-session only:** also use `state_write` at durable milestones (triage done, plan approved, worktree created, PR opened, PR merged, continuation chosen) to persist across sessions. Use whichever OMC mode is active (e.g. `ralph` if invoked via ralph, `autopilot` if via autopilot). Include these fields in the `state` object: `workflow` ("work_cycle"), `step`, `step_name`, `triage_class`, `tier`, `branch`, `worktree_path`, `tracking_issue`, `pr_number`, `updated_at`.

## Verification Tiers

Triage class describes scope; tier describes verification depth. They are orthogonal — a one-line edit to `auth/session.py` is small in scope but Thorough in tier.

| Tier | Use when | Minimum evidence |
|------|----------|------------------|
| **Light** | <5 files, <100 changed lines, no API/security/data behavior | focused test or direct command, plus diff review |
| **Standard** | default for most contained changes | focused tests + relevant lint/typecheck + repo-required check before commit |
| **Thorough** | security/auth, schemas, migrations, data-loss risk, public APIs, broad refactors, >20 files | full relevant suite + acceptance ledger review + regression checks + peer review when available |

Pick a tier in Step 1 and record it in the cursor. Escalate when the user says "careful", "critical", "production", "security", "migration", or similar.

**No-downgrade rule:** Never downgrade tier when touched paths include auth, credentials, secrets, schemas, package/dependency manifests, deployment config, or migration code — even under time pressure, even for "tiny" edits. The blast radius, not the diff size, sets the floor.

## Debugging Gate

When facing a bug, test failure, build failure, or unexpected behavior, do not patch symptoms first. Follow this sequence:

1. Reproduce or capture the exact failure.
2. Read the full error / log / stack trace.
3. Check recent changes and nearby working examples.
4. State **one** root-cause hypothesis with the evidence for it before writing a fix.
5. Make the smallest change that tests or fixes that hypothesis.
6. Verify with the narrowest command that proves the original symptom changed.

**Three-attempt cap:** if three fix attempts fail, stop and reassess the underlying design or ask the user for direction. Do not stack a fourth speculative fix on top of previous guesses. "One more try" is the symptom of the loop, not a way out.

## Step 1 — Triage

**Initialize the workflow cursor now** — `notepad_write_priority` with the full checklist and `[>] 1-Triage` active. This is the most important write; all subsequent updates build on it.

Quick scope assessment (~30 seconds of exploration):

- Which packages/subdirs are touched?
- Single file, single package, or cross-package?
- Any CLI entry points, arguments, or data paths changing?
- How ambiguous is the request? Could reasonable engineers disagree on approach?

### Vague Request Gate

Before classifying, check whether the request has at least one **concrete anchor**:

- file path, module, function, class, command, error text, failing test, issue/PR number, numbered steps, code block, or explicit acceptance criteria.

If no anchor exists and the task is more than a trivial edit, do not skip ahead — gather cheap codebase facts first, then ask one focused clarification. Do not start an implementation cycle on a request like "make the auth better" without anchoring it first.

### Classify

| Class | Criteria | Planning approach | Plan location |
|-------|----------|-------------------|---------------|
| **Small** | 1–3 files, single package, no API changes | `/omc-plan --direct` (omc-team claude) — skip interview, describe approach inline | Skip plan file — approach stated in Checkpoint 1 |
| **Single-session** | Contained feature/fix within one package | `/omc-plan --interactive` (omc-team claude) — thorough interview before planning | `.omc/plans/` |
| **Multi-session** | Cross-package, multi-phase, or architectural | `/omc-plan --interactive --consensus` (omc-team claude) — interview + planner/architect/critic loop | GitHub tracking issue + `.omc/plans/` |

Pick the verification tier (Light / Standard / Thorough — see Verification Tiers section). Present classification and tier together with reasoning. User can override either.

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

Adopts the discipline of `superpowers:brainstorming` inline — one question at a time, scope check, self-review — but does **not** invoke the skill (so no HARD GATE, no committed spec file required). Output is a conversational design brief, not an artifact on disk.

1. **Scope check (before questions)** — if the request describes multiple independent subsystems, flag it and propose decomposition before drilling into details. If the right scope is "broad", re-triage upward to Multi-session.
2. **Clarifying questions** — ask one at a time. Prefer multiple-choice. Gather codebase facts (read code, check patterns) *before* asking the user about them. Only ask the user for preferences, scope decisions, and constraints.
3. **Propose 2–3 approaches** — for each, 1–2 sentences, pros/cons, trade-offs. Lead with your recommendation and reasoning. Present one at a time.
4. **Design brief** — produce an inline summary:
   - **Goal:** one sentence
   - **Chosen approach:** what and why
   - **Alternatives considered:** what was rejected and why
   - **Assumptions:** what we're taking as given
   - **Open questions:** anything unresolved (ideally none)
   - **Acceptance Criteria Ledger** (see below)
   - **Verification:** tier + how we'll know it works — specific tests, checks, or expected outcomes

The all-classes Self-Review Checklist below covers placeholders, scope creep, ambiguity, and verification gaps before Step 3 — run it after the brief.

If the user asks for a durable artifact ("save the spec", "commit the design"), write it to `.omc/plans/<task-slug>/design.md` **inside the task worktree**. If no task worktree exists yet, run the Worktree Pre-Creation Safety checks from Step 5 first, create or enter the task worktree, and record `br:` / `wt:` in the cursor. Commit the artifact from that branch, never from an unisolated main checkout. Otherwise the brief lives in the conversation only.

### Multi-session: Deep Design Exploration

**Invoke `superpowers:brainstorming`** with the overrides below. The HARD GATE applies — no implementation work begins until the spec is written, self-reviewed, and the user has approved it.

When invoking, instruct the skill explicitly:

> Write the spec to `.omc/plans/specs/YYYY-MM-DD-<topic>-design.md`, **not** the default `docs/superpowers/specs/` path. After spec approval, return control to the `plan-do-review-renew` workflow at Step 3 — do **not** invoke `writing-plans`. OMC's `/omc-plan --interactive --consensus` is the planner here.

The brainstorming skill will run its standard flow: explore project context → clarifying questions → 2–3 approaches → present design → write spec → self-review → user-review gate. Cover architecture implications, phase decomposition (which parts can ship independently, what order reduces risk), and cross-cutting concerns (shared types, migration paths, backwards compatibility) — these are required at this tier.

After the user approves the spec:

1. Ensure the spec lives inside the task worktree. If no task worktree exists yet, run the Worktree Pre-Creation Safety checks from Step 5 first, create or enter the task worktree, and record `br:` / `wt:` in the cursor.
2. Create the GitHub tracking issue non-interactively:
   ```bash
   gh issue create --title "<topic> tracking" --label "tracking" --body-file <tracking-issue-body.md>
   ```
   The body file must link to the spec path and contain the phase checklist (`- [ ]` per phase), dependencies, and acceptance criteria.
3. Commit the spec file from the task branch alongside any durable tracking-issue note.
4. Continue to Step 3 with the spec file path, tracking issue number, branch, and worktree path captured in the workflow cursor.

### Acceptance Criteria Ledger

For every Single-session and Multi-session task (and any Small task with non-trivial behavior), produce an explicit ledger:

```markdown
Acceptance Criteria:
- [ ] AC1: <observable behavior> — Evidence: <test/command/manual check>
- [ ] AC2: <observable behavior> — Evidence: <test/command/manual check>
```

Rules:

- Each criterion must be testable or directly observable.
- Avoid generic criteria like "implementation works" or "tests pass" unless paired with behavior-specific checks (named tests, expected output, observable side effect).
- Mark criteria complete only after **fresh evidence** in the same session — a passing run from before cleanup or refactor is stale.
- If implementation reveals new required behavior, add or revise criteria before claiming completion.

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

Pass the design brief from Step 2 (including the Acceptance Criteria Ledger) as context to `/omc-plan` so the planner doesn't re-ask resolved questions.

Run `/omc-plan` with flags determined by triage:

- **Small:** `/omc-plan --direct` (omc-team claude) with the task description and assumption check. Go to Step 4 with the generated approach. Do NOT skip Steps 4–13 — worktree, test, simplify, final verify, commit, and checkpoints still apply.
- **Single-session:** `/omc-plan --interactive` (omc-team claude) with the design brief and acceptance ledger from Step 2 as context — the planner should build on the design decisions already made, not restart the interview from scratch.
- **Multi-session:** `/omc-plan --interactive --consensus` (omc-team claude) with the spec file (`.omc/plans/specs/<file>.md`) and acceptance criteria as context — the interview focuses on implementation details (the spec already settled the design), then planner/architect/critic deliberation loop until agreement.

**Offering consensus escalation:** For any class, if the task touches critical paths, has multiple viable approaches, or the user seems uncertain, ask:

> This could benefit from consensus planning (planner + architect + critic review loop). Want to upgrade to `--consensus`?

### Plan Quality Gate

Plans must be executable by a capable engineer with no chat history and "questionable taste" — assume they know nothing about our codebase, problem domain, or test conventions. Reject or revise the plan before Checkpoint 1 if any of these are present:

**Placeholders and hand-waving:**
- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling", "add validation", "handle edge cases" without naming the cases, validation rules, or error paths
- "Write tests for the above" without specifying what to test
- "Similar to Task N" instead of repeating the actual steps (the engineer may read tasks out of order)
- Steps that say *what* to do without showing *how* (code blocks required for code steps)

**Missing concreteness:**
- File changes without exact paths and ownership boundaries
- Undefined functions, types, methods, commands, flags, or dependencies
- Commands without expected output (engineer can't tell pass from fail)
- Missing rollback or recovery notes for migrations, destructive operations, or risky changes

**Granularity and consistency:**
- Steps coarser than 2–5 minutes of work each (split them — "write failing test", "run it to confirm failure", "implement", "verify pass", "commit" are five steps, not one)
- Type/signature/property-name drift between tasks (a function called `clearLayers()` in Task 3 but `clearFullLayers()` in Task 7 is a bug — flag it)
- For behavior changes: TDD shape (failing test → minimal implementation → verify → commit) unless the task is docs/config/generated-code

If the plan fails the gate, send it back to `/omc-plan` with specific feedback on what needs to be concrete.

**Self-review against the spec (Multi-session only):** after `/omc-plan` returns the plan, skim each section/requirement of the spec at `.omc/plans/specs/<file>.md`. Can you point to a task that implements it? List any gaps, then fix inline.

**Multi-session only — artifact layout:**
1. **Spec** → `.omc/plans/specs/YYYY-MM-DD-<topic>-design.md` (already written in Step 2)
2. **Tracking issue** → links to the spec file; body has `- [ ]` phase checkboxes, dependencies, and acceptance criteria per phase
3. **Session plan** → `.omc/plans/<phase-slug>.md` — current phase only, references the spec for design context

## Step 4 — Checkpoint 1: Plan Review

Present the Step 2 output (assumption check for small tasks, or design brief + Acceptance Criteria Ledger for larger ones) and the plan together. User options:

1. **Approve** → Step 5
2. **Request changes** → revise plan, re-present
3. **Request further review** → solicit critique, re-present
4. **Peer review** → `/peer-plan-review` — delegate review to an external model (Gemini, Codex, Cursor, or Claude), then re-present options with their feedback

## Step 5 — Execute

**Update workflow cursor now** — `notepad_write_priority` with `[>] 5-Execute`. This is the deepest phase; the cursor keeps remaining steps visible.

Signpost before proceeding:

> Creating isolated worktree on branch `<branch-name>`. Your main worktree stays untouched. All changes happen there.

Always use an isolated worktree unless the user explicitly says otherwise (e.g. "just do it here", "no worktree").

If Step 2 already created a task worktree for a durable spec or design artifact, reuse it here: verify `git status --short`, confirm the branch/base, and continue execution there instead of creating a second worktree.

### Worktree Pre-Creation Safety

Before creating the worktree, verify the destination directory is safe:

1. Choose a worktree location, in this order of preference:
   - existing `.worktrees/`
   - existing `worktrees/`
   - location named in `CLAUDE.md` / `AGENTS.md`
   - default to `.worktrees/`
2. For project-local locations, verify the directory is git-ignored: `git check-ignore -q <dir>`.
3. If it is not ignored:
   - If repo policy allows a tracked rule, add the directory to `.gitignore` before creating the worktree.
   - If a local-only rule is preferred, write to `.git/info/exclude` instead.
   - If neither is safe (dirty main worktree, mid-feature changes, or repo should not be touched), create the worktree **outside** the repo at `~/.claude/worktrees/<project>/<branch-slug>/`.
4. Never create a project-local worktree directory that can be accidentally tracked or committed.

Then: `EnterWorktree` → execute (direct for small tasks, `/team claude` for single/multi-session).

## Step 6 — Test

Run focused tests that prove the changed behavior. This run **locks behavior** before any cleanup.

```bash
# Single package
cd <package> && uv run pytest tests/ -x

# Cross-package: run for each affected package
```

If tests fail: invoke the **Debugging Gate** (3-attempt cap, root-cause hypothesis required). Do not proceed with failing tests.

Check each Acceptance Criterion against this run's evidence. Mark `[x]` only when the criterion observably passed in this session.

## Step 7 — Simplify / Deslop

Run after focused tests pass and before docs / final verify / commit. AI-written code accumulates bloat — wrappers, defensive checks, duplicated helpers, dead branches. Skip explicitly only if the change is docs-only, config-only, or generated code.

Use `/oh-my-claudecode:ai-slop-cleaner <changed-files>` scoped to the changed-file set. The cleaner will:

- protect behavior with regression tests
- run smell-focused passes one at a time (dead code → duplicate → naming → test reinforcement)
- preserve external behavior, public APIs, sync/async boundaries, error propagation, and side effects
- prefer deletion over addition

For higher-risk cleanup, follow with `/oh-my-claudecode:ai-slop-cleaner <files> --review` for a reviewer-only pass before continuing.

After Simplify, the focused tests from Step 6 must still pass. If cleanup changes behavior unintentionally, return to Step 6 with the Debugging Gate. Mark Step 7 `[-]` skipped when intentionally bypassed and record the reason in the cursor.

## Step 8 — Doc Check

Update docs in the same branch when behavior, CLI flags, public APIs, data paths, setup steps, or user workflows changed:

- CLI entry points, arguments, or flags changed? → update root `CLAUDE.md` quick reference table
- Pipeline invocations changed? → update package `CLAUDE.md`
- New vocabulary, reference proteins, or data paths? → update vocabulary translations
- New scripts or tools? → add to the appropriate table

Tell the user what you updated (or "no doc changes needed").

## Step 9 — Final Verify

After Simplify and Docs, run **fresh** tier-appropriate verification:

- Re-run the focused tests that locked behavior in Step 6 — Simplify or Docs may have edited code paths.
- Run tier-required checks before commit (Standard: relevant lint/typecheck; Thorough: full suite + regression checks).
- Re-check every Acceptance Criterion against this run's evidence; do not carry over green ticks from before Simplify.

If anything fails, return to Step 6 / Step 7 as appropriate. Do not proceed to commit on stale evidence.

## Step 10 — Commit

Stage and commit all changes from execution, simplify, doc updates, and any final-verify fixes. The branch must be clean before Step 11.

```bash
git add <changed-files>
git commit -m "<conventional commit message>"
```

If there are multiple logical changes, use multiple commits.

## Step 11 — Checkpoint 2: PR Strategy

> Branch: `<branch-name>` | Worktree: `<path>` | Tier: `<tier>` | Changes committed, ready to push.

**Recommend an option** based on task context:

- Small task, Light tier, tests pass, low risk → recommend **`push-mergesync`**
- Standard or Thorough tier touching shared code → recommend **`open-pr-review`** or **`open-pr-peer-review`**
- Standard case → recommend **`open-pr`**

Always state your recommendation with a one-line rationale before presenting options. User can always override.

Present options:

1. **`open-pr`** — push branch, open PR, enter check loop (Step 12) (default)
2. **`open-pr-review`** — push + PR + `/review-pr`, Claude reviews and fixes, then check loop (Step 12). Note: when invoked from here, `/review-pr` should end with `done` (not `mergesync`) — mergesync is handled by Step 12.
3. **`open-pr-peer-review`** — push + PR + `/peer-pr-review`, external model reviews (ask which: 1. gemini, 2. codex, 3. cursor, 4. claude), then await completion (see below) and enter check loop (Step 12)
4. **`push-mergesync`** — push + PR + mergesync immediately (no review, no check loop); then Step 13

**PR body must include the worktree path** so reviewers can check out directly:

> Worktree: `<absolute-worktree-path>`

**Awaiting peer review (option 3):** after spawning the worker, poll PR comments every 30s for a `## Review Summary` comment. When found (or worker signals done via `SendMessage`), run a **check** and present Step 12 options.

## Step 12 — PR Open: Check Loop

**Update workflow cursor now** — `notepad_write_priority` with `[>] 12-Check`. Update again on each loop iteration with current PR state (checks passing/failing, open comments).

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
2. **`check-fix`** — fetch, fix in worktree (use Debugging Gate for regressions), re-test, commit, push, re-present options
3. **`check-fix-mergesync`** — fetch, fix in worktree, re-test, commit, push, mergesync; then Step 13
4. **`mergesync`** — mergesync now (no check); then Step 13
5. **`abandon`** — close PR (`gh pr close <number>`), leave worktree intact; then Step 13

Repeat options 1 or 2 until user picks option 3, 4, or 5.

## Step 13 — Continuation

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
4. **Handoff** — generate a paste-ready handoff prompt for another session (see Handoff Prompt below).

**Multi-session only — before stopping:**
1. Update tracking issue: check off the completed phase checkbox (`gh issue edit`), add a session-notes comment (`gh issue comment`) covering decisions made, approaches tried and rejected, and surprising discoveries
2. Generate next session plan from the tracking issue into `.omc/plans/`
3. Present next session plan → loop to Step 4

## Handoff Prompt

Create a paste-ready handoff prompt when the user picks option 4 in Step 13, when context is getting long, or when another session should continue the work. Refresh cheap facts first (`pwd`, `git status --short`, `git branch --show-current`, `git worktree list`, recent commits, active PR/issue state) — do not invent state, mark unknowns explicitly.

### Auto-Suggest Triggers

Proactively offer a handoff prompt — without waiting to be asked — when any of these fire mid-cycle:

- **Context compaction just ran.** The conversation summary indicates context window was compressed; details may be lossy. Offer a handoff before any further multi-step work.
- **Same step has looped 3+ times** (e.g. Step 6 → Debugging Gate → Step 6 → Debugging Gate). State is fragile; capture it.
- **External blocker is pending** with no immediate user action (CI running 10+ min, peer review awaited, third-party API outage).
- **User signals end-of-session** ("wrapping up", "back tomorrow", "EOD", "have to step away", "let's pause here").
- **Multi-session phase boundary** between phases of a tracking issue, even if the user has not explicitly stopped.
- **Step 13 reached without explicit cleanup choice** — recommend handoff alongside the Continue/Stop/New options.

Format the proactive offer as a one-liner, not a wall of text:

> Want me to draft a handoff prompt? Context is getting tight after the last compaction.

Do not spam-suggest. One offer per trigger event; if declined, do not re-suggest until a *new* trigger fires.

Template:

```markdown
Continue this work using the `plan-do-review-renew` skill.

Repo: <repo-root>
Worktree: <absolute-worktree-path>
Branch: <branch>
Base: <base-ref>
PR: <number-or-url-or-none>
Tracking issue: <number-or-url-or-none>
Triage class: <small | single-session | multi-session>
Verification tier: <light | standard | thorough>

Current step: <step number and name>
Completed:
- <step / result>

Goal:
<one-paragraph task goal>

Approach and assumptions:
- <chosen approach>
- <important assumption or scope boundary>

Changed files:
- <path> — <what changed and why>

Simplify / deslop:
- <what was simplified, deleted, deduplicated, or why skipped>

Acceptance criteria:
- [x/ ] <criterion> — <evidence or missing evidence>

Verification:
- <command> — <result>

Known state and risks:
- <blocker, failing check, review comment, or risk>
- <unrelated dirty file or do-not-touch note>

Next actions:
1. cd <worktree-or-repo-path>
2. <exact next command or investigation>
3. <next implementation/review/test step>

Important instructions:
- Read CLAUDE.md and nearby project guidance before editing.
- Preserve unrelated changes; do not revert files you did not touch.
- Keep working in the listed worktree unless the user explicitly redirects.
- Run tier-appropriate verification before claiming completion.
```

If work is fully complete, still produce a handoff-style closeout when requested but set `Current step` to `13 - Done` and make `Next actions` focus on optional cleanup, follow-ups, or related work.
