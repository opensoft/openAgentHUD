This is a future implementation checklist, not authorization to implement during proposal creation. All boxes intentionally remain unchecked. Groups 1–10 establish the Foundation/MVP generic vertical slice; groups 11–15 deliver Near-term coverage; groups 16–17 complete the release gates for the whole change. Provider compatibility claims require sanitized local evidence and cannot be inferred from synthetic fixtures. See each named capability spec for acceptance scenarios and `design.md` for decisions.

## 1. Runtime and module boundaries

- [ ] 1.1 Create the TypeScript/ESM package, Node.js 24+ runtime declaration, executable entry point, source directories, and locked dependencies described in design decision 1. **Verification:** a clean dependency installation, compilation, and `openagenthud --help` smoke check succeed on the selected runtime.
- [ ] 1.2 Enforce contract/core/privacy/adapter/presentation/backend import boundaries and prohibit provider imports in common presentation. **Verification:** the boundary check passes for the initial modules and rejects a deliberate test import from presentation into a provider adapter.
- [ ] 1.3 Add reproducible typecheck, contract, replay, privacy, renderer, profile, integration, benchmark, and install-check scripts. **Verification:** each documented script runs against its initial harness, and the generic/fast harnesses require no provider binaries, credentials, or network.
- [ ] 1.4 Create a fixture manifest distinguishing synthetic contract traces from sanitized real provider samples and recording CLI/source versions and provenance. **Verification:** fixture validation rejects an advertised provider-compatibility claim backed only by a synthetic trace.

## 2. Normalized state and update contract

- [ ] 2.1 Define state-schema v1, nullable common fields, metadata dimensions, scope keys, capabilities, and namespaced extensions for `normalized-agent-state`. **Verification:** fixtures distinguish unknown from zero/false and reject invalid unit/scope combinations and common-path overwrite attempts.
- [ ] 2.2 Define and validate handshake compatibility and snapshot/patch/invalidate/source-health/capability envelopes. **Verification:** contract tests accept compatible additive input, reject incompatible majors/malformed handshakes, and require generation/epoch/ordering evidence.
- [ ] 2.3 Implement covered snapshot replacement, sparse patches, explicit clears, and capability withdrawal. **Verification:** the snapshot coverage, omitted-effort, explicit-clear, and atomic capability-removal scenarios produce the specified values and metadata.
- [ ] 2.4 Implement immutable revision publication with injected clocks and coherent field-group reduction. **Verification:** concurrent/replayed inputs cannot mutate prior revisions or produce a frame/export mixing binding generations.
- [ ] 2.5 Publish common field descriptors for units, scopes, quality, privacy, priority, and capability dependencies. **Verification:** descriptor completeness checks cover every common field and unknown extensions remain nondisclosed without a classified descriptor.

## 3. Configuration and privacy pipeline

- [ ] 3.1 Implement typed versioned YAML/JSON configuration, bounds checking, defaults, profiles, and the declared override precedence. **Verification:** `config validate` fixtures cover unknown keys/types, heights 1–10, invalid height 11, and conflicting file/profile/environment/CLI settings.
- [ ] 3.2 Implement candidate validation and atomic last-valid reload with safe origin/error reporting. **Verification:** malformed live edits leave the prior effective configuration active, while a valid candidate changes all relevant settings in one configuration revision.
- [ ] 3.3 Implement privacy classification and the shared ingestion/output projection for all listed field categories, including provider extensions and provenance. **Verification:** secret/content/path sentinels are absent from default rendered state, logs, diagnostics, exports, and caches, while redacted support remains distinguishable from unsupported fields.
- [ ] 3.4 Add compact/full/presentation/monochrome/privacy profiles with independent color and privacy controls. **Verification:** profile resolution preserves precedence and presentation/privacy profiles apply the documented disclosure restrictions consistently across sinks.
- [ ] 3.5 Implement bounded owner-private retention/cache storage, default-off persistent history, expiry, and purge behavior. **Verification:** enabled buffers stay within count/byte/age limits, disabling retention purges HUD-owned content, and provider-owned source files remain unchanged.
- [ ] 3.6 Reproject queued, retained, cached, and frozen values when privacy becomes stricter. **Verification:** restrictions remove previously permitted sentinels from every sink immediately and later relaxation does not resurrect discarded content.

## 4. Identity and source-controller lifecycle

- [ ] 4.1 Implement distinct HUD/launch/process-start/pane/session/turn/fork/subagent identities and verified scope keys. **Verification:** PID-reuse, unknown-account, process-only fallback, and parent/fork fixtures do not reuse incompatible state or fabricate provider session IDs.
- [ ] 4.2 Implement deterministic evidence matching, explicit-selection validation, and unresolved candidate reporting. **Verification:** two sessions in one repository remain unresolved without a discriminator, and newest file/process timestamps cannot resolve the test.
- [ ] 4.3 Implement discovering/binding/synchronized/degraded/disconnected/stopped transitions and causes for `session-synchronization`. **Verification:** stream failure, verified fallback, recovery, no-valid-source, and explicit-stop traces produce the expected lifecycle without changing selected identity.
- [ ] 4.4 Tag asynchronous work with binding generation/source epoch and reject obsolete results before reduction. **Verification:** late old-generation reads/events/health results cannot alter the current revision or counters.
- [ ] 4.5 Stage verified rollover and atomically clear session/turn-scoped state while retaining only matching worktree/account facts. **Verification:** readable-old-rollout, same-account/worktree, changed-account, and fork traces assert the first new-generation revision and its cleared/retained fields.
- [ ] 4.6 Implement periodic identity/root reconciliation independent of watcher health. **Verification:** missed-watch fixtures discover a correlated replacement within the configured reconciliation interval without selecting an unrelated root.

## 5. Ordering freshness and bounded recovery

- [ ] 5.1 Implement per-group source authority, source sequencing/deduplication, and durable-read watermarks. **Verification:** duplicate/out-of-order input and an old model read completing after a newer event cannot regress the selected value or double-count accounting.
- [ ] 5.2 Implement monotonic validation leases, cached/stale/disconnected metadata, observation ages, elapsed clocks, and suspend/resume invalidation. **Verification:** controlled-clock idle, wall-clock-jump, failed-validation, and suspension traces retain honest labels and never produce negative elapsed durations.
- [ ] 5.3 Add debounced file watchers, bounded poll fallback, watcher reattachment, partial-tail buffering, truncation, and rotation epochs. **Verification:** 50 ms debounce, incomplete JSONL, replacement, watcher-loss, and readable-old-file tests recover from a consistent baseline.
- [ ] 5.4 Enforce global queue record/byte limits, record/depth bounds, parser slices/watchdogs, connection limits, and retry backoff. **Verification:** oversized, deeply nested, burst, and retry-storm fixtures stay within design limits and expose bounded redacted errors.
- [ ] 5.5 Implement sequence-gap/overflow invalidation and authoritative resynchronization. **Verification:** affected totals remain uncertain until a covered baseline/replay succeeds, while independent project/model fields and user input stay responsive.

## 6. Generic producer contract and private ingress

- [ ] 6.1 Implement generic JSON handshake/snapshot bundles and JSONL envelopes from stdin/pipes and selected watched files. **Verification:** offline examples cover valid snapshots, sparse patches, clears, malformed input, duplicates, rotation, and reconnect without corrupting the last valid revision.
- [ ] 6.2 Implement owner-private single-launch ingress, authentication/current-generation negotiation, message bounds, and safe registry lifecycle. **Verification:** wrong-owner/type/token/session/generation fixtures are rejected and stale cleanup cannot delete an unrelated owner's runtime file.
- [ ] 6.3 Add `adapters validate` with semantic diagnostics and nonzero failure status. **Verification:** invalid units, scopes, versions, capabilities, and oversized records fail with safe field locations and without echoing raw payloads.
- [ ] 6.4 Build reusable adapter conformance fixtures and a reference fixture adapter. **Verification:** the fixture adapter renders common fields through unchanged presentation, and omission/clear/provenance/unsupported-field checks pass through every generic transport.
- [ ] 6.5 Publish the generic protocol, snapshot/JSONL examples, and offline demo producer. **Verification:** documented commands validate each example and run the same demo using both a pipe and a selected file without provider credentials or network queries.

## 7. Common agent project turn and context state

- [ ] 7.1 Add common agent/CLI/provider/model/role/effort descriptors and normalized displays. **Verification:** literal provider effort labels and unknown version/role fixtures retain their meanings without cross-provider conversion.
- [ ] 7.2 Implement launch/current/project/repository/worktree distinction and bounded local Git observation. **Verification:** directory-change, detached-head, dirty/file-count, and missing-upstream fixtures invalidate old worktree data and demonstrate that no fetch occurs.
- [ ] 7.3 Implement verified turn lifecycle and waiting reasons plus descriptive permission/sandbox/collaboration fields. **Verification:** approval-wait/interruption and process-only fixtures do not fabricate completion, working state, or enforcement claims.
- [ ] 7.4 Implement monotonic turn duration, response timestamps, and measured output rates with the documented interval/unit. **Verification:** token and character streams produce correctly labeled rates, insufficient intervals remain unknown, and reconnect/clock changes do not fabricate duration.
- [ ] 7.5 Implement scoped context operands, safe derivation, and contradictory/overflow anomalies. **Verification:** missing-capacity, mismatched-scope, zero-capacity, overflow, and session-totals-only fixtures produce the specified unknown or anomaly states.
- [ ] 7.6 Implement cumulative/delta turn/session token accounting, cache inclusion, reset/compaction evidence, checkpoints, and bounded dedup bookkeeping. **Verification:** inclusive-cache, repeated snapshot/delta, reset, ID eviction, and epoch-replay tests avoid double counts and invalidate unreconstructable totals.

## 8. Presentation and accessibility

- [ ] 8.1 Implement telemetry sanitizing and grapheme-aware display widths/truncation before styling. **Verification:** CJK/combining/emoji and ESC/OSC/CSI/DCS injection fixtures fit their widths, preserve clusters, and cannot invoke terminal actions.
- [ ] 8.2 Implement deterministic ordered/separated field layout at heights 1–10 with reserved freshness/anomaly and documented omission priority. **Verification:** renderer checks cover every height, configurable order, narrow/one-column cases, and priority preservation without overflow.
- [ ] 8.3 Add compact/expanded/full/presentation/plain field profiles and semantic themes including the Claude-like palette. **Verification:** profile snapshots preserve common-field meanings and color toggling changes styling without changing layout.
- [ ] 8.4 Add monochrome/high-contrast/ASCII/reduced-motion and `TERM=dumb` behavior. **Verification:** critical status remains textual/symbolic, plain/dumb output contains no terminal-control sequences, and reduced-motion retains live ingestion.
- [ ] 8.5 Implement inspection labels, frozen revision/time, expansion/collapse, and latest-revision resume. **Verification:** freeze across rollover remains visibly frozen, resume selects the latest generation, and stricter privacy immediately reprojects a frozen view.

## 9. Terminal backends and session-host lifetime

- [ ] 9.1 Implement standalone alternate-screen and append-only/plain output with resize and catchable-exit restoration. **Verification:** PTY checks restore cursor/raw modes, resize within bounds, emit safe plain lines, and document uncatchable-exit reset behavior.
- [ ] 9.2 Implement explicit tmux server/session/pane targeting and detached HUD splits with ownership tags. **Verification:** isolated tmux checks preserve agent focus, honor configured heights, and reject ambiguous/unverified targets.
- [ ] 9.3 Implement ownership-limited cleanup and server/pane identity revalidation. **Verification:** reused pane IDs, mismatched tags, and server restarts never delete an unowned or agent pane and produce actionable backend state.
- [ ] 9.4 Implement the run session host, child/PTY ownership when needed, privacy-safe revision stream, and recoverable registry. **Verification:** closing or killing only the HUD view leaves child I/O usable and permits explicit reattachment to the surviving launch.
- [ ] 9.5 Implement native child argv/cwd/environment preservation, documented correlation variables, once-only signal delivery, and exact exit propagation. **Verification:** a fixture child receives literal spaces/wildcards/substitution syntax, preserved environment/cwd, one intended interrupt, and returns status 7 unchanged.
- [ ] 9.6 Enforce separate standalone run output via `--hud-tty` or plain `--output`, and isolate HUD shortcuts from agent input. **Verification:** same-TTY interactive conflicts fail before launch, separate-TTY runs work, and agent-directed shortcut keys arrive unchanged.

## 10. Core CLI and Foundation/MVP milestone

- [ ] 10.1 Add command parsing/help and typed exit statuses for `attach`, `run`, `inspect`, `doctor`, `adapters list`, `adapters validate`, and `config validate`. **Verification:** command fixtures cover usage/config/unresolved/runtime failures and exact post-launch child exit propagation.
- [ ] 10.2 Implement deterministic adapter selection and explicit attach binding without agent launch/mutation. **Verification:** argument/config/child/environment evidence follows precedence, conflicts are explained, and ambiguous attach selects neither candidate.
- [ ] 10.3 Implement redacted `inspect`/`doctor --json`, local snapshot export, and targeted view actions. **Verification:** stdout is one valid JSON result, errors stay on stderr, all required identity/provenance/source/backend counters are present safely, and only the selected HUD responds to view controls.
- [ ] 10.4 Run the Foundation/MVP offline vertical slice across generic file/pipe input and standalone/tmux views. **Verification:** run/attach/resize/freeze/rollover/disconnect/recovery and all core privacy/replay checks pass with unknown unsupported fields and no provider credentials/network use; record this milestone without completing the full change.

## 11. Codex native observer

- [ ] 11.1 Define isolated Codex observer profile manifests and obtain sanitized local CLI-versioned rollout/state/log samples or mark absent evidence unsupported. **Verification:** manifest checks tie each advertised field/format to real sample provenance and unverified versions cannot enable a guessed parser.
- [ ] 11.2 Implement verified durable rollout reading with incremental bounded parsing and normalized field translation. **Verification:** profile fixtures cover supported model/turn/context/accounting fields, partial records, malformed input, duplicate replay, truncation, and rotation.
- [ ] 11.3 Implement verified read-only state-database discovery/reconciliation in a bounded worker. **Verification:** locked/unsupported database tests degrade safely, database checksums remain unchanged, and two concurrent roots cannot bind from cwd alone.
- [ ] 11.4 Implement supported fast log updates with per-field authority and read watermarks. **Verification:** a newer model-selection event survives an older durable read and unsupported log profiles withdraw only their own capabilities.
- [ ] 11.5 Integrate native launch/pane evidence and root rollover into the source controller. **Verification:** readable-old-rollout, fork/subagent, missed-watcher, PID-reuse, and changed-account fixtures publish the correct binding generations without native UI replacement.

## 12. Claude Gemini and Aider profiles

- [ ] 12.1 Define locally verified Claude hook/command profiles and an explicit correlation/chaining shim. **Verification:** real-sample provenance is recorded where available, uncorrelated hooks are rejected, and existing configured status-line output is preserved by the template check.
- [ ] 12.2 Implement Claude sparse-payload translation and generation refresh for short-lived producers. **Verification:** changing hook PIDs with one launch key stay in one binding, omissions preserve independently fresh fields, clears invalidate, and prior-generation hooks cannot affect a new session.
- [ ] 12.3 Add separate Gemini and Aider executable/version/process/project fallback profiles. **Verification:** launched fixture executables expose only verified fallback capabilities, report unsupported richer state explicitly, and pass the common conformance suite.
- [ ] 12.4 Generate the adapter compatibility matrix and actionable version/health reports from manifests. **Verification:** matrix entries include modes, tested versions, identity strength, fields, limitations, and recovery, and unverified installations remain visibly unverified.

## 13. Activity collaboration and optional content

- [ ] 13.1 Add stable tool-call activity, permitted targets, monotonic duration, completion/error state, and bounded completed-call records. **Verification:** concurrent repeated tool names remain separate by ID and default output/storage excludes arguments.
- [ ] 13.2 Add configured/loaded/active skill/plugin/MCP distinctions and verified subagent parent/model/status data. **Verification:** configured-but-unused fixtures are not labeled active and unsupported activity claims become visible anomalies.
- [ ] 13.3 Implement explicit parent/child inclusion metadata and safe optional aggregation. **Verification:** unknown or inclusive parent totals do not add child totals, while verified disjoint/exclusive scopes can aggregate exactly once.
- [ ] 13.4 Add scoped plan/todo completion, goals, titles, turn counts, and optional adapter-supplied summaries. **Verification:** default privacy hides content, enabled summary display requires the documented explicit content/retention controls, and no HUD-issued summarization request occurs.
- [ ] 13.5 Present context/accounting/drop/unsupported-activity anomalies through common descriptors and inspection. **Verification:** anomaly fixtures retain explanatory provenance and safe compact indicators in monochrome/narrow layouts without provider renderer branches.

## 14. Usage limits and opt-in providers

- [ ] 14.1 Implement arbitrary scoped limit/balance/cost records with stable IDs and explicit units/currency/metric kinds. **Verification:** three-window, two-account, credit/cost, mixed-currency, and overlapping-window fixtures do not merge incompatible records.
- [ ] 14.2 Add valid percent derivation, wall-clock reset countdown, observation TTL, keyed caches, and deadline reconciliation. **Verification:** zero/unknown capacity, stale cache, account switch, failed query, and expired deadline traces never fabricate replenishment or zero usage.
- [ ] 14.3 Implement explicitly enabled argv-based usage commands with timeout/output/concurrency/rate limits and safe credential references. **Verification:** default startup executes no usage command, literal argument tests avoid shell expansion, and timeout/oversized output produce bounded redacted failures.
- [ ] 14.4 Implement explicitly enabled HTTP providers with destination/redirect validation and scoped credentials. **Verification:** local test-server fixtures reject unapproved/private destinations and cross-origin credential forwarding and enforce response/time/concurrency limits without real provider credentials.
- [ ] 14.5 Add optional dated local pricing estimates and provider-namespaced usage extensions. **Verification:** estimates retain pricing source/date/currency/category evidence, missing pricing remains unknown, and a new request-count provider uses unchanged common rendering.

## 15. Explicit controlled mode and integration templates

- [ ] 15.1 Define a locally verified Codex app-server transport/profile and the compatible-client relay contract. **Verification:** protocol fixtures identify version/identity/event support and incompatible negotiation fails explicitly without altering native observer behavior.
- [ ] 15.2 Implement explicit controlled server/client launch and transparent bounded protocol relay in the session host. **Verification:** a local fixture client/server exchange preserves request/response ordering, IDs, terminal I/O, signals, and client exit status without provider access.
- [ ] 15.3 Translate verified app-server events and mirror approval waiting without creating approval responses. **Verification:** pending approval remains unresolved until the compatible client supplies its deliberate correlated response, and telemetry inputs cannot send control requests.
- [ ] 15.4 Verify controlled session lifetime under HUD view failure and reconnect. **Verification:** renderer loss leaves client/server I/O and approvals usable, reattachment obtains current generation, and no pending action is auto-approved or canceled by HUD teardown.
- [ ] 15.5 Add `yagent`/native-default `yodex` and inspectable zsh/bash/fish integration templates. **Verification:** shell checks preserve argv/env/cwd/status and once-only signals, sourcing/removing templates is reversible, and ordinary install/run commands leave startup files and existing agent hooks unchanged.

## 16. Whole-platform release verification

- [ ] 16.1 Run shared adapter conformance against every shipped mode/profile and add a fixture-only new adapter. **Verification:** all schema/capability/unit/scope/provenance/clear/privacy/unsupported-field scenarios pass and adding the adapter changes no common renderer/backend code.
- [ ] 16.2 Complete deterministic replay coverage for all listed recovery/accounting/freshness/reset/suspension traces. **Verification:** repeated input/clock traces yield identical revisions, generations, totals, anomalies, and redacted diagnostics.
- [ ] 16.3 Run isolated PTY/tmux integration across both backends and supported run modes. **Verification:** focus, width/heights, resize, signals, terminal restoration, child survival, server restart, and cleanup assertions pass without touching unrelated terminal resources.
- [ ] 16.4 Run adversarial input/privacy checks across every output and retained sink. **Verification:** injection, oversized/deep data, secret paths/env/log/error/export content, provider limits, and frozen/reload fixtures reveal no forbidden sentinels or untrusted terminal controls.
- [ ] 16.5 Implement/document reference resource benchmarks and enforce default ingestion/work limits. **Verification:** measured aggregate host/view/worker CPU and RSS meet the 1% idle/128 MiB budgets, p95 event-to-frame latency meets 250 ms, and burst recovery stays within 1024 records/8 MiB/256 KiB/depth-32 limits with responsive input.
- [ ] 16.6 Complete bounded local logging and operational diagnostic metrics. **Verification:** parser/retry storms increment accurate counters while log size/age stays bounded and reports include required versions, identity, generation, lag, reconciliation, latency, and resource evidence without upload.
- [ ] 16.7 Publish an acceptance mapping from every requirement/scenario to its fixture/check and the corresponding roadmap section. **Verification:** a coverage audit finds no uncovered Foundation/MVP/Near-term or cross-cutting gate and explicitly identifies all Later features as deferred.

## 17. Packaging migration and release readiness

- [ ] 17.1 Package the compiled CLI, schemas, examples/templates, Apache-2.0 and dependency notices, and checksums as a local release candidate. **Verification:** package inspection and checksum verification pass and no provider data/credentials or unpublished raw fixtures are included.
- [ ] 17.2 Implement explicit versioned configuration migration with atomic replacement, backup, and rollback guidance. **Verification:** predecessor-fixture migration preserves a restorable original, rejects unknown newer majors, and failed writes leave the prior valid file intact.
- [ ] 17.3 Run clean-environment install/upgrade/demo/run/attach/doctor/uninstall checks on Linux x86_64 and arm64 with Node.js 24+ and optional tmux 3.3+. **Verification:** prepared packages work without provider credentials or default network queries, native PTY dependencies load, startup files remain unchanged, and uninstall removes only installed HUD-owned files.
- [ ] 17.4 Document command/protocol/configuration/privacy/source recovery, standalone terminal ownership, controlled-client requirements, compatibility, and support/deprecation policy. **Verification:** every documented offline command is exercised, version/support claims match profile evidence, and rollback/uninstall instructions preserve active agents and agent-owned telemetry.
- [ ] 17.5 Run the complete release gate and prepare a reviewable local candidate without publication. **Verification:** all tasks and acceptance mappings are satisfied, typecheck/contracts/replay/privacy/render/profile/PTY/tmux/install/resource checks pass, and the candidate includes recorded compatibility and reference-system evidence; publishing remains a separately authorized action.
