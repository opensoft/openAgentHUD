## Purpose

Render compact, accessible, privacy-safe agent state in configurable terminal layouts while preserving the agent's focus and isolating HUD-owned terminal resources.

## ADDED Requirements

### Requirement: Configurable bounded line layouts

The HUD SHALL support selectable heights from one through ten lines, ordered fields, separators, configurable width, and compact, expanded, full, presentation, and plain-output profiles. Layout results MUST be deterministic for a given state revision, configuration, and terminal size. Overflow MUST truncate or omit fields by documented priority, preserving agent, model, turn, project, branch, and freshness before lower-priority details whenever space permits. At widths too small for those fields, truncation and freshness/anomaly state MUST remain explicit.

#### Scenario: Narrow one-line display
- **WHEN** a full-profile state is rendered into a narrow one-line terminal
- **THEN** lower-priority usage/activity details are removed first, output fits the display width, and freshness is retained or replaced by its documented compact marker

#### Scenario: Ten-line expansion
- **WHEN** the user selects ten lines and a terminal with sufficient height
- **THEN** at most ten HUD content lines are rendered with the configured order and no interference with agent input

### Requirement: Truthful status presentation

The HUD SHALL distinguish live, cached, stale, disconnected, frozen, inferred, estimated, unsupported, unavailable, and redacted information with text or symbols and inspection details. Critical lifecycle, waiting, freshness, and anomaly state MUST NOT rely on color alone. State inspection SHALL include source age and provenance subject to privacy policy.

#### Scenario: Stale estimated context in monochrome
- **WHEN** estimated context data becomes stale while colors are disabled
- **THEN** both estimation and staleness remain visible using labels or symbols and the measured age is inspectable

### Requirement: Independent themes and accessible modes

The HUD SHALL provide themeable status-line colors, a Claude-like semantic palette mapping, and independently selectable monochrome, high-contrast, ASCII-symbol, and reduced-motion modes. `TERM=dumb`, disabled color, and plain output MUST avoid terminal-control styling while preserving the selected field layout and essential state labels. Reduced-motion MUST disable animated indicators without stopping telemetry ingestion.

#### Scenario: Color disabled without layout changes
- **WHEN** the user disables color on an expanded profile
- **THEN** the expanded field arrangement remains intact and critical state is readable without ANSI color

#### Scenario: Dumb terminal
- **WHEN** the HUD runs with `TERM=dumb`
- **THEN** it emits plain bounded lines with no alternate-screen, cursor-motion, or color escape sequences

### Requirement: Unicode display width and truncation

Width calculation and truncation SHALL account for wide characters, combining marks, and common emoji grapheme sequences. Truncation MUST preserve complete grapheme clusters, respect the terminal display width, and avoid rendering a partial escape/control sequence. ASCII mode MUST replace HUD symbols with readable ASCII alternatives.

#### Scenario: Wide project label with emoji
- **WHEN** a project label contains CJK characters, combining accents, and a joined emoji sequence near the width limit
- **THEN** the line fits the configured display columns without splitting graphemes or overwriting neighboring content

### Requirement: Sanitized untrusted telemetry

All telemetry-derived text SHALL be sanitized before terminal rendering, including names, paths, branches, extension fields, and errors. Adapter content MUST NOT emit executable terminal controls, OSC hyperlinks/clipboard operations, cursor movement, or unexpected newlines. Only presentation-owned control sequences SHALL be emitted in supported interactive modes, and plain output MUST contain no such controls.

#### Scenario: Escape injection in branch name
- **WHEN** a branch label contains OSC clipboard commands and cursor-control sequences
- **THEN** those controls are removed or visibly escaped, no clipboard/cursor action occurs, and the safe branch label remains subject to privacy policy

### Requirement: Standalone terminal lifecycle

The standalone backend SHALL support alternate-screen and append-only/plain modes, resize events, and restoration of terminal modes/cursor state on normal exit and catchable termination. Non-TTY HUD output SHALL default to plain mode, group meaningful lines by revision, and remain bounded by coalescing display frames without dropping accounting input. The alternate-screen backend MUST own a terminal surface separate from the agent's input surface during `run`. Closing or failing the HUD MUST NOT terminate or corrupt the agent. Fatal process loss SHALL have a documented terminal-reset recovery path.

#### Scenario: Standalone resize and termination
- **WHEN** a standalone HUD is resized and then receives a stop signal
- **THEN** the layout is recomputed within bounds, terminal state is restored, and the observed agent remains running

#### Scenario: Redirected high-rate output
- **WHEN** a standalone HUD receives a burst while stdout is not a TTY
- **THEN** it emits bounded sanitized plain revision groups with no controls and coalesces only presentation while accounting remains complete or explicitly uncertain after overflow

### Requirement: Explicit tmux ownership and recovery

The tmux backend SHALL target an explicit server/socket, session, and pane or a verified launch pane. It MUST tag HUD-created resources with HUD-instance ownership, preserve agent focus by default, honor one-to-ten-line height, and clean up only resources whose ownership still matches. Server restart, pane replacement, and stale pane IDs MUST cause revalidation rather than deletion or attachment to an unrelated pane.

#### Scenario: Pane ID reused after server restart
- **WHEN** tmux restarts and an old HUD pane ID refers to an unowned pane
- **THEN** cleanup leaves the unowned pane intact and the HUD reports backend disconnection until its target is explicitly revalidated

#### Scenario: Split created beside an active agent
- **WHEN** the HUD creates its tmux pane next to an agent pane
- **THEN** keyboard focus remains with the agent and HUD teardown removes only the tagged HUD pane

### Requirement: Inspection expansion and frozen view

Users SHALL be able to expand, collapse, inspect, freeze, and resume the HUD through commands or keys delivered to the HUD's own focused input surface. The HUD MUST NOT intercept agent-directed keys. Freezing MUST pin the displayed revision and timestamp while ingestion and reconciliation continue; resume MUST publish the latest valid revision with current freshness. Privacy reload MUST redact even a frozen view immediately.

#### Scenario: Freeze across rollover and resume
- **WHEN** the user freezes a view, the live session rolls over, and the user resumes
- **THEN** the frozen view is labeled with its pinned age and resume displays the latest generation without replaying obsolete session state

#### Scenario: Agent receives an expansion key
- **WHEN** a key matching a HUD shortcut is typed into the agent's terminal surface
- **THEN** the key reaches the agent unchanged and does not invoke a HUD action
