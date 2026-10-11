## Purpose

Define useful common agent, project, turn, context, accounting, and activity measurements without treating unequal provider telemetry as interchangeable or inventing unsupported values.

## ADDED Requirements

### Requirement: Agent model and role identity

The HUD SHALL expose verified agent/CLI name and version, adapter, integration mode, provider, model name, model role, and reasoning effort/thinking level when supported. Provider effort labels MUST retain their provider meaning rather than being converted into an assumed common scale. Unknown versions, roles, or effort MUST remain unknown.

#### Scenario: Provider-specific reasoning label
- **WHEN** an adapter reports a model and a provider-specific thinking label
- **THEN** the HUD displays the reported label with provider provenance without claiming equivalence to another provider's effort level

### Requirement: Distinct project and Git scopes

The HUD SHALL distinguish launch directory, current agent directory, project root, repository root, worktree identity, and displayed folder. Locally verified Git data SHALL include branch or detached-head state, dirty state, and file counts when available. Ahead/behind MUST use available local references only, without an implicit fetch; unavailable comparisons MUST remain unknown. Privacy policy MUST govern all directory, repository, and branch displays.

#### Scenario: Directory change into another worktree
- **WHEN** verified agent directory telemetry moves into a different worktree
- **THEN** current-directory and worktree values change coherently, launch directory remains separately identified, and old worktree Git data is invalidated

#### Scenario: Detached head without upstream reference
- **WHEN** Git reports a detached head and no comparable upstream reference
- **THEN** detached-head state is shown, ahead/behind remains unknown, and no network fetch occurs

### Requirement: Turn lifecycle and waiting reasons

The HUD SHALL represent idle, thinking, working, waiting, completed, failed, interrupted, and disconnected turn states when evidence supports them. Waiting reasons SHALL distinguish approval, user input, tool completion, rate limit, and unknown. Process existence alone MUST NOT imply thinking or working, and completion MUST NOT be synthesized from a quiet telemetry interval.

#### Scenario: Approval wait followed by interruption
- **WHEN** a turn reports approval waiting and then interruption
- **THEN** the HUD transitions from waiting with an approval reason to interrupted without implying approval was granted

#### Scenario: Quiet process-only observation
- **WHEN** an alive generic process provides no turn telemetry
- **THEN** turn state remains unknown rather than being inferred as working or completed

### Requirement: Timing and output speed semantics

The HUD SHALL expose verified turn start time, monotonic elapsed duration, last response time, and output speed when supported. An attach mid-turn without trustworthy elapsed/start continuity MUST expose unknown or explicitly estimated elapsed time. Output speed MUST include a documented unit and measurement interval, and token rates MUST require actual timed token measurements. Bytes or characters MUST NOT be labeled tokens; an insufficient interval MUST remain unknown rather than zero. A final token total without timed samples MUST NOT produce a live rate; a whole-turn average SHALL be separately labeled with its interval.

#### Scenario: Character-only stream
- **WHEN** a source exposes output characters but no token counts
- **THEN** an enabled rate display identifies characters per second and its interval, while token-per-second rate remains unavailable

#### Scenario: Final total without streaming samples
- **WHEN** a completed turn reports token totals but no timed output observations
- **THEN** live output speed remains unknown and any permitted whole-turn average is distinctly labeled with its known or estimated timing

### Requirement: Scoped context measurements

Context used, remaining, total window, and percent SHALL identify the measured context scope and quality. Derived remaining and percent MUST require compatible known operands and a positive capacity; contradictory or overflowing values MUST remain visible as anomalies rather than being silently clamped. Session cumulative token totals MUST NOT substitute for context occupancy.

#### Scenario: Context occupancy exceeds capacity
- **WHEN** a verified measurement reports 1100 used tokens in a 1000-token window
- **THEN** the HUD exposes the contradiction/overflow, identifies its scope, and does not disguise it as a healthy full meter

#### Scenario: Session totals without occupancy
- **WHEN** session token totals are known but current context usage is not
- **THEN** session totals remain available and context occupancy and percent remain unknown

### Requirement: Token accounting and compaction

The HUD SHALL distinguish session and turn totals, input/output/cache categories, cumulative versus delta observations, and reported compaction count. Category inclusion relationships and reset semantics MUST be declared; cache tokens MUST NOT be added twice to an inclusive input total. Replayed deltas, cumulative snapshots, and counter resets MUST NOT inflate totals or create fabricated negative usage. A falling cumulative counter MUST require explicit reset evidence before beginning a new accounting epoch; compaction alone MUST NOT reset lifetime token totals. Unknown category inclusion MUST prevent an asserted combined total.

#### Scenario: Inclusive cached input
- **WHEN** an adapter reports 100 input tokens including 40 cached input tokens and 20 output tokens
- **THEN** the combined known total is 120, cached input remains separately visible, and the HUD does not report 160

#### Scenario: Counter reset after compaction
- **WHEN** an adapter reports a lower cumulative counter with separately verified scope/reset evidence after a compaction
- **THEN** the HUD applies the declared scope/reset semantics without subtracting negative usage or inferring unreported compaction totals

#### Scenario: Falling counter without reset proof
- **WHEN** a cumulative lifetime counter decreases or context compaction occurs without evidence of an accounting reset
- **THEN** the HUD exposes contradictory accounting and reconciles rather than starting an invented epoch or clearing lifetime usage

### Requirement: Descriptive permission and collaboration telemetry

Permission, approval policy, sandbox, and collaboration mode SHALL be displayed only when observed and labeled as descriptive telemetry. A missing policy MUST NOT become an assertion of permissive, enforced, or secure behavior.

#### Scenario: Unsupported sandbox telemetry
- **WHEN** an adapter cannot verify sandbox mode
- **THEN** sandbox mode remains unknown and the HUD makes no enforcement claim

### Requirement: Tools extensions and subagent activity

Observed tool activity SHALL carry stable call IDs, names, targets when permitted, start/duration, completion, and error states; arguments MUST be omitted or redacted by default. A completion without a verified start MUST have unknown duration, and duplicate completion MUST NOT invent another call or success. Synthetic IDs SHALL be labeled inferred and MUST NOT join uncertain cross-source calls. Skills, plugins, and MCP servers SHALL distinguish configured, loaded, and active states. Subagents SHALL include verified parent identity, model, and status where supported; parent and child totals MUST remain separate unless explicit inclusion/exclusion semantics and complete child coverage permit aggregation. Incomplete coverage SHALL be visible rather than presented as a complete session total.

#### Scenario: Repeated tool name with separate calls
- **WHEN** two calls to the same tool run concurrently under distinct call IDs
- **THEN** their durations and completions remain separate and arguments are hidden under default privacy settings

#### Scenario: Configured extension and active subagent
- **WHEN** a plugin is configured but unused and a subagent reports independent usage
- **THEN** the plugin is not labeled active and child usage is not added to a parent's possibly inclusive total

#### Scenario: Completion without start and incomplete children
- **WHEN** a tool completion has no known start and only some disjoint child totals are available
- **THEN** completion is shown with unknown duration and any child subtotal is labeled partial rather than a complete session total

### Requirement: Plans goals summaries and anomaly reporting

The HUD SHALL expose supported current plan/todo item, completion count, goal, session title, and turn count with explicit scope and privacy classification. Compact conversation summary display MUST require explicit field permission, while retention MUST require its own separate opt-in; summaries MUST originate from verified adapter input without implicit network summarization. Permitted plan counts SHALL remain usable when item/goal text is withheld. Context overflow, contradictory measurements, dropped-event accounting, and unsupported activity claims SHALL produce bounded typed anomaly records identifying the affected scope/revision, visible in inspection and compact indicators without raw telemetry bodies.

#### Scenario: Summary supplied with default privacy
- **WHEN** an adapter supplies a conversation summary, goal, and title under default configuration
- **THEN** sensitive content is hidden and absent from retained diagnostics and exports

#### Scenario: Plan counts without item content
- **WHEN** three of five plan items are complete but plan/goal text disclosure is disabled
- **THEN** the permitted completion count remains available while item text, goals, and titles stay withheld

#### Scenario: Unsupported activity claim
- **WHEN** an adapter sends active-skill data without declaring or validating that capability
- **THEN** the claim is rejected or marked unavailable and an actionable redacted anomaly is exposed
