---
name: syncpack-monorepo-setup
description: "Use when setting up syncpack in a JavaScript/TypeScript monorepo to control dependency version drift. Adds root scripts, optional postinstall wiring, and verifies the setup without treating syncpack as an app runtime dependency. Trigger: /syncpack-monorepo-setup"
allowed-tools: Bash(pwd:*), Bash(test:*), Bash(ls:*), Bash(find:*), Bash(git:*), Bash(node:*), Bash(npm:*), Bash(pnpm:*), Bash(yarn:*), Bash(corepack:*), Bash(npx:*), Bash(pnpx:*), Bash(jq:*), Read, Edit, MultiEdit
---

# syncpack-monorepo-setup — Monorepo Version Drift Guard

Set up [syncpack](https://syncpack.dev/) as a **version drift control layer** for a JavaScript/TypeScript monorepo. This is not just a convenience command: it keeps shared dependency ranges from slowly diverging across apps and packages.

## When to Use

- A pnpm/npm/yarn workspace has multiple `package.json` files.
- TypeScript, ESLint, Vite, React, or other shared tools have slightly different versions across packages.
- The project wants installs, lockfiles, and cache keys to be more reproducible.
- The user asks to implement the Illuwa monorepo template step for syncpack.

## Workflow

1. **Confirm repository root.**
   - Check for `package.json`.
   - Check workspace markers: `pnpm-workspace.yaml`, npm `workspaces`, `yarn.lock`, or `packageManager`.
   - Do not run from a subpackage unless the user explicitly asks.

2. **Detect package manager.**
   - Prefer `packageManager` in root `package.json`.
   - Else prefer lockfiles in this order: `pnpm-lock.yaml`, `yarn.lock`, `package-lock.json`.
   - For pnpm, prefer `pnpm dlx syncpack ...` or `pnpx syncpack ...` depending on local convention.

3. **Add syncpack as a root dev tool when appropriate.**
   - Preferred reproducible setup:
     ```bash
     pnpm add -D -w syncpack
     ```
   - For npm:
     ```bash
     npm install -D syncpack
     ```
   - For yarn:
     ```bash
     yarn add -D syncpack -W
     ```
   - Keep syncpack at the root. Do not add it to application runtime dependencies.

4. **Add root scripts.**
   Use stable script names so CI and humans have a clear entry point:

   ```json
   {
     "scripts": {
       "deps:check": "syncpack list-mismatches",
       "deps:fix": "syncpack fix"
     }
   }
   ```

   If the user explicitly wants automatic convergence after every install, add:

   ```json
   {
     "scripts": {
       "postinstall": "syncpack fix"
     }
   }
   ```

   For projects that prefer no installed devDependency, a pnpm-only variant is acceptable:

   ```json
   {
     "scripts": {
       "deps:check": "pnpx syncpack list-mismatches",
       "deps:fix": "pnpx syncpack fix",
       "postinstall": "pnpx syncpack fix"
     }
   }
   ```

5. **Avoid destructive surprise edits.**
   - Run `deps:check` first.
   - Show the expected `package.json` changes before running `deps:fix` if there are many mismatches.
   - Do not auto-update major versions unless the user asked for latest-version upgrades. The normal goal is internal consistency.

6. **Verify.**
   Run the smallest convincing checks:
   ```bash
   pnpm deps:check
   pnpm deps:fix
   git diff -- package.json '**/package.json' pnpm-lock.yaml package-lock.json yarn.lock
   ```

   Then run the existing lightweight project gate if present:
   ```bash
   pnpm typecheck || pnpm test || pnpm build
   ```

## Recommended Policy

- Treat syncpack as the **version drift guard** layer in the monorepo stack.
- Keep root `package.json` for repository-wide tools only.
- Keep app runtime dependencies in each app/package's own `package.json`.
- If installs differ between machines, diagnose dependency placement and lockfile drift before blaming Turborepo/Nx.

## Common Pitfalls

- Adding syncpack to a subpackage instead of the root.
- Hiding a large rewrite behind `postinstall` before the team has reviewed the first sync.
- Confusing version synchronization with dependency upgrades. `syncpack fix` aligns ranges; it is not a substitute for a deliberate upgrade plan.
- Using `pnpx syncpack fix` in `postinstall` can add network dependency to every install. Prefer a root `devDependency` when reproducibility matters.
