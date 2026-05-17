# illuwa-skills

Skills shared by Illuwa for Claude Code and other AI agents. This repository follows the simple `skills.sh` / `npx skills` layout: each skill lives under `skills/<skill-name>/SKILL.md`.

## Installation

```bash
npx skills add shoveller/illuwa-skills
```

## Available Skills

| Skill | Description |
|-------|-------------|
| [syncpack-monorepo-setup](#syncpack-monorepo-setup) | Add syncpack as a monorepo version-drift guard, including root scripts and postinstall wiring. |
| [syncpack-version-drift](#syncpack-version-drift) | Diagnose and fix dependency version drift across pnpm/npm/yarn monorepos with syncpack. |

---

### syncpack-monorepo-setup

Use this when a TypeScript/JavaScript monorepo needs a lightweight dependency version synchronization system. The skill adds `syncpack` at the repository root, wires safe scripts, and optionally adds `postinstall` so installs converge dependency versions.

```bash
/syncpack-monorepo-setup
```

Core behavior:

- Detect package manager and monorepo workspace files.
- Keep syncpack as a root dev tool, not an app runtime dependency.
- Add `deps:check`, `deps:fix`, and optionally `postinstall` scripts.
- Verify with `syncpack list-mismatches` and `syncpack fix --dry-run` when supported.

---

### syncpack-version-drift

Use this when packages in a monorepo have diverging dependency ranges or when CI/local installs behave differently across apps and packages.

```bash
/syncpack-version-drift
```

Core behavior:

- Inspect dependency mismatches without rewriting first.
- Use `syncpack list-mismatches` to locate drift.
- Use `syncpack fix` only after reviewing the intended changes.
- Reinstall and run the smallest available package-manager/typecheck validation.

## Source note

These skills are distilled from Illuwa's monorepo template notes about treating syncpack as a **version drift control** layer, not just a convenience script.

## License

MIT
