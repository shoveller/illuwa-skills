---
name: ts-functional-mind
description: "Use when refactoring long TypeScript files into smaller, testable functional units while preserving behavior and avoiding premature abstraction. Trigger: /ts-functional-mind"
allowed-tools: Bash(pnpm:*), Bash(graphify:*), Bash(git:*), Read, Edit, MultiEdit
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

## Conditional Logic Rules

- Replace conditional expressions with named pure state or message helpers plus guard-clause branching.
- Do not use ternary expressions, IIFEs, switch statements, `else`, or `else if` to hide branching inside expression-heavy paths.
- Prefer early returns, guard clauses, lookup maps, or named branch helpers.
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
- A refactor creates a generic abstraction used only once.

## Verification

Run the project-specific checks after each meaningful pass:

```bash
pnpm typecheck
pnpm test
pnpm build
```

After code changes, update the graph when graphify is available:

```bash
graphify update .
```
