# Terminal Streaming Buffer Pattern for MCP Servers

Use this when an MCP server also exposes an interactive browser/WebSocket terminal and the user asks whether MCP is acting as the stream buffer.

## Diagnostic checklist

Inspect the transport boundary before designing new MCP tools:

1. Find the MCP route/tool registration file.
   - Check whether `run_command` returns only a completed `exec()` result.
   - Check whether tools prepare a sandbox/session per call or share a named session.
2. Find the terminal WebSocket route.
   - If it directly returns `sandbox.getSession(sessionId).terminal(request, options)` or equivalent, it is pass-through, not a buffer.
   - If it creates a `WebSocketPair` and relays messages itself, it can observe and buffer input/output.
3. Check whether `sandboxId`, `sessionId`, and `runId` are distinct.
   - `sandboxId`: container/DO identity.
   - `sessionId`: shell/PTY context.
   - `runId`: artifact/review lifecycle identity.
4. Verify whether command execution is actually session-aware.
   - `sandbox.exec(command)` may not run in the same named PTY session as the browser terminal.
   - Do not claim same-session behavior until SDK/API support is verified.

## Minimal implementation shape

Prefer a small transcript interface before adding a full run registry:

```ts
type TerminalDirection = 'input' | 'output' | 'system'

interface TerminalBufferEvent {
  seq: number
  direction: TerminalDirection
  text: string
  timestamp: string
}

interface TerminalBufferSnapshot {
  sandboxId: string
  sessionId: string
  fromSeq: number
  nextSeq: number
  truncatedBeforeSeq: number
  events: TerminalBufferEvent[]
}
```

MCP tool shape:

```ts
read_terminal_buffer({ sandboxId, sessionId, offset?, limit? })
```

Rules:

- Reading the buffer should not prepare or start a sandbox.
- Start with an isolate-local ring buffer only if acceptable; document that it is not durable across cold starts, isolate splits, or deploys.
- If WebSocket pass-through is opaque and cannot be relayed, first buffer `run_command` input/output under the same interface and mark PTY transcript buffering as a follow-up spike.
- Keep low-level tools narrow: `run_command`, `read_file`, `write_file`, plus a read-only buffer tool. Remove agent-specific tools such as `opencode` when the goal is terminal control rather than agent execution.

## Planning guidance

When writing the plan, include:

- Current route/file evidence proving whether buffering exists.
- Baseline test commands and results before changing code.
- A spike task for SDK WebSocket relay observability.
- A fallback task that buffers completed command results if PTY relay cannot be observed.
- README updates that state buffer durability and session-awareness limits.

## Implementation notes from workflow-harness

A successful Phase 1 pattern was:

1. Remove agent-specific tools (`opencode`) from the low-level MCP surface.
2. Keep `run_command` backward-compatible with `command` only, then add optional `sandboxId` and `sessionId` for transcript capture.
3. Record two events only when both IDs are present:
   - `{ direction: 'input', source: 'run_command', text: command }`
   - `{ direction: 'output', source: 'run_command', text: JSON.stringify(result, null, 2) }`
4. Add a read-only `read_terminal_buffer({ sandboxId, sessionId, offset?, limit? })` tool that never prepares/starts a sandbox.
5. Put transcript types and store implementation behind a narrow module/export such as `server/transcript`.
   - Avoid importing a broad server barrel from MCP code if that barrel also exports UI/page/WebSocket helpers; it can pull HTML/assets into server-side tests or bundles.
6. Implement the first store as an isolate-local ring buffer with explicit `durability: 'memory'`, `seq`, `nextSeq`, and `truncatedBeforeSeq` fields.
7. Test both no-ID backward compatibility and ID-provided transcript recording.

Pitfalls:

- Do not imply `sandbox.exec(command)` runs in the same named PTY session as `/ws/terminal/{sandboxId}/{sessionId}` unless the SDK proves that relationship.
- If project lint forbids loops/imperative mutation, make ring-buffer trimming a single bounded drop per append or use slice-based trimming.
- Keep durable storage as a later adapter: DO memory for live tail, DO storage for small state/cursors/pointers, R2 for transcript chunks/artifacts.
