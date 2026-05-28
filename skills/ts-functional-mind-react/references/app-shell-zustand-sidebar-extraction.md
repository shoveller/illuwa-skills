# App shell + Zustand sidebar extraction pattern

Use this when a large React app shell mixes layout JSX, sidebar sections, feature state, and API refresh effects in one file.

## When it fits

- `App.tsx` or another route shell is large mainly because it renders navigation/sidebar sections.
- One sidebar feature has coherent state: list data, refresh action, and one persisted preference.
- The project already has Zustand or accepts a small feature-local store.
- The app shell still owns routing, selected path, modal opening, and cross-panel callbacks.

## Extraction shape

1. Create a feature store for the coherent data lifecycle.
   - Example state: `pages`, `showList`, `refresh()`, `toggleShowList()`.
   - Keep `localStorage` preference read/write inside the store when it belongs only to that feature.
   - Subscribe from components with narrow selectors.
2. Extract the sidebar layout into a component such as `AppSidebar`.
   - The sidebar reads feature store data directly.
   - The app shell passes boundary callbacks: navigation, create dialog open, active path, tree refresh, sort setter.
3. Keep reusable row/section/icon-button leaves controlled by props.
   - Do not couple all leaf components to Zustand just to avoid props.
4. Replace old parent state and derived sets with store refresh + local `useMemo` inside the extracted component.
5. Preserve persisted keys, DOM semantics, active-row styles, tree refs, aria labels, and event ordering.
6. Verify after the split with `git diff --check`, typecheck, and production build.

## Review cues

- Good: `App.tsx` shrinks and still owns app-wide routing and panel orchestration.
- Good: the extracted sidebar owns only sidebar rendering and feature-local store reads.
- Risk: moving API calls and JSX in the same pass without checks. Run typecheck before further extraction.
- Risk: singleton Zustand stores retaining stale state after workspace/account switches. Include explicit refresh dependencies or reset actions when switching contexts.
