# Agent-independent HUD platform

Author note: This is an independent alternative authored proposal using the requested author profile `gpt-6.1-sol` with `max` reasoning. It is grounded in the repository documents and now incorporates selected ideas from the locally reviewed Astra alternative; [peer-review.md](peer-review.md) records the evidence and remaining disagreements.

## Why

Coding-agent CLIs expose different, uneven telemetry, so a shared HUD must establish trustworthy identity, field semantics, synchronization, and privacy before displaying useful status. The expanded `FEATURES.md` roadmap now provides the product contract for turning this documentation-only repository into an agent-independent terminal HUD without implying telemetry parity across providers.

## What Changes

- Establish separate adapter, state-reduction, presentation, terminal-backend, and CLI modules with a versioned normalized contract, immutable revisions, explicit clears, keyed collection patches, negotiated additive compatibility, capability handshakes, and namespaced extensions.
- Add evidence-based session binding, controller-stamped ingress metadata, fresh generations on detach/rebind, a source-controller lifecycle, field-level source arbitration, periodic reconciliation, atomic rollover, bounded multi-slice recovery, and honest freshness and uncertainty indicators.
- Define common model, project/Git, turn, context, token, permission, tool, skill/plugin/MCP, subagent, plan, and optional content telemetry with explicit units, scopes, and aggregation rules.
- Represent usage limits as scoped records; distinguish quota, credits, measured cost, and estimates; support explicitly enabled external usage queries with bounded execution and credential-safe diagnostics.
- Provide a generic JSON/JSONL contract, Codex observer and separately enabled controlled app-server integrations with a minimal built-in input/approval client, Claude status-line ingestion, and Gemini/Aider discovery and fallback profiles. Reject stale decisions and unsafe request replay; publish support levels backed by real fixtures and smoke checks.
- Provide configurable one-to-ten-line presentation and standalone/tmux backends, including focus preservation, owned-resource cleanup, Unicode widths, accessible modes, terminal sanitizing, and explicit freeze/inspection behavior.
- Add typed configuration, atomic reload with staged rebinding, privacy controls applied to every output sink and pending export, explicit CLI and wrapper workflows, scoped local ingress, a second-terminal attach workflow, redacted inspection/export, and reversible shell templates. External usage providers require explicit opt-in, request-scoped credentials, and validated/pinned destinations.
- Require adapter contracts, deterministic replay, terminal integration and security fixtures, measured aggregate resource/source-I/O budgets at idle and 100 events/s, clean-environment lifecycle checks, and versioned release/support documentation before release.

The implementation sequence delivers Foundation and MVP first, then all Near-term sections, with the cross-cutting gates applied throughout. The completed change includes both stages; reaching MVP alone does not complete its tasks. Later roadmap items remain deferred: additional controlled frontends, a shared multi-consumer bus, multi-session overview/notifications, remote display, historical charts, public adapter SDK, cross-machine handoff, desktop output, and stronger process isolation. This change creates planning artifacts only and authorizes no implementation or publication.

## Capabilities

### New Capabilities

- `normalized-agent-state`: Versioned adapter handshake, normalized values and dependency metadata, scoped immutable revisions, and covered snapshot/keyed patch/invalidation semantics.
- `session-synchronization`: Binding evidence and ingress authority, source lifecycle, ordering, arbitration, freshness, bounded replay/recovery, and atomic session rollover.
- `agent-telemetry`: Common agent/project/model/turn/context/accounting/activity state with verified meanings and provider-specific limits.
- `usage-limits`: Scoped quota, balance and cost records, reset handling, freshness, and opt-in external usage providers.
- `adapter-integrations`: Generic transport and conformance tooling, Codex observation and explicit minimal controlled client, Claude hooks, fallback profiles, and compatibility reporting.
- `terminal-presentation`: Layouts, priority/truncation, themes, accessibility, safe Unicode rendering, standalone/tmux ownership, and interaction.
- `configuration-privacy`: Versioned configuration, precedence and reload, field privacy, bounded redacted retention, and observation-only defaults.
- `hud-cli`: Attach/run/inspect/doctor/config/adapter commands, selection, wrappers, signals, ingress, exports, and shell integration.
- `platform-verification`: Contract/replay/security/terminal suites, performance limits, diagnostic evidence, supported installation lifecycle, and release gates.

### Modified Capabilities

None. The canonical `openspec/specs/` inventory contains no existing capabilities.

## Impact

Future implementation introduces a TypeScript CLI on a chosen Node.js 24+ runtime floor, JSON state/configuration contracts, internal adapters, renderer and backends, local fixtures, tests, packaging, and documentation. It interacts with local agent processes, structured telemetry files, version-sensitive read-only Codex databases/logs, Git, terminal/PTY resources, and explicitly selected tmux resources. Only enabled external usage providers make network requests; credentials remain in agent/provider stores.

Existing roadmap and architecture documents remain canonical and unchanged. All artifacts for this alternative live in `openspec/changes/agent-hud-platform-sol/`; the other change and implementation files are outside this planning edit scope. No existing runtime interface is broken because no implementation currently exists.
