# Advanced Refactoring — Quick Reference

Quick lookup for common refactoring scenarios.

---

## Decision Matrix

| Problem | Solution |
|---------|----------|
| Can't test class (hidden deps) | Parameterize Constructor |
| Can't mock dependency | Extract Interface |
| Static method blocks testing | Extract and Override Call |
| Global variable | Encapsulate Global References |
| Singleton | Introduce Static Setter |
| Long method with many vars | Break Out Method Object |
| Utility class with only static methods | Introduce Instance Delegator |
| Need to add feature (can't test existing) | Sprout Method |
| Need to add substantial new code | Sprout Class |
| Need behavior before/after existing | Wrap Method |
| Need to wrap entire class | Wrap Class |

---

## Code Smell → Refactoring Quick Lookup

| Smell | Refactoring |
|-------|-------------|
| **Long Method** | Extract Method |
| **Large Class** | Extract Class |
| **Duplicate Code** | Extract Method → Move |
| **Switch Statements** | Replace Conditional with Polymorphism |
| **Long Parameter List** | Introduce Parameter Object |
| **Null Checks** | Introduce Null Object |
| **Feature Envy** | Move Method |
| **Data Clumps** | Introduce Parameter Object |
| **Primitive Obsession** | Replace Primitive with Object |
| **Parallel Inheritance** | Push Down / Move Method |
| **Shotgun Surgery** | Move Method |
| **Divergent Change** | Extract Class |
| **Lazy Class** | Inline Class |
| **Middle Man** | Remove Middle Man |

---

## The 5-Step Algorithm (Feathers)

1. **Identify change points** — Where needs to change?
2. **Find test points** — Where can detect effects?
3. **Break dependencies** — Get into test harness
4. **Write tests** — Characterization tests first
5. **Make changes** — Refactor safely

---

## Characterization Test Template

```java
@Test
void shouldBehaveTheWayItCurrentlyDoes() {
    // Setup
    ClassUnderTest cut = new ClassUnderTest();
    
    // Execute
    Result result = cut.doSomething(input);
    
    // Assert CURRENT behavior (not desired!)
    assertEquals(expectedCurrentBehavior, result.getValue());
}
```

---

## Pattern Introduction Timing

| Too Early | Just Right | Too Late |
|-----------|------------|----------|
| "We might need it" | Clear code smell appears | Code is unmaintainable |
| No duplication | Repeated pattern detected | Refactoring becomes rewrite |
| Team doesn't need it | Pattern solves real problem | High risk of breaking things |

---

## Safe Refactorings (No Tests Required)

These can be done safely without test coverage:
- Rename Symbol (variable, method, class)
- Extract Variable
- Inline Variable
- Extract Function/Method
- Split Declaration and Assignment
- Slide Statements (reorder within scope)
- Pull Down Common Tail of Conditional

**All other refactorings need tests first!**

---

## Key Commands/Methods

| Technique | Key Idea |
|-----------|----------|
| Extract Interface | `interface X { methods used }` + inject |
| Parameterize Constructor | `new Class(dep)` instead of `new Class()` |
| Subclass and Override | `class TestSub extends Prod` + override |
| Break Out Method Object | `new MethodObject(args).execute()` |
| Sprout Method | Add new method, call from old |
| Wrap Method | Rename old → new wrapping |

---

## Sources

- **Feathers**: Working Effectively with Legacy Code
- **Carlo**: UnderstandLegacyCode.com
- **Kerievsky**: Refactoring to Patterns (Industrial Logic)
- **Fowler**: Refactoring (refactoring.guru)