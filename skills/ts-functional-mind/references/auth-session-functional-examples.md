# Auth/session tutorial examples: functional TypeScript baseline

Use this reference when writing or refactoring TypeScript documentation snippets for auth/session layers (Better Auth, Hono handlers, D1/Drizzle adapters, browser auth clients). The goal is not to create a large architecture, but to keep examples copyable, testable, and aligned with the user's preference for small named helpers.

## Pattern

1. Keep the framework entrypoint thin.
   - Hono route handlers should mostly call named helpers and return their result.
   - Runtime factories should receive explicit `env`/bindings/options instead of reading ambient globals.
2. Separate configuration assembly from effect execution.
   - `createAuthDb(DB)`
   - `createDrizzleAuthAdapter(DB)`
   - `createSocialProviders(options)`
   - `createSecondaryStorage(KV)`
   - `createAuth(options)`
3. Put optional configuration behind guard-clause helpers.
   - Prefer `createGoogleProviderConfig(env)` returning config or `undefined` over inline ternary config blobs.
   - Prefer `createKvPutOptions(expirationTtl)` over embedding optional object properties at the call site.
4. Split protected API examples into three layers.
   - Parse request input: `parseCreateItemInput(request)`
   - Derive command from trusted session: `createSessionOwnedItemCommand(sessionUserId, input)`
   - Coordinate side effects in the handler: database insert, response, logging.
5. Client examples should not hide policy in JSX.
   - Use `signInWithGoogle()`, `redirectToHome()`, `getUserMenuLabel(session)`, `hasClientSession(session)`, `requireClientSession(session)` helpers.
   - JSX/event handlers should read as orchestration, not business logic.

## Pitfalls

- Do not trust `userId`, `ownerId`, or `accountId` from client payloads in examples. Derive ownership/scope from the verified server session.
- Do not let CLI/schema-generation auth entrypoints depend on Cloudflare runtime-only request context or secrets at module load time. Keep a separate auth export for schema generation when needed.
- Do not compress auth docs into a single login button. For a reusable guide, close the loop: auth factory → schema generation → migration → runtime route → protected API → browser smoke test.
- Do not treat client route guards as authorization. They are UX only; server handlers must call the auth/session API again.

## Verification ladder for docs that include auth/session snippets

- Schema generation command is present and isolated from runtime-only dependencies.
- Migration generation/application path is explicit.
- Hono `/api/auth/*` route wiring is shown as a thin adapter.
- Protected API example verifies session server-side.
- Browser client example shows sign-in, sign-out/session display, and redirects without embedding complex conditional logic in JSX.
