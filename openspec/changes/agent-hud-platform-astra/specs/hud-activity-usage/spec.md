## Purpose

Expose observed activity, collaboration, limits, and cost with stable correlation, explicit scopes, and honest uncertainty.

## ADDED Requirements

### Requirement: Correlated tool activity
Tool activity SHALL expose stable call identity, tool name, permitted target, start/duration, completion, and error status where observed. Arguments SHALL be omitted by default. Missing starts, duplicate completion events, and unobserved completions MUST NOT fabricate duration or success.

#### Scenario: Completion without start
- **WHEN** the first observation of a tool call is its completion
- **THEN** the HUD SHALL report the observed completion with unknown start/duration and MUST NOT invent a start timestamp.

### Requirement: Distinct integration activity states
Skills, plugins, and MCP servers SHALL distinguish configured, loaded, and actively used states with evidence and scope. Configuration presence alone MUST NOT imply loading, a call, success, or health.

#### Scenario: Configured MCP server is unused
- **WHEN** an adapter observes MCP configuration but no connection or call evidence
- **THEN** the server SHALL appear only as configured and active usage SHALL remain unverified.

### Requirement: Subagents and explicit aggregation
Subagents SHALL carry stable identity, parent identity, reported model/status, and inclusion semantics for any usage aggregation. Unknown ancestry or incomplete child coverage SHALL be visible. Parent totals SHALL never be augmented by children unless their scopes are documented as disjoint.

#### Scenario: Child usage may already be included
- **WHEN** a parent record does not state whether it includes child tokens
- **THEN** child totals SHALL be presented separately and aggregate usage SHALL remain unavailable.

### Requirement: Plans and optional content
The HUD SHALL expose observed plan/todo item, completion count, goal, title, turn count, and optional compact summary only when supported and permitted by field-level policy. Unsupported values SHALL remain unavailable; summary display and retention SHALL require separate consent. Summaries SHALL originate in verified adapter input; receiving content MUST NOT trigger a HUD-issued summarization or network request.

#### Scenario: Counts available but item content withheld
- **WHEN** a plan contains three completed items out of five and content permission is disabled
- **THEN** the HUD SHALL allow the completion count while withholding item text and goal/title content.

### Requirement: Visible measurement anomalies
Context overflow, contradictory measurements, dropped events, and unsupported claims of activity SHALL generate inspectable anomaly records with affected scope and state revision. The HUD MUST NOT silently repair contradictions by substituting plausible measurements.

#### Scenario: Unsupported activity claim
- **WHEN** an adapter claims active skill usage without the declared capability or supporting event
- **THEN** the HUD SHALL withhold the unsupported active claim and expose a bounded adapter-contract anomaly.

### Requirement: Extensible scoped usage records
Usage SHALL be a list keyed by provider, pseudonymous account, window, resource/unit, and limit kind rather than fixed primary/secondary slots. Records SHALL distinguish quota, credit balance, measured monetary cost, and estimated cost; expose used/remaining percentages, absolute reset time, countdown, and observation time where supported; and preserve unavailable fields as unknown.

Derived percentages SHALL require compatible units/scopes and positive known capacity. Overlapping windows, different accounts, units, or metric kinds MUST NOT be added into a combined limit without explicit compatible coverage semantics.

#### Scenario: Provider supplies three independent windows
- **WHEN** an account reports hourly, weekly, and model-specific limits
- **THEN** all three SHALL retain their separate keys and units without being collapsed into two display slots.

#### Scenario: Invalid quota denominator and mixed metrics
- **WHEN** a provider reports quota usage with zero or unknown capacity alongside credits and monetary cost
- **THEN** quota percentage SHALL remain unknown and the three metric kinds SHALL remain separate.

### Requirement: Reset reconciliation and cache age
Quota cache age and provider freshness SHALL remain visible. Reaching a reset deadline SHALL initiate permitted reconciliation or mark the window reset as unconfirmed. It MUST NOT fabricate a fresh zero-usage window. Cached account data SHALL be invalidated on changed or unknown account identity; matching unknown keys MUST NOT authorize reuse. Provider failure SHALL retain prior values only with their actual ages and degraded/cached status.

#### Scenario: Reset occurs offline
- **WHEN** a quota reset deadline passes while no fresh provider observation is available
- **THEN** the HUD SHALL show reset uncertainty and retain the prior measurement only as aged historical data.

### Requirement: Explicit bounded external usage providers
External command or HTTP providers SHALL be disabled by default and enabled by explicit provider configuration. Commands SHALL use executable-plus-argument arrays without shell interpolation; HTTP providers SHALL validate configured destinations and revalidate redirects. Both SHALL have time, output, parsing, retry, and rate limits, with redacted credential handling and cancellation. Concurrency and pending-work bounds SHALL apply globally across providers as well as per provider, so adding providers cannot create unbounded work.

#### Scenario: Redirect changes destination
- **WHEN** an opted-in provider redirects a request to an unapproved host
- **THEN** the request SHALL fail without forwarding credentials or following the redirect.

#### Scenario: Usage command hangs
- **WHEN** a configured usage command exceeds its timeout
- **THEN** its owned process group SHALL be terminated, state SHALL become degraded or stale as appropriate, and the agent process SHALL remain unaffected.

#### Scenario: Many providers request refresh together
- **WHEN** all configured providers become due while the shared query slots are occupied
- **THEN** pending work SHALL remain within the published global bound, repeated refreshes SHALL coalesce per provider, and prior quotas MUST NOT be marked fresh merely because a query was scheduled.

### Requirement: Cost estimate provenance
Cost estimates SHALL identify currency, pricing source, pricing date/version, token-category mapping, and estimated status. Missing or ambiguous prices SHALL produce unknown estimates. Monetary quantities in different currencies or overlapping scopes MUST NOT be summed implicitly.

#### Scenario: Price table omits a cache category
- **WHEN** usage contains a billable cache category without a configured price
- **THEN** the total estimate SHALL be unavailable or explicitly partial with the missing category identified, never presented as a complete measured cost.
