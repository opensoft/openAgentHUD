## Why

openAgentHUD currently has a roadmap and architecture but no implementation or behavioral contracts. A shared HUD needs explicit identity, consistency, freshness, and privacy guarantees before integrations with different agent CLIs can produce trustworthy displays.

## What Changes

- Define an agent-independent platform whose adapters publish versioned observations, whose reducer produces immutable revisions, and whose renderer consumes only normalized presentation data.
- Deliver the complete Foundation and MVP scope from `FEATURES.md`, followed by its Near-term capabilities in separately gated implementation stages within this change. The final change is complete only when all stages and release gates pass; an MVP milestone does not complete the change.
- Establish strong session binding, session-bound producer leases, bounded source reconciliation, atomic rollover and capability withdrawal, honest unknown/stale state, and checkpointed cumulative/delta accounting.
- Add accessible one- through ten-line layouts, standalone and tmux backends, targeted view controls, atomic configuration with explicit profile precedence, consistent privacy controls including frozen views, and read-only observation by default.
- Define generic JSON/JSONL integration, Codex observer and separately opted-in controlled integration, Claude hooks, and honest Gemini/Aider fallback profiles; publish evidence-based support tiers.
- Add activity, scoped quotas and cost, wrappers that preserve existing hook behavior, machine-readable diagnostics, installation verification, and aggregate resource/security release gates. Defer all roadmap “Later possibilities,” including public plugins, shared multi-consumer state buses, remote display, and historical charts.

This is an **alternative authored proposal** for independent review alongside the other proposal. It does not amend or supersede that change, `FEATURES.md`, or `docs/architecture.md`. No product code is implemented by this planning work.

The peer-review revision adopts selected Sol contract, synchronization, privacy, workflow, and verification ideas; [peer-review.md](peer-review.md) records the evidence, limitations, and choices. The nine Astra capability names remain unchanged.

Author note: This proposal and its peer-review revision were produced in the requested **gpt-6-astra with xhigh reasoning** sessions. The surrounding orchestration records the requested model/profile; this note makes no claim about provider-internal model identity beyond that invocation.

## Capabilities

### New Capabilities

- `hud-state-contract`: Versioned adapter handshake, observations, immutable revisions, field semantics, availability, and scoped accounting.
- `hud-session-synchronization`: Identity, binding generations, source lifecycle, ordering, freshness, bounded ingestion, and rollover.
- `hud-common-telemetry`: Agent, model, project/Git, turns, context, token totals, and descriptive permission state.
- `hud-presentation-backends`: Layouts, accessibility, safe terminal rendering, standalone output, tmux ownership, and inspection controls.
- `hud-configuration-privacy`: Typed configuration, reload and precedence, content consent, redaction, retention, and observation-only defaults.
- `hud-activity-usage`: Correlated activity, collaboration, anomalies, scoped limits, reset uncertainty, optional usage providers, and cost provenance.
- `hud-adapter-integrations`: Generic ingress, Codex integration modes, Claude hooks, Gemini/Aider fallbacks, and support-tier contracts.
- `hud-cli-workflows`: Attach/run commands, selection, wrappers, signals, scoped identity propagation, local inspection, and shell templates.
- `hud-release-assurance`: Replay/conformance, security and terminal tests, diagnostics, measured budgets, packaging, migrations, and support policy.

### Modified Capabilities

None. The main specification inventory is empty.

## Impact

Future implementation will introduce a local executable, adapter and normalized-state contracts, configuration and ingress schemas, provider fixtures, terminal backends, tests, and release documentation. The design selects TypeScript/Node.js 24 and initial Linux x86_64/macOS arm64 targets with optional tmux; other OS/architecture combinations are unverified. Runtime dependencies and source formats must be pinned and locally verified during implementation. Agent CLI source trees and credential stores remain owned by those agents. External usage requests and controlled sessions require explicit configuration or invocation. There is no existing executable or public configuration contract to break.

This change writes planning artifacts only inside `openspec/changes/agent-hud-platform-astra/`. Future source paths in the design and tasks are proposed implementation locations, not files created by this proposal.
