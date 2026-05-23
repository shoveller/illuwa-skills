---
name: mcp-sdk-typescript
description: "Use when building or refactoring a TypeScript MCP server with @modelcontextprotocol/sdk, designing tools/resources/prompts, and keeping transport-specific state/auth/caching separate. Trigger: /mcp-sdk-typescript"
allowed-tools: Bash(pnpm:*), Bash(npm:*), Bash(npx:*), Bash(git:*), Read, Write, Edit, MultiEdit
---

# MCP SDK TypeScript

Use this skill when implementing a Model Context Protocol server in TypeScript with the official SDK. Start here before choosing a Cloudflare-specific wrapper, FastMCP, or a stateful runtime.

## Core Rule

Design the MCP contract first: prompts guide the workflow, resources expose read-only context, and tools perform narrowly scoped actions.

Do not mix runtime concerns such as OAuth, Durable Objects, cache storage, or Cloudflare routing into the core MCP primitive design.

## When to Use

- You need a Node.js, stdio, HTTP, or runtime-agnostic MCP server.
- You are deciding which features should be tools, resources, or prompts.
- You need examples based on `McpServer`, `registerTool`, `registerResource`, `registerPrompt`, and `ResourceTemplate`.
- You are reviewing an MCP server for schema quality, side effects, or prompt/resource/tool boundaries.

Do not use this alone when:

- You need per-session state on Cloudflare Workers. Use `/cloudflare-mcpagent`.
- You need a stateless Cloudflare Worker with Agents SDK `createMcpHandler` or another edge helper. Use `/edgefastmcp-cloudflare` or a Cloudflare-specific skill.
- You are only configuring an MCP client.

## Primitive Map

| Primitive | Use for | Avoid |
| --- | --- | --- |
| Prompt | Reusable instructions and sequencing hints | Hiding dynamic data or credentials |
| Resource | Stable read-only context addressed by URI | Mutations, long computations, writes |
| Tool | Model-callable action with validated inputs | Broad arbitrary execution or vague side effects |

Recommended flow:

```text
Prompt -> search/list tool -> read resource -> specific action tool
```

## Minimal Server Shape

Prefer explicit registration methods and config objects. Recent SDK versions have moved away from older variadic overloads.

```ts
import { McpServer, ResourceTemplate } from '@modelcontextprotocol/sdk/server/mcp.js'
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js'
import { z } from 'zod'

const server = new McpServer({ name: 'project-guide', version: '0.1.0' })

server.registerTool(
  'search_docs',
  {
    description: 'Search project documentation for a natural language query.',
    inputSchema: z.object({
      query: z.string().min(1),
      limit: z.number().int().min(1).max(10).default(3),
    }),
  },
  async ({ query, limit }) => {
    const results = await searchDocs(query, limit)

    return {
      content: results.map((item) => ({
        type: 'text' as const,
        text: `[${item.path}]
${item.snippet}`,
      })),
    }
  },
)

server.registerResource(
  'doc-page',
  new ResourceTemplate('docs://page/{path}', { list: undefined }),
  { title: 'Project document', mimeType: 'text/markdown' },
  async (uri, { path }) => ({
    contents: [{ uri: uri.href, mimeType: 'text/markdown', text: await readDoc(String(path)) }],
  }),
)

server.registerPrompt(
  'use-project-docs',
  {
    description: 'Guide the client to search before answering from project docs.',
    argsSchema: z.object({ topic: z.string().optional() }),
  },
  async ({ topic }) => ({
    messages: [
      {
        role: 'user',
        content: {
          type: 'text',
          text: `Topic: ${topic ?? 'general'}
1. Call search_docs first.
2. Read docs://page/{path} for the best match.
3. Cite the document path in your answer.`,
        },
      },
    ],
  }),
)

await server.connect(new StdioServerTransport())
```

## Tool Rules

- Use verb-like names: `search_docs`, `read_page`, `create_issue`.
- Keep each tool's input schema narrow and validated with `z.object(...)` or another Standard Schema implementation.
- Put side effects in explicitly named tools; do not hide writes inside read/search tools.
- Return MCP content blocks, not arbitrary objects that clients may render inconsistently.
- Include idempotency keys, dry-run flags, or confirmation boundaries for expensive or destructive actions.

## Resource Rules

- Use stable URI schemes that make the data source obvious, such as `docs://page/{path}`.
- Keep resources read-only.
- Use `ResourceTemplate` when a parameterized URI maps cleanly to one document or object.
- Do not use resources for secrets, bearer tokens, or transient private session state.

## Prompt Rules

- Write prompts as workflow guides, not giant system prompts.
- Name the tools and resources the client should use.
- Keep prompts reusable across callers; avoid embedding one user's temporary request.
- Use `argsSchema` for optional focus parameters such as topic, path, or mode.

## Runtime Separation

Keep these out of the core MCP module until the adapter layer:

- Transport selection: stdio, Streamable HTTP, SSE, RPC.
- Runtime auth: API key, OAuth, Cloudflare Access, mTLS.
- Persistence: Durable Objects, D1, R2, Redis, filesystem.
- Cache policy: TTL, stale-while-revalidate, batching.

A good project shape:

```text
src/mcp/server.ts       # create/register McpServer primitives
src/mcp/tools/*.ts      # pure tool implementations and schemas
src/adapters/stdio.ts   # local stdio transport
src/adapters/worker.ts  # Worker/HTTP transport
src/runtime/cache.ts    # runtime-specific cache policy
```

## Verification

Run the narrow checks available in the project:

```bash
pnpm typecheck
pnpm test
pnpm build
```

For manual smoke testing, use an MCP client or inspector appropriate to the transport. For stdio servers, confirm the process starts, lists tools/resources/prompts, and each tool handles both valid and invalid schema inputs.

## Common Pitfalls

- Using one broad `run_anything` tool instead of narrow tools with schemas.
- Registering tools at request time in a way that duplicates names or leaks caller-specific state.
- Using a module-global `McpServer` instance in stateless HTTP code after SDK changes that require per-request server/transport isolation.
- Treating prompts as a replacement for resources; prompts should point the model to resources, not copy all data.
- Returning raw JSON without wrapping it in MCP content blocks.

## Checklist

- [ ] Tool names and descriptions make invocation timing obvious.
- [ ] Every tool has a narrow `inputSchema`.
- [ ] Resources are read-only and URI-addressable.
- [ ] Prompts describe the intended tool/resource sequence.
- [ ] Runtime auth, transport, persistence, and caching are isolated from primitive registration.
- [ ] Typecheck and at least one MCP smoke test pass.
