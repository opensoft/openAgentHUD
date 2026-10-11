## Purpose

Integrate diverse agent CLIs through verified, capability-declared sources while keeping provider differences outside common rendering.

## ADDED Requirements

### Requirement: Generic local snapshot and event integration
The generic adapter SHALL accept documented JSON snapshots and JSONL events through an explicitly selected file, pipe, or private session-scoped local ingress endpoint. Payloads SHALL use the shared handshake, observation operations, identity, ordering, scope, and provenance contracts. The endpoint MUST NOT listen on a network interface or accept mismatched launch/session identity.

Producer authorization SHALL bind verified session/source evidence to a controller-issued generation/epoch lease. Rollover SHALL retire old leases. A launch nonce alone MUST NOT authorize old session data for the current generation, including when a delayed producer reconnects; missing current-session evidence SHALL produce unresolved/rejected ingestion. File readers SHALL retain the generation/epoch captured when their source identity was validated.

#### Scenario: Mismatched ingress identity
- **WHEN** a producer sends a valid payload carrying another launch identity
- **THEN** the adapter SHALL reject it without mutating the bound session.

#### Scenario: Old hook reconnects after rollover
- **WHEN** a delayed producer from session A reconnects using the same launch nonce after session B has become current
- **THEN** the controller MUST NOT stamp its A payload as generation B and SHALL require verified B evidence before issuing a new lease.

### Requirement: Generic validation and reconnect
A local validation command SHALL check generic examples and streams without launching an agent. It SHALL report record number and redacted schema-path errors, enforce ingestion bounds, and return a nonzero status for invalid input. On reconnect, the adapter SHALL require a current snapshot or an explicitly contiguous resumable epoch before restoring trusted accounting.

#### Scenario: Reconnect loses event continuity
- **WHEN** a generic producer reconnects with a new source epoch and no snapshot
- **THEN** accounting SHALL remain uncertain until a snapshot arrives.

### Requirement: Codex native observer isolation
Codex observer mode SHALL attach without taking over the native TUI. Verified rollout sources SHALL provide durable state; verified state database evidence SHALL assist root-thread discovery; verified event/log evidence SHALL provide field-specific updates where supported. Version-sensitive parsing and read-only database access SHALL remain inside the adapter. Unsupported versions SHALL degrade visibly to verified fallbacks rather than guessing richer telemetry.

#### Scenario: State database format changes
- **WHEN** the observer cannot validate a state database layout
- **THEN** it SHALL report a version limitation, preserve independently verified observations, and MUST NOT mutate the database or select the newest unrelated rollout.

### Requirement: Codex controlled integration is explicit
Codex app-server mode SHALL require an explicit controlled-mode invocation and SHALL declare its distinct mode and responsibilities. Its client SHALL handle verified session/turn events, input and output, explicit user approval responses, cancellation, and connection lifecycle. Observer failure MUST NOT automatically switch to controlled mode. Displaying an approval request MUST NOT answer it.

The control surface SHALL be usable independently of the HUD and locally verified with the actual supported server. Its command and approval path SHALL be inaccessible to telemetry producers and view-control endpoints. Unsupported protocol negotiation SHALL fail explicitly; synthetic client/server tests alone MUST NOT establish usable provider support.

#### Scenario: Controlled approval request
- **WHEN** app-server requests approval
- **THEN** the agent SHALL remain waiting until an explicit user response reaches the control client; the HUD SHALL show descriptive waiting state only.

#### Scenario: Native observer is unsupported
- **WHEN** native observation fails for an installed Codex version
- **THEN** the adapter SHALL report limited support and MUST NOT launch app-server or replace the user's session automatically.

#### Scenario: Telemetry attempts to answer an approval
- **WHEN** a generic producer or view-control message contains an approval-response-shaped payload
- **THEN** it SHALL be rejected as an invalid message type and the actual request SHALL remain pending for the control client.

### Requirement: Controlled client survives presentation loss
The controlled integration SHALL keep the agent transport and approval owner separate from the renderer. If the presentation surface closes or crashes, the control client SHALL continue, preserve pending requests, and offer reconnect; if the control owner is lost, it SHALL fail closed on approvals and perform a documented disconnect or interruption without claiming completion.

#### Scenario: HUD closes with approval pending
- **WHEN** the HUD pane closes during a controlled approval request
- **THEN** the control client SHALL keep the request pending and allow it to be viewed after presentation reconnects.

### Requirement: Claude sparse hook correlation
The Claude adapter SHALL accept only locally verified status-line or command-hook payload formats. It SHALL correlate short-lived hook invocations using stable session/launch and hosting evidence rather than hook PID alone, with the same producer-lease validation as generic ingress. Sparse payload omission SHALL preserve unrelated fields, while explicit null SHALL clear them. Adapter health SHALL distinguish payload age from agent liveness. Explicitly installed templates SHALL preserve existing status-line behavior; telemetry failure MUST NOT suppress that output or block it beyond a documented forwarding deadline.

#### Scenario: Two hooks share a working directory
- **WHEN** hooks for different Claude sessions arrive from the same repository
- **THEN** their stable session identities SHALL determine routing and their state MUST NOT be merged.

#### Scenario: Existing status line during ingress failure
- **WHEN** an explicitly installed chaining template receives a verified payload but HUD ingress is unavailable
- **THEN** the original status-line command SHALL receive the original input and its stdout/exit status SHALL be preserved without HUD diagnostics being added to stdout.

### Requirement: Gemini and Aider fallback profiles
Gemini and Aider profiles SHALL provide explicit executable discovery, verified process identity, project/Git fallback fields, and a documented support tier. Model, effort, context, quotas, turns, and tool activity MUST remain unsupported or unknown unless a locally verified structured source is added with conformance fixtures.

#### Scenario: Executable exists without telemetry protocol
- **WHEN** a Gemini or Aider executable is discovered but no supported structured source is available
- **THEN** the HUD SHALL expose only verified fallback fields and clearly identify limited telemetry support.

### Requirement: Compatibility matrix and health
Each adapter SHALL publish supported CLI versions, mode, source formats, identity strength, available fields, limitations, and recovery behavior. Version claims SHALL require reproducible local evidence and contract fixtures; unknown versions SHALL report unverified support. Diagnostics SHALL identify adapter/source versions and actionable remediation without requiring renderer changes.

#### Scenario: Unsupported CLI upgrade
- **WHEN** a user upgrades beyond a verified source format
- **THEN** the adapter SHALL report the unsupported format and remediation or fallback options without claiming full compatibility.
