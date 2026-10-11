## Purpose

Provide an agent-independent, versioned state and update contract so adapters, consumers, and renderers share explicit meanings for values, identity, scope, uncertainty, and changes.

## ADDED Requirements

### Requirement: Compatible adapter handshake

Before accepting telemetry, the HUD SHALL validate a handshake containing adapter ID/version, agent/CLI identity and version when known, state-schema compatibility, integration mode, source identity, supported capabilities, and required protocol features. An incompatible major version, unsupported mandatory feature, or malformed handshake MUST produce an actionable compatibility error without replacing the last valid state. Bounded unknown optional telemetry fields in a compatible minor version MUST be skipped without retention or invented support; invalid known fields and unknown operations MUST remain errors.

#### Scenario: Unsupported major version
- **WHEN** an adapter requests an unsupported state-schema major version
- **THEN** its telemetry is rejected and inspection identifies the adapter and compatibility failure

#### Scenario: Compatible additive extension
- **WHEN** a compatible adapter includes a previously unknown optional field
- **THEN** known fields remain usable and the unknown field does not acquire an invented common meaning

#### Scenario: Additive common field and required feature
- **WHEN** a compatible minor version adds an unknown optional common field, then a handshake requires an unsupported operation
- **THEN** the optional field is safely skipped without retention while known fields remain usable, and the unsupported mandatory feature fails negotiation before its observations are accepted

### Requirement: Common field semantics and unknown values

The public contract SHALL define the type, unit, scope, and permitted reported/derived/inferred/estimated quality of each common field. Unknown values MUST be null or absent, and legitimate observed zero/false values MUST remain distinguishable from unknown. Measurements without required units or scope MUST be rejected or invalidated with a diagnostic rather than guessed.

#### Scenario: Missing context capacity
- **WHEN** used context tokens are reported but the context window is unknown
- **THEN** used tokens remain available while capacity, remaining tokens, and percent remain unknown

#### Scenario: Known zero and unknown count
- **WHEN** one source reports zero output tokens and another supplies no file-count measurement
- **THEN** output tokens are represented as measured zero and file count is represented as unknown

### Requirement: Scoped values and provenance

Every value or coherent field group SHALL identify its source, scope type and key, observation time when supplied, local receipt time, last successful validation, quality, and age. Scope keys MUST distinguish at least HUD instance, launch, process-start identity, worktree, account, session, and turn. Missing account or session identity MUST prevent reuse of that scope's cached data across bindings. Derived fields MUST expose their dependency revisions/scopes and inherit the weakest relevant availability, freshness, and input quality without concealing their derived origin.

#### Scenario: Account identity is unavailable
- **WHEN** a new binding has no verified account key
- **THEN** account-scoped values from the prior binding are excluded and their absence is explainable in inspection

#### Scenario: Provenance inspection
- **WHEN** a user inspects an estimated context measurement
- **THEN** the result includes its estimate label, unit, scope, redacted source description, and observation/validation ages

#### Scenario: Derivation from stale estimated input
- **WHEN** context percent is calculated from stale estimated usage and live measured capacity in a compatible scope
- **THEN** the result remains labeled derived, estimated, and stale with both dependency revisions inspectable

### Requirement: Immutable consistent revisions

The HUD SHALL expose monotonically increasing immutable state revisions within a HUD instance. Each revision MUST contain one binding generation and internally consistent values and metadata; every displayed frame and exported snapshot MUST be derived from exactly one revision.

#### Scenario: Concurrent updates during rendering
- **WHEN** model and turn updates arrive while a frame is being produced
- **THEN** the frame uses one published revision and cannot combine the new turn with incompatible prior-generation model metadata

### Requirement: Snapshot patch and invalidation semantics

The update contract SHALL support snapshots, patches, explicit field invalidation, source-health updates, and capability updates. A snapshot MUST declare coverage within the source's authorized paths for one scope and replace only that coverage, clearing covered omissions; a patch MUST preserve omitted paths and treat explicit null or clear operations as invalidation. Tools, subagents, and usage records SHALL support keyed upsert/remove patches using scope-qualified stable IDs, never positional array edits. Whole-collection replacement MUST declare complete snapshot coverage. Unauthorized coverage and conflicting collection operations MUST be rejected. Updates MUST carry validated binding generation, source epoch, and ordering/deduplication evidence. Capability removal MUST invalidate dependent values in the same published revision.

#### Scenario: Sparse patch followed by explicit clear
- **WHEN** a patch omits reasoning effort and a later patch explicitly clears it
- **THEN** the first patch preserves the prior effort and the second publishes unknown effort with its invalidation reason

#### Scenario: Snapshot coverage
- **WHEN** a model-only snapshot omits effort but previously available project data is outside its coverage
- **THEN** effort is cleared and project data remains unchanged

#### Scenario: Keyed collection patch
- **WHEN** a patch upserts tool call B and removes call A while another call C exists in the same scope
- **THEN** B is updated once, A is removed without implying cancellation or success, and C remains unchanged

#### Scenario: Unauthorized snapshot coverage
- **WHEN** a model-only source claims snapshot coverage over another source's account quota or submits conflicting replacement and upsert operations
- **THEN** the observation is rejected with a redacted contract error and the prior valid values remain unchanged

#### Scenario: Capability withdrawal
- **WHEN** an adapter withdraws quota support after a compatibility failure
- **THEN** quota capability and dependent value availability change together and old quota values cannot appear live

### Requirement: Independent status dimensions

The contract SHALL independently represent capability support, current availability, freshness, privacy disclosure, source connection health, and value quality. Freshness SHALL distinguish live, cached, and stale; disconnected SHALL be connection health and frozen SHALL be view state. These conditions and inferred/estimated quality MUST be identifiable together without overwriting one another. Unsupported, unavailable, redacted, and disconnected values MUST NOT be rendered as zero or healthy state.

#### Scenario: Redacted live value
- **WHEN** a supported live account measurement is hidden by privacy policy
- **THEN** its disclosure is redacted while its support and source health remain independently inspectable without revealing the value

#### Scenario: Healthy source without quota support
- **WHEN** a generic source is healthy but declares no quota capability
- **THEN** quota remains unsupported rather than displaying an empty healthy usage meter

#### Scenario: Disconnected frozen estimate
- **WHEN** an estimated value becomes stale after its source disconnects while the display is frozen
- **THEN** inspection preserves separate stale freshness, disconnected connection health, estimated quality, and frozen view state

### Requirement: Namespaced provider extensions and renderer independence

Provider-specific values SHALL be confined to adapter-owned namespaces with declared metadata and schema versions. The common presentation contract MUST consume common field descriptors or explicitly selected extension descriptors without requiring agent-specific rendering branches. Unknown extension namespaces MUST NOT overwrite common fields.

#### Scenario: Adding an adapter
- **WHEN** a new adapter supplies common model and turn values plus a namespaced vendor field
- **THEN** existing layouts render the common values through the unchanged common presentation contract and the vendor field is omitted unless explicitly selected

### Requirement: Descriptive observation contract

Permission, approval, sandbox, collaboration, and capability values SHALL describe observed agent state only. Consuming a snapshot, patch, or extension MUST NOT execute commands, approve requests, alter permissions, or change agent state.

#### Scenario: Approval state in telemetry
- **WHEN** a source reports that a tool is awaiting approval
- **THEN** the HUD displays a waiting state without sending an approval or executing the tool
