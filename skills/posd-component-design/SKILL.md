---
name: posd-component-design
description: Guide the design of new React/React-Native components, hooks, and modules using POSD principles. Use when the user asks to design, architect, or plan a new component, hook, feature, module, or API.
---

# POSD Component Design

Guide the design of new React/React-Native TypeScript components, hooks, and modules using principles from "A Philosophy of Software Design." Aim for depth, information hiding, and simplicity.

## Design Process

Follow these steps for any new component, hook, or module.

### Step 1: Define What It Hides

Before writing any code, answer: **What complexity does the consumer NOT need to know about?**

This is the most important question. A well-designed module hides:
- Data fetching strategy (which library, caching, retries)
- State management internals (how state is structured, when it updates)
- Formatting and validation logic
- Platform differences (iOS vs Android in React Native)
- Animation implementation
- Error recovery and edge case handling

### Step 2: Design the Interface First

Design the props (for components) or return type (for hooks) BEFORE the implementation. Aim for depth: the interface should be much simpler than the implementation.

Ask:
- Can a developer use this by looking at the types alone, without reading the source?
- Are there fewer than ~5 props/return values?
- Does each prop/return value map to a concept the consumer cares about (not an implementation detail)?

### Step 3: Design It Twice

Always propose at least two interface alternatives with different tradeoffs. Evaluate against:

- **Depth**: Which hides more complexity behind a simpler interface?
- **Generality**: Which is reusable in more contexts without modification?
- **Information hiding**: Which leaks fewer implementation details?
- **Re-render impact**: Which creates fewer unnecessary re-renders for consumers?

```tsx
// Option A: Minimal props, component owns everything
type DatePickerProps = {
  value: Date;
  onChange: (date: Date) => void;
  minDate?: Date;
  maxDate?: Date;
};

// Option B: Render-prop for custom display, more flexible but shallower
type DatePickerProps = {
  value: Date;
  onChange: (date: Date) => void;
  renderDay?: (date: Date, isSelected: boolean) => ReactNode;
  renderHeader?: (month: string, year: number) => ReactNode;
  minDate?: Date;
  maxDate?: Date;
};

// Evaluate: Option A is deeper (hides rendering decisions).
// Option B is more flexible but pushes complexity to the consumer.
// Prefer A unless customization is a proven requirement.
```

### Step 4: Define Errors Out of Existence

Use TypeScript's type system to make illegal states unrepresentable.

- Use discriminated unions for state with distinct modes:
  ```typescript
  type AsyncState<T> =
    | { status: 'idle' }
    | { status: 'loading' }
    | { status: 'success'; data: T }
    | { status: 'error'; error: Error };
  ```
- Use conditional/mapped types to prevent conflicting props
- Make required props required, optional props optional -- don't use `Partial<T>` when only some fields are truly optional
- Use branded types for IDs to prevent mixing different entity types:
  ```typescript
  type UserId = string & { __brand: 'UserId' };
  type TeamId = string & { __brand: 'TeamId' };
  ```

### Step 5: Decide State Ownership

For each piece of state the module needs:
- **Who owns it?** Component, parent, context, or external store?
- **Is it derived?** If it can be computed from other state/props, don't store it.
- **What's the re-render impact?** State should live as close to its consumer as possible.
- **Does it need to persist?** Across navigations (context/store), across refreshes (URL/storage)?

### Step 6: Choose Names

- Component: PascalCase noun describing what it renders
- Hook: `use` + what it provides to the consumer
- Props: named from the consumer's perspective, not the implementation's
- Callbacks: `on` + event for props, `handle` + event for internal handlers
- Booleans: `is`/`has`/`should` prefix

### Step 7: Plan for Re-Renders

- Will this hook return stable references? (Memoize objects/callbacks in the return value)
- Will this component re-render its children unnecessarily? (Consider composition, children-as-props)
- Does the state architecture minimize the blast radius of updates?

## Output Format

```
## Design: [Module Name]

### Purpose
[One sentence: what this module does]
[One sentence: what complexity it hides from the consumer]

### Option A: [Name]
```tsx
// Interface definition (props or hook signature + return type)
```
**Tradeoffs**: [depth, generality, information hiding assessment]

### Option B: [Name]
```tsx
// Alternative interface
```
**Tradeoffs**: [depth, generality, information hiding assessment]

### Recommendation
[Which option and why, evaluated against depth / generality / information hiding / re-render impact]

### State Architecture
- [State item]: owned by [who], because [reason]
- [Derived value]: computed from [source], not stored
- Re-render boundary: [where state updates are contained]

### Type Definitions
```typescript
// Final TypeScript types and interfaces
```

### Usage Example
```tsx
// How a consumer would use this module
```
```
