---
name: posd-refactoring
description: Refactor React/React-Native code by identifying POSD anti-patterns and proposing incremental, strategic improvements. Use when the user asks to refactor, simplify, reduce complexity, clean up, or improve code structure.
---

# POSD Refactoring

Identify structural complexity issues in React/React-Native TypeScript code and propose incremental refactoring steps grounded in "A Philosophy of Software Design."

## Guardrails

- **Incremental**: Each step should be independently shippable. Never propose rewriting an entire module at once.
- **Prioritize by impact**: Fix the issues that cause the most complexity first (change amplification > cognitive load > unknown unknowns).
- **Preserve behavior**: Refactoring changes structure, not functionality. Be explicit about what stays the same.
- **Explain the principle**: For each change, name the POSD concept and which complexity symptom it addresses.

## Anti-Pattern Detection

Scan the code for these categories of problems:

### Shallow Modules
- Components that accept many props and just arrange children
- Hooks that wrap another hook with little added logic
- Pass-through components that forward props without transformation
- **Fix**: Merge shallow wrappers into their parent or deepen them by adding real functionality

### Leaked Abstractions
- Hooks returning library-specific types (React Query, Zustand internals, Animated values)
- Components exposing implementation via props (store dispatchers, cache keys, internal IDs)
- Return types that mirror the internal data structure rather than what the consumer needs
- **Fix**: Define a clean return type that hides the implementation. Transform data inside the module.

### Wrong State Boundaries
- `useEffect` syncing state between two sources
- Derived state stored in `useState` instead of computed
- State held at a parent that's only used by one child
- Context used to avoid 2 levels of prop drilling
- **Fix**: Move state closer to its consumer, derive instead of store, restructure ownership

### Mixed Concerns
- Business logic in component bodies (calculations, transformations, validation)
- Components that both fetch data AND render UI
- Hooks that handle unrelated responsibilities
- **Fix**: Extract pure logic functions, split into container/presentational, separate hooks by responsibility

### Re-Render Cascades
- State too high in the tree causing unnecessary subtree re-renders
- Inline objects/functions in JSX creating new references every render
- Context providers with un-memoized values re-rendering all consumers
- Hooks returning new object references on every call
- **Fix**: Push state down, extract stateful sections, memoize where structurally necessary, stabilize hook returns

### Tactical Debt
- Copy-pasted components with minor variations
- `any` / `as` type assertions hiding type problems
- TODO comments indicating known shortcuts
- Inconsistent patterns (two different approaches to the same problem)
- **Fix**: Extract shared abstractions, fix types properly, establish one pattern and migrate

## Workflow

1. **Identify**: List all anti-patterns found, categorized by type above
2. **Prioritize**: Rank by complexity impact -- which issues cause the most change amplification, cognitive load, or unknown unknowns?
3. **Plan**: For each priority item, propose incremental steps with before/after code
4. **Verify**: After each refactoring step, confirm it reduces the target complexity symptom

## Output Format

```
## Refactoring Plan: [scope -- file, component, or feature name]

### Priority 1: [Title]
- **Anti-pattern**: [What's wrong -- e.g., "Shallow wrapper around useQuery"]
- **Principle**: [POSD concept -- e.g., "Deep Modules"]
- **Complexity symptom**: [change amplification / cognitive load / unknown unknowns]
- **Before**:
  ```tsx
  [current code snippet]
  ```
- **After**:
  ```tsx
  [proposed code snippet]
  ```
- **Steps**:
  1. [First incremental step]
  2. [Second incremental step]
  3. [Third incremental step -- each is shippable on its own]

### Priority 2: [Title]
...

### Summary
- Total issues found: [count]
- Estimated effort: [small / medium / large]
- Recommended order: [which to tackle first and why]
```
