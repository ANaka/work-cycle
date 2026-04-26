# OMC Optional Profile

Use this profile when oh-my-claudecode is available and helpful.

## Dependency Posture

OMC is an optional runtime and orchestration layer. It is not the portable
foundation for Work Cycle. The portable library should not require OMC commands,
state stores, HUDs, or worker runtimes.

## Good Uses

- Claude-specific planning commands
- Claude in-session team execution
- tmux worker orchestration
- advisor synthesis
- stateful Claude execution modes

## Avoid

- making `/omc-plan` the only planning path
- making notepad/state primitives required by portable workflows
- claiming Codex or Cursor adapters depend on OMC
- copying the full OMC runtime into this repo

## Current Claude Reality

The root Claude plugin still references OMC surfaces. That is valid for the
current Claude adapter and should be documented honestly until those references
are replaced or wrapped.
