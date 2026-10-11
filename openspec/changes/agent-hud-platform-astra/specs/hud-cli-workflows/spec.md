## Purpose

Provide explicit local workflows for launching or observing agents, selecting sessions, validating integrations, and inspecting redacted state.

## ADDED Requirements

### Requirement: Explicit command surface
The executable SHALL provide attach, run, inspect, doctor, adapters list, and config validate commands with documented exit statuses, safe argument parsing, and useful errors. It SHALL also expose generic payload validation and a credential-free local demo. Help and validation MUST NOT launch an agent, create tmux panes, or contact a provider.

Machine-readable invocations SHALL emit exactly one redacted JSON result on stdout and send diagnostics to stderr. Error results SHALL include a stable error kind and documented nonzero status; command codes MUST NOT replace a launched child's status.

#### Scenario: Configuration validation only
- **WHEN** a user invokes config validate
- **THEN** the command SHALL report configuration errors and exit without starting an adapter, agent, or backend.

#### Scenario: Structured diagnostic failure
- **WHEN** `doctor --json` detects an unresolved binding
- **THEN** stdout SHALL remain one parseable redacted result with an unresolved-binding error kind, stderr SHALL carry any progress diagnostics, and the command SHALL return the documented failure status.

### Requirement: Deterministic adapter selection
Adapter selection SHALL prioritize explicit command/configuration choices, then verified child executable identity, then corroborated environment/session evidence. Conflicting explicit selectors SHALL fail; ambiguous inferred candidates SHALL remain unresolved and request an explicit selector. Executable names alone MUST NOT imply a live session binding.

#### Scenario: Environment conflicts with explicit adapter
- **WHEN** a user explicitly selects the generic adapter in an environment suggesting Codex
- **THEN** the generic adapter SHALL be selected and incompatible session evidence SHALL be rejected rather than silently retargeted.

### Requirement: Safe attach semantics
Attach SHALL require enough evidence to resolve a target and SHALL expose the selected adapter, session, mode, and binding strength through inspection. Attaching or detaching MUST NOT change agent state, send signals to the observed agent, or take its input stream.

#### Scenario: Attach is interrupted
- **WHEN** a user interrupts a HUD attached to an existing native CLI
- **THEN** only HUD-owned observation and presentation resources SHALL stop and the agent SHALL continue.

### Requirement: Transparent wrappers and run
Run and yagent/yodex wrappers SHALL preserve the child argument vector, environment, cwd, signals, and exit status, apart from documented additive HUD identity variables. They MUST NOT interpolate child arguments through a shell. User-directed termination SHALL be forwarded exactly once according to the documented run mode; passive HUD view exit MUST NOT terminate the agent. Interactive HUD output SHALL use a terminal surface distinct from the native agent; conflicting output selection MUST fail before launch.

#### Scenario: Shell metacharacters in child arguments
- **WHEN** a child argument contains spaces, quotes, dollar signs, or semicolons
- **THEN** the child SHALL receive the exact argument bytes without shell evaluation.

#### Scenario: Child fails with a specific exit code
- **WHEN** the child exits with code 17
- **THEN** run SHALL return code 17 after cleaning up only owned HUD resources.

#### Scenario: Same terminal selected for HUD and agent
- **WHEN** a native run requests interactive HUD output on the child's input/output terminal
- **THEN** it SHALL fail before launching the child and explain the separate-terminal or explicit plain-output workflow.

### Requirement: Scoped identity propagation
Run SHALL generate a launch ID and propagate it using documented additive environment variables or a private local ingress descriptor. Child/subagent launch relationships SHALL preserve parent identity without conflating session IDs. Existing conflicting identity variables SHALL be rejected or explicitly replaced with an inspectable diagnostic; they MUST NOT silently attach to another HUD.

#### Scenario: Nested wrapper invocation
- **WHEN** a wrapper launches a child within an already identified agent session
- **THEN** it SHALL create a distinct launch ID and preserve the parent relationship rather than reuse the parent's session binding.

### Requirement: Inspection and local export
Inspect and doctor SHALL provide human-readable and structured local output including capabilities, availability, privacy state, binding evidence/generation, provenance/age, reconciliation, errors, and backend health. Snapshot export SHALL be explicit, redacted, schema-versioned, and tied to one revision. Diagnostics MUST NOT upload automatically.

#### Scenario: Export during rollover
- **WHEN** a user requests a snapshot as session rollover occurs
- **THEN** the output SHALL contain one complete pre-rollover or post-rollover revision and MUST NOT mix them.

### Requirement: Inspectable reversible shell integration
zsh, bash, and fish integration SHALL be supplied as inspectable templates with explicit installation and removal instructions. No default command SHALL silently rewrite startup files. Wrapper behavior SHALL be verified in each declared shell.

#### Scenario: Requesting a shell template
- **WHEN** a user requests the fish integration template
- **THEN** the command SHALL print or export the template without editing the user's fish startup files.

### Requirement: Explicitly targeted view controls
Inspect SHALL support expand, collapse, freeze, and resume actions targeting a HUD instance. With multiple matching views it SHALL require an explicit HUD ID. View actions SHALL affect only presentation and MUST NOT send agent commands or intercept agent input.

#### Scenario: Freeze one of two views
- **WHEN** `inspect --hud-id <id> --freeze` selects one of two running HUD views
- **THEN** only that view SHALL freeze while both sources continue ingestion and both agents retain input ownership.
