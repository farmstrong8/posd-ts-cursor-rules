# POSD TypeScript Cursor Rules

Cursor rules that apply principles from **"A Philosophy of Software Design"** by John Ousterhout to TypeScript development -- both React/React-Native frontend and Node.js backend.

These teach AI agents to think strategically about complexity -- designing deep modules, hiding information behind clean interfaces, structuring state and errors correctly, and catching anti-patterns before they compound.

## Installation

### Cursor (Remote Rules)

1. Open Cursor Settings (`Cmd+Shift+J` / `Ctrl+Shift+J`)
2. Navigate to **Rules & Command** > **Project Rules** > **Add Rule** > **Remote Rule (GitHub)**
3. Enter: `https://github.com/farmstrong8/posd-ts-cursor-rules.git`

The rules are loaded as agent-decided rules -- Cursor applies the relevant one based on context (React work vs Node/backend work).

### Manual

Copy the relevant `SKILL.md` into your project's `.cursor/skills/` directory, or paste its contents into a `.cursor/rules/` file.

## What's Included

### React / React-Native -- `skills/posd-react-rules/SKILL.md`

| Section | What it covers |
|---------|---------------|
| **Strategic Mindset** | Complexity symptoms. Strategic vs tactical. Never use `any`. |
| **Deep Modules** | Simple interfaces hiding complexity. Information hiding. Generality. Pulling complexity downward. |
| **React Component Structure** | Single responsibility. Composition. Meaningful layers. Re-render-aware design. Hook patterns. |
| **State & Types** | State placement and re-render impact. Derived state. Discriminated unions. Illegal states. |
| **Readability & Conventions** | Naming conventions. Meaningful comments. Codebase consistency. |
| **Red Flags** | Anti-pattern checklist: component size, props, state, re-renders, abstractions, hooks. |

### TypeScript / Node.js -- `skills/posd-node-rules/SKILL.md`

| Section | What it covers |
|---------|---------------|
| **Strategic Mindset** | Complexity symptoms. Strategic vs tactical. Never use `any`. |
| **Deep Modules** | Deep services and repositories. Information hiding. Generality. Pulling complexity downward. |
| **Module & Service Structure** | Single responsibility. Meaningful layers. Dependency injection. |
| **Error Handling** | Define errors out of existence. Result types. Typed error classes. |
| **Readability & Conventions** | Naming conventions. Meaningful comments. Codebase consistency. |
| **Red Flags** | Anti-pattern checklist: module size, abstractions, error handling, dependencies, data flow. |

## Philosophy

These rules are grounded in Ousterhout's core insight: **complexity is the root cause of most problems in software**. It manifests as:

- **Change amplification** -- a simple change touches many files
- **Cognitive load** -- you must understand too much to make a change safely
- **Unknown unknowns** -- it's not obvious what needs to change or what might break

This is opinionated guidance, not dogma. The principles are strong defaults -- break them deliberately when you have a clear reason.

## Attribution

Based on *A Philosophy of Software Design* by John Ousterhout. Read the book for the full framework.

## License

MIT
