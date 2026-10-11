## Purpose

Provide explicit, inspectable commands for launching or observing agents, controlling the HUD view, validating configuration/adapters, and exporting safe local diagnostics without changing agent behavior incidentally.

## ADDED Requirements

### Requirement: Explicit command surface and errors

The `openagenthud` executable SHALL provide `attach`, `run`, `inspect`, `doctor`, `adapters list`, `adapters validate`, `config validate`, explicit `config migrate`, an offline `demo`, and inspectable `shell-integration` templates. Commands MUST document options, mode, origins, and prelaunch exit statuses: 0 success, 1 operational failure, 2 invalid usage/configuration/input, 3 unsupported adapter/schema, and 4 unresolved binding. Once a child starts, run MUST report its exit/signal result instead of substituting HUD command codes. Machine-readable output MUST contain one valid redacted JSON result on stdout with diagnostics on stderr. Help and validation MUST NOT launch agents/backends or query providers.

#### Scenario: Machine-readable inspection
- **WHEN** a user invokes `inspect --json` for a running HUD
- **THEN** stdout contains a valid permitted snapshot/diagnostic result from one revision and no progress text or raw provider records

#### Scenario: Ambiguous noninteractive attach
- **WHEN** noninteractive attach finds multiple matching sessions
- **THEN** it exits with status 4 and redacted explicit-selection guidance without creating a backend or choosing a candidate

### Requirement: Safe attach and deterministic adapter selection

`attach` SHALL observe an existing session without launching or modifying the agent. Adapter selection SHALL prefer explicit arguments/configuration, then verified child identity, correlated environment/session evidence, and compatible executable evidence. Conflicting evidence or multiple valid candidates MUST produce an unresolved result with explicit selection guidance. Plain executable names MUST NOT override an explicit verified adapter/session selection.

#### Scenario: Explicit selection conflicts with environment
- **WHEN** an explicit Codex session is selected but environment hints identify another agent
- **THEN** the explicit selection is validated and conflicting hints are reported without silently selecting the other agent

#### Scenario: Attach without sufficient identity
- **WHEN** `attach` discovers two otherwise matching sessions
- **THEN** it returns unresolved selection information and neither launches a new agent nor displays one candidate's state as selected

### Requirement: Transparent native run and wrapper commands

`run -- <executable> <arguments...>` SHALL launch the exact argument vector without implicit shell evaluation. Native wrapper mode MUST preserve the child's existing environment and working directory, add only documented HUD correlation variables, relay signals without duplicate delivery, and report the child's exit status or signal termination. `yagent` SHALL delegate to generic run behavior and `yodex` SHALL select Codex native observer mode by default. Controlled app-server mode MUST require a separate explicit option and literal server argv; its minimal built-in control client MUST remain separate from the HUD view. Without tmux or an explicit separate HUD TTY/output, native run SHALL print a redacted second-terminal attach instruction before child startup and preserve the child's original terminal, rather than inserting ongoing HUD output into its TUI.

#### Scenario: Literal arguments and nonzero exit
- **WHEN** a child is launched with spaces, wildcard characters, and command-substitution-looking literal arguments and exits with status 7
- **THEN** it receives the exact argument bytes, no shell expansion occurs, and the wrapper returns status 7

#### Scenario: Interrupt forwarded once
- **WHEN** a user interrupts a running child through the supported wrapper path
- **THEN** the child receives one intended interrupt and the HUD reports the child's resulting termination behavior

#### Scenario: Native run outside tmux
- **WHEN** native run has no separate HUD output surface configured
- **THEN** a copyable launch-scoped attach command is printed before startup, the exact child argv is launched, and agent input/output remain free of ongoing HUD rendering

### Requirement: HUD and child lifetime separation

An attached HUD SHALL stop independently of its observed agent. A launched session's host MUST preserve the child's terminal/control transport if the HUD view closes or crashes; ending only the HUD MUST NOT send termination signals to the agent. The wrapper MUST distinguish an explicit agent stop from a display stop, release HUD-owned resources, and retain recoverable session identity while the child remains active.

#### Scenario: HUD pane closes during native run
- **WHEN** the user closes only the HUD pane while its launched agent is still running
- **THEN** the agent continues with usable input/output and a new HUD can explicitly reattach using the surviving session identity

### Requirement: Session-scoped correlation and ingress

The CLI SHALL propagate documented HUD-instance, launch/session correlation, and a private local ingress descriptor through supported wrappers/templates. Authentication material MUST remain in an owner-private descriptor rather than ordinary inherited environment values or visible argv. Nested launches MUST receive a new launch ID and preserve parent launch identity without asserting provider-session or subagent identity from environment inheritance. Conflicting active endpoint identities MUST be rejected with redacted guidance. Ingress discovery MUST validate ownership, source leases, and generation; short-lived producers MUST correlate without PID-only discovery, and arbitrary environment content MUST NOT be retained or exported.

#### Scenario: Short-lived status producer
- **WHEN** a hook inherits the current launch key and ingress address
- **THEN** it can publish a validated sparse update to that launch without becoming the durable agent identity

#### Scenario: Nested wrapper lineage
- **WHEN** run is invoked inside another identified launch
- **THEN** the new launch preserves the parent relationship, receives a distinct private descriptor, and does not reuse the parent's provider session or authentication lease

### Requirement: Local inspection doctor and export

`inspect` SHALL expose redacted binding evidence, generation, scopes, provenance, availability, freshness, source age, controller state, and current layout. `doctor` SHALL report adapter/source/backend versions, compatibility, last reconciliation, retries, parse errors, dropped updates, lag, render latency, resource use, and actionable recovery. Users SHALL be able to copy redacted diagnostics and explicitly export a local structured snapshot. These commands MUST NOT upload data, mutate agent state, or dump raw credentials/logs.

#### Scenario: Diagnosing a stale source
- **WHEN** `doctor --json` examines a degraded HUD
- **THEN** it identifies the failed source, redacted binding evidence, age, retries, last reconciliation, and recovery guidance without transmitting the report

### Requirement: View controls respect input ownership

`inspect` SHALL support explicit expand, collapse, freeze, and resume actions against a selected HUD instance. Commands and keyboard actions MUST target only that HUD, keep the agent's input ownership intact, and use the same privacy projection as rendering/export.

#### Scenario: Freeze selected instance
- **WHEN** two HUD instances exist and `inspect --hud-id <id> --freeze` targets one
- **THEN** only the selected view freezes and neither agent receives a control command or intercepted key

### Requirement: Reversible shell integration

The distribution SHALL include inspectable zsh, bash, and fish templates for wrappers and supported hook correlation. Templates MUST preserve argument quoting, child status, environment, and working directory; installation/removal MUST be explicit and reversible. No ordinary command or installation step MUST silently rewrite shell startup files or overwrite an existing agent status-line command.

#### Scenario: Removing a sourced template
- **WHEN** a user removes the documented template include from a shell configuration
- **THEN** HUD wrapper functions disappear without altering the agent executable, unrelated shell configuration, or existing status-line integration
