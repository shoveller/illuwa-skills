---
name: edgefastmcp-cloudflare
description: "Use when building a stateless MCP server on Cloudflare Workers with EdgeFastMCP or a plain Worker MCP handler, avoiding Durable Objects unless session state is required. Trigger: /edgefastmcp-cloudflare"
allowed-tools: Bash(pnpm:*), Bash(npm:*), Bash(npx:*), Bash(wrangler:*), Bash(git:*), Read, Write, Edit, MultiEdit
---

# EdgeFastMCP Cloudflare

Use this skill when you need a stateless MCP server on Cloudflare Workers. The goal is to avoid Durable Objects, migrations, and storage-cost design unless the MCP server actually needs per-session state.

## Core Rule

One request in, one response out. If a tool needs durable session state, OAuth token persistence, elicitation state, or multi-step workflow state, switch to `/cloudflare-mcpagent`.

## When to Use

- The MCP server exposes read-only search, fetch, transform, or API-wrapper tools.
- Each tool can complete within a single request.
- You want a Cloudflare Worker deployment without Durable Object bindings or migrations.
- Authentication can be handled by Cloudflare Access, Hono middleware, an API gateway, or another stateless boundary.

Do not use this for long-running POSIX work, repository mutation, package installs, browser automation, or anything requiring bash/git/npm execution. Use a sandbox/container or external runner for those tasks.

## Choosing the Stateless Handler

| Option | Use when |
| --- | --- |
| `createMcpHandler()` from `agents/mcp` | You want Cloudflare's official Agents SDK stateless Worker handler. |
| `EdgeFastMCP` from `fastmcp/edge` | You prefer FastMCP's compact tool API and edge export model. |
| Raw MCP SDK transport | You need full transport control and accept more boilerplate. |

Cloudflare's official docs currently emphasize `createMcpHandler()` for stateless Workers. Use EdgeFastMCP when the project has already standardized on FastMCP or its API better fits the codebase.

## EdgeFastMCP Minimal Shape

```ts
import { EdgeFastMCP } from 'fastmcp/edge'
import { z } from 'zod'

const server = new EdgeFastMCP({
  name: 'stateless-edge-mcp',
  version: '0.1.0',
  description: 'Stateless MCP server running on Cloudflare Workers',
})

server.addTool({
  name: 'greet',
  description: 'Return a greeting from the edge.',
  parameters: z.object({ name: z.string().default('world') }),
  execute: async ({ name }) => `Hello, ${name}.`,
})

export default server
```

No `server.start()` is needed in Workers. Export the Worker-compatible handler/module.

## Official Agents SDK Stateless Shape

When using Cloudflare's `agents/mcp`, create fresh server/transport state per request if the SDK version requires it.

```ts
import { createMcpHandler } from 'agents/mcp'
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js'
import { z } from 'zod'

const createServer = () => {
  const server = new McpServer({ name: 'stateless-worker-mcp', version: '0.1.0' })

  server.registerTool(
    'hello',
    {
      description: 'Return a greeting.',
      inputSchema: z.object({ name: z.string().optional() }),
    },
    async ({ name }) => ({
      content: [{ type: 'text' as const, text: `Hello, ${name ?? 'world'}.` }],
    }),
  )

  return server
}

export default {
  fetch(request, env, ctx) {
    const server = createServer()
    return createMcpHandler(server, { route: '/mcp' })(request, env, ctx)
  },
} satisfies ExportedHandler<Env>
```

## Wrangler Configuration

A stateless MCP Worker does not need Durable Object bindings or migrations.

```jsonc
{
  "$schema": "node_modules/wrangler/config-schema.json",
  "name": "stateless-mcp",
  "main": "src/index.ts",
  "compatibility_date": "2026-02-24",
  "compatibility_flags": ["nodejs_compat"]
}
```

Add R2, KV, D1, AI, or service bindings only when tools actually need them.

## Hono and Auth Integration

If the chosen library exposes or can be wrapped by a Hono app, keep auth in middleware and MCP registration in a separate module.

```ts
app.use('/mcp/*', async (c, next) => {
  const token = c.req.header('authorization')

  if (token !== `Bearer ${c.env.MCP_API_KEY}`) {
    return c.json({ error: 'Unauthorized' }, 401)
  }

  await next()
})
```

For production or third-party clients, prefer Cloudflare Access or OAuth-capable flows over a shared static key.

## Stateless Tool Rules

- Do not store user-critical state in module-level variables.
- Do not rely on isolate memory for permission revocation or workflow progress.
- Prefer upstream HTTP caching, Cloudflare Cache API, or deterministic recomputation.
- Keep secrets in Worker secrets/bindings, not tool arguments or MCP resources.
- Return compact content blocks; store large outputs in the right backing service and return links/ids when appropriate.

## Escalation Triggers

Switch to `/cloudflare-mcpagent` when:

- The client needs per-session state across requests.
- The MCP server needs elicitation, sampling state, or durable transport state.
- OAuth tokens or user grants must be persisted in the runtime.
- You are adding Durable Object storage or alarms.

Switch to a sandbox/container when:

- The tool needs filesystem writes, git, package managers, browser automation, or long-running processes.

## Verification

```bash
pnpm wrangler types
pnpm typecheck
pnpm test
pnpm build
pnpm wrangler deploy --dry-run
```

Then smoke test the deployed or dev URL with an MCP client or adapter:

```bash
npx mcp-remote http://localhost:8787/mcp
```

Confirm invalid auth fails before tool execution and valid calls do not require DO bindings.

## Common Pitfalls

- Adding Durable Objects to a stateless server “just in case”.
- Keeping a global `McpServer`/transport instance across requests when the SDK version requires per-request instances.
- Treating module-level `Map` as durable storage.
- Forgetting that Workers do not provide a persistent filesystem.
- Exposing a public MCP endpoint without Access, OAuth, or another auth boundary.

## Checklist

- [ ] Every tool completes in one request.
- [ ] No per-session durable state is required.
- [ ] No Durable Object binding or migration is present.
- [ ] Auth is handled before MCP tool execution.
- [ ] Worker bindings are minimal and purpose-specific.
- [ ] Typecheck, build, and deploy dry-run pass.
