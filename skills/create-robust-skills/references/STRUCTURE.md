# Skill Structure Reference

Detailed breakdown of the optimal folder structure for a robust skill.

## Minimal Structure (Single Topic)

```
my-skill/
└── SKILL.md
```

## Standard Structure (Core + Depth)

```
my-skill/
├── SKILL.md
└── references/
    ├── CONCEPTS.md
    └── CHEATSHEET.md
```

## Complete Structure (Full Documentation)

```
my-skill/
├── SKILL.md
├── references/
│   ├── CONCEPTS.md
│   ├── PATTERNS.md
│   ├── ALTERNATIVES.md
│   └── CHEATSHEET.md
└── examples/
    ├── scenario-1.md
    └── scenario-2.md
```

## File Purposes

| File | Purpose | Required |
|------|---------|----------|
| SKILL.md | Main guide, quick reference, decision trees | ✅ Yes |
| references/CONCEPTS.md | Deep dive on core concepts | Optional |
| references/PATTERNS.md | Detailed pattern explanations | Optional |
| references/ALTERNATIVES.md | Trade-offs, comparisons | Optional |
| references/CHEATSHEET.md | Quick lookup, syntax reference | Optional |
| examples/scenario-1.md | Step-by-step use case | Optional |
| examples/scenario-2.md | Another use case | Optional |

## When to Use Each Level

### Minimal (SKILL.md only)
- Skill is simple, single topic
- No deep dives needed
- Example: `find-skills`

### Standard (SKILL.md + references/)
- Core knowledge in main file
- Depth available in references
- Most skills fit here
- Example: `clean-ddd-hexagonal`

### Complete (Full structure)
- Complex domain
- Multiple audiences (beginner → advanced)
- Real-world examples needed
- Example: `modern-javascript`

## Anti-Patterns

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| Everything in SKILL.md | Too long, hard to navigate | Split into references/ |
| References without SKILL.md | No quick start | Always start with SKILL.md |
| Too many files | Over-engineering | Keep it simple, add as needed |
| No examples | Theory only | Add examples/ for common cases |