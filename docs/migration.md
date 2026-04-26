# Migration Plan

This migration keeps the Claude plugin stable while introducing a broader
agentic skills library.

## Phase 1: Add Reference Model

- Add `library/`, `profiles/`, `adapters/`, and `docs/`.
- Keep root `.claude-plugin/`, `skills/`, and `commands/` unchanged.
- Update README to distinguish current Claude implementation from portable
  reference docs.

## Phase 2: Align Claude Docs

- Compare root Claude skills against `library/` contracts.
- Fix stale dependency links and overly broad dependency language.
- Keep command names and marketplace paths stable.

## Phase 3: Codex Adapter

- Add Codex-native skills under an adapter plugin path.
- Validate with Codex skill validation and local invocation.
- Document install and update commands.
- Avoid Claude slash-command parity claims.

## Phase 4: Cursor Adapter

- Add Cursor rule files or prompt packs after the rule shape is validated.
- Keep Cursor guidance scoped to context/rules and prompts.
- Document manual invocation patterns.

## Phase 5: Optional Generation

Only add a generator after the hand-maintained adapters have stabilized. The
first useful generator should check drift and shared metadata, not try to
compile every workflow detail mechanically.

## Compatibility Rule

The current Claude plugin remains the stable implementation until a major
release announces otherwise.
