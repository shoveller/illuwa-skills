# illuwa-skills

Skills shared by Illuwa for Claude Code, OpenCode, and other skills.sh-compatible AI agents. This repository follows the `skills.sh` / `npx skills` layout: each skill lives under `skills/<skill-name>/SKILL.md`.

## Installation

```bash
npx skills add shoveller/illuwa-skills
```

Install a specific skill:

```bash
npx skills add shoveller/illuwa-skills --skill ts-functional-mind
npx skills add shoveller/illuwa-skills --skill ts-functional-mind-react
npx skills add shoveller/illuwa-skills --skill skill-moment-capture
npx skills add shoveller/illuwa-skills --skill mcp-sdk-typescript
npx skills add shoveller/illuwa-skills --skill cloudflare-do-storage-caching
npx skills add shoveller/illuwa-skills --skill cloudflare-mcpagent
npx skills add shoveller/illuwa-skills --skill edgefastmcp-cloudflare
npx skills add shoveller/illuwa-skills --skill hono-cloudflare-worker
```

## Available Skills

| Skill | Description |
|-------|-------------|
| [ts-functional-mind](#ts-functional-mind) | Refactor long TypeScript files into smaller, testable functional units while preserving behavior. |
| [ts-functional-mind-react](#ts-functional-mind-react) | Refactor long TypeScript React files while preserving rendering contracts, state ownership, and side-effect ordering. |
| [skill-moment-capture](#skill-moment-capture) | Capture reusable session judgments, checklists, workflows, and failure-prevention rules as skills. |
| [mcp-sdk-typescript](#mcp-sdk-typescript) | Build runtime-agnostic TypeScript MCP servers with tools, resources, prompts, and clean transport separation. |
| [cloudflare-do-storage-caching](#cloudflare-do-storage-caching) | Review Durable Objects storage costs and design L1/L2 caches for Workers, Agents, and MCP servers. |
| [cloudflare-mcpagent](#cloudflare-mcpagent) | Build stateful remote MCP servers on Cloudflare Workers with Agents SDK `McpAgent` and Durable Objects. |
| [edgefastmcp-cloudflare](#edgefastmcp-cloudflare) | Build stateless MCP servers on Cloudflare Workers without Durable Objects. |
| [hono-cloudflare-worker](#hono-cloudflare-worker) | Wire Cloudflare Worker entrypoints with Hono routes, Wrangler config, Vite, bindings, and route tests. |

---

### ts-functional-mind

Use when a TypeScript file has grown long because side effects, data conversion, domain logic, and variants are mixed together.

```bash
/ts-functional-mind
```

Core behavior:
- Identify behavioral contracts before moving code.
- Extract pure functions before extracting effect coordinators.
- Avoid premature abstractions and generic helpers used only once.
- Preserve API payloads, ids, path strings, persisted data, and side-effect ordering.
- Verify with `pnpm typecheck`, `pnpm test`, `pnpm build`, and `graphify update .` when available.

---

### ts-functional-mind-react

Use when a TypeScript React file has grown long because rendering, state ownership, side effects, data conversion, and UI variants are mixed together.

```bash
/ts-functional-mind-react
```

Core behavior:
- Extract pure data/view helpers before hooks or context.
- Split stateless leaves before moving API calls or state ownership.
- Treat multiple direct `useState` or `useEffect` calls as a signal to inspect workflow ownership.
- Preserve ids, keys, aria labels, drag payloads, and event ordering.
- Verify with `pnpm typecheck`, `pnpm test`, `pnpm build`, and `graphify update .` when available.

---

### skill-moment-capture

Use when a session reveals a reusable judgment, checklist, workflow, or failure-prevention rule that should become a skill or be added to an existing skill.

```bash
/skill-moment-capture
```

Core behavior:
- Identify the future trigger phrase or situation.
- Decide whether to update an existing skill or create a new one.
- Extract commands, paths, safety rules, and verification steps.
- Avoid one-off implementation details and secrets.
- Write operational instructions, not a retrospective.

---

### mcp-sdk-typescript

Use when building or refactoring a TypeScript MCP server with `@modelcontextprotocol/sdk`.

```bash
/mcp-sdk-typescript
```

Core behavior:
- Separate prompts, resources, and tools before choosing runtime adapters.
- Use explicit `registerTool`, `registerResource`, and `registerPrompt` config objects.
- Keep auth, persistence, transport, and caching out of the primitive registration layer.

---

### cloudflare-do-storage-caching

Use when reviewing Cloudflare Durable Objects storage costs or cache design.

```bash
/cloudflare-do-storage-caching
```

Core behavior:
- Make hot requests finish from Durable Object instance memory.
- Treat `ctx.storage` as a durable recovery layer, not a per-request cache lookup.
- Avoid request-path `list()` and repeated alarm writes.
- Use current Cloudflare pricing vocabulary: SQLite rows read/written or legacy KV request units.

---

### cloudflare-mcpagent

Use when building a stateful remote MCP server on Cloudflare Workers with Agents SDK `McpAgent`.

```bash
/cloudflare-mcpagent
```

Core behavior:
- Choose `McpAgent` only when state across requests is required.
- Keep Worker entrypoints thin: route, authenticate, delegate to `serve('/mcp')`.
- Export the Durable Object class and configure Wrangler migrations/bindings.
- Apply the Durable Objects storage caching checklist before using `ctx.storage` on hot paths.

---

### edgefastmcp-cloudflare

Use when building a stateless MCP server on Cloudflare Workers.

```bash
/edgefastmcp-cloudflare
```

Core behavior:
- Keep tools request-local and avoid Durable Objects.
- Use EdgeFastMCP, `createMcpHandler`, or raw MCP transport depending on project conventions.
- Add authentication before tool execution.
- Escalate to `McpAgent` when durable session state appears.

---

### hono-cloudflare-worker

Use when wiring Cloudflare Worker entrypoints with Hono.

```bash
/hono-cloudflare-worker
```

Core behavior:
- Keep Wrangler config as the source of truth for bindings and migrations.
- Let Hono own HTTP routing and small status/help pages.
- Preserve Worker named exports such as Durable Object classes.
- Verify with route tests, `wrangler types`, typecheck, and build.

---

## Moved Skills

The KiwiFS MCP skill now lives in a dedicated public package:

```bash
npx skills add shoveller/kiwifs-agent-skills --skill kiwifs
```

Repository: https://github.com/shoveller/kiwifs-agent-skills

## Source note

These skills are distilled from Illuwa's TypeScript scaffold skill-set recipe, Cloudflare/MCP implementation notes, and public upstream documentation. They are adapted to the public `skills.sh` repository layout and avoid private wiki links in the package itself.

## License

MIT
