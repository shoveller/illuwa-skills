---
name: syncpack-version-drift
description: "Use when diagnosing or fixing dependency version drift in JavaScript/TypeScript monorepos. Inspects syncpack mismatches, applies minimal fixes, and verifies install/typecheck/build health. Trigger: /syncpack-version-drift"
allowed-tools: Bash(pwd:*), Bash(test:*), Bash(ls:*), Bash(find:*), Bash(git:*), Bash(node:*), Bash(npm:*), Bash(pnpm:*), Bash(yarn:*), Bash(corepack:*), Bash(npx:*), Bash(pnpx:*), Bash(jq:*), Read, Edit, MultiEdit
---

# syncpack-version-drift — Diagnose and Fix Monorepo Dependency Drift

Use syncpack to find and repair dependency version drift across a JavaScript/TypeScript monorepo. The aim is reproducibility: fewer "works in this package but not that one" failures caused by subtly different dependency ranges.

## When to Use

- Multiple packages depend on the same library with different version ranges.
- CI, local installs, or cache behavior differ across apps/packages.
- TypeScript/ESLint/Vite/React versions are not aligned across the workspace.
- A monorepo already has syncpack scripts and the user wants drift fixed.

## Workflow

1. **Start with a clean snapshot.**
   ```bash
   git status --short
   ```
   If unrelated user changes exist, avoid broad rewrites. Report them and limit edits to dependency metadata.

2. **Find the package manager and existing scripts.**
   ```bash
   node -e "const p=require('./package.json'); console.log(p.packageManager||'no packageManager'); console.log(p.scripts||{})"
   ```

3. **Run a read-only mismatch check first.**
   Prefer existing scripts:
   ```bash
   pnpm deps:check
   ```

   Otherwise use the package manager directly:
   ```bash
   pnpm exec syncpack list-mismatches
   # or: pnpm dlx syncpack list-mismatches
   # or: npx syncpack list-mismatches
   ```

4. **Classify the drift before fixing.**
   - **Shared toolchain drift:** TypeScript, ESLint, Prettier, Vite, Vitest, React types.
   - **Runtime dependency drift:** React, Hono, Zod, database clients, SDKs.
   - **Package-manager policy drift:** workspace protocol, peer ranges, root-only tools.

   Do not blindly align peer dependencies or intentionally divergent runtime versions without checking package ownership.

5. **Apply the smallest fix.**
   Prefer existing script:
   ```bash
   pnpm deps:fix
   ```

   Or direct command:
   ```bash
   pnpm exec syncpack fix
   ```

   If the user asked only for a report, stop after `list-mismatches` and summarize.

6. **Reinstall if lockfile or package metadata changed.**
   ```bash
   pnpm install
   ```

7. **Verify project health.**
   Use available scripts in this order, choosing the smallest convincing check:
   ```bash
   pnpm deps:check
   pnpm typecheck
   pnpm test
   pnpm build
   ```

8. **Summarize exact changes.**
   Include:
   - files changed
   - dependency names/ranges aligned
   - commands run
   - verification result
   - any intentionally unresolved drift

## Decision Rules

- If a dependency appears in many packages with tiny range differences, align it.
- If one app intentionally pins an older runtime because of framework compatibility, do not force it without user approval.
- If root `package.json` owns app runtime libraries, recommend moving runtime dependencies down to app/package manifests before treating the issue as a syncpack problem.
- If `syncpack fix` creates unexpectedly large changes, revert and propose a smaller allowlist/config approach.

## Common Pitfalls

- Treating every mismatch as wrong. Some peer ranges or compatibility pins are intentional.
- Running `syncpack fix` before collecting a readable mismatch report.
- Forgetting that version drift control is separate from Turborepo/Nx orchestration.
- Assuming references or built packages will fix package-manager drift. They will not; dependency ownership must be correct first.
