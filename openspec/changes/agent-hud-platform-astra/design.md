## Context

See `proposal.md` for motivation and the alternative-proposal status. The repository contains documentation, an Apache-2.0 license, OpenSpec configuration, and scaffolds. There is no executable, dependency manifest, source tree, test suite, or existing main capability specification to preserve. `FEATURES.md` is the canonical expanded roadmap; the architecture document supplies layering, source preference, rollover, and privacy constraints. Neither is changed by this proposal. The peer-review revision uses Sol's planning ideas as identified in `peer-review.md`; neither proposal establishes provider protocol facts. No web or external sources were used for this review.

The nine delta specifications define acceptance behavior. Provider payload examples, database schemas, app-server methods, and exact supported CLI versions are not established by the supplied documents. Implementation must obtain local, redacted fixtures and verify each format before enabling it. This is a required bounded integration task, not permission to guess protocol fields. The architecture's description of Codex sources is a proposed integration direction, not evidence that an arbitrary installed version implements a particular schema.

## Goals / Non-Goals

**Goals:**

- Make the normalized contract and deterministic reducer the stable integration boundary.
- Keep provenance, identity, scope, ordering, privacy, and source health available independently of layout.
- Permit recovery without unbounded rescans, accidental rebinding, duplicate accounting, or cross-session frames.
- Choose concrete process ownership, configuration, wire formats, budgets, delivery stages, and acceptance gates before implementation.

**Non-Goals:**

- No agent CLI patches, terminal-pixel scraping, provider credential replication, or invented cross-provider parity.
- No general agent frontend in attach/native-run workflows. The explicitly invoked Codex app-server client is a separate limited integration with its own approval/input responsibilities.
- No roadmap “Later” features: shared multi-consumer service, remote transport, notifications, history charts, public plugin SDK, additional controlled frontends, cross-machine handoff, desktop widgets, or sandbox helper. Private per-launch IPC is an internal lifecycle mechanism, not a public state bus.
- Initial supported OS/architecture pairs are Linux x86_64 and macOS arm64. Other pairs, including native Windows, remain unverified; do not infer support from the operating-system name alone.

## Decisions

### 1. TypeScript core with explicit module boundaries

Use strict TypeScript, ESM, a Node.js 24 baseline, npm with a committed lockfile, and the built-in Node test runner. This is a selected implementation baseline, not a claim about a currently installed runtime. Initial clean-platform/reference targets are Ubuntu 24.04 x86_64 and macOS 14 arm64, with tmux 3.3+ optional. Pin and verify exact runtime/backend/dependency builds on both native targets during bootstrap; broad Node-major or tmux-version syntax is not proof that every later version is tested. Use JSON Schema 2020-12 as the published wire/config contract, generated TypeScript types, and a pinned schema validator. Use a small pinned terminal-width implementation with grapheme segmentation; application code owns layout and sanitization. Read version-sensitive SQLite sources through a read-only connection inside an adapter worker with bounded busy timeout. Verify the baseline API and dependency compatibility during bootstrap.

Proposed ownership:

| Path | Responsibility |
| --- | --- |
| `src/contracts/`, `schemas/` | Schema versions, field dictionary, handshake, observation envelopes, compatibility |
| `src/core/` | Identity registry, source controller, immutable reducer, accounting, clocks |
| `src/adapters/` | Generic, Codex observer, Codex app-server, Claude, Gemini, Aider |
| `src/privacy/`, `src/config/` | Field policy, sink projection, validation, reload, explicit migration |
| `src/presentation/` | Provider-independent field catalog, layout, width, themes, sanitization |
| `src/backends/` | Standalone and tmux ownership, resize, terminal restoration |
| `src/cli/`, `src/runtime/` | Commands, supervisor, private IPC, signals, inspection |
| `src/usage/`, `src/diagnostics/` | Opt-in usage queries, cost estimation, bounded logs/metrics |
| `tests/fixtures/`, `tests/contract/`, `tests/replay/`, `tests/integration/` | Protocol evidence and reproducible gates |

The renderer cannot import adapters, filesystem observers, provider databases, or provider-specific types. Enforce this dependency rule with an import-boundary check. Built-in adapters run as trusted application code with worker isolation for blocking or untrusted parsing; worker isolation is a resource-control boundary, not a security sandbox.

Alternative: Rust/Go could reduce resident memory and simplify binary distribution, but would add integration/schema tooling cost before the source formats are stable. A large component TUI framework would simplify widgets but obscure width, control-sequence, and focus behavior. The small renderer and separate backend are the deliberate choices for a ten-line maximum HUD.

### 2. Contract version 1 uses explicit operations and field cells

The wire handshake contains `schemaVersions`, `adapterId`, `adapterVersion`, `cliName`, nullable `cliVersion`, `integrationMode`, and a list of capabilities with supported field groups and scope kinds. Compatibility is same major plus supported mandatory features; minor additions cannot reinterpret existing fields. Unknown namespaced extensions are size-checked and retained only if policy permits; unknown common keys are rejected to catch misspellings.

Each observation envelope contains:

```text
schemaVersion, adapterId, sourceId, sourceEpoch, eventId,
generation, scope { kind, key }, sequence,
observedAt?, receivedAt, operation, payload
```

`receivedAt` and monotonic receipt/validation ticks are stamped by the trusted ingress. External producers cannot set the active binding generation: the controller stamps only the generation/epoch captured in a lease issued after source/session evidence is validated. It never stamps a delayed payload with whichever generation happens to be current. Sequence is increasing within `(sourceId, sourceEpoch)`; no cross-source sequence comparison is permitted. Source epoch changes require a covered baseline or verified complete replay before delta accounting becomes authoritative. Event IDs remain stable on replay.

Operations are `snapshot`, `patch`, `invalidate`, `source-health`, and `capabilities`. A snapshot declares its owned field groups and replaces only those groups for one scope. Fields omitted within that declared snapshot become unknown. Patch payloads are explicit maps of field paths to values; omission is no update, and null clears. Lists such as tools, agents, and quotas use keyed upsert/remove operations, not positional array patches. Removal requires an explicit key. Invalidation names groups and a reason; last-known values can remain visibly stale but cannot feed live-derived values.

Validate path ownership, allowed namespace, complete coherent groups, and mutually exclusive set/clear/upsert/remove operations before committing. Reject prototype-mutating keys such as `__proto__` and traversal through `constructor`/`prototype` at the mutation boundary. Capability removal invalidates that source's dependent cells and derived results in one transaction; independent conforming source evidence remains eligible under arbitration. Source-health renewals affect only applicable validation metadata, never observation timestamps. Sol's covered snapshots and atomic withdrawal make these failure cases explicit without replacing Astra's keyed collections.

Each normalized cell contains `value`, `availability` (`known`, `unknown`, `not-applicable`, `error`), independent `disclosure` (`visible`, `redacted`), `quality` (`reported`, `derived`, `estimated`, `inferred`), and provenance. Capability support (including unsupported) and connection health are independent metadata. Redaction removes the value and restricted provenance from projections without conflating support, availability, or health; safe metadata can remain inspectable. Freshness has `live`, `cached`, or `stale`; disconnected is connection state, frozen is view state, and estimated/inferred are quality labels. Their independent representation avoids impossible single-enum combinations; presentation can show combinations such as `estimated · stale`.

State contains `schemaVersion`, `revision`, `generation`, identity, project, model, turn, context, accounting, permissions, activity, usage, capabilities, anomalies, and source health. Reducer transactions validate complete coherent groups and publish a new frozen object with structural sharing. A frame/export captures a revision reference once. Derived cells record input revisions and inherit the weakest availability/freshness/quality needed for the calculation.

Alternative: one mutable object and JSON merge-patch are simpler initially but obscure ownership, explicit clears, keyed collections, and atomic rollover. The explicit operation contract is chosen to make replay and third-party generic producers testable.

### 3. Scope keys and binding are evidence-based

Generate a random HUD instance ID and a random launch ID. Track OS boot/process-start identity plus PID; use platform-specific process-start probes whose output is bounded. Track tmux server socket identity, server-start evidence, session ID, pane ID, and ownership token separately. Working-directory equality is corroborating evidence only.

An explicit adapter/session selector is authoritative when its source ownership can be verified. A launched child is associated through launch ID, process-start identity, and adapter-reported session evidence. For attach, intersect exact constraints; never use a “latest rollout” tie breaker. Zero matches remains discovering; multiple matches yields unresolved binding and a redacted candidate list. Noninteractive ambiguous attach exits with usage/selection error code 2. Interactive selection is confined to the HUD input surface.

Maintain scope keys for account, worktree, session, turn, and subagent. Keys use opaque adapter-issued identity or per-instance salted hashes; raw paths/account identifiers are not exported. Account identity must be revalidated before retaining account quotas across rollover. Worktree keys include canonical repository/worktree identity, not branch name alone. Unknown account identity disables durable quota reuse.

Lifecycle transitions: discovering → binding after a unique candidate; binding → synchronized after a validated baseline snapshot; source loss or inconsistent continuity → degraded; lost transport/process with no valid live source → disconnected; explicit shutdown → stopped. Recovery passes through binding/reconciliation. Binding generation increments on any detach/rebind, even to the same session; accounting epoch can remain continuous only after validated reconciliation.

Rollover is staged: verify candidate identity, allocate generation, cancel old subscriptions, load a bounded baseline, assemble cleared session/turn/activity state, retain matching independent scopes, publish one revision, then activate generation-tagged subscriptions. Buffer candidate events during staging with the same queue limits. If the baseline fails, publish an unresolved/degraded new binding with unknown session fields or keep the old binding visibly disconnected until identity is confirmed; never relabel old values as the new session.

Every file read, database job, poll, retry completion, and health probe captures both generation and source epoch. Check them before reduction or renewal of health/counters; stale work contributes only to a separate bounded rejected-result diagnostic. Rotation or reconnect can retire an epoch without changing the session. Producer leases retire with their captured generation/epoch and cannot be refreshed from the launch nonce alone. A delayed hook must provide evidence of the current session or remain unresolved; process inheritance only establishes launch lineage.

Alternative: choosing newest process/file is convenient for one session but unsafe with two sessions in one repository and PID reuse. Strong evidence and explicit ambiguity are mandatory.

### 4. Reconciliation and field precedence are explicit

Default file debounce is 50 ms (validated range 25–100 ms), fallback polling 1 s (250 ms–30 s), identity/source reconciliation 5 s (1–60 s), and stable-field freshness TTL 15 s (at least twice its validation cadence). Poll timers are jittered by up to 10 percent, with one active operation per source. Successful reconciliation validates only the groups that the source can attest to.

| Group | Preferred evidence | Fallback and overwrite rule |
| --- | --- | --- |
| Session identity | Explicit controlled session or correlated launch/session evidence | Verified source ownership; no timestamp-only inference |
| Model/effort | Verified model-selection event for current session | Durable record only if it can prove it is not older; otherwise preserve event and age it |
| Turn/activity | Contiguous live events or documented hook | Validated snapshot; process liveness alone cannot prove turn completion |
| Context/accounting | Scoped authoritative usage snapshot/events | Replay with event IDs/epochs; a gap invalidates delta-derived totals |
| Project/Git | Verified agent cwd and local worktree query | Launch cwd labeled inferred; query failures cannot mean clean |
| Account quota | Verified provider observation for the same account/window | Cache retains original observation age; deadline alone is no new data |

Adapter rules translate source-specific ordering into a monotone group watermark or mark it incomparable. A lower-priority source may fill unknown fields but cannot replace a newer higher-priority field without a comparable version or a baseline reconciliation that retires the old authority. Wall-clock timestamps alone never resolve cross-source races.

Adopt Sol's read-start guard: capture the current group revision/watermark and monotonic start when dispatching a durable read, and reject regression of fields advanced while it was in flight. This is necessary but not sufficient: a read begun later may still contain old data. Superseding the event requires a checkpoint that covers it or a verified authority transition; incomparable contradictions retain the last trustworthy selection with an anomaly and its actual age.

Stable fields can remain live during idle periods when their authoritative source revalidates them. Process liveness only validates process/connection evidence. A gap-free healthy event transport with a verified heartbeat can revalidate current turn state; unsupported heartbeat assumptions are prohibited. Hook fields are live when received, become cached after 5 s without validation, and stale after their TTL. Quotas retain a provider-specific TTL (default 5 min) and receipt/observation ages. Source failure leaves values cached within TTL and stale afterward while connection state independently reports degradation/disconnection.

Use an injectable clock with monotonic ticks for intervals and separate wall time for timestamps/deadlines. Detect resume by a platform wake signal, a wall/monotonic discrepancy, or a scheduling gap greater than the stable-field validation TTL, covering systems where both clocks advance during suspension. Invalidate transport confidence before reconciliation; wall discontinuity recalculates reset countdowns, not elapsed turn duration. Freeze pins a sanitized view revision, not ingestion or privacy enforcement.

Alternative: one global heartbeat or last-file-mtime hides stale fields and misclassifies healthy idle sessions. Per-group validation costs more metadata but prevents that ambiguity.

### 5. Bounded ingestion and recovery

Initial defaults and hard ceilings are the same until measured tuning is reviewed:

| Resource | Bound and recovery |
| --- | --- |
| Record size / nesting | 1 MiB / 32 levels; reject with redacted error |
| Event queue | 2048 entries or 8 MiB, whichever first; invalidate affected stream and reconcile |
| Partial JSONL buffer | 1 MiB; oversized unfinished record invalidates source epoch |
| Parser work | 25 ms worker deadline per record; terminate/restart parser and reconcile on breach |
| Active activity records | 256 per collection and 4 MiB globally; overflow count and incomplete-coverage marker |
| Completed activity/history/cache | 256 entries, 4 MiB globally, or 15 min, whichever first; stricter provider TTL wins and content retention is off by default |
| Accounting event dedup | 4096 IDs per source epoch plus committed sequence watermark |
| Source workers/handles | At most 8 active telemetry sources; at most 2 concurrent queries globally across local subprocesses and command/HTTP usage providers, with observer work given priority |
| Initial durable replay | 16 MiB / 2 s per reconciliation slice; remain degraded with progress and continue bounded slices |
| Local subprocess | 2 s, 1 MiB combined output; kill only owned observer process group |
| External usage query | 5 s, 256 KiB output, one concurrent query/provider within the global query cap, minimum 60 s cadence; at most 16 configured providers and one pending refresh/provider |
| Retry | 250 ms exponential backoff capped at 30 s, maximum 5 attempts per reconciliation cycle |
| Structured logs | 2 files × 1 MiB, repeated errors rate-limited with exact counters |

Sequence watermarks reject old replay after dedup-ring eviction. Out-of-order future records trigger reconciliation rather than indefinite reordering buffers. Snapshot replay accumulates into staging state before publication, so partial history does not masquerade as a complete total. Sources without a snapshot can complete bounded replay across slices only if a verified start/reset and contiguous coverage prove completeness; replaying an arbitrary suffix or an uncheckpointed new epoch never restores complete totals. Overflow clears queued deltas for the affected epoch, not unrelated field groups. Query scheduling coalesces pending refreshes per provider and counts deferred work; being scheduled does not refresh a value's age. The history byte cap is needed because count-only limits can exceed the total RSS budget with large records.

Readers retain file identity, byte offset, epoch, and partial-record buffer. On truncation/rotation/replacement they re-establish identity and replay a baseline. Watchers are hints; periodic reconciliation detects missed events. Read-only SQLite access has a 100 ms busy limit per attempt inside the overall 2 s query budget; format mismatch disables that source. No repair writes, migrations, or VACUUM touch agent databases.

Alternative: unlimited replay provides apparently complete data but can block the HUD and exhaust memory. Bounded staging plus visible uncertainty preserves responsiveness and correctness.

### 6. Common telemetry and activity semantics

Publish a field dictionary containing units, ranges, scopes, clearing behavior, source capabilities, privacy class, and derivation rules. Context uses tokens; percent is `100 × used / capacity` only for compatible scope with positive capacity. Remaining can be negative during overflow and is accompanied by an anomaly. Never cap actual data to make a gauge look valid.

Turn timestamps distinguish wall-clock start and monotonic observed start. If attaching mid-turn without a trustworthy monotonic start, expose reported start time and an explicitly estimated elapsed duration or unknown elapsed, not an invented precise duration. Streaming output speed uses a rolling 5 s sample interval and labels tokens/s or characters/s; whole-turn average is a different derived field. No timed samples means no live rate.

Accounting keys contain scope, source/metric, category, cumulative/delta mode, inclusion semantics, and epoch. Cumulative snapshots replace a baseline with explicit included-event coverage; apply only compatible deltas above that checkpoint. For example, total 100 through sequence 20 plus a replay of 19 and a 5-token delta at 21 yields 105. If snapshots and deltas cannot establish nonoverlap, keep separate evidence and mark the aggregate uncertain. A decreasing cumulative counter starts an epoch only when the source proves reset; otherwise it is an anomaly and requires reconciliation. Compaction changes context occupancy but not lifetime totals. Cache categories are disjoint only when declared. Parent/child aggregation requires explicit disjoint coverage and a complete child set. Session cumulative tokens never stand in for context occupancy.

Tool calls use stable call IDs; synthetic local IDs are marked inferred and cannot be used to join uncertain cross-source records. Missing-start completion has unknown duration. Configured, loaded, and active integration states are separate. Plan counts can be displayed without item text; goals/titles/summaries follow content policy. Summaries are optional verified adapter input, never a HUD-issued summarization request. Anomalies are bounded typed records keyed by affected scope/revision, never free-form raw telemetry dumps.

Alternative: normalize every provider into a superficially uniform field set. Rejected because apparent parity would erase uncertainty and accounting differences.

### 7. Presentation and process ownership

Render from a sanitized immutable view model. Layout first reserves health/freshness text, then agent, model, turn, project, and branch, then optional fields by user order. When even core fields do not fit, truncate core values deterministically and expose omissions in inspect; at widths below 4 cells show a one-cell textual health symbol. Line count is a hard maximum; renderer-owned padding fills requested height in interactive mode. Plain mode emits only meaningful lines, grouped by revision.

Use grapheme segmentation plus cell-width tables, and test common emoji/wide/combining sequences. Strip or visibly escape all telemetry control sequences, bidi formatting controls, newlines, tabs, and carriage returns before width measurement. Only renderer-generated ANSI can reach interactive output. TERM=dumb selects ASCII/plain; NO_COLOR disables colors without changing layout. Reduced motion disables animations and uses only state-driven redraws. High contrast uses text labels as well as color. The Claude-like palette is a selectable theme, not a renderer dependency on the Claude adapter.

The renderer runs separately from the source/supervisor owner. In attach mode, observation never owns the agent process. In native run, a small supervisor launches the agent with inherited stdin/stdout/stderr and exact argv/environment/cwd; the renderer uses an owned tmux pane or a second terminal. With no tmux, run prints an attach instruction before spawning the child, exposes a private per-launch endpoint, and does not mix HUD output into the child's TUI. Explicit plain output goes to a separately selected file/descriptor. A fatal renderer error only detaches presentation.

Validate interactive terminal identity before launch and reject two interactive surfaces on the same TTY. A reconnectable owner-private registry records opaque HUD/launch IDs, endpoint and protocol version, and owner PID/start identity, never credentials. Attach hosts stop observation when their view closes; run supervisors survive view loss until child exit. Stale cleanup revalidates owner start identity and endpoint ownership. Per-view commands use a dedicated validated message class and cannot access producer or control-client actions.

Run owns child exit accounting and forwards user-directed SIGINT/SIGTERM exactly once, using terminal foreground-process-group behavior to avoid double delivery. SIGHUP/terminal-close behavior follows the documented child job lifetime; the survival guarantee is about passive HUD loss, not a user terminating the agent's terminal or supervisor. Native-run supervisor itself uses no alternate screen. PTY mode, if required by the controlled client, owns and restores only its own input surface.

Entire tmux-server loss can also destroy the agent's hosting terminal and is outside the view-only guarantee. Report actual process/terminal failure and revalidate before reattachment. Catchable exits restore terminal state; uncatchable presentation loss has documented `stty sane`/terminal-reset recovery. Do not claim cleanup can resurrect a terminated hosting process.

Tmux calls use argument arrays, a verified server socket/instance identity, target pane ID, and `@openagenthud-owner` token on created panes. Create with focus-preserving behavior, save no global options, debounce resizes, and verify the token plus server identity before cleanup. Never kill a tmux session/server. If minimum pane size prevents the requested split, explain the constraint and offer standalone attach rather than resizing the agent's unrelated panes. After server restart, rediscover without assuming IDs persist.

Alternative: render in the native agent's PTY or let a pane process own the agent. Rejected because it captures agent keystrokes and couples agent survival to presentation.

### 8. Configuration and privacy are centralized

Use JSON configuration at `$XDG_CONFIG_HOME/openagenthud/config.json`, falling back to `~/.config/openagenthud/config.json`. `--config` adds an explicitly selected file; never auto-load repository configuration. Precedence is defaults → user file → explicit file → selected profile → allowlisted `OPENAGENTHUD_*` environment settings → flags. Select the profile name by flags, allowlisted environment, explicit file, user file, then default compact; a profile cannot recursively select another profile. Profile bodies may set layout/theme/privacy fields only, never providers, commands, or binding identity. Arrays replace rather than merge; object fields merge by schema path. Display effective values and origins through redacted inspection. Environment variables cannot implicitly enable external usage commands or HTTP providers.

Validate the initial effective configuration before child/provider launch, ingress creation, or terminal changes. Watch selected files, debounce, parse/validate the complete candidate, build a new configuration revision and policy projection, then swap together. Fields requiring rebinding trigger a staged generation change after validation. Invalid reload preserves the last valid revision. Add explicit `config migrate --write` with an adjacent private backup; unknown major versions remain untouched. Use a declared synthetic predecessor configuration fixture for first-release migration tests, without implying an existing deployed v0 format.

Privacy is a typed field policy, not regex-only postprocessing. Default visible values are agent/model/status, basename project label, branch, and numeric usage/quota/cost. Default hidden values are full paths, usernames, raw account IDs, prompt/response text, tool arguments, goals, titles, and summaries. The privacy/presentation profile hides project labels, branches, and quota/cost too. Distinguish telemetry secrets from user-permitted content: credentials/tokens never become retained HUD fields regardless of consent.

Adapters hold raw records only in bounded transient parsing memory. Extract minimal internal identity material, replace it with opaque scope keys where possible, and apply field policy before observations enter retained queues/reducer/history. Unclassified extensions default to nondisclosure until a descriptor and explicit permission exist. All sinks consume a second policy projection, including source labels, scope keys, provenance, errors, tool targets, and extension values; debug logs are not exempt. Tightened policy rebuilds current/frozen views, purges prohibited cache/history entries and rotated HUD logs, and resets buffered exports before further output. Relaxing policy admits new observations only and cannot revive discarded frozen/history values. Previously exported user files cannot be recalled; state this on export. Physical secure erasure is not claimed.

Use private runtime directories (0700), files/socket endpoints (0600), ownership checks, and symlink-safe creation. Default caches are memory-only. Optional durable non-content quota cache requires explicit enablement, a stable verified account key, and a finite TTL within the shared cache limits. Optional content retention requires both per-field display permission and separate retention opt-in; default maximum when enabled is 15 min/256 entries/4 MiB shared with other retained records. Display-only permission permits the current summary without past summaries or disk persistence. Disabling retention purges owned retained content. No automatic upload.

Alternative: redact only rendered text. Rejected because logs, frozen views, caches, and inspection otherwise leak the same data.

### 9. Built-in adapters and support tiers

| Adapter/mode | Implemented approach | Evidence required before structured support |
| --- | --- | --- |
| Generic | Private file/FIFO/Unix socket; JSON snapshot or newline-framed events | Published v1 examples and full conformance suite |
| Codex observer | Read-only rollout parser; isolated optional state/log readers; process/tmux correlation | Sanitized fixtures from at least one locally available CLI version, including rollover |
| Codex controlled | Separate explicit app-server client and source adapter | Local protocol transcript including turn/input, approval, cancel, reconnect, child exit |
| Claude | Explicit status-line/command integration feeding launch-scoped ingress | Local hook payloads showing stable identity, sparse updates, null clear, concurrent sessions |
| Gemini / Aider | Executable/version probe, process-start binding, project/Git fallbacks | Local discovery/process fixtures; richer telemetry remains unsupported |

Tiers are `structured-verified`, `fallback-verified`, `unverified-version`, and `unavailable`, scoped by adapter mode and CLI version rather than brand alone. Every enabled adapter has a handshake and contract suite, including fallbacks. A missing local provider executable blocks claiming that version as verified; the generic demo remains fully testable offline. This change is not complete until structured Codex and Claude modes meet their specified fixture/smoke gates; inability to obtain evidence is reported as an implementation blocker, never solved by invented fixtures or silent deferral.

Generic wire v1 uses a handshake record followed by operation records. JSON snapshot files contain handshake plus a single scoped snapshot. JSONL file/FIFO/socket producers start with handshake and a baseline snapshot. The endpoint is private and bound to a launch nonce through a 0600 descriptor file; producer messages cannot nominate arbitrary sessions. Only one producer is accepted per source ID. Authentication proves access to the launch, not that payloads describe its current session. Issue an opaque lease after matching explicit session/turn evidence or adapter-verified source lineage to the current binding. File/FIFO readers capture the same lease internally when validating their source. On reconnect require both current-session evidence and a baseline or validated resume cursor; reject stale leases and unresolved evidence. Expose `openagenthud adapters validate generic --input <path> --format json|jsonl`; it reports record/schema path and never echoes raw values.

Claude's explicitly installed chaining template preserves the existing status-line command's exact input, stdout, and exit status. It forwards a bounded copy to HUD ingress with a 100 ms forwarding deadline; failure is a redacted stderr diagnostic, never status-line stdout. Existing commands remain user-selected and are never inferred from telemetry. Establish the hook's session evidence before acquiring a lease; when locally verified payloads cannot identify the successor session, report unresolved correlation instead of restamping old data. Template removal restores the original invocation without editing unrelated configuration.

Codex controlled mode is invoked only as `openagenthud run --adapter codex --mode controlled -- codex app-server ...`, with the locally verified app-server arguments explicitly present in the user's child command. The wrapper preserves that argv rather than silently converting a native command. It replaces neither a native TUI attach nor a failed observer automatically. The control client owns app-server input/output, permission prompts, explicit user responses, cancellation, transport reconnect, and child lifetime; the HUD subscribes read-only. Unknown approval types remain pending and produce an actionable control-client error. Losing the renderer preserves the client; losing the client never auto-approves and follows the verified server disconnect semantics, showing interrupted/disconnected rather than completed. A minimal line-oriented control surface is sufficient; native-TUI parity is outside scope.

Adopt Sol's strict separation of transport/control and telemetry, but retain a bundled limited client rather than depend on an unspecified relay-compatible external frontend. The client surface supports session selection/creation, turn input/output, explicit correlated approval responses, cancellation, and disconnect/reconnect only; no general editor or native-TUI replacement is planned. Resolve verified protocol feasibility in task 13.1 before building it. Use offline fixture server/client traces for ordering, IDs, unknown messages, backpressure, and approval isolation, then require a real local server smoke check with the usable client. Synthetic success cannot complete controlled support. An unsupported handshake returns a clear controlled-mode error without falling back to observer or guessing methods. Producer and view-control channels have disjoint schemas and no route to control RPCs; client-side user responses alone can authorize them. Blocking protocol work uses the same bounded query/worker accounting as other HUD components.

Alternative: provider adapters directly render fragments or always invoke a controlled server. Rejected because it defeats shared layouts and imposes control responsibilities on passive observation.

### 10. Usage providers and CLI workflows

Usage records are keyed by provider/account/window/resource/unit/kind. Percentages require compatible units/scopes and known positive capacity; overlapping windows and mixed metric kinds are never implicitly summed. A reset timer produces `reset-unconfirmed` and schedules one allowed provider refresh within global query bounds; it never zeros usage. Unknown account identity prevents cache reuse, and failed queries preserve original observation/validation ages. Provider-specific parsers emit normalized records and namespaced details. Estimated cost uses a user-supplied, versioned local price table with currency, date, source, and disjoint category mapping; no bundled claim of current prices and no automatic pricing fetch.

External providers require an explicit config entry. Commands use fixed executable/argv arrays, a minimal documented environment, no shell, and no substitutions from agent text. Credential references resolve only for the request and never become argv or state. HTTP defaults to HTTPS and exact configured hosts/ports; loopback HTTP requires explicit allowance. Reject URL credentials, non-HTTP schemes, unapproved redirects, and private/link-local destinations unless explicitly allowlisted; resolve and validate destinations on every attempt and pin the approved address during connection to avoid DNS rebinding. Do not forward credentials across origin changes. Default redirects are disabled.

CLI executable name is `openagenthud`. Commands: `attach`, `run`, `inspect`, `doctor`, `adapters list`, `adapters validate generic`, `config validate`, `config migrate`, `demo`, and `shell-integration <zsh|bash|fish>`. `inspect --json` and `doctor --json` print exactly one local redacted result on stdout, including structured failures; progress/errors go to stderr. Stable error kinds distinguish invalid-configuration, unresolved-binding, unsupported-adapter/schema, and backend/provider failure. `inspect --export <path>` writes a private schema-versioned snapshot only when explicitly requested. `inspect --hud-id <id> --expand|--collapse|--freeze|--resume` sends an explicit view action; multiple matches without an ID fail selection. These operations cannot send agent-control messages. Exit codes: 0 success, 1 operational/health failure, 2 invalid configuration/selection/input, 3 unsupported adapter/schema. Run returns child status, with documented conventional signal mapping if the host cannot reproduce signal termination; prelaunch errors use the preceding command codes.

`yagent` delegates to run; `yodex` supplies the explicit Codex observer profile unless the user explicitly selects controlled mode. Templates preserve literal argv and last exit status. Add `OPENAGENTHUD_LAUNCH_ID`, `OPENAGENTHUD_PARENT_LAUNCH_ID`, and private descriptor location only; do not replace user HOME/PATH/cwd. Existing parent launch identity becomes lineage; a conflicting active target nonce is rejected. Subagent identity comes from verified telemetry, not merely inheritance of parent environment.

Alternative: implicit account polling, shell command strings, or startup-file mutation. Rejected because hidden side effects violate offline defaults and make wrappers unsafe.

### 11. Roadmap coverage and stage gates

| Roadmap area | Capability | Delivery stage / task group |
| --- | --- | --- |
| Foundation: state and rendering boundaries | `hud-state-contract` | Foundation / 1–2 |
| Foundation: layout, accessibility, terminal backends | `hud-presentation-backends` | Foundation / 6 |
| Foundation: configuration and privacy | `hud-configuration-privacy` | Foundation / 3 |
| MVP: session identity and synchronization | `hud-session-synchronization` | MVP / 4–5 |
| MVP: common state and accounting | `hud-common-telemetry` | MVP / 7 |
| Near-term: activity/collaboration | `hud-activity-usage` | Near-term / 10 |
| Near-term: usage, quotas, and cost | `hud-activity-usage` | Near-term / 11 |
| Near-term: generic and provider adapters | `hud-adapter-integrations` | Generic pulled into MVP / 8; provider breadth / 12–13 |
| Near-term: commands, wrappers, inspection, shells | `hud-cli-workflows` | Core commands pulled into MVP / 9; completion / 14 |
| Cross-cutting release gates | `hud-release-assurance` | Continuous and final / 15–16 |
| Later possibilities | Explicitly excluded above | Subsequent changes |

Foundation gate: type/schema/import-boundary checks, reducer and privacy fixtures, accessible rendering, backend ownership tests. MVP gate: generic demo and real local attach/run, strong concurrent identity, rollover/fault replay, common state, inspection, and baseline budgets. Near-term gate: activity/usage, all specified adapter modes with honest support tiers, controlled approval isolation, full shell/CLI behavior. Final release gate: all tasks complete, fixture evidence, clean platform smoke tests, measured budgets, packaging and support notes. Do not mark later-stage tasks optional because the first useful release can ship earlier.

### 12. Reproducible quality and performance gates

Define npm scripts during bootstrap: `check`, `test:unit`, `test:contract`, `test:replay`, `test:security`, `test:render`, `test:integration`, `bench`, and `test:package`. Unit/replay tests use an injectable clock and fixture-only I/O. Integration tests run in disposable tmux servers/PTYs and temporary home/runtime directories. Failure injection must assert state metadata as well as visible text. Fixtures exclude real secrets/content and retain CLI/source version provenance. Manifests distinguish synthetic contract traces from sanitized provider samples; require the latter plus a reproducible local integration check for structured support. Add explicit cases for capability withdrawal, overlapping baseline/deltas after ID eviction, later-started stale reads, retired-epoch health, old hooks acquiring a new lease, hostile mutation paths, privacy relax-after-purge, and JSON stdout purity.

Initial reference workload: one HUD plus generic producer, 256 tools/agents maximum, ten-line view at 120 columns, no external query, five-minute warmup then ten-minute idle or 100 events/s active run. The active trace uses 1 KiB records covering model, turn, context, and accounting updates with fixed replay data; record actual bytes and latency samples. Oversized/deep-record, maximum-history, many-provider, and burst tests are separate stress workloads. Reference targets are native Ubuntu 24.04 x86_64 on a documented four-vCPU/8-GiB runner and native macOS 14 arm64 on a documented 8-GiB-or-greater machine; record CPU model, kernel, Node, tmux, terminal, and filesystem in results. Emulation is insufficient for performance claims. These are selected test targets, not claims that tests have run.

Budgets: idle average CPU ≤1 percent of one logical core; total HUD-owned RSS ≤192 MiB excluding agent and provider processes; idle source reads ≤16 KiB/s and ≤4 source probes/s averaged over ten minutes; no steady-state full-log rescans; observation-receipt-to-frame p95 ≤100 ms at 100 events/s. Discovery/recovery is ≤two configured reconciliation periods after the source stabilizes for a valid baseline of at most 1 MiB plus at most 2048 1-KiB replay events. Larger verified histories use bounded slices with visible progress and uncertain totals; a size-independent completion deadline is not promised. Overload memory remains within the declared queue/record/history limits. Report source-change-to-frame separately including file debounce; receipt-to-frame starts at trusted ingress and excludes upstream transport/provider delay. GPU activity is not required. Fix regressions or revise thresholds explicitly with evidence; never change a threshold inside a benchmark to pass it.

Measure costs of worker/supervisor/control-client/renderer together. Plain output is rate-limited to 10 revisions/s with latest-frame coalescing; accounting ingestion is never coalesced. Logs have exact counters but rate-limited repeated messages. Package tests build an npm tarball, verify checksum and license contents, install into clean homes on both declared native targets, exercise demo/attach/run and shell templates, migrate and roll back config, and uninstall without touching agent data or a running child's owner. Preparing that local candidate does not authorize publication.

Alternative: manual visual checks and unspecified “low CPU” targets. Rejected because identity/privacy regressions and source I/O growth are otherwise difficult to detect consistently.

## Risks / Trade-offs

- [Internal provider formats change] → Isolate readers, gate versions on local fixtures, fail visibly, and keep fallback capabilities honest. This reduces immediate coverage rather than guessing.
- [Broad roadmap increases delivery size] → Use explicit stage gates and dependency-ordered tasks; completion still requires every in-scope stage.
- [Node processes and workers consume memory] → Measure aggregate RSS/CPU, share pure code, start workers lazily, and enforce the stated budgets before release.
- [Process identity differs between Linux and macOS] → Implement platform probes with PID-reuse fixtures; unsupported evidence yields unresolved binding.
- [Sparse hooks cannot prove liveness of every field] → Keep process health separate from field validation and expose age/cached state.
- [Lossy or unavailable provider usage] → Mark incomplete accounting; do not infer missing categories, child inclusion, prices, or reset success.
- [Terminal widths vary for some emoji] → Use tested tables, grapheme-safe truncation, ASCII mode, and explicit supported-terminal test evidence.
- [Controlled app-server mode introduces action responsibilities] → Separate control client and read-only HUD, require explicit invocation, and block release without approval/cancel/disconnect tests.
- [A usable controlled client is more work than a relay fixture] → Keep the bundled line-oriented surface bounded, verify protocol feasibility first, and block full completion on a real-server smoke check instead of declaring an unspecified external client sufficient.
- [Launch-authenticated old hooks can cross rollover] → Validate current session/source evidence before issuing leases; unresolved correlation remains unavailable even when authentication succeeds.
- [Runtime privacy tightening cannot recall user exports] → Purge HUD-owned state, explain export boundaries, and never claim secure erasure of outside files.
- [Same-user code can access private runtime endpoints] → Treat filesystem permissions/nonces as accidental cross-session protection; built-in integrations share the user's trust domain. Stronger sandbox isolation remains a later feature.

## Migration Plan

1. There is no deployed executable or public schema to migrate. Implement schema/config v1 and the generic demo first, keeping this alternative unmerged with the other proposal until one implementation plan is selected.
2. Deliver Foundation and MVP stages through an experimental package; require all applicable security/identity/backend gates before declaring the MVP milestone.
3. Add Near-term modes behind explicit configuration and support-tier metadata. Unverified source versions remain labeled and cannot satisfy structured integration release tasks.
4. Prepare a reviewable local release candidate only after both declared native-platform gates and the final roadmap coverage audit. Include checksums, Apache-2.0 and dependency notices, compatibility matrix, and support policy. Publication is a separate explicitly authorized action. Support the latest minor release on the declared Node baseline; older provider versions retain only the listed verified fixtures, and security fixes follow the documented maintained release line.
5. For future configuration upgrades, provide a deterministic version-to-version migration that validates output, writes a private backup, and atomically replaces only on `config migrate --write`. Preserve unknown newer versions unchanged. Back up optional cache metadata; caches are disposable and can be invalidated rather than migrated.
6. Roll back by reinstalling the prior package and restoring its configuration backup. Stop views and inactive observation resources only; do not stop a run supervisor/control client or remove its endpoint while it owns a live child. A compatible view may reattach; an incompatible view reports the protocol limitation and can use supported passive observer attach without replacing the active owner. Uninstall removes package-owned binaries and selected expired runtime/cache files; a live owner must retain the loaded resources needed to continue and cleans its own matching registry after child exit. Retain user configuration/exports unless separately selected for removal. Supply inspectable shell-template removal instructions.

## Open Questions

None that change this design or its task breakdown. Exact verified provider-version rows and terminal-width exceptions are outputs of the specified local integration and release tests, not unresolved architecture choices. Visual palette values can be tuned during the already scoped theme task without changing behavior.
