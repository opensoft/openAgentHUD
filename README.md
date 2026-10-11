# openAgentHUD

An agent-CLI-independent terminal HUD for showing live coding-agent state in a
compact, configurable information area.

openAgentHUD separates the display layer from the data source. A common HUD
can show the same useful information for Codex, Claude, Gemini, Aider, and
other agent CLIs while adapters translate each CLI's available telemetry into a
shared state model.

## Goals

- Keep the HUD outside any individual agent CLI's source tree.
- Make model, context, turn, quota, project, and Git state visible at a glance.
- Prefer live event streams, then file/database observers, then process and
  shell fallbacks.
- Detect session rollover and reconnect automatically.
- Keep unknown or stale data visibly honest instead of displaying old values as
  current.
- Run in a tmux pane, standalone terminal, or other TUI host.

## Planned adapters

- Codex: app-server events when controlled by the HUD; rollout, state, log,
  process, and tmux observers when attached to the native CLI.
- Claude: status-line input and command integration.
- Generic: process, environment, Git, terminal, and user-provided JSON state.
- Additional adapters can be added without changing the renderer.

See [FEATURES.md](FEATURES.md) for the roadmap and
[docs/architecture.md](docs/architecture.md) for the proposed design.

## License

Apache-2.0. See [LICENSE](LICENSE).

