---
name: advanced-refactoring
description: Proactively apply when refactoring legacy code, applying patterns, or improving code structure. Triggers on refactor, legacy code, technical debt, patterns, extract method, dependency breaking, seams, monster method, large class. Use when working with untested code, complex dependencies, or applying design patterns. Comprehensive refactoring from Feathers, Carlo, Kerievsky, and refactoring.guru.
---

# Advanced Refactoring

A concise skill for refactoring legacy code and applying patterns safely. Detailed content is in `references/` and `examples/`.

## Core Philosophy

> **Legacy code is code without tests.** — Michael Feathers

> **Legacy code is profitable code that you're afraid to change.** — Nicolas Carlo

Without tests, you cannot verify changes preserve behavior.

---

## Decision Trees

### "What type of refactoring do I need?"

What's your starting point?
├─ Code has no tests                    → Go to "How to add tests to legacy code?"
├─ Code has tests but is messy          → Go to "Which code smell needs fixing?"
├─ Large class / monster method         → Go to "How to break down a monster?"
└─ Complex dependencies                 → Go to "How to break dependencies?"

### "How to add tests to legacy code?"

What's your constraint?
├─ Can't instantiate the class           → See `references/DEPENDENCY-BREAKING.md` (lines 85-120: Parameterize Constructor)
├─ Can't call the method                 → See "Seams" below (lines 71-81)
├─ Method has side effects               → See "Characterization Tests" below (lines 83-98)
├─ No time for comprehensive tests       → See `references/CARLO-STRATEGIES.md` (lines 16-86: Sprout Method/Class)
├─ Just need to understand it            → Scratch Refactoring (see Feathers Ch.16, `DEPENDENCY-BREAKING.md`)
└─ Need quick safety net                → Approval Testing → `references/CARLO-STRATEGIES.md` (lines 148-180)

### "Which code smell needs fixing?"

What problem do you see?
├─ Duplicate code across classes        → Extract Method → Move Method (see `references/REFACTORING-GURU.md`: 68-196)
├─ Large class with many responsibilities → Extract Class → see `references/REFACTORING-GURU.md` (lines 50-52)
├─ Long method with nested conditionals → Replace Nested Conditionals with Guard Clauses (`REFACTORING-GURU.md`: 241-243)
├─ Repeated null checks                  → Introduce Null Object → see `references/REFACTORING-TO-PATTERNS.md` (lines 113-145)
├─ Complex conditional logic            → Replace Conditional with Polymorphism → see `references/REFACTORING-TO-PATTERNS.md` (lines 81-109)
├─ Long parameter list                   → Introduce Parameter Object → see `references/REFACTORING-TO-PATTERNS.md` (lines 149-170)

### "How to break down a monster method?"

What extraction opportunity exists?
├─ Clear block of functionality        → Extract Method (prefer 0 params)
├─ No clear sections                    → Introduce Sensing Variable first
├─ Too many local variables             → Break Out Method Object → `references/DEPENDENCY-BREAKING.md` (lines 224-262)

---

## Key Concepts (See References for Details)

### Feathers: Legacy Code Change Algorithm

```
1. Identify change points      → Where does behavior need to change?
2. Find test points            → Where can we detect effects?
3. Break dependencies          → Get code into test harness
4. Write tests                 → Characterization tests first
5. Make changes and refactor   → Safe incremental improvement
```

### Seams

A **seam** is a place where you can alter behavior without editing in that place.

| Seam Type | Best For |
|-----------|----------|
| **Object Seam** | Override method in subclass (OOP preferred) |
| **Preprocessor Seam** | #ifdef, environment variables (C/C++) |
| **Link Seam** | Different binaries at link time |

Finding seams: Look for dependency injection points, extension points, parameterization, delegation.

### Characterization Tests

For code you don't understand:

1. **Observe** — Run code, see what it actually does
2. **Record** — Write tests capturing current behavior
3. **Verify** — Tests should pass (documenting behavior)
4. **Refactor** — Now you have a safety net

```java
@Test
void shouldBehaveTheWayItCurrentlyDoes() {
    Result result = new ClassUnderTest().doSomething(input);
    assertEquals(expectedCurrentBehavior, result.getValue());  // NOT what you wish!
}
```

### Pinch Points

A **pinch point** is a narrow place in the effect graph where testing a few methods detects changes in many methods. Sketch the effect diagram, find bottlenecks, test there for maximum coverage.

### Monster Methods

1. **Start with sensing variables** — Track interesting values
2. **Extract what you know** — Methods with clear variables
3. **Target low-coupling extractions** — 0 parameters are safest
4. **Work systematically** — Small extractions accumulate

### SRP Heuristics (Feathers)

How to identify responsibilities in large classes:

1. **Group Methods** — Look for similar names
2. **Look at Hidden Methods** — Many private methods often = another class
3. **Look for Decisions That Can Change** — Hard-coded decisions = responsibilities
4. **Look for Internal Relationships** — Which variables used by which methods
5. **Describe Primary Responsibility** — Can you describe in one sentence?
6. **Do Scratch Refactoring** — Extract freely, throw away after understanding

---

## Integration: Complete Workflow

```
START: Need to change legacy code
│
├─► Have tests?
│   ├─ YES → Safe to refactor
│   │
│   └─ NO → Can we add tests easily?
│       ├─ YES → Add tests, then refactor
│       │
│       └─ NO → Use dependency-breaking techniques
│           ├─ 1. Identify change points
│           ├─ 2. Find test points (sensing/separation)
│           ├─ 3. Break dependencies → see references/DEPENDENCY-BREAKING.md
│           ├─ 4. Write characterization tests
│           └─ 5. Refactor safely
│
└─► Apply appropriate pattern → see references/REFACTORING-TO-PATTERNS.md
    ├─ Large class → Extract Class
    ├── Duplicated code → Extract Method, Move Method
    ├── Complex conditionals → Replace Conditional with Polymorphism
    └── Long method → Break Out Method Object, Extract Method
```

### Safety Principles

1. **Always have a test safety net** before refactoring
2. **Characterization tests first** — document what code actually does
3. **Small steps** — refactor incrementally, run tests after each
4. **Use IDE refactoring** — let tools do mechanical work
5. **Don't refactor and add features simultaneously** — separate concerns
6. **Commit frequently** — small commits, verify each

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| Refactoring without tests | Risk of breaking behavior | Add characterization tests first |
| Big bang refactoring | High risk, hard to verify | Incremental refactoring |
| Ignoring failing tests | Hidden regressions | Fix or understand failures |
| Refactoring for "cleanliness" only | No immediate benefit | Refactor when there's a reason |
| Pattern addiction | Over-engineering | Apply patterns only when needed |

---

## When to Use This Skill

| Use When | Skip When |
|----------|-----------|
| Working with legacy code (no tests) | Greenfield development |
| Adding features to untested code | Code already well-tested |
| Breaking down large classes | Small, focused classes |
| Applying design patterns | Code doesn't need patterns |

---

## Quick Reference

| Problem | Solution |
|---------|----------|
| Can't test class (hidden deps) | Parameterize Constructor → `references/DEPENDENCY-BREAKING.md` (lines 85-120) |
| Can't mock dependency | Extract Interface → `references/DEPENDENCY-BREAKING.md` (lines 47-84) |
| Long method with many vars | Break Out Method Object → `references/DEPENDENCY-BREAKING.md` (lines 224-262) |
| Need to add feature (can't test existing) | Sprout Method → `references/CARLO-STRATEGIES.md` (lines 16-53) |
| Need to add substantial new code | Sprout Class → `references/CARLO-STRATEGIES.md` (lines 55-86) |
| Need behavior before/after existing | Wrap Method → `references/CARLO-STRATEGIES.md` (lines 88-115) |
| Complex conditional logic | Replace Conditional with Polymorphism → `references/REFACTORING-TO-PATTERNS.md` (lines 81-109) |
| Null checks everywhere | Introduce Null Object → `references/REFACTORING-TO-PATTERNS.md` (lines 113-145) |
| Long parameter list | Introduce Parameter Object → `references/REFACTORING-TO-PATTERNS.md` (lines 149-170) |
| Code smell identification | See `references/REFACTORING-GURU.md` (lines 5-63: 21 code smells) |
| Practical examples | See `examples/SCENARIOS.md` |

---

## Sources

### Primary References
- [Working Effectively with Legacy Code](https://www.amazon.com/Working-Effectively-Legacy-Michael-Feathers/dp/0131177052) — Michael Feathers → `references/DEPENDENCY-BREAKING.md`
- [Understand Legacy Code](https://understandlegacycode.com/) — Nicolas Carlo → `references/CARLO-STRATEGIES.md`
- [Refactoring to Patterns Catalog](https://www.industriallogic.com/refactoring-to-patterns/catalog/) — Joshua Kerievsky → `references/REFACTORING-TO-PATTERNS.md`
- [Refactoring.guru](https://refactoring.guru/refactoring) → `references/REFACTORING-GURU.md`

### Additional Resources
- [Refactoring to Patterns (PDF)](http://www.tarrani.net/RefactoringToPatterns.pdf) — Joshua Kerievsky
- [Martin Fowler: Refactoring](https://martinfowler.com/books/refactoring.html) — Foundation text
- [Your Code as a Crime Scene](https://www.adamtornhill.com/) — Adam Tornhill
- [The Mikado Method](https://www.mikadomethod.info/) — See `references/CARLO-STRATEGIES.md`