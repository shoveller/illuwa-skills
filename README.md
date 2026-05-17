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
```

## Available Skills

| Skill | Description |
|-------|-------------|
| [ts-functional-mind](#ts-functional-mind) | Refactor long TypeScript files into smaller, testable functional units while preserving behavior. |
| [ts-functional-mind-react](#ts-functional-mind-react) | Refactor long TypeScript React files while preserving rendering contracts, state ownership, and side-effect ordering. |
| [skill-moment-capture](#skill-moment-capture) | Capture reusable session judgments, checklists, workflows, and failure-prevention rules as skills. |

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

## Source note

These skills are distilled from Illuwa's TypeScript scaffold skill-set recipe and adapted to the public `skills.sh` repository layout.

## License

MIT
