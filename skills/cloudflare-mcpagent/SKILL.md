---
name: cloudflare-mcpagent
description: "Use when building a stateful remote MCP server on Cloudflare Workers with Agents SDK McpAgent, Durable Objects, Streamable HTTP/SSE transport, and storage-aware caching. Trigger: /cloudflare-mcpagent"
allowed-tools: Bash(pnpm:*), Bash(npm:*), Bash(npx:*), Bash(wrangler:*), Bash(git:*), Read, Write, Edit, MultiEdit
---

# Cloudflare McpAgent

Use this skill when an MCP server needs per-session state on Cloudflare Workers. `McpAgent` gives each MCP server instance durable state backed by Durable Objects and integrates with Cloudflare's Agents SDK transports.

## Core Rule

Choose `McpAgent` for state. If the server is stateless, use `createMcpHandler` or a stateless edge MCP pattern instead.

## When to Use

- MCP clients need per-session counters, workflow state, elicitation state, or resumable transport state.
- OAuth/user state must survive across requests.
- You want Cloudflare Workers deployment with Streamable HTTP and optional SSE compatibility.
- You need Durable Object locality and state APIs.

Do not use this for a read-only or stateless tool bundle just because it is convenient. Stateless servers avoid Durable Object migrations and storage-cost design.

## Approach Selection

| Need | Prefer |
| --- | --- |
| Stateless tools in a plain Worker | `createMcpHandler()` from `agents/mcp` |
| Stateful MCP with Durable Object state | `McpAgent` |
| Full manual transport control | Raw MCP SDK transport |
| Edge helper without DO state | `/edgefastmcp-cloudflare` |

## Minimal File Layout

```text
src/
  mcp.ts       # McpAgent subclass and MCP primitive registration
  index.ts    # Worker entrypoint, auth/routing, serve('/mcp')
wrangler.jsonc
package.json
```

## McpAgent Class

Prefer `registerTool`, `registerResource`, and `registerPrompt` for the underlying `McpServer` primitives.

```ts
import { McpAgent } from 'agents/mcp'
import { McpServer, ResourceTemplate } from '@modelcontextprotocol/sdk/server/mcp.js'
import { z } from 'zod'

export class ProjectMCP extends McpAgent<Env> {
  server = new McpServer({ name: 'project-mcp', version: '0.1.0' })

  async init() {
    this.server.registerTool(
      'search_project',
      {
        description: 'Search project knowledge for a query.',
        inputSchema: z.object({
          query: z.string().min(1),
          limit: z.number().int().min(1).max(10).default(3),
        }),
      },
      async ({ query, limit }) => {
        const results = await this.searchWithCache(query, limit)

        return {
          content: results.map((result) => ({
            type: 'text' as const,
            text: `[${result.source}]
${result.text}`,
          })),
        }
      },
    )

    this.server.registerResource(
      'project-page',
      new ResourceTemplate('project://page/{path}', { list: undefined }),
      { title: 'Project page', mimeType: 'text/markdown' },
      async (uri, { path }) => ({
        contents: [{ uri: uri.href, mimeType: 'text/markdown', text: await this.readPage(String(path)) }],
      }),
    )
  }

  private async searchWithCache(query: string, limit: number) {
    // Use L1 memory first, then ctx.storage only on miss if durable recovery is needed.
    return searchOrigin(this.env, query, limit)
  }

  private async readPage(path: string) {
    return readOriginPage(this.env, path)
  }
}
```

## Worker Entrypoint

Keep the entrypoint thin: route, authenticate, and delegate to `serve()`.

```ts
import { ProjectMCP } from './mcp'

export { ProjectMCP } from './mcp'

const readBearer = (value: string | null) => {
  if (value?.startsWith('Bearer ')) {
    return value.slice('Bearer '.length)
  }

  return null
}

export default {
  async fetch(request, env, ctx): Promise<Response> {
    const url = new URL(request.url)

    if (!url.pathname.startsWith('/mcp')) {
      return new Response('Not Found', { status: 404 })
    }

    const incomingToken =
      url.searchParams.get('apiKey') ??
      request.headers.get('apiKey') ??
      readBearer(request.headers.get('Authorization'))

    if (env.MCP_API_KEY && incomingToken !== env.MCP_API_KEY) {
      return Response.json({ error: 'Unauthorized' }, { status: 401 })
    }

    return ProjectMCP.serve('/mcp').fetch(request, env, ctx)
  },
} satisfies ExportedHandler<Env>
```

For third-party clients, prefer OAuth with Cloudflare's Workers OAuth provider or Cloudflare Access over a shared API key.

## Wrangler Configuration

Export the `McpAgent` class and bind it as a Durable Object. New classes should generally use SQLite-backed migrations.

```jsonc
{
  "$schema": "node_modules/wrangler/config-schema.json",
  "name": "project-mcp",
  "main": "src/index.ts",
  "compatibility_date": "2026-02-24",
  "compatibility_flags": ["nodejs_compat"],
  "migrations": [
    { "tag": "v1", "new_sqlite_classes": ["ProjectMCP"] }
  ],
  "durable_objects": {
    "bindings": [
      { "name": "MCP_OBJECT", "class_name": "ProjectMCP" }
    ]
  }
}
```

If the project uses Wrangler autoconfig or framework-managed config, verify that `wrangler types` and deploy still see the Durable Object export and migration.

## Storage and Cost Rules

`McpAgent` state is backed by Durable Objects. Before adding storage calls, use `/cloudflare-do-storage-caching`.

- Do not call `ctx.storage.get()` on every tool invocation unless there is no cheaper state boundary.
- Keep hot cache in class fields or `Map` values.
- Use `ctx.storage` for cold-start recovery, transport state, or durable user/session state.
- Avoid request-path `storage.list()`.
- Set alarms only when absent or when a meaningful schedule changes.
- Store large bodies in R2/D1/origin systems; keep DO storage small.

### Transcript and Run Artifact Pattern

When using `McpAgent` as a terminal/sandbox control plane, do not assume it automatically makes sandbox PTY or command transcripts durable. The application must explicitly observe output and append it to a transcript store.

Use this layering:

- **McpAgent / DO instance memory:** live tail ring buffer, hot session/run maps, recent events for `read_terminal_buffer`-style tools.
- **DO storage:** small durable boundaries only — run/session metadata, cursors, sequence numbers, R2 chunk pointers, status transitions.
- **R2:** long transcript chunks, stdout/stderr bodies, patches, review artifacts, and other large append-only output.

Avoid writing every terminal event directly to `ctx.storage`. Prefer batching or chunk flushing (for example by byte threshold, short interval, or run/session close), then store only the latest pointer/cursor in DO storage. If a WebSocket pass-through is opaque and cannot be observed, record visible `run_command` results first and document that interactive PTY transcript capture is deferred.

## Async Sandbox Run Pattern

When an MCP server fronts Cloudflare Sandbox or other long-running command execution, keep MCP as the control plane and move execution/feedback to durable Cloudflare primitives. See `references/sandbox-async-run-feedback.md` for the detailed pattern.

Recommended MVP:

```text
Hermes -> MCP enqueue_run -> Command Queue -> Workspace DO/Sandbox
Sandbox -> DO/D1 state + R2 logs/artifacts -> Feedback Queue -> Hermes Webhook/Gateway -> original reply target
```

Rules:

- `enqueue_run` should return quickly with `{ runId, status: "queued" }`; do not hold the MCP request open for terminal completion.
- Use MCP for short control/query operations: `enqueue_run`, `get_run_status`, `get_run_logs`, `cancel_run`, and later `send_run_input`.
- Put full logs and artifacts in R2; queue only small event payloads, summaries, cursors, and references.
- Let Hermes (or the owning agent) produce the final Discord/GitHub/user-facing response. MCP should not become the feedback poster.

## Transport Notes

Cloudflare's Agents SDK supports multiple MCP paths. For production remote clients, Streamable HTTP is the normal default. SSE is legacy compatibility. RPC is useful for internal Agent-to-McpAgent connections in the same Worker/runtime and does not replace public authentication.

## Verification

```bash
pnpm wrangler types
pnpm typecheck
pnpm test
pnpm build
pnpm wrangler deploy --dry-run
```

Also test with an MCP client or adapter:

```bash
npx mcp-remote https://<worker-host>/mcp
```

Verify:

- The Worker exports the `McpAgent` class by name.
- The DO binding and migration reference the exact class name.
- `/mcp` responds through the expected transport.
- Unauthorized requests fail before reaching MCP tools.
- Repeated hot tool calls do not repeatedly hit `ctx.storage` or expensive origins.

## Common Pitfalls

- Using `McpAgent` for stateless tools and inheriting DO cost/complexity unnecessarily.
- Forgetting the named export for the DO class.
- Changing the DO class name without adding a correct Wrangler migration.
- Putting auth, routing, and tool implementation all in `fetch()` instead of keeping the entrypoint thin.
- Treating API-key query parameters as production-grade auth for third-party clients.
- Writing cache cleanup code that costs more than the data it cleans.

## Checklist

- [ ] The server truly needs state across requests.
- [ ] Stateless alternatives were rejected explicitly.
- [ ] MCP primitives are registered inside the agent class.
- [ ] Worker entrypoint only handles route/auth/delegation.
- [ ] Durable Object binding and migration are present.
- [ ] Storage access follows the DO caching checklist.
- [ ] Type generation, typecheck, build, and deploy dry-run pass.
