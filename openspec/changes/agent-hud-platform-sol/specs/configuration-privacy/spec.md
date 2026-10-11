## Purpose

Make configuration predictable and reloadable while applying explicit content, identifier, and retention policy consistently to every place HUD telemetry can be displayed or stored.

## ADDED Requirements

### Requirement: Versioned typed configuration and precedence

Configuration SHALL have an explicit schema version, typed fields, validation errors with field locations, and documented precedence: command-line overrides, allowlisted environment overrides, selected profile, base configuration file, then defaults. Unknown fields and invalid types MUST produce actionable errors rather than silent coercion. The HUD MUST NOT automatically load repository-local configuration. Environment hints MUST NOT enable external command/HTTP providers without explicit configuration or invocation. Initial invalid configuration MUST fail safely before altering a terminal or launching a child. Version migration MUST be explicit and preserve a backup when a file is rewritten.

#### Scenario: Conflicting height settings
- **WHEN** defaults, a base file, a profile, an environment override, and a CLI flag set different heights
- **THEN** the CLI height wins and inspection can explain the selected value's configuration origin without exposing secret environment data

#### Scenario: Invalid initial height
- **WHEN** initial configuration requests eleven lines or uses a string where a number is required
- **THEN** validation returns a field-specific error and no HUD pane or agent process is created

#### Scenario: Inherited provider setting and repository file
- **WHEN** an inherited environment value suggests enabling a usage command and the cwd contains a HUD configuration file
- **THEN** neither enables that command or supplies automatic trusted configuration, and ordinary offline defaults remain effective

### Requirement: Atomic last-valid configuration reload

The HUD SHALL validate a complete candidate configuration before applying a reload atomically. Invalid candidates MUST preserve the last valid configuration and expose a bounded redacted error. Identity-affecting reloads MUST stage a new binding generation and publish the selected target with cleared or validated new-target state; old values MUST NOT be relabeled as the new target. A valid privacy restriction MUST immediately reproject displayed, frozen, cached, queued, and retained values, purge prohibited HUD-owned logs including rotated files, and govern all subsequent outputs. Pending sink writes and exports MUST check the effective privacy revision immediately before writing and cancel or rebuild obsolete projections; a later relaxation MUST NOT resurrect discarded sensitive content.

#### Scenario: Invalid edit during a live turn
- **WHEN** the watched configuration becomes malformed during observation
- **THEN** the previous layout/privacy settings remain active and a redacted reload error is available

#### Scenario: Stricter privacy while frozen
- **WHEN** the user disables branch and goal display while the HUD is frozen
- **THEN** both fields are hidden immediately in the frozen view, exports, diagnostics, and retained records

#### Scenario: Export races with privacy restriction
- **WHEN** an export is prepared under a permissive policy and the user restricts it before the next write
- **THEN** the obsolete projection is canceled or rebuilt before writing, prohibited HUD-owned log entries are purged, and no later sink write uses the old policy

#### Scenario: Target changes during reload
- **WHEN** a valid reload selects another session while old-source work is outstanding
- **THEN** target activation uses a new generation with cleared or verified new-session fields and old-source completions cannot contaminate it

### Requirement: Credential-free offline defaults

Default operation SHALL require no provider credentials, network access, startup-file edits, or prompt/response content. Prompt bodies, response bodies, summaries, goals, session titles, tool arguments, usernames, account identifiers, credential-bearing environment values, and authentication tokens MUST be hidden or discarded by default. Paths MUST use the configured reduced disclosure form rather than exposing an absolute home path. Network queries and content retention MUST require separate explicit settings.

#### Scenario: Sensitive default payload
- **WHEN** a producer sends prompt text, response text, tool arguments, an account email, and a home-directory path with ordinary model state
- **THEN** only allowed reduced-disclosure common state is rendered and sensitive payload content is absent from default diagnostics, exports, and storage

### Requirement: Field-level privacy controls

Privacy controls SHALL independently govern prompts, responses, summaries, goals/titles, usernames, paths, repository names, branches, tool targets/arguments, account identifiers, and quota/cost details. Unknown provider extensions MUST default to nondisclosure until classified and explicitly allowed. Privacy profiles MUST use the same field policy as individual controls, and redacted availability MUST remain distinguishable from unsupported capability.

#### Scenario: Privacy profile hides usage details
- **WHEN** a privacy profile hides account and quota/cost fields but an adapter supports live quotas
- **THEN** the HUD identifies those fields as redacted without publishing their amounts, account identifiers, or reset details

### Requirement: Redaction before every output sink

The HUD SHALL apply one effective privacy policy before rendering, logging, diagnostics, snapshot export, caching, and retained history. Secret values and raw sensitive provider records MUST NOT be written to error messages, debug logs, temporary diagnostics, or automatic crash reports. Exports MUST contain only schema-validated permitted projections; source paths and binding evidence MUST be reduced or replaced with opaque references according to policy.

#### Scenario: Parser error includes a secret-bearing record
- **WHEN** a malformed record contains a credential string and a sensitive path
- **THEN** errors identify safe source/field locations without echoing the record or revealing the restricted path in any sink

#### Scenario: Redacted snapshot export
- **WHEN** the user exports a snapshot while repository and branch disclosure are disabled
- **THEN** those fields are absent or redacted in the export and cannot be recovered from provenance or diagnostic evidence

### Requirement: Bounded local retention and explicit export

Persistent content/history retention SHALL be off by default. Current content display SHALL require explicit per-field permission and MUST NOT itself enable history or disk retention; retaining that content SHALL require a separate opt-in with finite size/age bounds. Enabled retention and caches MUST use owner-restricted storage and privacy-safe contents; in-memory history MUST also be bounded. Diagnostics/logs MUST stay local unless the user explicitly exports them, and the HUD MUST never upload them automatically. Disabling retention MUST purge HUD-managed retained content without deleting agent-owned data. Explicit exports SHALL explain that completed user-owned copies cannot be recalled by a later privacy change; the HUD MUST NOT claim physical secure erasure.

#### Scenario: Retention disabled after use
- **WHEN** the user disables previously enabled summary/history retention
- **THEN** HUD-managed retained content is purged, live observation continues, and agent rollout files are left untouched

#### Scenario: Current summary without retained history
- **WHEN** a user explicitly permits summary display and leaves content retention disabled
- **THEN** only the current permitted summary can appear in memory and no prior summary or durable conversation history is retained

### Requirement: Safe owner-private local resources

HUD-owned runtime directories SHALL use owner-only access; endpoint descriptors, logs, caches, migration backups, and explicit exports SHALL use private files with validated ownership/type and symlink-safe creation. Unsafe targets MUST fail closed without clobbering unrelated files. Credentials and known credential fields MUST never become retained telemetry or visible argv, even when content display is permitted. Private endpoint permissions and nonces SHALL be documented as routing protection within the user's trust domain, not isolation from other code running as that user.

#### Scenario: Export or backup follows a hostile symlink
- **WHEN** a selected export or migration-backup path is an unsafe symlink or has an unverified owner
- **THEN** the operation refuses the target with a redacted error and leaves its referent unchanged

### Requirement: Observation cannot authorize actions

Configuration, telemetry, or a rendered field SHALL NOT implicitly execute an embedded command, approve an agent request, alter agent permissions, install hooks, modify startup files, or change agent state. Explicit external usage-command configuration and deliberate wrapper/client actions MUST remain separate from telemetry consumption.

#### Scenario: Command-looking telemetry
- **WHEN** a tool target or namespaced extension contains shell syntax
- **THEN** it is treated as sanitized data and never evaluated as a command
