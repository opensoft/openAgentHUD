## Purpose

Gate releases on reproducible correctness, privacy, terminal safety, resource use, and clean installation evidence for declared support tiers.

## ADDED Requirements

### Requirement: Adapter contract and deterministic replay gate
Every enabled adapter/mode SHALL pass contract fixtures covering schema versions, capabilities, units, scope, null/omission semantics, provenance, and unsupported fields. A deterministic clock/replay suite SHALL cover ordering, idle freshness, elapsed time, suspend/resume, quota reset uncertainty, and repeatable revision output.

Fixture manifests SHALL distinguish synthetic contract traces from sanitized real provider samples and record CLI/source versions and provenance. Generic schema success or a synthetic server/client exchange MUST NOT establish structured provider compatibility. Capability withdrawal, patch-path rejection, and checkpoint coverage SHALL have explicit conformance cases.

#### Scenario: Adapter violates null semantics
- **WHEN** an adapter fixture clears a field but its result retains the previous value
- **THEN** the release gate SHALL fail for that adapter and it MUST NOT be published as conformant.

### Requirement: Concurrency and recovery fixture gate
Release fixtures SHALL cover two concurrent sessions in one repository, ambiguous discovery, PID reuse, rollover with a readable old source, out-of-order/duplicate events, gaps, partial writes, rotation, truncation, watcher loss, database locks, reconnect, process exit, malformed input, and queue overflow. Each fixture SHALL assert state, trust metadata, and bounded recovery.

Fixtures SHALL also cover same-generation retired-epoch callbacks, old hooks reconnecting after rollover, snapshot/delta overlap after deduplication eviction, later-started stale reads, and suspend with clocks advancing together.

#### Scenario: Queue overflow loses accounting
- **WHEN** a replay exceeds queue capacity
- **THEN** the gate SHALL fail unless memory stays bounded, loss is diagnosed, totals become uncertain, and resynchronization restores trustworthy state.

### Requirement: Terminal and accessibility gate
PTY and tmux integration SHALL verify resize, focus, literal arguments, signals, child exit status, terminal restoration, restart, ownership-limited cleanup, and agent survival after HUD loss. Rendering SHALL cover one through ten lines, narrow widths, Unicode graphemes, monochrome, high contrast, ASCII, reduced motion, redaction, and TERM=dumb.

#### Scenario: Cleanup affects an unrelated pane
- **WHEN** a tmux restart fixture reuses a HUD pane ID for an unrelated pane
- **THEN** the gate SHALL fail if HUD cleanup modifies that pane.

### Requirement: Security and privacy gate
Security fixtures SHALL test terminal escape injection, oversized/deep records, secret-bearing paths, environment values, parser errors, logs, exports, frozen frames, retained caches, hostile ingress ownership, external-provider destinations, and command argument injection. All sink outputs SHALL be checked against the effective privacy policy.

#### Scenario: Debug output exposes a token
- **WHEN** a secret-bearing fixture is processed with debug logging enabled
- **THEN** any token disclosure in a HUD-owned output SHALL fail the release gate.

### Requirement: Measured resource budgets
Each release SHALL report a reproducible reference system and benchmark workload for idle CPU, peak memory, queue capacity, source I/O, reconciliation latency, and render latency. Numeric thresholds SHALL be fixed before the gate runs and MUST NOT be silently relaxed to accept regressions. Overflow and overload SHALL preserve the same boundedness and trust semantics as normal operation.

CPU and memory measurements SHALL aggregate all HUD-owned supervisors, workers, clients, and views, excluding only agent/provider processes. The reference active trace SHALL declare record byte sizes, field coverage, and update frequency; latency SHALL be measured from actual events rather than inferred from idle runs. Large-source recovery SHALL declare a byte/event envelope and report progress beyond it instead of promising size-independent convergence. Native OS/architecture evidence SHALL support platform performance claims; emulation alone MUST NOT establish those claims.

#### Scenario: Idle resource regression
- **WHEN** an unchanged verified idle session exceeds the declared CPU or I/O budget
- **THEN** the resource gate SHALL fail with measured evidence and the responsible workload.

### Requirement: Bounded actionable diagnostics
Diagnostics SHALL expose adapter/source/backend versions, redacted binding evidence, generation, field provenance, last reconciliation, retries, parse errors, dropped updates, source lag, render latency, and resource use. Structured logs SHALL be local, bounded, redacted, and explicitly exportable with no automatic upload.

#### Scenario: Persistent malformed input
- **WHEN** a source repeatedly sends malformed records
- **THEN** counters SHALL remain accurate while repeated log entries are rate-limited and storage remains within its configured bound.

### Requirement: Clean installation and migration gate
Initial supported targets SHALL be Linux x86_64 and macOS arm64 on the declared Node.js 24 baseline with optional tmux; other OS/architecture pairs SHALL be labeled unverified. Each declared target SHALL verify clean-environment install, upgrade, explicit configuration migration and rollback, run, attach, generic demo, and uninstall. Uninstall and rollback SHALL remove only owned inactive installation/runtime artifacts selected for removal and MUST NOT remove agent data, user exports, or the lifetime owner/endpoint of a still-running child.

#### Scenario: Upgrade has a newer configuration schema
- **WHEN** an older executable encounters a configuration version it cannot read
- **THEN** it SHALL fail with a version error and preserve the file so that rollback or explicit migration remains possible.

#### Scenario: Uninstall while a launched agent is active
- **WHEN** a user uninstalls the HUD while a run supervisor still owns an active child
- **THEN** the agent's I/O and lifetime owner SHALL remain usable until child exit, and only that owner's matching expired resources SHALL be cleaned afterward.

### Requirement: Versioned release and support evidence
Releases SHALL include semantic version, checksums, license notices, configuration/ingress schema compatibility, CLI/mode support matrix, known limitations, and a documented support policy. Claims of structured provider support SHALL require a verified local fixture and integration test; unsupported environments SHALL be labeled rather than implied supported.

Completing this change SHALL require real local structured Codex observer, Codex controlled, and Claude evidence with their specified smoke checks; advertising only fallbacks MUST NOT satisfy those planned modes. Package preparation and verification SHALL NOT authorize publication.

#### Scenario: Unverified provider version
- **WHEN** no reproducible fixture or smoke test exists for an installed provider version
- **THEN** release documentation and adapter health SHALL identify it as unverified instead of claiming structured compatibility.

#### Scenario: Controlled mode has only a synthetic client test
- **WHEN** relay/approval fixtures pass but no usable control surface has completed a verified local server smoke check
- **THEN** controlled support and the full-change completion gate SHALL remain unsatisfied.
