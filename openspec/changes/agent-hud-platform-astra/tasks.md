## 1. Foundation bootstrap and contract tooling

- [ ] 1.1 Create the TypeScript/ESM package and module directories from design decision 1, pin Node/dependency baselines and npm lockfile; verify clean install, build, and `npm run check` on native Linux x86_64 and macOS arm64 with the selected Node 24 baseline and record exact runtime/optional tmux versions.
- [ ] 1.2 Add JSON Schema validation/type generation and the documented npm test scripts; verify valid schema examples compile, invalid examples fail with field-path errors, and generated types are reproducible without a diff.
- [ ] 1.3 Add an import-boundary check between adapters, core, presentation, and backends; verify a deliberately forbidden renderer-to-adapter import fails the check and the intended module graph passes.
- [ ] 1.4 Create fixture metadata distinguishing synthetic traces from sanitized provider samples, injectable monotonic/wall clocks, deterministic replay harness, and isolated temporary-home utilities; verify identical replay results and reject provider-support claims backed only by synthetic fixtures.

## 2. Versioned normalized state

- [ ] 2.1 Define the v1 handshake/envelope schemas and capability negotiation for `hud-state-contract`; verify incompatible major versions fail and compatible additive namespaced extensions preserve common-field behavior in `test:contract`.
- [ ] 2.2 Publish the common-field dictionary with units, scopes, inclusion rules, quality, privacy classes, and clearing semantics; verify every normalized field in the schemas has a dictionary entry and a valid example.
- [ ] 2.3 Implement field-covered scoped snapshots, path patches, null clears, keyed collections, invalidation, health, and atomic capability withdrawal; verify model-only snapshots preserve project fields and withdrawal invalidates dependent values/derivations in one revision while independent sources remain usable.
- [ ] 2.4 Implement immutable revision publication and coherent group derivation; verify frames/exports cannot mix revisions and attempted consumer mutations cannot change published state.
- [ ] 2.5 Represent capability, availability, quality, freshness, privacy, connection, and frozen-view status independently; verify unsupported, redacted, disconnected, and stale fixtures never become fabricated zero or healthy values.
- [ ] 2.6 Implement accounting epochs, bounded event deduplication/watermarks, checkpointed cumulative/delta coverage, and declared category inclusion; verify total 100 through sequence 20 plus replayed 19 and delta 5 at 21 yields 105 even after ID eviction, and unproven resets/epoch continuity remain uncertain.
- [ ] 2.7 Validate mutation paths, source coverage, namespace ownership, and conflicting operations atomically; verify prototype-mutating keys, unauthorized extension writes, and simultaneous set/clear reject the whole envelope without state or prototype changes.

## 3. Configuration and privacy foundation

- [ ] 3.1 Define configuration v1 and implement defaults/user/explicit-file/profile/environment/flag precedence with origin tracking and bounded profile fields; verify conflicting origins, unknown keys/versions, and invalid height fail before child/provider/endpoint/terminal changes.
- [ ] 3.2 Implement atomic reload and an explicit notification for identity-affecting settings; verify malformed edits preserve the active config revision and valid candidates publish exactly one configuration revision and notification.
- [ ] 3.3 Implement typed privacy projection for all documented field classes before retention and at every sink; verify secret-bearing fixtures leave no prohibited values in reducer state, logs, diagnostics, exports, or caches.
- [ ] 3.4 Implement separate content-display and retention permissions with shared count/byte/age bounds; verify display-only summaries retain no history, retention disabling purges owned content, and oversized retained records cannot exceed the 4 MiB bound.
- [ ] 3.5 Implement atomic policy/configuration tightening that purges buffered/frozen/current/history/log data and rebuilds sanitized views; verify restrictions remove sentinels from all sinks and later relaxation cannot revive them without new observations.
- [ ] 3.6 Add private directory/file creation, ownership/symlink checks, and safe credential-reference handling; verify hostile runtime paths fail closed and credentials never appear in argv, state, or errors.
- [ ] 3.7 Verify offline defaults with a denied-network test environment and watched temporary shell/credential files; confirm default startup makes no outbound requests and modifies none of those files.
- [ ] 3.8 Apply default-deny extension classification and privacy to source labels, scope keys, provenance, and tool targets; verify hidden content cannot escape through those alternate paths and safe support/health metadata remains independently inspectable.

## 4. Identity and source-controller lifecycle

- [ ] 4.1 Implement separate HUD/launch/process/pane/session/turn/fork/subagent identities and Linux process-start probes; verify PID reuse and two sessions in the same repository do not merge state.
- [ ] 4.2 Implement macOS process-start/hosting probes with the same identity contract; verify platform fixtures reject PID reuse and report unresolved evidence rather than guessing.
- [ ] 4.3 Implement explicit selectors, corroborated candidate matching, ambiguity output, and account/worktree scope keys; verify explicit selection resolves the intended candidate and newest-file/PID-only candidates never bind silently.
- [ ] 4.4 Implement discovering/binding/synchronized/degraded/disconnected/stopped transitions with reasons; verify process exit, failed baseline, source recovery, and explicit stop follow the documented lifecycle.
- [ ] 4.5 Implement generation/epoch capture for every read, probe, callback, and retry plus atomic rollover staging; verify old-readable-rollout and same-generation retired-epoch results cannot alter new values, validation leases, health, or active-source counters.
- [ ] 4.6 Implement scope-preserving reconciliation for matching account/worktree identity; verify rollover retains only matching independent scopes and clears prior session/turn/activity fields in one revision.
- [ ] 4.7 Connect identity-affecting configuration reloads to staged rebinding; verify a valid target change creates a new generation and cannot mix previous-target fields into the new session.

## 5. Ordering, freshness, and bounded source recovery

- [ ] 5.1 Implement source epochs, group watermarks, read-start guards, field precedence, and baseline reconciliation; verify older reads begun before or after an event cannot overwrite it without coverage proof and gaps invalidate affected totals.
- [ ] 5.2 Implement monotonic freshness/elapsed timers, wall deadlines, stable-field validation, and wake/divergence/scheduling-gap resume detection; verify healthy idle, clock jumps, and suspension with both clocks advancing cannot falsely reaffirm live state.
- [ ] 5.3 Implement 50 ms default file debounce, fallback polling, jitter, periodic source/identity reconciliation, and cadence validation; verify a missed watcher replacement is discovered within the configured reconciliation bound.
- [ ] 5.4 Implement file-identity offsets and bounded partial-record parsing for append, truncation, rotation, replacement, and malformed input; verify no partial record publishes state and each replacement establishes the correct epoch.
- [ ] 5.5 Implement parser workers, record/depth deadlines, queue/history/handle limits, dedup eviction, and overflow recovery from design decision 5; verify overload stays bounded, marks accounting uncertain, and recovers from a baseline.
- [ ] 5.6 Implement cancellable capped retries and bounded observer subprocesses; verify timeout/oversized output kills only the owned query process group and stop cancels outstanding retries.
- [ ] 5.7 Add bounded multi-slice replay with verified starting baseline/coverage and progress diagnostics; verify arbitrary suffixes remain uncertain, complete replay converges, and the declared small recovery envelope meets its deadline without promising that deadline for unbounded histories.

## 6. Rendering and terminal backends

- [ ] 6.1 Implement the provider-independent view model and one-through-ten-line layout with field order, separators, priorities, and profiles; verify deterministic snapshots for all heights and widths from one cell through the reference 120 columns.
- [ ] 6.2 Implement grapheme-safe cell measurement and telemetry/control-text sanitization; verify wide/combining/emoji boundaries and ANSI/OSC/DCS/newline/bidi injection fixtures in `test:render` and `test:security`.
- [ ] 6.3 Add theme selection, Claude-like palette, independent NO_COLOR, monochrome, high contrast, ASCII, reduced-motion, and TERM=dumb behavior; verify every critical state retains a textual indication without color or animation.
- [ ] 6.4 Implement standalone alternate-screen and append-only plain modes with bounded output/coalescing; verify non-TTY output has no control sequences and catchable exits restore cursor/raw/alternate-screen state in PTY tests.
- [ ] 6.5 Implement tmux server/target validation, ownership tags, focus-preserving pane creation, resize, and ownership-limited cleanup; verify server restart/ID reuse leaves unrelated panes intact and agent focus remains unchanged.
- [ ] 6.6 Implement focused HUD expand/collapse/freeze/inspect actions with continuing ingestion and privacy updates; verify unfocused keys reach the agent and unfreeze displays the latest coherent revision.
- [ ] 6.7 Implement separate renderer lifetime and reconnectable presentation using a fixture lifetime owner; verify closing/killing the view leaves fixture I/O usable and a new view receives current state, then validate the real native supervisor in task 9.4.
- [ ] 6.8 Implement hosting-failure diagnosis and document uncatchable terminal-reset recovery; verify whole-server loss revalidates process identity without claiming child survival and catchable presentation loss still restores terminal modes.

## 7. Common agent and project telemetry

- [ ] 7.1 Implement agent/CLI/adapter/mode/provider/model/role/effort mapping; verify unknown effort stays unknown and unlike provider effort labels remain unchanged.
- [ ] 7.2 Implement distinct launch/current/project/repository/worktree locations and cwd-dependent invalidation; verify a repository switch never displays the old branch in the new project revision.
- [ ] 7.3 Implement bounded local Git queries for branch, detached HEAD, dirty state, counts, worktree, and optional local ahead/behind; verify inaccessible/non-repository/missing-upstream cases and confirm no network fetch occurs.
- [ ] 7.4 Implement turn lifecycle, waiting reasons, observed/reported start, last response, and measured/estimated elapsed duration; verify inactivity never fabricates completion and approval display has no action side effect.
- [ ] 7.5 Implement 5 s output-rate sampling and separately labeled whole-turn averages; verify tokens and characters retain different units and untimed final totals do not yield a live rate.
- [ ] 7.6 Implement context used/remaining/capacity/percent, compaction, and token-category presentation; verify session-only totals never become occupancy, overflow stays visible, missing capacity yields unknown percentage, and compaction does not reset lifetime usage.
- [ ] 7.7 Implement descriptive permission/approval/sandbox/collaboration fields with provenance; verify stale sandbox metadata cannot render as a current enforcement guarantee.

## 8. Generic adapter and local ingress

- [ ] 8.1 Publish generic JSON/JSONL handshake, snapshot, event, clear, invalidation, and reconnect examples; verify all examples pass the v1 schema and a new sample adapter renders without presentation changes.
- [ ] 8.2 Implement explicit file and FIFO inputs with generation-stamped producer identity; verify malformed/partial records, replacement, wrong launch ID, EOF, and reconnect semantics in `test:contract`.
- [ ] 8.3 Implement the private Unix endpoint/descriptor, one-producer-per-source handshake, and controller-issued session/source leases; verify ownership/nonce/collision checks and a delayed old hook with a valid launch nonce cannot acquire the replacement session's generation without current evidence.
- [ ] 8.4 Implement `adapters validate generic` for both formats with safe location errors and exit statuses; verify valid fixtures exit zero and invalid fixtures identify record/schema path without echoing secrets.
- [ ] 8.5 Implement the credential-free generic demo with rollover, stale-state, quota uncertainty, and optional activity examples; verify it runs offline in standalone and isolated tmux modes.
- [ ] 8.6 Implement lease retirement and reconnect snapshot/resume-cursor checks; verify stale leases fail, reauthentication alone cannot promote old payloads, and changed-epoch accounting requires proven baseline/replay coverage.

## 9. MVP commands and native run supervision

- [ ] 9.1 Implement command parsing/help for attach, run, inspect, doctor, adapters list, config validate, and demo with documented prelaunch codes and structured error kinds; verify help/validation launch no agent/backend/provider and failure JSON remains one stdout value.
- [ ] 9.2 Implement deterministic explicit/config/child/environment adapter selection and noninteractive ambiguity errors; verify conflicting selectors fail and environment hints cannot override explicit selection.
- [ ] 9.3 Implement the native-run supervisor with literal argv, inherited environment/cwd/stdio, additive launch lineage, and child exit status; verify metacharacter arguments, exit code 17, and nested wrapper identity in child probes.
- [ ] 9.4 Implement signal forwarding without duplicate foreground-group delivery and isolate HUD shutdown from agent termination; verify SIGINT/SIGTERM, renderer crash, attach interruption, and documented SIGHUP behavior using PTYs.
- [ ] 9.5 Implement automatic owned-tmux rendering and the no-tmux second-terminal attach workflow; verify conflicting same-TTY interactive output fails before launch, child stdio stays untouched, and plain output uses only an explicitly selected destination.
- [ ] 9.6 Implement redacted revision-bound inspect/export and baseline doctor output; verify export during rollover is coherent and diagnostics contain provenance, ages, binding strength, and no raw private identity.
- [ ] 9.7 Implement the recoverable owner-private registry and owner-start/protocol checks before cleanup or reattachment; verify live run-owner entries survive view loss and stale/reused process identities cannot delete unrelated resources.
- [ ] 9.8 Implement `inspect --hud-id` expand/collapse/freeze/resume actions with disjoint view-message schemas; verify only the selected view changes, ambiguity fails, and producer/view messages cannot invoke agent control.
- [ ] 9.9 Run the Foundation/MVP milestone gate across generic contracts, privacy, identity/recovery replay, render, PTY/tmux, and baseline resources; verify all applicable npm scripts pass and record the milestone without marking Near-term tasks complete.

## 10. Near-term activity and collaboration

- [ ] 10.1 Implement keyed tool lifecycle, permitted targets, observed duration/completion/errors, and bounded completed records; verify duplicate completion, missing start, and unknown completion do not invent duration or success.
- [ ] 10.2 Implement separate configured/loaded/active states for skills, plugins, and MCP; verify configuration-only fixtures never imply active use or health.
- [ ] 10.3 Implement subagent ancestry, model/status, incomplete coverage, and explicit aggregate eligibility; verify parent-includes-child, disjoint, and unknown-inclusion fixtures prevent double counting.
- [ ] 10.4 Implement plan counts/items, goal/title/turn count, and opt-in adapter-supplied summary projection; verify content-disabled output preserves permitted counts, display alone creates no history, and summary reception triggers no network or summarization request.
- [ ] 10.5 Implement typed bounded anomalies for context contradiction, unsupported activity, gaps, and dropped updates; verify each anomaly identifies its scope/revision and contains no raw source body.

## 11. Near-term usage providers and cost

- [ ] 11.1 Implement scoped usage records and quota/credit/measured-cost/estimate presentation; verify three windows and mixed kinds stay separate and zero/unknown capacity or mismatched scope cannot produce a percentage.
- [ ] 11.2 Implement provider-age display, reset-unconfirmed reconciliation, account-key invalidation, and bounded optional cache; verify offline reset/query failure never fabricates freshness and changed or unknown accounts cannot reuse old quotas.
- [ ] 11.3 Implement the shared two-query scheduler, 16-provider limit, one pending refresh/provider, and observer priority; verify simultaneous resets/retry storms stay bounded, duplicate refresh requests coalesce, and scheduled work does not renew quota age.
- [ ] 11.4 Implement explicitly configured command providers with executable/argv arrays, minimal environment, cancellation, output/depth/timeout limits, and private credential references; verify hanging/injectable/oversized command fixtures preserve agent lifetime and expose no credentials.
- [ ] 11.5 Implement opt-in HTTP providers with exact destination allowlists, address validation/pinning, no default redirects, origin-safe credentials, and bounded retries; verify redirect, DNS-rebinding, private-address, timeout, and denied-network fixtures.
- [ ] 11.6 Implement a provider-normalizer registration boundary with one verified fixture-backed quota mapping where local telemetry is available; verify provider-specific windows/details map into common records without renderer changes or invented live support.
- [ ] 11.7 Implement user-supplied price tables with source/date/currency/category provenance and partial/unknown estimates; verify missing cache prices, mixed currencies, and overlapping scopes cannot yield an unlabeled complete total.

## 12. Verified observer and fallback adapters

- [ ] 12.1 Capture and redact local Codex rollout/state/log/process fixtures with version and source provenance, including concurrent roots and rollover; verify the fixture manifest establishes supported formats and contains no secrets or conversation bodies before enabling readers.
- [ ] 12.2 Implement the version-gated Codex durable rollout observer and session correlation; verify native TUI attachment remains passive and old-readable-rollout, duplicate accounting, and malformed-format fixtures recover correctly.
- [ ] 12.3 Implement isolated read-only Codex state-database root discovery and verified model-selection log updates; verify lock timeouts, format mismatch, late durable reads, and database byte hashes confirm bounded recovery without database modification.
- [ ] 12.4 Capture verified Claude payloads and explicitly installed chaining templates; verify stable identity/sparse/null/concurrent fixtures and preserve the existing status-line command's input, stdout, and exit status during ingress failure with forwarding bounded to 100 ms.
- [ ] 12.5 Implement Claude session-bound leases, sparse patches, hook age, and separate process health; verify concurrent sessions and late hooks reconnecting after rollover stay separate and omissions preserve unrelated fields.
- [ ] 12.6 Implement Gemini executable/version discovery and process/project/Git fallback profile; verify unavailable/runnable/ambiguous cases and confirm model/context/quota/activity are not claimed without evidence.
- [ ] 12.7 Implement Aider executable/version discovery and the same honest fallback contract; verify its mode/version/identity limitations are visible and common rendering needs no adapter branches.
- [ ] 12.8 Publish adapter support-tier rows and health/remediation messages from local evidence; verify every enabled mode passes `test:contract`, unknown versions are labeled unverified, and missing mandatory structured-mode evidence remains an explicit blocker.

## 13. Explicit Codex controlled integration

- [ ] 13.1 Establish protocol feasibility using a local app-server transcript for the bounded client's session/turn, input/output, approval, cancel, reconnect, and exit surface; verify real method/version evidence before client construction and record missing evidence as a full-change blocker.
- [ ] 13.2 Add offline fixture client/server protocol checks for ordering, correlated approval IDs, unknown negotiation, backpressure, and control-plane isolation; verify producer/view payloads cannot answer approvals and fixture success alone cannot complete task 13.7; use this harness while building tasks 13.3–13.6.
- [ ] 13.3 Implement the explicit controlled-mode command routing and separate persistent control client; verify observer failure never launches app-server and closing the renderer leaves the client and test agent alive.
- [ ] 13.4 Implement verified app-server lifecycle/event translation through the common observation contract; verify turn ordering, generation change, context/accounting scopes, disconnect, and resnapshot with replay fixtures.
- [ ] 13.5 Implement the minimal input/output and explicit approval-response surface, including unknown approval handling; verify every approval remains pending until a user response and HUD display code has no approval-action path.
- [ ] 13.6 Implement cancellation, control-client transport reconnect, and documented control-owner loss behavior; verify no request auto-approves, no interrupted turn appears completed, and renderer reconnect preserves pending requests.
- [ ] 13.7 Run the controlled-mode PTY/process integration gate on a verified local CLI version; verify exact arguments where forwarded, intended signal behavior, terminal restoration, child exit reporting, and separation from native observer mode.

## 14. Complete user workflows and diagnostics

- [ ] 14.1 Supply yagent/yodex wrappers and inspectable zsh/bash/fish templates with removal instructions; verify all three shells preserve literal argv, cwd, environment, signals, and exit status without modifying startup files automatically.
- [ ] 14.2 Complete structured diagnostics/export with versions, provenance, generations, reconciliation, lag, aggregate resource use, and counters; verify healthy/degraded/error paths each emit one redacted JSON stdout result, keep diagnostics on stderr, and never upload.
- [ ] 14.3 Implement bounded structured log rotation and repeated-error rate limiting; verify a persistent malformed source stays within 2 × 1 MiB while exact counters remain correct.
- [ ] 14.4 Complete compact/expanded/full/presentation/plain/monochrome/privacy profile documentation and selection; verify examples validate, privacy defaults match implementation, and changing color does not change field layout.
- [ ] 14.5 Implement explicit config migration with backup, validation, and atomic replacement using a declared synthetic predecessor fixture; verify failed writes preserve the original, newer schemas remain untouched, and rollback recovers the prior valid configuration.
- [ ] 14.6 Document attach/run/control-mode differences, capability limitations, ingress examples, reset uncertainty, export privacy boundaries, and all exit codes; verify documented commands against the generic demo and verified adapter smoke tests.

## 15. Cross-cutting release gates

- [ ] 15.1 Complete a requirement-to-test manifest for all nine capabilities and adapter/mode conformance cases; verify every requirement/scenario maps to a passing test or a reproducible explicit platform smoke procedure.
- [ ] 15.2 Run the complete recovery suite including retired-epoch health, old-hook lease renewal, stale later-started reads, checkpoint overlap/ID eviction, and both-clocks-advance suspension alongside existing concurrency/source faults; verify trust metadata, small-envelope deadlines, and bounded large-replay progress.
- [ ] 15.3 Run the all-sink security/privacy suite for terminal escapes, huge/deep input, secrets in environment/paths/errors/extensions, privacy reload, unsafe endpoints, usage commands, and destinations; verify no prohibited fixture marker appears in any output or retained state.
- [ ] 15.4 Run Linux/macOS PTY and disposable-tmux integration for widths/heights, Unicode/accessibility, focus, input ownership, signals, restart, cleanup, and renderer loss; verify unrelated panes and agent processes remain intact.
- [ ] 15.5 Run native reference idle/100-events-per-second 1-KiB-trace benchmarks with aggregate supervisor/worker/client/view costs; verify CPU ≤1 percent/core, RSS ≤192 MiB, idle I/O/probes, p95 receipt-to-frame ≤100 ms, separate source-change latency, and declared recovery envelope. Include maximum retained bytes, provider fan-out, and hostile-record stress without silently changing thresholds.
- [ ] 15.6 Record the reference-machine/runtime/terminal/source versions and benchmark results; verify the workload, commands, fixed thresholds, and overload behavior are reproducible from the report.
- [ ] 15.7 Audit Foundation/MVP/Near-term and cross-cutting roadmap coverage against proposal/specs/design/tasks; verify every in-scope roadmap item has acceptance evidence and only the explicitly listed Later/non-goal items are deferred.

## 16. Packaging, migration, and supported release

- [ ] 16.1 Build the versioned npm package with executable, schemas, examples, Apache-2.0 and dependency notices, compatibility notes, and checksum; verify `test:package` rejects missing files and checksums match the distributable bytes.
- [ ] 16.2 Verify clean install, demo, attach, native run, and uninstall on native Linux x86_64/macOS arm64 with optional tmux present/absent; confirm no credential/network prerequisite and active-child I/O/owner endpoints survive uninstall without agent/export deletion.
- [ ] 16.3 Exercise upgrade, explicit migration, rollback, and inactive-runtime cleanup on both targets; verify restoration of prior package/config, preservation of newer configs and active lifetime owners, and safe cleanup after their child exits.
- [ ] 16.4 Publish the evidence-backed CLI/mode compatibility matrix, known limitations, maintained-version/support policy, and reversible shell setup instructions in release documentation; verify every structured support claim links to a local fixture and successful integration record.
- [ ] 16.5 Run the final complete check/test/package/resource gate and assemble a local candidate report; verify all tasks and real structured Codex observer/controlled/Claude gates have evidence, and keep all planning boxes unchecked until implementation occurs. Candidate preparation does not authorize publication.
