# openAgentHUD feature roadmap

This is the product roadmap for an agent-CLI-independent terminal HUD. The
HUD presents the state that an adapter can verify; it must not invent parity
between CLIs that expose different telemetry.

Priority language:

- **Foundation** means the feature is required for trustworthy core behavior.
- **MVP** means the feature is required before the first generally useful
  release.
- **Near-term** means it follows the MVP but is part of the supported product
  direction.
- **Later** means it is intentionally deferred until observation is reliable.

## Foundation — trustworthy HUD core

### Agent-independent state and rendering

- [ ] Keep adapters, state reduction, presentation, and terminal backends in
      separate modules; no agent-specific branches in common rendering.
- [ ] Define a versioned normalized state schema with additive,
      namespaced provider extensions.
- [ ] Define field semantics, units, scopes, and whether a value is reported,
      derived, or estimated.
- [ ] Publish immutable, internally consistent state revisions so one rendered
      frame cannot mix incompatible updates.
- [ ] Add snapshot, patch, invalidation, source-health, and capability update
      semantics; distinguish omitted fields from explicit clears.
- [ ] Support adapter handshake metadata: adapter ID/version, CLI version,
      schema compatibility, integration mode, and capabilities.
- [ ] Distinguish capability, availability, freshness, privacy redaction, and
      connection health instead of collapsing them into one status.
- [ ] Preserve unknown values as `unknown`/`null`; never render unavailable
      measurements as zero or healthy state.
- [ ] Keep provider-specific fields namespaced so they do not pollute common
      fields or require renderer changes.

### Layout, colors, and terminal backends

- [ ] Configurable line layout with field order, separators, width, and a
      selectable height from one through ten lines.
- [ ] Compact, expanded, full, presentation, and plain-output profiles.
- [ ] Deterministic field priority and truncation rules that preserve agent,
      model, turn, project, branch, and freshness indicators first.
- [ ] Themeable colors matching terminal status-line conventions, including a
      Claude-like palette mapping, with color disabled independently of layout.
- [ ] Monochrome, high-contrast, ASCII-symbol, reduced-motion, and `TERM=dumb`
      modes; never communicate critical state by color alone.
- [ ] Correct display-width handling for wide characters, combining marks, and
      common emoji sequences.
- [ ] Standalone terminal backend with alternate-screen and append-only/plain
      modes.
- [ ] Tmux backend with explicit server/session/pane ownership, resize support,
      restart detection, and cleanup limited to HUD-owned resources.
- [ ] Preserve agent focus by default; closing or crashing the HUD must not
      terminate or corrupt the agent.
- [ ] Safe rendering of untrusted telemetry after terminal-control sanitizing.

### Configuration and privacy defaults

- [ ] Versioned configuration schema with typed validation, useful errors, and
      documented precedence.
- [ ] Defaults that work without provider credentials, network access, or a
      shell-startup-file modification.
- [ ] Atomic configuration reload that retains the last valid configuration if
      a new file is invalid.
- [ ] Field-level privacy controls for prompts, responses, summaries, goals,
      usernames, paths, repository names, branches, tool arguments, account
      identifiers, and quota/cost details.
- [ ] Apply redaction consistently to the HUD, logs, diagnostics, exports,
      caches, and retained history.
- [ ] Never execute a command, approve an action, or change agent state merely
      because the HUD displayed telemetry.

## MVP — identity, synchronization, and common state

### Session identity and ownership

- [ ] Separate HUD instance, launch, child process, terminal pane, agent
      session/thread, turn, fork, and subagent identities.
- [ ] Bind with explicit session IDs where available, plus adapter identity,
      process-start identity, working directory, and hosting evidence; PID
      alone is insufficient.
- [ ] Detect ambiguous matches and show an unresolved state or request an
      explicit selection; never silently choose the newest file or process.
- [ ] Track a binding generation and reject late events from old bindings.
- [ ] Report parent/fork relationships without assuming that parent totals
      include child totals.

### Synchronization and freshness

- [ ] Implement a source-controller lifecycle: discovering, binding,
      synchronized, degraded, disconnected, and stopped.
- [ ] Prefer live event/API streams, then documented hooks/status lines,
      structured files, local databases/logs, and finally process/project/Git
      fallbacks.
- [ ] Support event-driven updates with a bounded poll fallback and configurable
      cadence; use a short file-watch debounce window of roughly 25–100 ms.
- [ ] Maintain per-field or coherent field-group provenance: source, scope,
      observation time, receipt time, last validation, quality, and age.
- [ ] Use monotonic time for elapsed durations and freshness; use wall-clock
      time only for displayed timestamps and reset deadlines.
- [ ] Keep idle-but-healthy sessions from becoming stale solely because no turn
      event was emitted.
- [ ] Define source precedence per field and prevent an older durable read from
      overwriting a newer event.
- [ ] Handle duplicates, out-of-order events, sequence gaps, partial writes,
      truncation, rotation, replacement, watcher loss, database locks, and
      malformed records.
- [ ] Reconcile periodically even when event watchers appear healthy.
- [ ] Stage and publish session rollover atomically; clear old session-scoped
      state before publishing the new generation.
- [ ] Preserve independently scoped data only when its account/worktree/session
      key still matches.
- [ ] Bound queues, history, record size, nesting, parser time, retries, and
      subprocess output; resynchronize after overflow rather than silently
      dropping accounting events.
- [ ] Show live, cached, stale, disconnected, frozen, inferred, and estimated
      state explicitly, with source age available in inspection.

### Common agent and project state

- [ ] Current agent/CLI name, version, adapter, integration mode, and provider.
- [ ] Current model, model role, and reasoning effort/thinking level when
      reported; do not equate unlike providers' effort labels.
- [ ] Launch directory, current agent directory, project root, repository root,
      worktree, and folder display with configurable path redaction.
- [ ] Git branch, detached-head state, dirty state, worktree, file counts, and
      optional ahead/behind based on locally available references.
- [ ] Turn lifecycle: idle, thinking, working, waiting, completed, failed,
      interrupted, and disconnected.
- [ ] Waiting reason when known: approval, user input, tool completion, rate
      limit, or unknown.
- [ ] Turn start time, elapsed time, last response time, and output speed with a
      documented unit and measurement interval.
- [ ] Context used, remaining, total window, and percent with the measured
      scope; distinguish measured, estimated, unknown, and stale values.
- [ ] Session and turn token totals with input/output/cache categories,
      cumulative-versus-delta semantics, and compaction count where available.
- [ ] Permission, approval, sandbox, and collaboration-mode telemetry as
      descriptive state, never as an enforcement claim.

## Near-term — activity, usage, and adapter coverage

### Agent activity and collaboration

- [ ] Active tool names, stable call IDs, targets, duration, completion, and
      error state when observed; redact or omit arguments by default.
- [ ] Distinguish configured, loaded, and actively used skills, plugins, and
      MCP servers.
- [ ] Subagents with parent ID, model, status, and explicit aggregation rules.
- [ ] Current plan/todo item, completion count, goal, session title, and turn
      count where supported.
- [ ] Optional compact conversation summary only behind explicit content and
      retention settings.
- [ ] Visible anomalies for context overflow, contradictory measurements,
      dropped events, and unsupported activity claims.

### Usage, quotas, and cost

- [ ] Represent scoped usage limits as a list of provider/account/window
      records, not fixed primary/secondary slots.
- [ ] Percent used/remaining, reset countdown, absolute reset time, unit, and
      observation time where supported.
- [ ] Distinguish quota, credit balance, monetary cost, and estimated cost.
- [ ] Keep quota cache age and provider freshness visible; reaching reset time
      triggers reconciliation or uncertainty rather than fabricated refresh.
- [ ] External usage providers through explicit, bounded, opt-in commands or
      HTTP queries with validated destinations and safe credential handling.
- [ ] Provider-specific quota adapters without polluting normalized fields.
- [ ] Optional pricing provenance, currency, pricing date, and estimate labels.

### CLI adapters and support tiers

- [ ] Generic JSON snapshot and JSONL event adapter through a documented local
      pipe, file, or session-scoped ingress endpoint.
- [ ] Generic adapter validation command, example payloads, malformed-input
      handling, reconnect behavior, and conformance fixtures.
- [ ] Codex native-TUI observer adapter using isolated, version-sensitive
      rollout/state/log/process sources where verified.
- [ ] Codex app-server adapter for a HUD-controlled session, kept separate from
      observer mode and its additional PTY/approval responsibilities.
- [ ] Claude status-line/command adapter with correlation for short-lived hook
      processes and sparse payload updates.
- [ ] Gemini adapter profile, executable discovery, and documented fallback
      status before claiming richer structured telemetry.
- [ ] Aider adapter profile, executable discovery, and documented fallback
      status before claiming richer structured telemetry.
- [ ] Publish a compatibility matrix for CLI versions, integration modes,
      identity strength, fields, limitations, and recovery behavior.
- [ ] Adapter health and version-compatibility reporting with actionable errors.
- [ ] Ensure adding an adapter never requires changing the common renderer.

### Integration and usability

- [ ] Explicit `attach`, `run`, `inspect`, `doctor`, `adapters list`, and
      `config validate` commands with safe argument and signal handling.
- [ ] Wrapper commands such as `yagent` and `yodex` that preserve the child
      command's arguments, environment, working directory, signals, and exit
      status.
- [ ] Automatic adapter selection from explicit configuration, child process,
      environment, and session identity, with ambiguity reported.
- [ ] Shared session ID propagation through environment variables or a scoped
      local ingress mechanism.
- [ ] Keyboard actions for expanding, collapsing, freezing, and inspecting HUD
      details without intercepting keys intended for the agent.
- [ ] Multiple profiles for compact, full, presentation, monochrome, and
      privacy-focused layouts.
- [ ] Copyable redacted diagnostics and a local structured snapshot export.
- [ ] zsh, bash, and fish integration templates that are inspectable and
      reversible rather than silently rewriting startup files.

## Cross-cutting release gates

- [ ] Contract tests for every adapter covering schema versions, capabilities,
      units, omission/clear semantics, provenance, and unsupported fields.
- [ ] Deterministic reducer/replay tests with a controllable clock for
      freshness, ordering, reset countdowns, and suspend/resume.
- [ ] Fixtures for two concurrent sessions in one repository, rollover while
      the old source remains readable, file rotation, missed watchers,
      reconnect, process exit, malformed input, and queue overflow.
- [ ] Tmux and PTY integration tests for resize, focus, signal forwarding,
      terminal restoration, restart, cleanup, and narrow widths.
- [ ] Renderer tests for Unicode, monochrome, redaction, accessibility, and
      one- through ten-line layouts.
- [ ] Security fixtures for terminal escape injection, oversized records,
      secret-bearing paths, logs, environment data, and debug reports.
- [ ] Resource budgets for idle CPU, memory, queue size, source I/O, and render
      latency, measured on a documented reference system.
- [ ] Diagnostics for adapter/source/backend versions, binding evidence,
      generation, provenance, last reconciliation, retries, parse errors,
      dropped updates, source lag, render latency, and resource use.
- [ ] Structured, bounded, redacted logs with explicit local export and no
      automatic upload.
- [ ] Clean-environment install, upgrade, config migration, run, attach,
      generic demo, and uninstall verification on declared platforms.
- [ ] Versioned releases, checksums, license notices, compatibility notes, and
      a documented support policy.

## Later possibilities

- [ ] Native controlled-session frontends for additional CLIs.
- [ ] Shared local socket/state bus for multiple HUD consumers.
- [ ] Read-only multi-session overview and threshold notifications.
- [ ] Remote HUD display over SSH or a local dashboard.
- [ ] Historical turn, context, and usage charts.
- [ ] Public SDK or plugin system for third-party adapters, with an explicit
      user-privilege trust boundary.
- [ ] Cross-machine session handoff and reconnect.
- [ ] Structured export for desktop status bars and widgets.
- [ ] Stronger adapter isolation or a separately sandboxed helper process.

## Non-goals

- Do not scrape terminal pixels as the primary integration method.
- Do not fork or patch every agent CLI to render the HUD.
- Do not promise identical telemetry or feature parity across CLIs.
- Do not expose prompt or response contents by default.
- Do not claim that a cached, estimated, inferred, or stale value is live.
- Do not silently approve actions, modify agent state, or turn observation into
      a full agent frontend.
