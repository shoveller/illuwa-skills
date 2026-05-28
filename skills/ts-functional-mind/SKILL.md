---
name: ts-functional-mind
description: "Use when refactoring long TypeScript files into smaller, testable functional units while preserving behavior and avoiding premature abstraction. Trigger: /ts-functional-mind"
allowed-tools: Bash(pnpm:*), Bash(git:*), Read, Edit, MultiEdit
---

# TypeScript Functional Mind

Use this skill when a TypeScript file has grown long because side effects, data conversion, domain logic, and variants are mixed together.

## Core Rule

Make behavior smaller before making architecture bigger.

Separate by volatility and testability, not by aesthetic symmetry.

## Workflow

1. Identify the behavioral contracts first.
2. Mark side effects that must not change: API calls, persistence, routing, drag-and-drop ids, optimistic updates, timers, and browser APIs.
3. Extract pure functions before extracting effect coordinators.
4. Keep state ownership in the original module until ownership boundaries are proven.
5. Add tests around extracted pure functions.
6. Run typecheck before making the next extraction pass.

## Extraction Order

1. Pure data conversion: parsing, formatting, sorting, grouping, filtering, export text generation.
2. Presentation-only helpers: labels, colors, status text, empty-state messages, small view models.
3. Event handlers with stable contracts: only after the caller and callee boundaries are clear.
4. Effect coordinators: only when they own a coherent side-effect lifecycle.

## Strict Functional Baseline

Reference files:

- `references/profile-binding-branching.md` — typed registry maps and guard clauses for profile/config dispatch.
- `references/auth-session-functional-examples.md` — functional TypeScript patterns for auth/session tutorial snippets, including Better Auth/Hono-style runtime factories, schema generation boundaries, protected API helpers, and browser auth client helpers.

- Prefer `const` for local bindings; treat `let` as a refactor smell until a pure value-returning helper has been considered.
- Do not mutate function parameters or parameter-object properties; create and return a new value or copy.
- Prefer returning new values over accumulator/state mutation.
- Allow a little duplication when it keeps helpers pure and avoids hidden mutation.
- Prefer `const helper = (...) => {}` for new local helpers when project conventions allow it; keep existing public APIs, framework contracts, or hoisting-dependent declarations intact.
- Apply this baseline to documentation and tutorial snippets too. When writing sample TypeScript for the user, avoid sneaking policy into compact ternaries or expression-heavy examples; model the desired style with named pure helpers, guard clauses, or thin entrypoints plus shared common logic. For auth/session tutorials specifically, use `references/auth-session-functional-examples.md` so examples cover runtime factory, schema-generation boundary, server-side session verification, command creation, and client action helpers without turning the route/component into a logic blob.

## Collection Transformation Rules

- Prefer `map`, `filter`, `reduce`, `flatMap`, spread, and `slice` for collection transformations.
- Avoid mutating array methods such as `push`, `pop`, `shift`, `unshift`, `splice`, `reverse`, and `fill` on caller-owned data.
- Prefer `toSorted` and `toReversed` when available; otherwise copy first with `[...items].sort(...)` or `[...items].reverse()`.
- Allow imperative loops for performance hot paths, stream/iterator control, or intentional long-running loops, but leave a clear reason and tests.

## Conditional Logic Rules

- Replace conditional expressions with named pure state or message helpers plus guard-clause branching.
- Do not use ternary expressions, IIFEs, switch statements, `else`, or `else if` to hide branching inside expression-heavy paths.
- Prefer early returns, guard clauses, lookup maps, or named branch helpers.
- For config/profile dispatch, prefer a typed registry map checked with `satisfies Record<...>` plus a separate guard clause for runtime absence; see `references/profile-binding-branching.md`.
- Prefer block-bodied guard clauses when the formatter/linter allows it, so later side-effect additions are visible in review.
- Name booleans positively when possible: prefer `succeeded`, `isValid`, and `enableCheck` over `notFailed`, `isNotValid`, and `disableCheck`.
- Prefer pure helpers that return state, labels, messages, or data objects instead of framework output.
- Preserve side-effect ordering when extracting conditionals from handlers.
- Classify each branch as data selection, state transition, or view selection before extracting it.

## TypeScript Rules

- Export types only when another file must name the contract.
- Keep domain types in `lib` and caller-only contracts near the caller.
- Prefer discriminated unions for UI modes that already exist in state.
- Avoid compatibility shims unless persisted data or external callers require them.
- Let `noUnusedLocals` and `noUnusedParameters` shape the final boundary.

## JSDoc Rules

- Use JSDoc to explain why a function exists, what each named parameter means, and what the return value represents.
- In TypeScript files, do not duplicate TypeScript types in JSDoc tags with brace-wrapped type annotations.
- Prefer `@param name - Description.` and `@returns Description.` so TypeScript remains the source of truth for types.
- Use `@async` when it clarifies that the function coordinates asynchronous side effects.
- For functions that return nothing, prefer `@returns Nothing.` only when keeping a uniform documented-function format is useful.

## Test Rules

- Test extracted pure functions first.
- Do not add integration tests merely to justify a file split.
- For drag-and-drop, test payload guards and reorder calculations separately from UI wiring.
- For parsers and exporters, test small examples with stable output strings.
- Run the narrow test first, then full typecheck, then build.

## Red Flags

- A module coordinates multiple unrelated lifecycle concerns.
- A local state shape implicitly models a workflow state machine.
- A helper needs framework imports but claims to be domain logic.
- A rename changes API payload shape, sortable ids, path strings, or persisted frontmatter.
- A refactor mutates input parameters, caller-owned arrays, or parameter-object properties.
- A refactor creates a generic abstraction used only once.

## Verification

Run the project-specific checks after each meaningful pass:

```bash
pnpm typecheck
pnpm test
pnpm build
```
