# Plan-Do-Review-Renew

This is the portable Work Cycle contract. Adapters may rename tools and commands,
but should preserve the state sequence and exit evidence.

## States

| State | Purpose | Exit Evidence | Normal Next |
| --- | --- | --- | --- |
| S0 Intake | Understand the request and local guidance. | Goal and context are clear enough to triage. | S1 or S16 for direct read-only answers |
| S1 Triage | Classify scope, risk, anchors, and verification tier. | Scope class, risk notes, and verification tier are named. | S2, S3 for tiny anchored work, or S16 for read-only answers |
| S2 Brainstorm | Check assumptions and choose an approach. | Approach, assumptions, and verification shape are stated. | S3 or S1 if scope changes |
| S3 Plan | Build executable tasks and acceptance criteria. | Plan passes quality gate; criteria are observable. | S4 |
| S4 Plan Checkpoint | Present or record the plan before execution. | Approval, autonomous go-ahead, or revision path is explicit. | S5, S3 for revisions, or S16 if stopped |
| S5 Worktree | Create or enter the execution workspace. | Branch, base, path, and setup state are known. | S6 |
| S6 Execute | Make scoped changes. | Intended files changed with no unrelated cleanup. | S7, or S2 if the design breaks |
| S7 Test | Run focused checks for changed behavior. | Checks pass or failures are diagnosed. | S8, or S6 on failures |
| S8 Simplify | Remove avoidable bloat and confusing structure. | Cleanup is complete or skipped with reason. | S9, or S7 if cleanup changes behavior |
| S9 Docs | Update docs for changed workflows or commands. | Docs updated or no-docs-needed is recorded. | S10 |
| S10 Final Verify | Run the required verification tier. | Fresh evidence covers acceptance criteria. | S11, or S6 on failures |
| S11 Review | Inspect diff for scope, quality, and leftovers. | Diff is understood and risks are accounted for. | S12, or S6 if review finds fixes |
| S12 Commit | Commit only when the user or shipping path asks for it. | Commit exists or uncommitted state is intentional. | S13 or S15 for local-only work |
| S13 PR Checkpoint | Choose PR, draft, local-only, or merge strategy. | Strategy is explicit. | S14 or S15 |
| S14 Check Loop | Fetch PR checks, reviews, comments, and commits. | Checks are clean, fixed, waiting, or explicitly bypassed. | S14, S6 for fixes, or S15 |
| S15 Continue/Handoff | Decide cleanup, continue, new task, or handoff. | Worktree disposition and next action are explicit. | S1 for next task or S16 |
| S16 Done | Report outcome, verification, risks, and residual state. | User has the concrete state of the work. | End |

## State Rules

- Keep one active state at a time.
- Advance on evidence, not intention.
- Loop failures back to the state that owns them.
- Do not jump from execution to final response.
- Do not claim completion without fresh verification.
- Do not close the workflow until the worktree and continuation path are
  accounted for.

## Skip Rules

- Read-only answers may go from S0 to S16 without opening a worktree.
- Tiny anchored edits may skip a durable plan artifact, but not assumption
  checking, verification, review, or continuation accounting.
- Docs-only changes may skip behavior tests, but still need link/path checks,
  diff review, and dependency-claim review.
- Commit and PR states are skipped unless the user asked to ship or the workflow
  has an explicit publishing path.
- Any skipped state needs a reason in the cursor, handoff, or final summary.

## Failure Routes

- Discovery of new scope returns to S1 or S2.
- Plan ambiguity returns to S3.
- Test or verification failure returns to S6 after diagnosis.
- Review findings return to S6 when fixes are valid and in scope.
- PR comments or CI failures return to S6 after the check loop captures the
  failing state.
- Repeated debugging failures trigger the debugging gate reassessment instead
  of another speculative fix.

## Quality Gate

Plans must include exact paths, concrete tasks, named checks, and known rollback
or recovery notes for risky changes. Reject plans with placeholders, generic
"handle edge cases" language, or tests that do not specify what they prove.

## Relationship To Superpowers

Superpowers can provide strong phase implementations for brainstorming,
planning, TDD, debugging, worktrees, review, and branch finishing. Work Cycle
still owns the state machine and continuation policy.
