## Purpose

Define comparable agent and project fields without implying that different providers expose equivalent measurements or permission guarantees.

## ADDED Requirements

### Requirement: Agent and model identity
The HUD SHALL expose agent/CLI name and version, adapter, integration mode, provider, model name, model role, and reasoning effort when reported. Provider-specific effort labels MUST retain their original meaning without conversion to a fabricated common scale.

#### Scenario: Unsupported effort field
- **WHEN** an adapter reports a model but no effort information
- **THEN** the model SHALL be displayed and effort SHALL remain unknown rather than being inferred from the model name.

### Requirement: Distinct project locations
The HUD SHALL distinguish launch directory, current agent directory, project root, repository root, and worktree. Displayed paths and folder names SHALL follow privacy settings; a cwd change SHALL invalidate or refresh dependent Git state before a coherent project revision is published.

#### Scenario: Agent changes repository
- **WHEN** verified cwd changes to a different repository
- **THEN** the next project revision SHALL use the new scope and MUST NOT display the old branch as belonging to it.

### Requirement: Local Git state
Git telemetry SHALL distinguish branch, detached HEAD, dirty status, worktree, and file counts. Optional ahead/behind SHALL use locally available references without fetching. Missing repositories, inaccessible indexes, and unavailable refs SHALL produce explicit unknown or not-applicable values.

#### Scenario: Detached checkout without tracking reference
- **WHEN** the worktree is detached and no usable upstream reference exists
- **THEN** detached state SHALL be visible, ahead/behind SHALL remain unavailable, and the HUD MUST NOT perform a network fetch.

### Requirement: Turn lifecycle and waiting reasons
The turn model SHALL distinguish idle, thinking, working, waiting, completed, failed, interrupted, and disconnected. A known waiting reason SHALL identify approval, user input, tool completion, or rate limit; otherwise it SHALL be unknown. Completion MUST require evidence and MUST NOT be inferred from inactivity alone.

#### Scenario: Waiting for approval
- **WHEN** verified telemetry reports an approval request
- **THEN** the turn SHALL display waiting with approval as the reason and displaying that state MUST NOT approve the request.

### Requirement: Timing and output rate
The HUD SHALL expose turn start, monotonic elapsed duration, last response timestamp, and output speed when measurable. Attaching mid-turn without a verified elapsed-time basis SHALL expose unknown or explicitly estimated duration rather than exact elapsed time. Output speed SHALL identify its unit and observation interval; tokens and characters MUST NOT be treated as interchangeable. A final token total without timed samples MUST NOT imply a live generation rate.

#### Scenario: Only final usage is available
- **WHEN** a provider reports a turn token total after completion but no timed output observations
- **THEN** live output speed SHALL remain unknown; any explicitly derived whole-turn average SHALL be separately labeled with its interval.

### Requirement: Context scope and consistency
Context measurements SHALL include used, remaining, total window, percent, scope, and reported or estimated quality where available. Derived remaining and percent SHALL require compatible scope and a known positive capacity. Session cumulative token totals MUST NOT substitute for current context occupancy. Overflow or contradictory measurements SHALL remain visible as anomalies and MUST NOT be silently clamped into a healthy range.

#### Scenario: Context exceeds reported capacity
- **WHEN** used context is 110000 tokens and capacity is 100000 for the same scope
- **THEN** the HUD SHALL expose the overflow and a derived 110 percent with an anomaly rather than displaying 100 percent as a valid measurement.

#### Scenario: Capacity is unknown
- **WHEN** used context is known but capacity is absent
- **THEN** used tokens SHALL remain available while remaining and percent SHALL be unknown.

#### Scenario: Only cumulative session tokens are available
- **WHEN** lifetime session tokens are known but no context occupancy measurement exists
- **THEN** context used, remaining, and percentage SHALL remain unknown while session accounting remains separately available.

### Requirement: Token categories and compaction
Turn and session totals SHALL identify input, output, cache-read, and cache-write categories where supported, with provider inclusion semantics and compaction count. Overlapping categories MUST NOT be summed into a false total. Compaction SHALL update context independently of lifetime token accounting.

#### Scenario: Cache overlaps input
- **WHEN** reported input includes cache-read tokens
- **THEN** total input SHALL retain the reported inclusive value and MUST NOT add cache-read tokens a second time.

### Requirement: Descriptive permissions and collaboration
Permission, approval, sandbox, and collaboration-mode fields SHALL be shown only as reported descriptive telemetry, including unknown or stale status. They MUST NOT be represented as guarantees that the HUD enforces the agent's security boundary.

#### Scenario: Stale sandbox information
- **WHEN** sandbox metadata has not been validated within its freshness interval
- **THEN** it SHALL be marked stale and MUST NOT appear as a current safety assurance.
