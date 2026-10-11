# openAgentHUD feature roadmap

This is the initial product list for a universal agent CLI HUD. Items are
ordered by the value they provide and the foundation they require.

## v0.1 — universal HUD foundation

- [ ] Agent-independent renderer with a normalized state model.
- [ ] Configurable line layout, field order, height, width, separators, and
      compact/expanded modes.
- [ ] Themeable colors matching terminal status-line conventions, including
      Claude-like color mapping.
- [ ] Tmux backend with a dedicated HUD pane.
- [ ] Standalone terminal backend.
- [ ] Safe fallback rendering when an adapter or field is unavailable.
- [ ] Explicit `unknown`, `stale`, and `disconnected` indicators.
- [ ] Update cadence controls with event-driven refresh and bounded polling.
- [ ] Privacy controls for prompts, responses, usernames, paths, and quota
      details.

## v0.2 — common agent state

- [ ] Current agent/CLI name and version.
- [ ] Current model and provider.
- [ ] Reasoning effort or thinking level when available.
- [ ] Current working folder and project root.
- [ ] Git branch, dirty state, file counts, ahead/behind, and worktree.
- [ ] Turn state: idle, thinking, working, waiting, completed, failed, or
      interrupted.
- [ ] Turn start time, elapsed time, last response time, and output speed.
- [ ] Context used, remaining, total window, and token breakdown.
- [ ] Session token totals, input/output/cache tokens, and compaction count.
- [ ] Permission, approval, sandbox, and collaboration-mode status.

## v0.3 — agent activity

- [ ] Active tool names and targets.
- [ ] Tool duration, completion, and error state.
- [ ] Loaded skills and plugins.
- [ ] MCP server activity.
- [ ] Subagents and their model/status.
- [ ] Current plan/todo item and completion count.
- [ ] Session title, goal, session ID, and branch/fork relationship.
- [ ] Recent turn count and optional compact conversation summary.

## v0.4 — usage and account data

- [ ] Primary and secondary usage windows.
- [ ] Percent used and percent remaining.
- [ ] Reset countdown and absolute reset time.
- [ ] Account balance or credit label where supported.
- [ ] External usage providers through configured commands or HTTP queries.
- [ ] Usage freshness timestamps and cached-data indicators.
- [ ] Provider-specific quota adapters without polluting common fields.

## v0.5 — reliable synchronization

- [ ] Per-launch session identity and adapter ownership.
- [ ] Automatic session-rollover detection.
- [ ] Rebinding when a CLI creates a new thread or rollout file.
- [ ] Atomic source switching with no mixed old/new state in one frame.
- [ ] File-watch updates with a short debounce window.
- [ ] Fast event-log updates for model and effort changes.
- [ ] Periodic reconciliation against the durable session source.
- [ ] Process-exit and pane-disconnect detection.
- [ ] Diagnostics showing the active source, session ID, age, and last update.
- [ ] Test fixtures for stale bindings, concurrent sessions, and model
      switches.

## v0.6 — CLI adapters

- [ ] Codex native-TUI observer adapter.
- [ ] Codex app-server adapter for a HUD-controlled session.
- [ ] Claude status-line adapter.
- [ ] Gemini CLI adapter.
- [ ] Aider adapter.
- [ ] Generic JSON event adapter.
- [ ] Adapter capability negotiation so unsupported fields disappear cleanly.
- [ ] Adapter health and version compatibility reporting.

## v0.7 — integration and usability

- [ ] Wrapper commands such as `yagent`, `yodex`, or explicit adapter selection.
- [ ] Automatic adapter selection from the child process and environment.
- [ ] Shared session ID passed through environment variables.
- [ ] Config reload without restarting the agent.
- [ ] Keyboard actions for expanding, collapsing, freezing, and inspecting
      details.
- [ ] Copyable diagnostics and a redacted debug report.
- [ ] Multiple HUD profiles for compact, full, and presentation layouts.
- [ ] Shell integration for zsh, bash, and fish.

## Later possibilities

- [ ] Native app-server frontends for CLIs that expose a supported event API.
- [ ] WebSocket or local-socket state bus for multiple HUD consumers.
- [ ] Remote HUD display over SSH or a local dashboard.
- [ ] Historical turn and usage charts.
- [ ] Plugin SDK for third-party adapters.
- [ ] Cross-machine session handoff and reconnect.
- [ ] Structured export for status bars and desktop widgets.

## Non-goals

- Do not scrape terminal pixels as the primary integration method.
- Do not fork or patch every agent CLI to render the HUD.
- Do not expose prompt or response contents by default.
- Do not claim that a cached or stale value is live.
