## Purpose

Keep the HUD attached to the intended agent session with recoverable source selection, coherent rollover, bounded ingestion, and explicit freshness rather than accidental attachment or stale measurements.

## ADDED Requirements

### Requirement: Evidence-based identity binding

The HUD SHALL distinguish HUD instance, launch, child process, terminal pane, agent session/thread, turn, fork, and subagent identities. Binding MUST combine adapter identity with an explicit session ID when available and corroborating process-start, working-directory, launch, or hosting evidence. PID alone, file recency alone, and repository membership alone MUST NOT establish session identity. Identity strength and parent/fork evidence SHALL be inspectable.

#### Scenario: Reused process ID
- **WHEN** the original agent exits and an unrelated process reuses its PID
- **THEN** the HUD rejects the replacement process-start identity and does not transfer the old session state

#### Scenario: Process-only fallback
- **WHEN** a wrapper owns a child process whose CLI exposes no session ID
- **THEN** the HUD binds to the launch and process-start identity, labels identity strength as process-only, and leaves unsupported session/turn identity unknown

### Requirement: Ambiguity remains unresolved

When multiple candidates satisfy incomplete binding evidence, the HUD SHALL enter an unresolved discovery state and expose redacted candidates for explicit selection. It MUST NOT choose the newest file, newest process, or apparently most active session. Explicit selections MUST still be validated against their supplied identity evidence.

#### Scenario: Two sessions in one repository
- **WHEN** two Codex sessions share a repository and no explicit session, child, or pane discriminator is provided
- **THEN** neither session's telemetry is displayed as selected and the user receives selection guidance

### Requirement: Binding generations reject obsolete updates

Every binding establishment, detach/rebind, or identity-affecting selector change SHALL advance a generation token, including rebinding the same verified session. Updates and asynchronous completion results MUST identify their generation, and obsolete generations MUST be discarded before affecting state, accounting, source health, or the current view. Source epochs SHALL distinguish transport reconnects, rotations, and sequence restarts within an otherwise unchanged binding. The trusted controller SHALL stamp local receipt/validation times and binding generation only after validating a source's session-scoped lease; producer timestamps or claimed generations MUST NOT establish authority. A retired producer connection MUST require a new handshake rather than being relabeled with the current generation.

#### Scenario: Delayed prior-session read
- **WHEN** a file read from generation 4 completes after generation 5 has been published
- **THEN** its values and health result do not modify generation 5

#### Scenario: Same session is rebound
- **WHEN** the HUD detaches and rebinds the same session ID while an old callback and producer connection remain
- **THEN** the new binding has a fresh generation and neither the callback nor the retired connection can update it without validated renegotiation

#### Scenario: Producer claims a current receipt time
- **WHEN** a producer supplies a forged recent receipt time or a generation belonging to another binding
- **THEN** the claim cannot refresh values or select a binding, and accepted observations use trusted local timing and the validated connection lease

### Requirement: Source-controller lifecycle and selection hierarchy

The HUD SHALL expose discovering, binding, synchronized, degraded, disconnected, and stopped lifecycle states with causes. For each capability it MUST prefer verified live event/API streams, then documented hooks/status lines, structured files, local databases/logs, and process/project/Git fallbacks. Unsupported or unhealthy higher-ranked sources MUST permit a verified lower-ranked source with visible provenance. Observer attachment MUST NOT start a controlled frontend to obtain richer telemetry.

#### Scenario: Stream failure with durable fallback
- **WHEN** an active stream disconnects and a correctly bound structured file remains valid
- **THEN** the HUD enters degraded operation, identifies the fallback, and does not label the disconnected stream's old values live

#### Scenario: Permanent shutdown
- **WHEN** the user stops observation
- **THEN** the source controller enters stopped, cancels retries/watchers, and preserves the agent process

### Requirement: Ordered field arbitration and accounting recovery

Source precedence SHALL be declared per field or coherent field group. Duplicates and out-of-order records MUST NOT roll values backward or double-count usage; an older durable snapshot MUST NOT supersede newer event evidence. Sequence gaps and uncertain accounting MUST mark affected fields unavailable/stale and trigger reconciliation before claiming complete totals. Ordering MUST NOT depend on comparing untrusted clocks from unrelated sources.

#### Scenario: Delayed durable model snapshot
- **WHEN** a durable read began before a live model-change event and completes afterward
- **THEN** the newer model selection remains visible and inspection explains the rejected older evidence

#### Scenario: Missing usage event
- **WHEN** a source sequence skips an accounting record
- **THEN** the affected totals become uncertain, the HUD requests an authoritative snapshot or replay, and duplicate recovery records do not inflate totals

### Requirement: Event observation and periodic reconciliation

The HUD SHALL support event-driven ingestion, configurable bounded polling, a file-watch debounce between 25 and 100 milliseconds by default, and periodic identity/source reconciliation even while watchers appear healthy. Reconciliation MUST detect replacement sources while an old file remains readable. Polling, retries, and watcher reattachment MUST be bounded and visible in diagnostics.

#### Scenario: Missed filesystem notification
- **WHEN** a watcher misses the creation of the new rollout while the old rollout still exists
- **THEN** periodic reconciliation discovers the correctly correlated replacement and initiates rollover without requiring the old file to disappear

### Requirement: Monotonic freshness and elapsed time

The HUD SHALL calculate durations and source-validation age with monotonic time and reserve wall-clock time for displayed timestamps and reset deadlines. Successful field-specific validation of an unchanged healthy source MUST keep applicable stable facts live without inventing a new observation time. Process liveness alone MUST NOT revalidate model, context, quota, or turn completion. Failed validation MUST transition affected values through cached/stale freshness according to documented thresholds while connection health independently reports degradation/disconnection. Idle turns MUST NOT become stale solely because no turn event was emitted; suspend/resume MUST force revalidation before resumed data appears live.

#### Scenario: Healthy idle session
- **WHEN** an idle session emits no new turn events but process/source reconciliation continues to succeed
- **THEN** stable session facts remain live and their original observation time remains available

#### Scenario: Clock change and suspend
- **WHEN** wall time jumps backward and the host later resumes after suspension
- **THEN** elapsed durations do not become negative and affected sources are reconciled before their cached data regains a live label

#### Scenario: Alive process with silent unsupported stream
- **WHEN** process validation succeeds but no supported heartbeat, contiguous catch-up, or authoritative snapshot validates turn and model fields
- **THEN** process health remains known while those fields retain their own validation ages and cannot be renewed solely by process liveness

### Requirement: Atomic rollover with scoped retention

The HUD SHALL stage a verified replacement binding before publishing it. The first new-generation revision MUST clear previous session/turn/model/context/activity/accounting state; independently scoped worktree/account values MUST be preserved only when their verified keys still match and their ages remain visible. Parent/fork relationship changes MUST NOT imply inheritance of context or token totals.

#### Scenario: Rollover in the same worktree and account
- **WHEN** a verified new root session replaces the old session in the same worktree and account
- **THEN** one atomic revision exposes the new session with old session fields cleared and only still-keyed worktree/account data retained with provenance

#### Scenario: Account changes during rollover
- **WHEN** the replacement session uses a different or unknown account key
- **THEN** the old account's quota and balance records are removed from the new generation

### Requirement: Recoverable local source failures

The HUD SHALL handle partial records, malformed records, truncation, rotation, replacement, watcher loss, database locks, process exit, and reconnect without crashing or silently switching identity. Complete valid records preceding a partial write MUST remain usable. Recovery MUST revalidate identity and obtain a consistent baseline before marking recovered accounting complete.

#### Scenario: Partial JSONL write and rotation
- **WHEN** the current file ends mid-record and is subsequently rotated
- **THEN** the partial record is not applied, prior valid data is labeled according to age, and the replacement file is validated under a new source epoch

#### Scenario: Locked database
- **WHEN** a local state database is temporarily locked
- **THEN** the HUD uses bounded retries or verified fallback, reports degradation, and neither writes the database nor loses the selected session identity

### Requirement: Bounded ingestion and overflow resynchronization

The HUD SHALL enforce published limits for queue size, record bytes, nesting depth, parser work, retained history, retries, subprocess output, replay staging, and total recovery work. Queue overflow or a rejected accounting-bearing record MUST invalidate affected totals and trigger resynchronization rather than silently dropping events while claiming accuracy. Oversized or deeply nested input MUST produce bounded redacted errors. Large baselines SHALL be replayed in bounded cancellable slices with progress and fair access for other sources. Partial replay MUST NOT publish complete totals; exceeding the total recovery ceiling SHALL leave affected fields uncertain and report a checkpoint/selection remedy instead of restarting unlimited scans.

#### Scenario: Event queue overflow
- **WHEN** incoming accounting events exceed the configured queue limit
- **THEN** memory remains bounded, dropped-update diagnostics increase, and affected totals stay uncertain until an authoritative recovery succeeds

#### Scenario: Oversized hostile record
- **WHEN** a source sends a record above the documented byte or nesting limit
- **THEN** the record is rejected within the parser budget and other bounded sources and presentation remain responsive

#### Scenario: Baseline exceeds a replay slice
- **WHEN** an authoritative log needs several slices to reach its validated checkpoint
- **THEN** replay progress is inspectable, unrelated sources remain responsive, and totals become complete only after the coherent baseline and contiguous buffered events are reconciled

#### Scenario: Recovery cannot catch up within its ceiling
- **WHEN** recovery exceeds its published total byte or time ceiling
- **THEN** the source remains degraded with uncertain totals and actionable checkpoint guidance, and retries do not silently restart the same unbounded scan
