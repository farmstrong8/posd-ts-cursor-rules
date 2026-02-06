---
name: posd-design-review
description: Review React/React-Native code for design quality using principles from "A Philosophy of Software Design." Use when the user asks to review code, evaluate a PR, assess component quality, or identify complexity problems.
---

# POSD Design Review

Systematically review React/React-Native TypeScript code for structural complexity issues using principles from "A Philosophy of Software Design."

## Guardrails

- Focus on **structural** issues that affect complexity. Skip cosmetic nitpicks.
- Don't flag thin utility wrappers or adapter components that serve a deliberate purpose.
- Prioritize findings by impact. A component with the wrong abstraction boundary matters more than a missing `useCallback`.
- Be specific: cite file, line, and provide concrete code suggestions.

## Review Checklist

Work through each category. Skip any that don't apply.

### 1. Module Depth

- Are component props interfaces simple relative to what the component does internally?
- Are hook return types simple relative to the complexity they encapsulate?
- Are there shallow wrappers that just forward props/calls with no transformation?

### 2. Information Hiding

- Do hooks leak their implementation (e.g., returning library-specific types like `UseQueryResult`)?
- Do components expose internal concerns through props (animation drivers, store dispatchers, query keys)?
- Could a consumer use this module without knowing how it works internally?

### 3. Layer Value

- Does each component in the tree add meaningful abstraction?
- Are there pass-through components that take the same props as their children?
- Are there hooks that wrap another hook and add nothing?

### 4. State Architecture

- Is state placed as close to its consumer as possible?
- Is any state being synced between two sources via `useEffect`? (Wrong boundary)
- Is derived state being stored instead of computed?
- Is context being used appropriately (app-wide concerns only, not avoiding 2 levels of props)?
- Are context providers memoizing their values?

### 5. Re-Render Awareness

- Are inline objects/functions in JSX creating new references every render?
- Are hooks returning new object references on every call?
- Is state held too high, causing unnecessary subtree re-renders?
- Is `useMemo`/`useCallback` being used as a bandage for structural problems?

### 6. Pattern Consistency

- Does this code follow the codebase's established conventions?
- Are naming patterns consistent (handlers, booleans, hooks, components)?
- Does it introduce a new pattern where one already exists?

### 7. Red Flag Scan

- Components over ~150 lines?
- More than ~5 props?
- Prop drilling through 3+ levels?
- Multiple `useState` for related state?
- `useEffect` with many dependencies?
- Business logic in the component body?
- `any` or `as` type assertions?

## Output Format

Structure your review as follows:

```
## POSD Review: [file or feature name]

### Findings

#### [STRUCTURAL] [One-line title]
- **Principle**: [Which POSD concept -- e.g., "Information Hiding," "Deep Modules"]
- **Location**: [file:line or component name]
- **Issue**: [What's wrong and why it adds complexity]
- **Suggestion**: [Concrete code change or refactoring direction]
- **Impact**: [Which complexity symptom this addresses: change amplification / cognitive load / unknown unknowns]

#### [MODERATE] [One-line title]
...

#### [MINOR] [One-line title]
...

### Summary
- Structural issues: [count]
- Moderate issues: [count]
- Minor issues: [count]
- **Top priority**: [The single most impactful thing to fix first and why]
```

Severity levels:
- **STRUCTURAL**: Wrong abstraction boundaries, leaked information, state architecture problems. Fix these.
- **MODERATE**: Re-render issues, missing memoization where it matters, consistency violations. Should fix.
- **MINOR**: Naming improvements, comment suggestions, minor opportunities. Nice to have.
