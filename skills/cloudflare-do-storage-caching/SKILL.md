---
name: cloudflare-do-storage-caching
description: "Use when reviewing Cloudflare Durable Objects storage costs, reducing ctx.storage reads/writes/lists/alarms, and designing L1/L2 caches for Workers or MCP servers. Trigger: /cloudflare-do-storage-caching"
allowed-tools: Bash(pnpm:*), Bash(npm:*), Bash(npx:*), Bash(git:*), Read, Write, Edit, MultiEdit
---

# Cloudflare DO Storage Caching

Use this skill when a Cloudflare Worker, Agent, or MCP server uses Durable Objects and you need to control storage cost, latency, and cleanup behavior.

## Core Rule

Make hot requests finish from Durable Object instance memory. Treat `ctx.storage` as a durable recovery layer, not a per-request cache lookup.

## When to Use

- A Durable Object reads or writes `ctx.storage` during normal request handling.
- A Cloudflare `McpAgent` stores session state, OAuth state, counters, search caches, or transport state.
- A Worker uses alarms to clean up cached data.
- You need to estimate whether `get`, `put`, `delete`, `list`, SQL, or alarm operations can become expensive.

Do not use this when the app is stateless and does not use Durable Objects. Prefer a stateless Worker pattern instead.

## Current Cost Vocabulary

Do not describe new Durable Objects only as “Class A/B ops”. Current Cloudflare docs describe Durable Objects storage by backend:

| Backend | Cost vocabulary |
| --- | --- |
| SQLite-backed DO storage | rows read, rows written, stored data |
| Legacy KV-backed DO storage | read request units, write request units, delete requests, stored data |

Important billing facts to re-check against current pricing before committing an estimate:

- KV-style methods such as `get()`, `put()`, `delete()`, and `list()` on SQLite-backed storage use a hidden SQLite table and can count as rows read/written.
- `setAlarm()` is a write-like storage operation.
- Deletes count as writes for SQLite-backed storage.
- Legacy KV-backed `list()` is charged by data examined/returned and can have a minimum read unit even for empty results.
- Stored data can keep billing until storage is fully removed. Use `deleteAll()` for full cleanup; for compatibility dates before 2026-02-24, delete alarms separately or enable the relevant compatibility behavior.

## Anti-Patterns

### Per-request storage read

```ts
const config = await this.ctx.storage.get('config')
```

If every request does this, storage cost scales with request volume. Prefer an instance field that loads once and refreshes on TTL.

### Request-path list

```ts
const cacheEntries = await this.ctx.storage.list({ prefix: 'cache:' })
```

Move listing to an admin endpoint or a low-frequency alarm. Never scan a prefix on every user request.

### Tiny write on every call

```ts
await this.ctx.storage.put(`counter:${Date.now()}`, value)
```

Batch counters or write only when state crosses a durable boundary.

### Alarm reset on every request

```ts
await this.ctx.storage.setAlarm(Date.now() + 60_000)
```

Check whether an alarm already exists before setting one.

## Cache Layers

| Layer | Purpose | Cost profile | Rule |
| --- | --- | --- | --- |
| L1 memory | Hot request path | no storage operation | default for reads |
| L2 DO storage | Cold-start recovery and durable cache | storage rows/units | touch only on miss or flush |
| L3 origin | API, R2, D1, AutoRAG, database | external cost/latency | call only after L1/L2 miss |

## L1/L2 Memoization Pattern

```ts
type CacheEntry<T> = { data: T; expiry: number }

type MemoizeOptions<Args extends unknown[]> = {
  prefix: string
  ttlMs: number
  key: (...args: Args) => string
}

export const createDurableMemo = <Args extends unknown[], T>(
  storage: DurableObjectStorage,
  load: (...args: Args) => Promise<T>,
  options: MemoizeOptions<Args>,
) => {
  const l1 = new Map<string, CacheEntry<T>>()

  return async (...args: Args): Promise<T> => {
    const storageKey = `${options.prefix}${options.key(...args)}`
    const now = Date.now()
    const memoryHit = l1.get(storageKey)

    if (memoryHit && memoryHit.expiry > now) {
      return memoryHit.data
    }

    const durableHit = await storage.get<CacheEntry<T>>(storageKey)

    if (durableHit && durableHit.expiry > now) {
      l1.set(storageKey, durableHit)
      return durableHit.data
    }

    const data = await load(...args)
    const entry = { data, expiry: now + options.ttlMs }
    l1.set(storageKey, entry)
    await storage.put(storageKey, entry)
    return data
  }
}
```

Use this only when durable recovery is needed. If losing cache data is acceptable, use L1 memory alone or Cloudflare Cache API instead.

## Alarm Cleanup Pattern

Cleanup costs money too. Keep it low-frequency and bounded.

```ts
async alarm() {
  const now = Date.now()
  const entries = await this.ctx.storage.list<CacheEntry<unknown>>({ prefix: 'cache:' })
  const expiredKeys = Array.from(entries.entries())
    .filter(([, value]) => value.expiry < now)
    .slice(0, 100)
    .map(([key]) => key)

  if (expiredKeys.length > 0) {
    await this.ctx.storage.delete(expiredKeys)
  }

  await this.ensureCleanupAlarm()
}

private async ensureCleanupAlarm() {
  const currentAlarm = await this.ctx.storage.getAlarm()

  if (currentAlarm !== null) {
    return
  }

  await this.ctx.storage.setAlarm(Date.now() + 24 * 60 * 60 * 1000)
}
```

## MCP-Specific Rules

- Keep tool definitions static; do not rebuild tool metadata from storage every request.
- Cache expensive search results by `(user scope, query, limit, version)` with short TTL.
- Cache negative results briefly to stop repeated misses.
- Avoid storing entire large documents in DO storage; use R2/D1/KV/origin stores and keep only small indexes or session state in the DO.
- If a tool is stateless, do not put it behind `McpAgent` only for convenience. Consider a stateless Worker handler.

## Review Process

1. List each request path and count storage calls: `get`, `put`, `delete`, `list`, SQL, `getAlarm`, `setAlarm`.
2. Classify each call as hot-path, cold-start, flush, cleanup, or admin.
3. Remove hot-path `list()` entirely.
4. Add L1 memory for hot reads.
5. Batch or debounce writes that do not need exact real-time persistence.
6. Bound cleanup: prefix, max keys per run, and minimum alarm interval.
7. Re-check Cloudflare pricing for the current backend and date.

## Verification

```bash
pnpm typecheck
pnpm test
pnpm build
```

Add or run tests that prove repeated calls hit the loader once per TTL window when practical. If no tests exist, instrument local logs or counters around storage methods in a narrow dev run.

## Common Pitfalls

- Assuming Cloudflare's internal in-memory storage cache eliminates billing. Reduce storage API calls at the application layer.
- Using multi-key operations to assume one billable operation; backend pricing may still account per key, row, or data size.
- Cleaning expired cache keys more often than the cache is used.
- Forgetting that `setAlarm()` is write-like.
- Deleting user keys but leaving alarm/storage metadata that keeps the object allocated; use full cleanup semantics when retiring objects.

## Checklist

- [ ] Hot request paths do not call `list()`.
- [ ] Hot reads use L1 memory before `ctx.storage`.
- [ ] L2 storage is only for cold recovery or durable state.
- [ ] Writes are batched, debounced, or tied to durable boundaries.
- [ ] Alarm setup avoids resetting on every request.
- [ ] Cleanup has a bounded prefix, batch size, and interval.
- [ ] Pricing terminology matches the actual DO backend.
