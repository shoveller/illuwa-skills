# EdgeFastMCP + external sandbox runner pattern

Session-derived pattern from refactoring a Cloudflare Worker MCP package from raw MCP SDK transport assembly to `fastmcp/edge`.

## Shape

- Keep the Worker route/module stateless: construct/register `EdgeFastMCP` tools without opening sandbox sessions or doing expensive per-request setup.
- Register each tool with compact `zod` parameters and an `execute` function.
- If a tool delegates to an external sandbox/container/runner, call the sandbox preparation function **inside the tool execution path**, not while the route is being created.
- When a tool accepts an explicit runner identity such as `sandboxId`, route execution through an explicit selector (`prepareSandboxById(sandboxId, env)` / `getSandboxById`) instead of the request-derived default (`prepareSandbox(request, env)`). Otherwise the ID may only label transcripts/logs while the command runs in a different sandbox.
- This preserves cheap MCP discovery/initialization and avoids spending sandbox setup cost on requests that never invoke a heavy tool.

## Example sketch

```ts
const mcp = new EdgeFastMCP({ name: 'runner-mcp', version: '0.1.0' })

mcp.addTool({
  name: 'run_command',
  description: 'Run a command in the configured sandbox.',
  parameters: z.object({
    command: z.string(),
    sandboxId: z.string().optional(),
    sessionId: z.string().optional(),
  }),
  execute: async ({ command, sandboxId, sessionId }) => {
    const { sandbox, sandboxId: resolvedSandboxId } = sandboxId
      ? await prepareSandboxById(sandboxId, env)
      : await prepareSandbox(request, env)

    const result = await sandbox.exec(command)

    if (sessionId) {
      await appendTranscript({ sandboxId: resolvedSandboxId, sessionId }, result)
    }

    return result.stdout
  },
})

export default mcp
```

## Verification focus

- Unit-test that expected tools are registered on the `EdgeFastMCP` instance.
- Mock the sandbox and assert `prepareSandbox()` is called by heavy tool execution, not during module/route construction.
- For tools with optional `sandboxId`, add a regression test that passes `sandboxId` and asserts the explicit selector (`prepareSandboxById`) is called while the request-derived default is not. Also assert transcript/log keys use the resolved normalized sandbox ID returned by the selector.
- Run package-scoped tests/typecheck/build before pushing. In a monorepo, if workspace hooks fail elsewhere, follow `github-repo-management`'s scoped-verification-before-bypassing-hooks rule.
