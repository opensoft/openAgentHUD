## Purpose

Deliver verified, independently selectable adapter integrations with a public generic telemetry contract, honest fallback support, and clear separation between read-only observation and controlled-session transport.

## ADDED Requirements

### Requirement: Generic JSON and JSONL ingestion

The generic adapter SHALL accept documented JSON snapshots and JSONL event envelopes from a local pipe/stdin, explicitly selected file, or session-scoped local ingress endpoint. It MUST validate handshake, schema, generation lease, source ordering, authorized coverage, scope, units, and capabilities before publication. The trusted ingress MUST stamp the validated generation and receipt time; producers MUST NOT nominate arbitrary bindings or validation times. Ingress MUST be local and owner-restricted with no network listener, and each source identity SHALL have at most one active producer. Unknown, duplicate-owner, or mismatched session/generation senders MUST be rejected. A malformed record MUST NOT corrupt the last valid revision.

#### Scenario: Valid generic snapshot and patch
- **WHEN** a producer supplies a compatible handshake, a covered snapshot, and an ordered sparse patch through a selected pipe
- **THEN** the HUD publishes validated common state and preserves patch omissions according to the normalized contract

#### Scenario: Wrong-session ingress sender
- **WHEN** a producer connects with a session or generation different from the endpoint's binding
- **THEN** the record is rejected and no selected-session value is changed

### Requirement: Generic validation and conformance tools

The CLI SHALL provide `adapters validate` for generic payloads, example snapshot/event files, and reusable adapter conformance fixtures. Validation MUST report record number and schema/capability/units/scope errors with redacted field locations and a nonzero exit status without launching an agent. Examples SHALL cover explicit clears, keyed collections, malformed input, duplicates, reconnects, and unsupported fields. A new source epoch or reconnect without proven contiguous resume SHALL require a covered baseline before restoring trusted delta totals.

#### Scenario: Invalid unit in an example producer
- **WHEN** `adapters validate` receives a token measurement declared in bytes
- **THEN** validation identifies the semantic mismatch, returns failure, and prints no raw sensitive payload

#### Scenario: Reconnect without continuity
- **WHEN** a producer reconnects with a new source epoch or an unverifiable resume cursor and sends deltas without a baseline
- **THEN** totals remain uncertain until a covered authoritative baseline is reconciled and replay cannot double-count prior usage

### Requirement: Read-only Codex native observer

The Codex observer SHALL use verified version-sensitive rollout, state database, event log, and process/hosting evidence to observe the native CLI without replacing its UI. Durable rollout data SHALL supply supported session telemetry; verified database/thread evidence SHALL aid root-session discovery and rollover, and verified log events SHALL supplement supported fast changes. Internal formats MUST be isolated in versioned adapter profiles, accessed read-only, and disabled or degraded when compatibility cannot be established.

#### Scenario: Native session rollover with readable old rollout
- **WHEN** verified Codex root-thread evidence identifies a successor session while the prior rollout remains readable
- **THEN** the observer validates the successor's identity and publishes the atomic rollover rather than continuing to display the prior session

#### Scenario: Unsupported database format
- **WHEN** the installed Codex version's database profile cannot be verified
- **THEN** database parsing is disabled with an actionable compatibility report while verified rollout or process fallback remains available

### Requirement: Explicit Codex controlled integration

Codex app-server integration SHALL be separately selected as a controlled mode for a HUD-launched session and MUST prefer verified app-server events. A minimal built-in control client SHALL provide deliberate user input, safely presented output, explicit approval responses, cancellation, and connection status on its own input surface; native-TUI parity is outside scope. The server SHALL be launched from an explicit literal child argv, never by silently transforming a native observer command. The HUD telemetry path MUST remain observation-only. Control decisions MUST refer to verified session/generation/request identity; stale or unknown requests MUST NOT be answered, and unsupported approval types SHALL remain pending with actionable guidance. Observer attachment MUST never silently enter this mode. Loss of the HUD view MUST NOT terminate the agent/client or auto-approve pending requests.

#### Scenario: Controlled request awaits client approval
- **WHEN** the app-server reports a request requiring approval
- **THEN** the HUD shows approval waiting while only an explicit user response on the control client's focused surface can resolve the verified request

#### Scenario: HUD disconnects during a controlled turn
- **WHEN** the renderer exits or its transport disconnects while the client and server are active
- **THEN** the client/server session continues under its session host and the HUD does not synthesize responses or kill the agent

### Requirement: Controlled recovery cannot replay uncertain actions

The persistent control owner SHALL preserve pending requests across presentation loss and reconcile transport/session continuity before accepting decisions after reconnect. It MUST NOT automatically replay input, cancellation, or approval responses whose delivery outcome is unknown unless a verified protocol explicitly guarantees idempotency and confirms the same request/session. Control-owner loss SHALL leave approvals unresolved and report the verified disconnect/interruption outcome without claiming success.

#### Scenario: Old approval response after reconnect
- **WHEN** a request ID is reused in another session or a user response targets a request retired before reconnect
- **THEN** the control client rejects the stale response and sends no decision for the replacement request

#### Scenario: Input delivery outcome is unknown
- **WHEN** transport disconnects after input or an approval response is sent but before acknowledgment
- **THEN** the client reports the uncertain outcome and reconciles without automatically sending the action again

### Requirement: Claude sparse status-line integration

The Claude adapter SHALL accept verified status-line/command payloads correlated with a propagated session/launch key and process/hosting evidence. Short-lived hook process IDs MUST NOT be treated as durable agent-session identities. Sparse payloads MUST follow patch omission rules, explicit clears MUST invalidate fields, and uncorrelated hooks MUST be rejected. Integration templates MUST preserve any configured status-line behavior and require explicit installation.

#### Scenario: Hook process changes with stable correlation
- **WHEN** two short-lived hooks carry the same verified launch/session key but different PIDs
- **THEN** they update the same binding without creating new agent sessions

#### Scenario: Sparse model-only hook
- **WHEN** a correlated Claude hook reports only model data
- **THEN** previously valid project/context values remain subject to their own freshness instead of being overwritten by absent payload fields

### Requirement: Gemini and Aider honest fallback profiles

Gemini and Aider adapters SHALL provide explicit profiles, executable discovery, CLI version reporting when verified, and process/project/Git fallback. Richer model/context/usage/activity capabilities MUST remain unsupported until a documented or locally verified structured profile passes conformance checks. CLI name detection alone MUST NOT confer telemetry parity.

#### Scenario: Aider executable without structured telemetry
- **WHEN** a launched Aider process is recognized but no verified structured telemetry is available
- **THEN** the HUD reports process/project support and identifies context, quotas, and tools as unsupported rather than guessing them

### Requirement: Source support tiers and compatibility matrix

Each adapter SHALL publish tested CLI versions, integration modes, identity strength, common fields, source types, limitations, compatibility failures, and recovery behavior. Support tiers SHALL distinguish structured-verified, hook-verified, fallback-verified, unverified-version, and unavailable, scoped to the mode/source rather than the brand. Structured/hook verification SHALL require sanitized real provider fixtures and a reproducible local CLI smoke check; synthetic conformance traces alone MUST NOT establish compatibility. Completion of this change SHALL require at least one verified version for each required Codex observer, Codex controlled, and Claude hook mode. Missing evidence SHALL remain an explicit implementation blocker rather than silently completing those modes as unsupported. Untested versions MUST remain unverified and health errors MUST provide actionable recovery guidance.

#### Scenario: Untested CLI version
- **WHEN** an adapter detects a CLI version outside its verified profile ranges
- **THEN** inspection reports the version as unverified and uses only sources whose compatibility can be independently established

### Requirement: Internal adapter addition without renderer changes

An additional internal adapter SHALL integrate through the existing normalized handshake/update contract and pass shared conformance checks. It MUST NOT require agent-specific changes to the common renderer, backend, or CLI selection policy. A public third-party plugin SDK is outside this capability.

#### Scenario: New fixture adapter
- **WHEN** a test adapter emits valid common telemetry and namespaced optional extensions
- **THEN** existing renderer/backend tests succeed without common presentation changes
