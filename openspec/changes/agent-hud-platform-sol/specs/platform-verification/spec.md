## Purpose

Gate releases on reproducible contract, recovery, terminal, security, resource, and installation evidence so supported HUD behavior is trustworthy across its declared adapters and platforms.

## ADDED Requirements

### Requirement: Adapter contract and support evidence

Every shipped adapter/profile SHALL pass a common conformance suite covering schema versions and mandatory features, handshake/capabilities, units/scopes, unknowns, coverage/keyed collection/clear semantics, dependency provenance, privacy, and unsupported fields. Structured/hook compatibility claims MUST cite versioned sanitized real fixtures and reproducible local CLI smoke checks and match the published support matrix. The required Codex observer/controlled and Claude hook modes MUST each obtain at least one verified CLI-version row before full change completion; absent evidence MUST remain a blocker. Adding a fixture adapter SHALL verify that common rendering remains unchanged. Every requirement/scenario SHALL map to a passing test or reproducible explicit smoke procedure and its roadmap item.

#### Scenario: Profile advertises unverified quota
- **WHEN** a profile advertises quota support without a conforming verified fixture/check
- **THEN** the release check rejects that support claim rather than treating generic schema acceptance as telemetry evidence

### Requirement: Deterministic replay and recovery suite

Release verification SHALL use a controllable monotonic/wall clock and deterministic replay for source ordering, duplication, gaps, accounting, freshness, reset deadlines, and suspend/resume. Required fixtures SHALL include two concurrent sessions in one repository, same-session detach/rebind, selector reload, rollover with readable old sources, partial writes, truncation/rotation, missed watchers, locked databases, reconnect, process exit/PID reuse, malformed input, queue overflow, and multi-slice/ceiling-limited recovery. Controlled tests SHALL include stale/reused approval IDs and unknown action-delivery outcomes across reconnect. Identical input and clock traces MUST produce identical revisions and uncertainty outcomes.

#### Scenario: Replay a rollover trace twice
- **WHEN** the same rollover/late-event trace is replayed twice with the same controlled clock
- **THEN** both runs yield the same generations, cleared session fields, retained keyed facts, and rejected-event diagnostics

### Requirement: Terminal and presentation integration suite

Release verification SHALL include tmux and PTY tests for resize, focus, argument/signal forwarding, child survival after HUD loss, terminal restoration, server restart, ownership-limited cleanup, and narrow widths. Renderer checks SHALL cover Unicode/grapheme width, monochrome/high-contrast/ASCII/reduced-motion modes, `TERM=dumb`, privacy, frozen views, and every height from one through ten lines.

#### Scenario: Unowned pane after restart
- **WHEN** integration tests simulate a tmux restart with an unowned replacement pane
- **THEN** no unowned resource is deleted and the restart/focus/reattachment outcome is asserted

### Requirement: Security and privacy fixtures

Release verification SHALL include terminal escape injection, oversized/deeply nested records, secret-bearing paths, source errors, rotated logs, environment data, owner/symlink misuse, forged ingress timing/generation, usage-command/HTTP limits, DNS rebinding and mixed address families, exports racing privacy changes, caches, and debug reports. Fixtures MUST verify that forbidden control sequences and configured secret/content sentinels are absent from every output and retained sink, including after privacy reload and a frozen view. Tests MUST require no real provider credentials or default network access.

#### Scenario: Restrict privacy after retaining a fixture
- **WHEN** a test enables bounded retention and then disables sensitive fields while the view is frozen
- **THEN** all HUD-owned sinks and retained snapshots omit the sensitive sentinels and the agent's source files remain unchanged

### Requirement: Published measurable resource limits

Default operation SHALL bound ingestion to 1024 queued records and 8 MiB queued bytes globally, 256 KiB per telemetry record, and 32 nesting levels; replay staging, total recovery, retry/subprocess/parser work SHALL also have published limits. On the documented reference system with one idle session, after five minutes warmup, average aggregate HUD CPU SHALL be at most 1% of one logical core over ten minutes, aggregate peak host/view/worker RSS at most 128 MiB, and idle source reads/probes at most 16 KiB/s and four probes/s averaged over that interval. Active benchmarks SHALL use 100 events/s with 1 KiB records and require trusted local notification/ingress-to-frame latency at most 250 milliseconds at the 95th percentile, including local queueing/debounce. Discovery/recovery of a stable reference source with a baseline at most 8 MiB SHALL converge within two configured reconciliation periods; larger or unrecoverable sources SHALL honor recovery ceilings and remain visibly uncertain. Stress runs MUST verify responsive input, accounting resynchronization, and absence of steady-state full-log rescans. Workloads, cadence, hardware/versions, all HUD-owned process costs, and excluded provider/agent transport time SHALL be reported. Thresholds MUST be fixed before measurement and MUST NOT be silently relaxed.

#### Scenario: Burst above default queue limits
- **WHEN** the benchmark injects a burst above either queue bound
- **THEN** queued records and bytes stay within limits, affected accounting becomes uncertain, and recovery restores verified totals without blocking agent input

#### Scenario: Idle reference benchmark
- **WHEN** a supported build observes one healthy idle fixture on the declared reference hardware/runtime
- **THEN** measured CPU, memory, latency, and I/O are included in the release evidence and satisfy the published budgets

### Requirement: Bounded local operational diagnostics

The HUD SHALL expose structured bounded redacted local diagnostics/logs for adapter/source/backend versions, identity evidence, binding generation, provenance, last reconciliation, retry/parse/drop counts, source lag, render latency, and resource use. Diagnostic retention MUST respect privacy and size/age limits, and export MUST require a local explicit action without automatic upload.

#### Scenario: Repeated parser failures
- **WHEN** a source repeatedly emits malformed records
- **THEN** counters reflect the failures while stored diagnostic detail remains bounded and never retains raw failed payloads

### Requirement: Declared platform installation and release lifecycle

The first supported release SHALL declare Linux x86_64 and arm64, Node.js 24+ as the chosen runtime floor, and tmux 3.3+ for the optional tmux backend; other platforms SHALL be explicitly unverified. Clean-environment checks SHALL cover install, upgrade, configuration migration/rollback, generic offline demo, run, attach, diagnostics, and uninstall without provider credentials or implicit startup-file changes. Versioned releases SHALL include checksums, Apache-2.0 notices and dependency license notices, compatibility notes, and a support/deprecation policy. Publication SHALL remain a separate explicit release action.

#### Scenario: Offline clean install and uninstall
- **WHEN** a prepared release package is installed in a clean supported environment and used for the generic demo, run, and attach checks
- **THEN** all documented workflows work without provider credentials/network queries, upgrade preserves a recoverable configuration, and uninstall removes only installed HUD-owned files
