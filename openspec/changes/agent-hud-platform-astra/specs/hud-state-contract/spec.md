## Purpose

Provide an agent-independent, versioned observation contract that preserves the meaning and trustworthiness of every displayed field.

## ADDED Requirements

### Requirement: Versioned adapter handshake
Each adapter SHALL declare adapter ID/version, CLI name/version when known, integration mode, supported schema versions, and capabilities before publishing state. The HUD MUST reject incompatible major schema versions with an actionable error and accept compatible additive minor versions without requiring provider-specific renderer changes.

#### Scenario: Incompatible schema
- **WHEN** an adapter offers only an unsupported major version
- **THEN** the HUD SHALL report incompatibility and MUST NOT publish its observations as valid state.

#### Scenario: Additive extension
- **WHEN** a compatible adapter adds a namespaced extension unknown to the HUD
- **THEN** existing common fields SHALL remain usable and the unknown extension MUST NOT change their interpretation.

### Requirement: Explicit observation operations
The contract SHALL support snapshot, patch, invalidation, source-health, and capability updates with source identity/epoch, binding generation, scope key, and ordering metadata. A snapshot SHALL declare covered field groups within one scope and replace only that coverage, clearing covered omissions; omission in a patch SHALL preserve the previous value; explicit null SHALL clear the value to unknown. Invalidation SHALL mark specified values untrusted without inventing replacement measurements. Capability withdrawal SHALL invalidate observations dependent on that capability and their derivations in the same revision; independently supported sources SHALL retain their own evidence. Source-health updates MUST NOT invent measurement observations.

#### Scenario: Sparse patch and explicit clear
- **WHEN** a patch omits model effort and explicitly sets context usage to null
- **THEN** effort SHALL retain its prior value and context usage SHALL become unknown.

#### Scenario: Scoped snapshot
- **WHEN** a new snapshot replaces session state but omits an account-scoped quota
- **THEN** omitted session fields SHALL become unknown and the independently valid account quota SHALL remain associated with its original account key.

#### Scenario: Model-only snapshot
- **WHEN** a snapshot covers only model fields and omits effort
- **THEN** effort SHALL become unknown while project fields outside that coverage SHALL remain unchanged.

#### Scenario: Capability withdrawal is atomic
- **WHEN** the only context source withdraws its context capability
- **THEN** the same revision SHALL mark the capability unsupported and invalidate its occupancy and derived percentage without retaining a live meter.

### Requirement: Immutable coherent publication
Published state SHALL carry schema version, binding generation, and monotonically increasing revision. Each revision MUST be immutable and internally consistent; a rendered frame and exported snapshot SHALL each reference one complete revision.

#### Scenario: Multi-field context update
- **WHEN** context usage and total window change in one coherent observation
- **THEN** no consumer SHALL observe a revision combining the new usage with the old window.

### Requirement: Independent trust dimensions
For each field or coherent group the HUD SHALL distinguish capability support, availability, privacy redaction, freshness, measurement quality, and connection health. Unknown values SHALL be null or absent with an explicit availability reason. Unsupported, withheld, stale, or disconnected values MUST NOT become zero, false, idle, or healthy by default.

#### Scenario: Redacted but supported quota
- **WHEN** an adapter supports quota but the user's policy withholds it
- **THEN** inspection SHALL report supported capability and redacted disclosure, preserve independently reportable availability and health, and MUST NOT disclose the quota or treat it as unsupported.

#### Scenario: Unknown dirty state
- **WHEN** Git status cannot be obtained
- **THEN** dirty state SHALL remain unknown rather than showing a clean repository.

### Requirement: Measurement dictionary and provenance
The contract SHALL define units, valid ranges, ownership scopes, and reported/derived/estimated/inferred quality for common fields. Provenance SHALL include source ID/kind, scope, observation and receipt timestamps, last successful validation, and inspectable age. Derived fields MUST retain the dependency revision and weakest relevant trust state.

#### Scenario: Derived percentage
- **WHEN** a context percentage derives from a stale usage measurement and live capacity
- **THEN** the percentage SHALL be identified as derived and stale with both dependencies inspectable.

### Requirement: Scoped accounting semantics
Token and usage records SHALL identify account/session/turn/subagent scope, source/metric, category, unit, and cumulative or delta semantics. Repeated event IDs MUST NOT add usage twice, including after bounded deduplication eviction. A cumulative baseline SHALL declare which deltas it includes through a verified checkpoint; only subsequent compatible deltas SHALL augment it. Unprovable overlap or replay continuity across epochs SHALL make affected totals uncertain until a covered baseline or complete verified replay succeeds. Only source-proven counter resets SHALL begin a new accounting epoch; a decrease alone MUST NOT establish reset. Totals with incompatible scopes or overlapping categories MUST NOT be added implicitly.

#### Scenario: Duplicate delta
- **WHEN** a token delta with the same source epoch and event ID arrives twice
- **THEN** the second observation SHALL leave all totals unchanged.

#### Scenario: Parent and child totals
- **WHEN** parent totals have unknown child inclusion semantics
- **THEN** the HUD SHALL show parent and child totals separately and MUST NOT claim their sum is a session total.

#### Scenario: Baseline overlaps replayed deltas
- **WHEN** a cumulative total of 100 includes events through sequence 20 and replay supplies sequence 19 plus a 5-token delta at sequence 21
- **THEN** the result SHALL be 105 and sequence 19 MUST NOT be added, even if its event ID has left the deduplication buffer.

#### Scenario: Reconnect cannot establish accounting coverage
- **WHEN** a source restarts sequencing in a new epoch without a checkpoint or verified complete replay
- **THEN** old and new deltas MUST NOT be summed into a claimed complete total.

### Requirement: Safe observation paths
Observation paths and collection operations SHALL be limited to declared writable fields, namespaces, and source coverage. Prototype-mutating keys, conflicting set/clear operations, and common-field overwrite through extensions MUST be rejected atomically with redacted errors; no partial update SHALL be published.

#### Scenario: Malicious or conflicting patch
- **WHEN** a patch contains a prototype-mutating key or both sets and clears the same field
- **THEN** the entire patch SHALL be rejected without changing the current revision or runtime object prototypes.

### Requirement: Provider-independent presentation contract
Common rendering SHALL consume normalized fields and capability metadata without agent-specific conditionals. Provider extensions SHALL be namespaced by adapter ID, bounded by the common ingestion limits, and presented only through generic supported extension descriptors or inspection.

#### Scenario: New conforming adapter
- **WHEN** a new adapter supplies existing normalized fields
- **THEN** those fields SHALL render correctly without modifying the common renderer.
