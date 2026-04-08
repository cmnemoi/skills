# Create Robust Skills — Quick Reference

## Skill Structure

```
skill-name/
├── SKILL.md                    # Required
├── references/                  # Optional
│   ├── CONCEPTS.md
│   └── CHEATSHEET.md
└── examples/                   # Optional
    └── scenario.md
```

## YAML Header Template

```yaml
---
name: skill-name
description: Proactively apply when [condition]. Triggers on [k1], [k2], [k3]. Use when [use cases]. [Summary].
---
```

## Decision Tree Template

```
### "Question?"

What do you need?
├─ [Case A] → [Solution 1]
├─ [Case B] → [Solution 2]
└─ [Case C] → [Solution 3]
```

## Code Example Template

```markdown
// ❌ Legacy
[bad code]

// ✅ Modern
[good code]
```

## Anti-Pattern Table Template

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| [Pattern] | [Issue] | [Solution] |

## Quality Checklist

- [ ] YAML header with name, triggers, use cases
- [ ] 2-4 decision trees
- [ ] ❌/✅ code examples
- [ ] Anti-patterns table
- [ ] When to use / skip section
- [ ] references/ for depth (if needed)
- [ ] examples/ for scenarios (if needed)

## Writing Rules

| Do | Don't |
|----|-------|
| Be direct | Be verbose |
| Lead with answer | Explain first |
| Use tables | Walls of text |
| Show complete code | Incomplete snippets |