---
name: create-robust-skills
description: Proactively apply when creating, improving, or reviewing agent skills. Triggers on skill creation, SKILL.md writing, skill patterns, skill structure, skill templates, or when user asks "how to write a skill", "create a skill", "skill template", "make a better skill". Use when designing new skills, refactoring existing skills, or ensuring skills follow best practices. Comprehensive guide for creating high-quality, maintainable agent skills.
---

# Create Robust Skills

A comprehensive guide for creating maintainable, well-documented agent skills following established patterns from the skills ecosystem.

## When to Use (and When NOT to)

| Use When | Skip When |
|----------|-----------|
| Creating a new skill from scratch | Just need quick help with code |
| Improving existing skills | Writing一次性 scripts |
| Reviewing skill quality | Simple one-off tasks |
| Teaching others about skills | Already have a working skill |
| Standardizing skill format | Emergency hotfix |

**Start with the core SKILL.md.** Add references and examples only when needed for depth.

---

## Critical: What Makes a Skill Effective

A skill must be **actionable**, **discoverable**, and **maintainable**.

### Actionable
- Clear decision trees for common questions
- Concrete code examples (before/after)
- Anti-patterns to avoid
- Step-by-step workflows

### Discoverable
- Comprehensive triggers in YAML header
- Natural language keywords users would say
- Decision trees for navigation

### Maintainable
- Modular: core in SKILL.md, depth in references/
- Well-organized structure
- Clear file naming

---

## Quick Decision Trees

### "What should this skill cover?"

```
Scope?
├─ Single domain (React, DDD, Docker)          → 1 skill, focused
├─ Multiple related topics (frontend patterns)  → 1 skill with sections
├─ Unrelated topics                             → Separate skills
└─ Very broad (all of Python)                  → Multiple skills + meta-skill
```

### "What goes in SKILL.md vs references/?"

```
Content placement?
├─ Core concepts, quick reference               → SKILL.md
├─ Detailed explanations, deep dives             → references/
├─ Real-world usage scenarios                   → examples/
├─ Copy-paste templates                         → references/ or examples/
└─ Alternative solutions, trade-offs            → references/
```

### "How should triggers be written?"

```
Trigger style?
├─ Technical terms (react, docker, ddd)         → Include in triggers
├─ User questions ("how do I", "can you")       → Include in description
├─ Related concepts (refactor, modernize)        → Include as triggers
├─ Opposite concepts                            → Skip (not relevant)
└─ Overly specific                              → Broaden or remove
```

---

## Skill Structure

```
skill-name/
├── SKILL.md                    # Required: Main guide
├── references/                 # Optional: Deep dives
│   ├── TOPIC-1.md
│   ├── TOPIC-2.md
│   └── CHEATSHEET.md          # Quick reference
└── examples/                  # Optional: Concrete use cases
    ├── example-1.md
    └── scenario-1.md
```

---

## YAML Header Requirements

```yaml
---
name: skill-name
description: Proactively apply when [trigger condition]. Triggers on [keywords]. Use when [use cases]. [One sentence summary].
---
```

**Field breakdown:**
- `name`: kebab-case, descriptive
- `description`: 1-2 sentences, triggers + use cases

### Example Headers

```yaml
# Good - specific triggers
name: react-typescript
description: Proactively apply when writing React with TypeScript. Triggers on react, typescript, react-ts, component typing, hooks types. Use when creating React components, typing props, or migrating JS to TS. Type-safe React development patterns.

# Good - clear use case
name: docker-optimization
description: Proactively apply when optimizing Docker images and containers. Triggers on docker, container, Dockerfile, image size, build time. Use when creating Dockerfiles, optimizing builds, or reducing image sizes. Docker best practices and optimization techniques.

# Bad - too vague
name: js
description: Helps with JavaScript. Use when writing code.
```

---

## Decision Trees

Include 2-4 decision trees covering common questions.

```
### "Which [pattern] should I use?"

What do you need?
├─ [Scenario A] → [Solution 1]
├─ [Scenario B] → [Solution 2]
├─ [Scenario C] → [Solution 3]
└─ [Scenario D] → [Solution 4]
```

### Tree Example (from modern-javascript)

```
### "Which array method should I use?"

What do you need?
├─ Transform each element           → .map()
├─ Keep some elements               → .filter()
├─ Find one element                 → .find() / .findLast()
└─ Reduce to single value           → .reduce()
```

---

## Code Examples: Before/After

Always use ❌ for bad, ✅ for good.

```javascript
// ❌ Legacy pattern
const last = arr[arr.length - 1];

// ✅ Modern pattern
const last = arr.at(-1);
```

```typescript
// ❌ Any type
const data: any = fetchData();

// ✅ Proper typing
const data = await fetchData() as UserResponse;
```

### Example Patterns to Include

| Pattern | Show | Include |
|---------|------|---------|
| Common mistake | ❌ code | Problem explanation |
| Modern solution | ✅ code | Why it's better |
| Trade-off | Both | When to use each |
| Evolution | Legacy → Modern | Migration path |

---

## Anti-Patterns Table

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| Too many triggers | Hard to match correctly | Focus on 5-10 key terms |
| No decision trees | Users can't navigate | Add 2-4 quick trees |
| Missing examples | Theory only | Include ❌/✅ pairs |
| No references | No depth | Add references/ for details |
| Vague description | Can't discover | Include triggers + use cases |
| Single monolithic file | Hard to maintain | Split into SKILL.md + references/ |

---

## When to Use / Not Use Section

Critical for setting boundaries. Include a table like:

| Use When | Skip When |
|----------|-----------|
| Complex domain with many rules | Simple single task |
| Long-lived system | Prototype/MVP |
| Team collaboration | Solo work |
| Need to swap implementations | Fixed technology |

---

## References Folder

Create `references/` for deep dives that aren't essential for the main guide.

### When to Add References

- Detailed explanation of a concept
- Alternative approaches with trade-offs
- Language/framework-specific implementations
- Extended examples
- Cheatsheets for quick lookup

### Reference File Naming

```
references/
├── CONCEPTS.md           # Core concepts deep dive
├── TOOLS.md              # Tool-specific guidance
├── PATTERNS.md           # Common patterns
├── CHEATSHEET.md         # Quick reference
└── ALTERNATIVES.md       # Trade-offs
```

---

## Examples Folder

Create `examples/` for concrete, copy-pasteable scenarios.

### When to Add Examples

- Real-world use cases users might face
- Step-by-step scenarios
- Before/after full implementations
- Edge cases and how to handle them

### Example File Structure

```markdown
# Example: Creating a React Component

## Scenario
User needs to create a type-safe component with props.

## Step-by-step

1. Define prop types
2. Create component
3. Add default props

## Result

```tsx
interface Props {
  name: string;
  onClick?: () => void;
}

export const Button: React.FC<Props> = ({ name, onClick }) => (
  <button onClick={onClick}>{name}</button>
);
```

---

## Writing Style Guidelines

### Tone
- Direct, actionable
- Solution-oriented
- Clear and concise

### Structure
- Use headers for navigation
- Use tables for comparisons
- Use code blocks for examples
- Use lists for quick reference

### Formatting
```markdown
### Section Title

| Table | Header |
|-------|--------|
| Content | Content |

```language
code here
```
```

---

## Quality Checklist

Before publishing a skill, verify:

- [ ] YAML header with name, description, triggers
- [ ] At least 2 decision trees
- [ ] Code examples with ❌/✅ pairs
- [ ] Anti-patterns table
- [ ] When to use / skip section
- [ ] References for deeper topics (if needed)
- [ ] Examples for common scenarios (if needed)
- [ ] Clear, actionable content
- [ ] Consistent formatting

---

## Research: Using External Sources

A **robust skill** is grounded in current best practices and authoritative sources. When writing a skill, leverage external resources to ensure accuracy and relevance.

### Using Web Search for State-of-the-Art

When creating a skill on a specific topic, use web search to find:

- **Latest patterns** — What's currently recommended (not legacy approaches)
- **Official documentation** — Language/framework docs, RFCs, specs
- **Community consensus** — Stack Overflow votes, GitHub discussions
- **Expert resources** — Blog posts from recognized authorities

```bash
# Example searches when writing a skill
"React best practices 2024"
"Docker multi-stage build optimization"
"DDD aggregate design patterns"
"TypeScript generic best practices"
```

**Tip:** Prefer recent sources (2023-2026) and verify multiple sources agree.

### User-Provided Sources

Users may provide sources in two ways:

#### 1. Prompt References

The user mentions specific works in their request:

```
"Create a skill for refactoring, inspired by Clean Code by Robert C. Martin"
"Write a skill about testing, based on 'Working Effectively with Legacy Code'"
"Use the principles from 'The Pragmatic Programmer'"
```

**Action:** Take these references as the foundation for the skill. Read the key concepts and incorporate them.

#### 2. Filesystem Sources

The user provides actual files or documents:

```
/var/home/cmnemoi/.agents/Working Effectively with Legacy Code.pdf
/var/home/cmnemoi/.agents/books/refactoring.pdf
/var/home/cmnemoi/.agents/notes/architecture-patterns.md
```

**Action:** 
1. Read the provided files first
2. Extract key concepts and patterns
3. Integrate them into the skill
4. Reference them as sources

### Integrating Sources into Skills

When using external sources:

1. **Credit the source** — Add to "Sources" section at the end
2. **Adapt, don't copy** — Convert concepts to skill format (decision trees, examples)
3. **Verify currency** — Check if information is still relevant
4. **Cross-reference** — Compare multiple sources for consensus

```markdown
## Sources

### Primary References
- [Clean Code](https://example.com) — Robert C. Martin
- [Working Effectively with Legacy Code](https://example.com) — Michael Feathers

### Web Resources
- [React Docs: Best Practices](https://react.dev)
- [Martin Fowler: Refactoring](https://martinfowler.com)
```

---

## Reference Documentation

| File | Purpose |
|------|---------|
| [references/STRUCTURE.md](references/STRUCTURE.md) | Detailed folder structure |
| [references/HEADER.md](references/HEADER.md) | YAML header best practices |
| [references/WRITING.md](references/WRITING.md) | Writing style guide |
| [references/EXAMPLES.md](references/EXAMPLES.md) | Example scenarios |
| [references/CHEATSHEET.md](references/CHEATSHEET.md) | Quick reference |

---

## Sources

### Skills Ecosystem
- [Skills CLI](https://skills.sh/) — Package manager for skills
- [Skills Leaderboard](https://skills.sh/) — Top skills by installs

### Inspiration
- [modern-javascript](skills/modern-javascript/) — ES6-ES2025 patterns
- [clean-ddd-hexagonal](skills/clean-ddd-hexagonal/) — Architecture patterns
- [find-skills](skills/find-skills/) — Skill discovery workflow