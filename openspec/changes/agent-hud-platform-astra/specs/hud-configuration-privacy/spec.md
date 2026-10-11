## Purpose

Make configuration predictable and privacy consistent across every visible or retained output without turning observation into agent control.

## ADDED Requirements

### Requirement: Typed versioned configuration and precedence
Configuration SHALL have a schema version, typed validation, field-path errors, and documented precedence of built-in defaults, user configuration, explicitly selected configuration file, selected profile, supported environment overrides, and command-line flags from lowest to highest priority. Profile selection SHALL be explicit and its effective field origins inspectable. Unknown core keys and unsupported major versions SHALL fail validation. Invalid initial configuration MUST fail before child launch, provider queries, ingress creation, or terminal changes. Repository files MUST NOT be loaded automatically as executable or trusted configuration.

#### Scenario: Conflicting height values
- **WHEN** user configuration sets height two and a command-line flag sets height four
- **THEN** the effective height SHALL be four and inspection SHALL identify the command-line origin.

#### Scenario: Profile precedence and invalid startup
- **WHEN** a selected profile overrides a file height and a flag then requests height eleven
- **THEN** validation SHALL identify the invalid flag origin and MUST NOT start an agent, provider, endpoint, or HUD pane.

### Requirement: Atomic reload and migration
Reload SHALL validate the complete merged candidate before replacing active configuration. Invalid candidates SHALL leave the last valid configuration effective and report a redacted field-path error. Supported migration SHALL be explicit, create a backup, and never silently rewrite a newer schema version.

#### Scenario: Invalid live edit
- **WHEN** a running HUD encounters malformed configuration or an invalid enum
- **THEN** it SHALL continue with the previous valid configuration and display an actionable error without exposing secret values.

### Requirement: Offline and noninvasive defaults
Default startup SHALL require no credentials, outbound network, shell-startup-file edits, or account-store access. Enabling an external provider SHALL require explicit configuration. Telemetry display MUST NOT execute commands named in telemetry, approve actions, or change agent state; fixed local observers and explicitly configured usage commands SHALL be the only observation-related subprocess sources.

#### Scenario: Telemetry includes a command
- **WHEN** a tool record contains a shell command or a proposed approval action
- **THEN** the HUD SHALL treat it as data and MUST NOT execute or approve it.

### Requirement: Field-level privacy policy
Privacy policy SHALL separately control prompts, responses, summaries, goals/titles, usernames, full paths, repository names, branches, tool targets/arguments, account identifiers, and quota/cost details. Prompt/response bodies, tool arguments, raw account IDs, usernames, full paths, goals, titles, and summaries SHALL be hidden by default. Unknown extensions SHALL remain undisclosed until classified and explicitly allowed. Scope keys, provenance, and source labels SHALL follow the same disclosure policy as values. Repository basename, branch, and numeric quota/cost visibility SHALL have explicit defaults and SHALL be disabled by the privacy profile.

#### Scenario: Default observation contains content
- **WHEN** an adapter receives prompt text, tool arguments, a goal, and an account identifier
- **THEN** default outputs SHALL omit their values and record only non-sensitive availability reasons.

#### Scenario: Secret carried in an extension or source label
- **WHEN** a producer places hidden account/path content in an unknown extension or provenance label
- **THEN** no display, diagnostic, export, or retained sink SHALL disclose that content through the alternate location.

### Requirement: Redaction across all sinks
A single effective policy SHALL apply before HUD presentation, logs, diagnostics, exports, caches, and retained history. Credentials and tokens MUST never be persisted or exported by the HUD, including when content display is enabled. Debug mode MUST NOT bypass policy. Tightening privacy SHALL purge prohibited queued, current, frozen, and HUD-owned retained values before publishing further output; configuration and policy projection SHALL change atomically. Later relaxation MUST NOT resurrect discarded values without new permitted observations.

#### Scenario: Secret in parser error
- **WHEN** malformed input contains a bearer token and a private path
- **THEN** logs and diagnostics SHALL report a bounded location/category error without echoing the raw input or token.

#### Scenario: Privacy tightened during a session
- **WHEN** the user disables branch and repository display
- **THEN** subsequent output and retained HUD-owned state SHALL omit those values, including frozen views and export snapshots.

#### Scenario: Privacy relaxed after purging a frozen view
- **WHEN** branch and goal display are disabled while frozen and later re-enabled without new observations
- **THEN** the frozen view and every other sink MUST NOT recover the discarded branch or goal.

### Requirement: Content and retention consent
Optional content display SHALL require explicit field permission; content retention SHALL require a separate explicit setting with finite count, byte, and age limits. Disabling retention SHALL purge HUD-owned retained content while leaving agent-owned records untouched. Default operation SHALL retain only bounded redacted operational state and MUST NOT create durable conversation history. Exports SHALL require explicit local invocation and SHALL explain that already exported files cannot be recalled by later policy changes.

#### Scenario: Summary enabled without retention
- **WHEN** a user permits compact summaries but leaves retention disabled
- **THEN** the HUD SHALL allow the current permitted summary in memory and MUST NOT write it to disk or retain prior summaries.

### Requirement: Safe local resources
HUD-owned runtime directories, ingress endpoints, caches, logs, and exports SHALL use private permissions and validated ownership. Credentials SHALL remain in provider-managed stores; external providers SHALL reference user-configured credentials without logging, copying them into state, or passing them in visible command-line arguments.

#### Scenario: Unsafe ingress path
- **WHEN** an ingress endpoint path resolves to an unowned directory or unsafe symlink target
- **THEN** startup SHALL refuse the endpoint with an actionable redacted error.
