# Example: Creating a TypeScript Skill

## Scenario

You want to create a skill for "TypeScript best practices".

## Step-by-Step

### Step 1: Define Scope

Is this a single skill or multiple?

```
TypeScript best practices
├─ Basic types
├─ Advanced types
├─ Generics
├─ Configuration
└─ Migration from JS
```

**Decision**: Single skill with references/ for depth.

### Step 2: Write Header

```yaml
---
name: typescript-best-practices
description: Proactively apply when writing TypeScript code. Triggers on typescript, ts, types, generics, interface, type-alias, type-guards, type-inference. Use when writing new TS code, improving existing types, or migrating JS to TS. TypeScript patterns, best practices, and advanced types.
---
```

### Step 3: Add Decision Trees

```
### "Which type should I use?"

What do you need?
├─ Object shape,可能被扩展     → interface
├─ Union, primitive type      → type alias
├─ Runtime check              → type guard
├─ Reusable type parameter     → generic
└─ Pick specific properties   → Pick<T, K>
```

### Step 4: Add Code Examples

```typescript
// ❌ Any type hides errors
const data: any = fetchData();

// ✅ Proper type
const data = await fetchData() as UserData;
```

### Step 5: Create References

```
typescript-best-practices/
├── SKILL.md
└── references/
    ├── GENERICS.md      # Deep dive on generics
    ├── UTILITY-TYPES.md # Pick, Omit, etc.
    └── CHEATSHEET.md    # Quick reference
```

### Step 6: Add Anti-Patterns Table

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| `any` type | No type safety | Use proper types |
| `interface` for primitives | Confusing | Use `type` |
| Missing null checks | Runtime errors | Use optional chaining |

## Resulting Structure

```
typescript-best-practices/
├── SKILL.md
└── references/
    ├── GENERICS.md
    ├── UTILITY-TYPES.md
    └── CHEATSHEET.md
```

## Key Takeaways

1. **Start simple** — SKILL.md first, add references as needed
2. **Be specific** — Clear triggers, specific use cases
3. **Show, don't tell** — ❌/✅ code examples
4. **Help navigation** — Decision trees
5. **Set boundaries** — When to use / skip