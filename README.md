# POSD React/React-Native Cursor Rules

Cursor rules and agent skills that apply principles from **"A Philosophy of Software Design"** by John Ousterhout to React and React-Native TypeScript development.

These teach AI agents to think strategically about complexity -- designing deep components, hiding information behind clean interfaces, structuring state correctly, and catching anti-patterns before they compound.

## Installation

### Cursor (Remote Rules)

1. Open Cursor Settings (`Cmd+Shift+J` / `Ctrl+Shift+J`)
2. Navigate to **Rules & Command** > **Project Rules** > **Add Rule** > **Remote Rule (GitHub)**
3. Enter: `https://github.com/farmstrong8/posd-ts-cursor-rules.git`

The `AGENTS.md` rules are automatically applied. Skills are auto-discovered and used by the agent when your questions match their descriptions.

### Manual

Copy `AGENTS.md` into your project root. Copy the `skills/` directory into your project.

## What's Included

### AGENTS.md -- Rules

All rules live in a single `AGENTS.md` file. They cover 6 areas:

| Section | What it covers |
|---------|---------------|
| **Strategic Mindset** | Complexity symptoms (change amplification, cognitive load, unknown unknowns). Strategic vs tactical programming. |
| **Deep Modules** | Simple interfaces hiding significant complexity. Information hiding. Generality. Pulling complexity downward. |
| **React Component Structure** | Single responsibility. Composition over configuration. Meaningful layers. Re-render-aware design. Hook patterns. |
| **State & Types** | State placement and re-render impact. Derived state. Discriminated unions. Making illegal states unrepresentable. |
| **Readability & Conventions** | Naming conventions. Meaningful comments. Codebase consistency. |
| **Red Flags** | Anti-pattern checklist: component size, prop count, state problems, re-render issues, abstraction smells, hook smells. |

### Skills -- Active Workflows

3 skills for design, review, and refactoring tasks.

| Skill | When it activates |
|-------|------------------|
| **posd-design-review** | When you ask the agent to review code, evaluate a PR, or assess quality. Produces structured findings with severity levels. |
| **posd-refactoring** | When you ask to refactor, simplify, or reduce complexity. Identifies anti-patterns and proposes incremental before/after changes. |
| **posd-component-design** | When you ask to design a new component, hook, or feature. Walks through interface-first design with two alternatives and tradeoff analysis. |

## Philosophy

These rules are grounded in Ousterhout's core insight: **complexity is the root cause of most problems in software**. It manifests as:

- **Change amplification** -- a simple change touches many files
- **Cognitive load** -- you must understand too much to make a change safely
- **Unknown unknowns** -- it's not obvious what needs to change or what might break

Every rule and skill is designed to reduce one or more of these symptoms in the context of React/React-Native development.

This is opinionated guidance, not dogma. The principles are strong defaults -- break them deliberately when you have a clear reason.

## Attribution

Based on *A Philosophy of Software Design* by John Ousterhout. These rules are an interpretation of the book's principles applied to React/React-Native TypeScript. Read the book for the full framework.

## License

MIT
