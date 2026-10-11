# Architecture

## Layers

```text
agent CLI ──┐
            ├── adapter ──> normalized state ──> renderer ──> terminal backend
agent CLI ──┘                                  ├── tmux pane
                                               └── standalone TUI
```

### Adapters

An adapter owns discovery and translation for one agent CLI. It emits a common
state object and may expose provider-specific fields under a namespaced area.
An adapter should prefer sources in this order:

1. A supported live event/API stream.
2. A documented CLI hook or status-line protocol.
3. A structured rollout or state file with filesystem watching.
4. A local database or event log, treated as internal and version-sensitive.
5. Process, terminal, Git, and shell fallbacks.

### Normalized state

The shared state should include at least:

```json
{
  "agent": "codex",
  "cliVersion": "0.0.0",
  "sessionId": "...",
  "source": {
    "kind": "rollout",
    "path": "...",
    "updatedAt": "...",
    "freshness": "live"
  },
  "project": {
    "cwd": "...",
    "root": "...",
    "branch": "...",
    "dirty": false
  },
  "model": {
    "name": "...",
    "provider": "...",
    "effort": "..."
  },
  "turn": {
    "state": "idle",
    "startedAt": "...",
    "durationMs": 0
  },
  "context": {
    "usedTokens": 0,
    "totalTokens": 0,
    "percent": 0
  },
  "usage": {},
  "permissions": {},
  "activity": {
    "tools": [],
    "skills": [],
    "mcp": [],
    "agents": []
  },
  "capabilities": []
}
```

Unknown values must be `null` or omitted. A source age and freshness state are
required so the renderer can distinguish live, cached, stale, and disconnected
data.

## Synchronization

The HUD should maintain a source controller rather than a permanent file path:

1. Establish identity at launch using an explicit session ID, child PID, and
   adapter name.
2. Subscribe to the source's live events where possible.
3. Watch the current structured file and debounce updates for approximately
   25–100 ms.
4. Reconcile the active thread/rollout periodically, even when the old source
   still exists.
5. Atomically replace the source when the agent creates a new session or
   rollout.
6. Mark fields stale when their source stops advancing.

For Codex native-TUI observation, the rollout is the durable source, while the
state database detects new root threads and the event log can provide faster
model-selection updates. These database interfaces should be isolated in the
Codex adapter because they are implementation details and may change.

For a HUD-controlled Codex session, app-server events are preferred because the
HUD can receive the same thread and turn events as the client. A controlled
app-server mode is an integration choice, not a requirement for the observer
mode.

## Backends

The renderer should not know how the agent is hosted. Backends handle pane or
terminal concerns:

- tmux split/pane lifecycle and resize
- standalone alternate-screen TUI
- future local socket or desktop status-bar output

## Security and privacy

- Never display prompt or response bodies by default.
- Redact paths, account identifiers, and tokens in diagnostics.
- Treat provider logs and local databases as sensitive.
- Make network usage explicit and opt-in.
- Keep account credentials in the agent's credential store; adapters receive
  only the minimum data needed to display a status.
