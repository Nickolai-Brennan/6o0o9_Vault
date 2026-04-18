---
applyTo: "**/*.{ts,tsx}"
---

# TypeScript Development Standards

## Type Safety

- Avoid using `any` type—use `unknown` or specific types instead
- Use strict null checks (no `null` or `undefined` without explicit handling)
- Define interfaces for all object shapes

```typescript
// Avoid
function processData(data: any) {
    return data.value;
}

// Prefer
interface DataShape {
    value: string;
}

function processData(data: DataShape): string {
    return data.value;
}
```

## Naming Conventions

- Use PascalCase for types, interfaces, and classes
- Use camelCase for variables, functions, and methods
- Use UPPER_CASE for constants

## Modern TypeScript Patterns

- Use optional chaining (`?.`) and nullish coalescing (`??`)
- Prefer `const` over `let`; never use `var`
- Use arrow functions for callbacks and short functions

## React-Specific (for .tsx files)

- Use functional components with TypeScript props interfaces
- Type all props and state explicitly
- Use proper event types (e.g., `React.ChangeEvent<HTMLInputElement>`)
- 
