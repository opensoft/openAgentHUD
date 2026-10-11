## Purpose

Bind observations to the correct running agent and maintain honest, recoverable state across source failure, rollover, and concurrency.

## ADDED Requirements

### Requirement: Strong and separate identities
The HUD SHALL distinguish HUD instance, launch, process-start identity, pane, agent session/thread, turn, fork, and subagent IDs. Binding SHALL use explicit session identity when available and corroborating adapter, process-start, cwd, and hosting evidence. PID or newest-file time alone MUST NOT establish identity. Parent/fork relationships SHALL remain explicit.

#### Scenario: Concurrent sessions in one repository
- **WHEN** two live sessions share a repository and neither is uniquely identified by the available evidence
- **THEN** binding SHALL remain unresolved with redacted candidate evidence and an explicit selection path.

#### Scenario: Reused process ID
- **WHEN** a process ID is reused with a different process-start identity
- **THEN** the HUD MUST NOT transfer the previous session binding to the replacement process.

### Requirement: Source-controller lifecycle
The source controller SHALL expose discovering, binding, synchronized, degraded, disconnected, and stopped states with reasons. It SHALL prefer supported live events, documented hooks, structured files, version-sensitive local databases/logs, then process/project/Git fallbacks on a per-field basis. Loss of one source MUST NOT imply loss of all independently valid fields.

#### Scenario: Primary stream fails
- **WHEN** a stream disconnects while a verified durable source remains readable
- **THEN** the controller SHALL report degradation, reconcile that source, and retain each field's actual provenance and freshness.

### Requirement: Generational isolation and atomic rollover
Every binding SHALL carry a generation, advanced on detach/rebind even to the same session. All asynchronous work SHALL retain its captured binding generation and source epoch. Retired-generation or retired-epoch results MUST be rejected before changing values, accounting, validation leases, or active-source health/counters; a separate bounded rejected-result diagnostic MAY count the rejection. A newly verified session SHALL publish identity and cleared or reloaded session/turn state in one atomic revision. Account and worktree data SHALL survive only when their verified scope keys still match; unknown keys MUST NOT compare equal for reuse.

#### Scenario: Old source remains readable
- **WHEN** a verified replacement session becomes active while the old rollout remains readable
- **THEN** the HUD SHALL switch to the replacement generation atomically and MUST NOT show its identity with the previous session's model, turn, context, or token totals.

#### Scenario: Late callback
- **WHEN** a watcher callback from a retired generation arrives after rollover
- **THEN** it SHALL have no effect on current state.

#### Scenario: Late validation from the same session's retired source
- **WHEN** a database job captured source epoch 2 and finishes after epoch 3 is active in the same binding generation
- **THEN** its values and health result MUST NOT renew epoch 3 validation or change its accounting or active-source counters.

### Requirement: Ordering and field authority
Sources SHALL supply or derive ordering within an explicit source epoch. Duplicate or older observations SHALL be ignored; sequence gaps SHALL trigger invalidation and reconciliation of affected accounting. Cross-source precedence SHALL be specified per field and MUST prevent an older durable observation from overwriting a newer authoritative event. Starting or completing a read later MUST NOT alone prove its content newer; replacement SHALL require compatible coverage/watermark evidence or an explicit reconciled authority transition. Incomparable contradictory evidence SHALL remain diagnosable rather than being ordered by provider wall clocks.

#### Scenario: Delayed durable model record
- **WHEN** an old rollout model record arrives after a newer verified model-selection event
- **THEN** the newer model SHALL remain displayed and the rejected update SHALL be diagnosable.

#### Scenario: Later read contains old data
- **WHEN** a durable read begins after a model-change event but cannot prove it includes that event
- **THEN** its older model MUST NOT overwrite the selected event merely because the read completed later.

#### Scenario: Missing accounting event
- **WHEN** an accounting stream jumps from sequence 20 to sequence 23
- **THEN** the affected totals SHALL become uncertain until reconciled from an authoritative snapshot.

### Requirement: Time and freshness semantics
Elapsed durations and freshness SHALL use a monotonic clock; displayed timestamps and provider reset deadlines SHALL use wall time. Live, cached, stale, disconnected, frozen, inferred, and estimated conditions SHALL be inspectable and visibly distinguishable where displayed. Successful identity/liveness validation SHALL keep unchanged idle session metadata current; it MUST NOT falsely validate event-dependent measurements. Suspend/resume SHALL trigger reconciliation before previously live state is reaffirmed.

#### Scenario: Healthy idle session
- **WHEN** no turn event occurs but the bound process and stable metadata are successfully revalidated
- **THEN** stable metadata SHALL remain live while event-dependent measurements SHALL retain their own validation ages.

#### Scenario: Wall-clock adjustment
- **WHEN** the wall clock jumps backward during a turn
- **THEN** elapsed duration SHALL remain nondecreasing and reset countdowns SHALL be recalculated from the provider deadline.

#### Scenario: Suspension with no wall-clock divergence
- **WHEN** the host resumes after a long scheduling gap while both clocks have advanced together
- **THEN** source confidence SHALL require reconciliation before the HUD reaffirms previously live values.

### Requirement: Event-driven updates and periodic reconciliation
File changes SHALL be debounced within configurable bounds of 25–100 ms. The controller SHALL provide bounded polling when watches or events are unavailable and SHALL reconcile identity and active source periodically even when watchers appear healthy. Cadences and last reconciliation SHALL be inspectable.

#### Scenario: Missed watcher event
- **WHEN** a watcher misses a source replacement
- **THEN** the next scheduled reconciliation SHALL detect it and recover without requiring a HUD restart.

### Requirement: Recoverable partial and replaced sources
Ingestion SHALL handle incomplete records, truncation, rotation, atomic file replacement, malformed records, watcher loss, database locks, and source disappearance without crashing the HUD or publishing partial state. Offsets SHALL be associated with file identity and source epoch. Malformed accounting records SHALL invalidate affected totals until resynchronization.

#### Scenario: Partial JSONL append
- **WHEN** the last JSONL record is incomplete
- **THEN** the reader SHALL hold it within its bounded buffer and publish nothing from that record until it is complete and valid.

#### Scenario: Database lock and file rotation
- **WHEN** a database is locked while a rollout rotates
- **THEN** the HUD SHALL retry within configured bounds, report degraded evidence, and resume reading the replacement file without mixing epochs.

### Requirement: Bounded recovery and ingestion
Queue length, history, record bytes, nesting depth, parsing work, retries, subprocess output, and retained source handles SHALL have finite configured limits. Overflow MUST be visible, invalidate affected event-derived state, and initiate snapshot reconciliation rather than silently dropping accounting updates. Recovery SHALL use capped backoff and remain cancellable.

#### Scenario: Producer exceeds consumer capacity
- **WHEN** the event queue reaches its limit
- **THEN** memory SHALL remain bounded, the drop counter SHALL increase, affected totals SHALL become uncertain, and recovery SHALL request or reread a snapshot.

### Requirement: Process exit and reconnect
Confirmed process exit SHALL mark the connection disconnected and preserve only explicitly labeled historical last-known values. Reconnect to the same verified session SHALL reconcile before returning to synchronized; reconnect to a different session SHALL use a new binding generation.

#### Scenario: Agent exits mid-turn
- **WHEN** the bound process exits without a final turn event
- **THEN** the HUD SHALL display disconnection and MUST NOT infer successful turn completion.
