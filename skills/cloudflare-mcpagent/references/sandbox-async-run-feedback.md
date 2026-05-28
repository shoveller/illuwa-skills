# Sandbox async run feedback pattern

Use this when an MCP server is the agent-facing control plane for Cloudflare Sandbox or long-running command execution.

## Shape

```txt
Hermes / agent client
  -> MCP tool: enqueue_run(command, origin/replyContext)
  <- { runId, status: "queued" }

Command Queue
  -> Queue Consumer or Workflow
  -> Workspace Durable Object / Sandbox
  -> execute command
  -> DO/D1: run metadata and state
  -> R2: full logs, log chunks, artifacts
  -> Feedback Queue: run.completed | run.failed | run.needs_input

Feedback Consumer
  -> Hermes Webhook / Gateway
  -> Hermes optionally calls MCP get_run_status/get_run_logs
  -> original chat/thread or external feedback target
```

## Durable boundaries

- MCP response should be short and synchronous: enqueue, return `runId`, and stop.
- Queue messages should carry status, event type, ids, small summary/tail references, and artifact references — not full stdout/stderr.
- DO/D1 should own status transitions, cursors, idempotency keys, and reply context.
- R2 should own full logs, transcript chunks, diffs, and artifacts.
- Hermes should create the human-facing summary/reply; MCP should not try to post the final Discord/GitHub response itself.

## Minimal MCP tools

- `enqueue_run(command, origin | replyContext)`
- `get_run_status(runId)`
- `get_run_logs(runId, cursor?)`
- `cancel_run(runId)`
- later: `send_run_input(runId, data)` for `run.needs_input`

## Why

This avoids tying long-running stdout/stderr streams to a single MCP tool response. It also gives retries, cancellation, reconnect, idempotency, and artifact retention a Cloudflare-native owner instead of hiding them inside a client request lifecycle.
