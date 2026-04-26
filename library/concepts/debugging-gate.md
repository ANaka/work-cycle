# Debugging Gate

The debugging gate prevents speculative fix loops.

## Sequence

1. Reproduce or capture the exact failure.
2. Read the full error, logs, or stack trace.
3. Inspect recent changes and nearby working examples.
4. State one root-cause hypothesis with evidence.
5. Make the smallest fix or experiment that tests the hypothesis.
6. Verify with the narrowest command that proves the original symptom changed.

## Three-Attempt Cap

After three failed fix attempts, stop and reassess. Do not stack a fourth
speculative fix on top of failed guesses.

## Applies To

- failing tests
- build failures
- CI failures
- unexpected runtime behavior
- flaky or timing-sensitive behavior
- validation and schema errors
