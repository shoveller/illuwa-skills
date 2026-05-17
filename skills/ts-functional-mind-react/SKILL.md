---
name: ts-functional-mind-react
description: "Use when refactoring long TypeScript React files into smaller, testable functional units while preserving behavior, rendering contracts, and state ownership. Trigger: /ts-functional-mind-react"
allowed-tools: Bash(pnpm:*), Bash(git:*), Read, Edit, MultiEdit
---

# TypeScript Functional Mind React

Use this skill when a TypeScript React file has grown long because rendering, state ownership, side effects, data conversion, and UI variants are mixed together.

## Core Rule

Make behavior smaller before making architecture bigger.

Separate React concerns by volatility and testability, not by aesthetic symmetry.

## Workflow

1. Identify the behavioral contracts first.
2. Mark side effects that must not change: API calls, persistence, routing, drag-and-drop ids, optimistic updates, timers, and browser APIs.
3. Extract pure functions before extracting hooks.
4. Extract stateless leaf components before introducing context.
5. Keep state ownership in the original container until prop flow becomes a proven problem.
6. Add tests around extracted pure functions.
7. Run typecheck before making the next extraction pass.

## Extraction Order

1. Pure data conversion: parsing, formatting, sorting, grouping, filtering, export text generation.
2. UI-only helpers: labels, colors, badges, empty-state messages, small view models.
3. Stateless components: dialogs, panels, lists, rows, toolbar sections, skeletons.
4. Event handlers with stable contracts: only after the caller and callee boundaries are clear.
5. Hooks: only when they own a coherent side-effect lifecycle.
6. Context: only when prop threading crosses unrelated subtrees.

## React Rules

- Allow at most one direct `useState` and at most one direct `useEffect` in a React component body.
- Treat two or more direct `useState` calls as a strong signal that local props/state has grown into a workflow; check existing project store patterns, then consider Zustand, Recoil, Jotai, Redux, or a feature-local store.
- Treat two or more direct `useEffect` calls as a strong signal that the component owns too many roles and is leaning on side effects; split the role into named custom hooks or smaller components.
- Do not hide multiple unrelated effects in one generic hook; each hook name must describe the lifecycle it owns.
- Keep one state owner for one workflow until a split has clear ownership boundaries.
- Prefer explicit props over new context during the first refactor pass.
- Do not move API calls while also moving JSX unless the behavior is already tested.
- Do not introduce `useMemo` or `useCallback` merely because code moved.
- Preserve existing ids, keys, aria labels, and click/drag event ordering.
- Preserve Storybook-visible visual states when splitting components.

## Props Drilling and State Store Rules

- Before adding a state library, inspect `package.json` and existing imports; if the project already uses Zustand, Recoil, Jotai, Redux, or another store, follow that established pattern.
- If no state library is established and one feature has clear props drilling across board/dialog/leaf layers, prefer Zustand over Recoil for a small feature-local state machine.
- Prefer a feature-scoped Provider store over a module-level singleton when screen re-entry could otherwise preserve stale dialogs, errors, loading flags, or draft form data.
- When a project already uses a state library such as Zustand or Recoil for the feature, let feature-bound descendants subscribe to the store directly instead of passing store state and actions through props.
- Keep simple presentational components controlled by explicit props; do not couple reusable UI leaves to a feature store just to avoid props.
- Remove parent-to-child prop threading when the parent only reads store values to forward them through unrelated intermediate components.
- Keep application boundary callbacks such as routing, navigation, close handlers, and analytics outside the store; store actions can return success data for the screen to handle.
- Keep reusable leaf components controlled with props when that preserves Storybook, tests, and reuse; not every descendant needs to subscribe to the store.
- Subscribe with narrow selectors for the state/action each component needs instead of passing or selecting the full store object.
- In async store actions, read current values through the store getter near the side effect to avoid stale closure bugs.

## Conditional Logic Rules

- Replace JSX ternaries with named pure view-state helpers, extracted leaf components, or explicit conditional rendering.
- Do not use ternary expressions, IIFEs, switch statements, `else`, or `else if` in render paths, handlers, or helpers.
- Prefer `return null`, early returns, guard clauses, lookup maps, and small leaf components.
- Prefer pure helpers that return view state, labels, messages, or data objects instead of JSX.
- Preserve side-effect ordering when extracting conditionals from event handlers.
- Classify each branch as data selection, state transition, or view selection before extracting it.

## TypeScript Rules

- Export types only when another file must name the contract.
- Keep domain types in `lib` and component-only props near the component.
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
- Do not add component tests merely to justify a file split.
- For drag-and-drop, test payload guards and reorder calculations separately from UI wiring.
- For parsers and exporters, test small examples with stable output strings.
- Run the narrow test first, then full typecheck, then build.

## Red Flags

- A component owns multiple direct `useEffect` blocks for different lifecycle concerns.
- A component owns multiple direct `useState` calls that implicitly model a workflow state machine.
- A new hook returns more than one unrelated lifecycle.
- A helper needs React imports but claims to be domain logic.
- A new component receives the whole parent state object because its boundary is unclear.
- A rename changes API payload shape, sortable ids, path strings, or persisted frontmatter.
- A refactor creates a generic abstraction used only once.

## Complex Interaction Pattern

For long React files with drag-and-drop, sortable lists, dialogs, or mutation-heavy flows:

- Keep provider boundaries and drag payload contracts stable.
- Keep sortable item identity fields unchanged.
- Keep API mutation calls in the container on the first pass.
- Move parsing, formatting, export text generation, and list transformation logic to pure helpers.
- Move dialog and panel JSX to feature-local components while keeping state in the existing owner until ownership is proven.
- Extract reorder math, row updates, payload conversion, and empty-state labels before moving drag handlers or API calls.
- Add test coverage for pure helpers, and preserve existing visual-regression or Storybook workflows when present.

## Verification

Run the project-specific checks after each meaningful pass:

```bash
pnpm typecheck
pnpm test
pnpm build
```
