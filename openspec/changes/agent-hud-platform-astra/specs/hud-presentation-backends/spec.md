## Purpose

Present trustworthy agent state in accessible terminal layouts while preserving the agent's focus, process lifetime, and terminal integrity.

## ADDED Requirements

### Requirement: Configurable bounded layout
The HUD SHALL support one through ten lines, configurable field order, separators, width, and compact, expanded, full, presentation, plain-output, monochrome, and privacy profiles. Layout SHALL deterministically prioritize freshness/health, agent, model, turn, project, and branch over optional details. At widths too small for those fields, it SHALL preserve a textual health marker and expose omitted fields through inspection.

#### Scenario: Narrow one-line display
- **WHEN** the terminal cannot fit all configured fields on one line
- **THEN** the HUD SHALL drop lower-priority optional fields, truncate at display-cell boundaries, and preserve a health marker without wrapping beyond the assigned width.

#### Scenario: Invalid height
- **WHEN** configuration requests eleven lines
- **THEN** validation SHALL reject the value with the supported one-through-ten range.

### Requirement: Color-independent accessibility
Themes SHALL support terminal status-line colors including a Claude-like palette mapping. Color SHALL be independently disableable. Monochrome, high-contrast, ASCII-symbol, reduced-motion, and TERM=dumb behavior SHALL preserve textual meanings; critical state MUST NOT depend only on color or animation.

#### Scenario: Dumb terminal
- **WHEN** TERM is dumb
- **THEN** output SHALL use plain ASCII text without cursor movement, alternate screen, color, or animation controls.

### Requirement: Unicode-safe cell measurement
Layout SHALL measure terminal display cells and preserve grapheme boundaries for wide characters, combining marks, and common emoji sequences. Truncation MUST NOT split a grapheme or exceed the configured cell width; ASCII mode SHALL provide stable alternatives where terminal glyph support differs.

#### Scenario: Combining mark at truncation boundary
- **WHEN** a project label contains a wide character and a combining sequence at the available width boundary
- **THEN** truncation SHALL preserve complete graphemes and stay within the width budget.

### Requirement: Terminal-control sanitization
All telemetry and configurable text SHALL be sanitized before terminal output, including ANSI/OSC/DCS sequences, carriage returns, embedded newlines, and bidi/control characters that could misrepresent status. Only trusted renderer-generated control sequences SHALL affect terminal state.

#### Scenario: Malicious branch name
- **WHEN** a branch value contains an OSC clipboard sequence followed by a newline and forged status text
- **THEN** the HUD SHALL render an inert bounded label and MUST NOT change the clipboard, execute controls, or inject extra status lines.

### Requirement: Standalone terminal modes
The standalone backend SHALL provide an interactive alternate-screen view and append-only plain output. Non-TTY output SHALL default to plain mode. Orderly exits and catchable signals SHALL restore terminal modes and cursor state; unsupported interactive operation SHALL return an actionable error or use plain output.

#### Scenario: Redirected output
- **WHEN** stdout is redirected to a file without an explicit compatible output selection
- **THEN** the HUD SHALL emit bounded plain records without terminal control sequences.

#### Scenario: Interactive interrupt
- **WHEN** the user interrupts an interactive standalone HUD
- **THEN** the backend SHALL restore terminal state before exiting.

### Requirement: Tmux ownership and lifecycle
The tmux backend SHALL bind to an explicit server/session/target pane and mark created HUD resources with an instance ownership token. It SHALL resize only owned HUD panes, preserve agent focus by default, detect server restart, and validate identity before cleanup. It MUST NOT kill, reconfigure, or reuse an unowned pane based solely on a reused pane number.

#### Scenario: Pane identifier is reused
- **WHEN** the tmux server restarts and assigns an old HUD pane ID to an unrelated pane
- **THEN** cleanup SHALL leave that pane intact and report that ownership could not be verified.

#### Scenario: Creating and closing a HUD pane
- **WHEN** a HUD pane is created beside an agent and later closed
- **THEN** agent focus SHALL remain unchanged at creation and the agent process SHALL remain alive after closing the HUD.

### Requirement: Passive presentation lifetime
Rendering failure, HUD view closure, and restart of HUD presentation code MUST NOT terminate or corrupt the agent; the surviving session owner SHALL preserve usable agent I/O and permit explicit view reattachment. The backend SHALL own only presentation resources; a controlled-session transport SHALL have a separate lifetime owner. Loss of the agent's hosting terminal, entire tmux server, or session supervisor is a distinct failure outside the view-loss survival guarantee and SHALL be diagnosed without claiming continued agent health. Terminal recovery instructions SHALL cover uncatchable presentation-process loss.

#### Scenario: Renderer crashes during a turn
- **WHEN** the renderer process fails during an active agent turn
- **THEN** the agent SHALL continue and presentation SHALL be reconnectable to a reconciled snapshot.

#### Scenario: Entire hosting tmux server is lost
- **WHEN** the tmux server hosting both agent and HUD terminates
- **THEN** recovery SHALL revalidate process and terminal identity, report the actual hosting failure, and MUST NOT claim the agent survived merely because HUD reconnection is possible.

### Requirement: Local interaction and freeze
Expand, collapse, freeze, unfreeze, and inspect actions SHALL consume keyboard input only when the HUD owns the focused input surface. Freeze SHALL pin a displayed revision with a visible frozen marker and continuing age indication while ingestion continues; unfreeze SHALL show the latest coherent revision.

#### Scenario: Agent receives input while HUD is unfocused
- **WHEN** a key bound to a HUD action is typed in the agent pane
- **THEN** the key SHALL reach the agent unchanged and MUST NOT trigger a HUD action.

#### Scenario: State changes while frozen
- **WHEN** a turn completes while the display is frozen
- **THEN** the frozen frame SHALL retain its revision and marker, and unfreezing SHALL reveal the completed state without replaying stale intermediate frames.
