# Acceptance Ledger

An acceptance ledger turns a plan into observable completion criteria.

## Format

```markdown
Acceptance Criteria:
- [ ] AC1: <observable behavior> -- Evidence: <test/command/manual check>
- [ ] AC2: <observable behavior> -- Evidence: <test/command/manual check>
```

## Rules

- Criteria must be directly observable.
- Evidence must name a command, test, diff check, or manual inspection.
- Do not use "implementation complete" as a criterion by itself.
- Mark a criterion complete only after fresh evidence exists in the current
  cycle.
- Revise the ledger if implementation reveals a required behavior that was
  missing from the plan.

## Good Criteria

- "PR check failures are reproduced or classified from logs."
- "README links resolve to tracked files."
- "Auth schema migration has rollback notes."
- "No root Claude plugin paths moved."

## Bad Criteria

- "Clean up docs."
- "Make it better."
- "Tests pass."
- "Should work."
