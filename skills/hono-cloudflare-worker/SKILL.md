---
name: hono-cloudflare-worker
description: "Use when wiring a Cloudflare Worker entrypoint with Hono routes, bindings, Wrangler config, Vite dev/build, small JSX UI/status pages, and route-level tests. Trigger: /hono-cloudflare-worker"
allowed-tools: Bash(pnpm:*), Bash(npm:*), Bash(npx:*), Bash(wrangler:*), Bash(git:*), Read, Write, Edit, MultiEdit
---

# Hono Cloudflare Worker

Use this skill when a Cloudflare Worker owns runtime routes and Hono should provide the HTTP entrypoint. This is for API Workers, MCP gateways, WebSocket gateways, Durable Object front doors, and small server-rendered status/help pages.

## Core Rule

Let Wrangler config own Cloudflare resources, let Hono own HTTP routing, and keep runtime adapters testable outside the Worker entrypoint.

## When to Use

- A Worker package needs API routes, health checks, or small HTML pages.
- You want Hono route tests with `app.request()`.
- A Worker must forward specific routes to MCP, WebSocket, Durable Object, Queue, Workflow, or service-binding handlers.
- You want Vite dev/build with `@cloudflare/vite-plugin` while keeping `wrangler.jsonc` as the resource source of truth.

Do not use this when a full-stack framework already owns the Worker entrypoint, such as React Router framework mode, OpenNext, or another Worker framework adapter.

## Dependencies

```bash
pnpm add hono vite-ssr-components
pnpm add -D @cloudflare/vite-plugin vite wrangler vitest typescript
```

For an API-only Worker, omit `vite-ssr-components` and TSX renderer files.

## Package Scripts

```json
{
  "scripts": {
    "dev": "vite dev",
    "start": "vite dev",
    "build": "vite build",
    "deploy": "vite build && wrangler deploy",
    "cf-typegen": "wrangler types",
    "typecheck": "pnpm cf-typegen && tsc --noEmit",
    "test": "vitest run"
  }
}
```

Preserve project-specific lint and format scripts if they already exist.

## Wrangler Config

Keep Cloudflare bindings, migrations, routes, compatibility date, and class exports visible to Wrangler.

```jsonc
{
  "$schema": "node_modules/wrangler/config-schema.json",
  "name": "worker-app",
  "main": "src/index.tsx",
  "compatibility_date": "2026-02-24",
  "compatibility_flags": ["nodejs_compat"]
}
```

If the Worker does not render JSX, use `src/index.ts`.

## Vite Config

```ts
import { cloudflare } from '@cloudflare/vite-plugin'
import { defineConfig } from 'vite'
import ssrPlugin from 'vite-ssr-components/plugin'

export default defineConfig({
  plugins: [cloudflare(), ssrPlugin()],
})
```

For API-only Workers:

```ts
import { cloudflare } from '@cloudflare/vite-plugin'
import { defineConfig } from 'vite'

export default defineConfig({ plugins: [cloudflare()] })
```

## TSX Settings

Only add Hono JSX settings when rendering server-side HTML from Hono.

```json
{
  "compilerOptions": {
    "jsx": "react-jsx",
    "jsxImportSource": "hono/jsx"
  },
  "include": ["src/**/*.ts", "src/**/*.tsx", "src/**/*.d.ts", "vite.config.ts"]
}
```

Preserve generated Cloudflare type includes and existing path/module settings.

## Minimal Hono Entrypoint

```tsx
import { Hono } from 'hono'

export type WorkerEnv = {
  MCP_API_KEY?: string
}

const app = new Hono<{ Bindings: WorkerEnv }>()

app.get('/health', (c) => c.json({ ok: true }))

app.all('/mcp/*', async (c) => {
  return handleMcpRequest(c.req.raw, c.env, c.executionCtx)
})

app.notFound(() => new Response('Not Found', { status: 404 }))

export default app
```

Keep named exports for Durable Object classes or other Worker module exports:

```ts
export { SessionObject } from './SessionObject'
```

## Small HTML Renderer

```tsx
import { jsxRenderer } from 'hono/jsx-renderer'
import { Link, ViteClient } from 'vite-ssr-components/hono'

export const renderer = jsxRenderer(({ children }) => (
  <html lang="en">
    <head>
      <meta charset="utf-8" />
      <meta name="viewport" content="width=device-width, initial-scale=1" />
      <title>Worker App</title>
      <ViteClient />
      <Link href="/src/style.css" rel="stylesheet" />
    </head>
    <body>{children}</body>
  </html>
))
```

Use the renderer only for routes that return HTML:

```tsx
app.use('/ui', renderer)
app.use('/ui/*', renderer)
app.get('/ui', (c) => c.render(<main><h1>Worker App</h1></main>))
```

## Route Adapter Rule

Wrap runtime-specific handlers instead of mixing them directly into Hono routes.

```ts
app.all('/api/search', (c) => handleSearch(c.req.raw, c.env))
app.all('/ws/terminal/*', (c) => handleTerminalWebSocket(c.req.raw, c.env))
app.all('/mcp/*', (c) => handleMcpRequest(c.req.raw, c.env, c.executionCtx))
```

This keeps route tests small and makes Worker bindings explicit.

## Testing

Use `app.request()` for routes without Cloudflare-only imports:

```ts
import { describe, expect, test } from 'vitest'
import app from '../src/index'

describe('worker routes', () => {
  test('serves health check', async () => {
    const response = await app.request('http://example.test/health')

    expect(response.status).toBe(200)
    await expect(response.json()).resolves.toMatchObject({ ok: true })
  })
})
```

If imports require Cloudflare bindings at module load time, split pure route adapters from binding adapters or mock the Worker-only modules.

## Verification

```bash
pnpm test
pnpm typecheck
pnpm build
```

For deployment confidence:

```bash
pnpm wrangler types
pnpm wrangler deploy --dry-run
```

In monorepos, run the same commands through the relevant package filter and then run root quality gates if they exist.

## Failure Branches

- `wrangler types` cannot find a Durable Object class: check `main` and named exports.
- TSX fails typecheck: check `jsx`, `jsxImportSource`, and `src/**/*.tsx` includes.
- Vite cannot find Wrangler config: keep `wrangler.jsonc`, `wrangler.json`, or `wrangler.toml` in the package root.
- Tests fail on Worker-only imports: split adapters or mock runtime bindings.
- CSS is missing in server-rendered pages: ensure `ViteClient` and `Link` are in the renderer and the CSS path is Vite-managed.

## Common Pitfalls

- Treating Hono JSX as a React SPA. It is server-rendered HTML in the Worker.
- Moving Cloudflare resource definitions into Vite-only config where Wrangler commands cannot see them.
- Importing heavy runtime clients at module load time and breaking tests.
- Forgetting to export Durable Object classes after converting a `fetch` object to Hono.
- Adding this layer to a package that is only a library and has no Worker entrypoint.

## Checklist

- [ ] Hono owns HTTP routes, not resource definitions.
- [ ] Wrangler config remains the source of truth for Cloudflare bindings/migrations.
- [ ] Worker named exports are preserved.
- [ ] Route handlers are adapter functions where practical.
- [ ] `app.request()` tests cover health and key routes.
- [ ] `wrangler types`, typecheck, tests, and build pass.
