# My agent skills

My agent skills for opencode. Backend patterns, modern JavaScript, refactoring techniques, and skill creation guides.

```bash
npx skills add https://github.com/cmnemoi/skills
```

---

## Available Skills

### clean-ddd-hexagonal

Apply Clean Architecture + DDD + Hexagonal patterns to backend services. Use when designing APIs, microservices, domain models, aggregates, repositories, bounded contexts, or scalable backend structure. Language-agnostic (Go, Rust, Python, TypeScript, Java, C#).

```bash
npx skills add https://github.com/cmnemoi/skills --skill clean-ddd-hexagonal
```

### modern-javascript

Modern JavaScript (ES6-ES2025) patterns and best practices. Use when writing new JavaScript, refactoring legacy code, modernizing codebases, or implementing functional patterns. Covers `.at()`, `.toSorted()`, `.toReversed()`, `Object.groupBy()`, optional chaining, nullish coalescing, async/await, and more.

```bash
npx skills add https://github.com/cmnemoi/skills --skill modern-javascript
```

### advanced-refactoring

Comprehensive refactoring techniques for legacy code. Use when working with untested code, large classes, monster methods, complex dependencies, or applying design patterns. Based on Feathers, Carlo, Kerievsky, and refactoring.guru.

```bash
npx skills add https://github.com/cmnemoi/skills --skill advanced-refactoring
```

### create-robust-skills

Guide for creating high-quality, maintainable agent skills. Use when designing new skills, refactoring existing skills, or ensuring skills follow best practices. Covers SKILL.md structure, templates, and patterns.

```bash
npx skills add https://github.com/cmnemoi/skills --skill create-robust-skills
```

### find-skills

Helps discover and install skills from the open agent skills ecosystem. Use when searching for functionality, asking "how do I do X", or exploring available capabilities.

```bash
npx skills add https://github.com/cmnemoi/skills --skill find-skills
```

### dsl-driven-testing

DSL-driven testing with a four-layer architecture (SimpleDSL, ATDD, Protocol Drivers). Use when writing acceptance tests that must run at multiple levels (unit/integration/E2E), structuring test layers, or when tests are too coupled to infrastructure. Write tests in pure business DSL, then plug in different drivers (in-memory, HTTP, UI) — the same scenarios run in milliseconds or as full E2E tests.

```bash
npx skills add https://github.com/cmnemoi/skills --skill dsl-driven-testing
```

### spec-driven-development

Pragmatic Spec-Driven Development centered on a lightweight spec-test-code triangle. Use when defining feature behavior before implementation, keeping specs aligned with tests and code, or introducing lightweight spec workflows. Covers spec types (feature, bugfix, design-first), stable IDs format `(bounded-context).subdomain.feature::rule-name`, and traceability best practices.

```bash
npx skills add https://github.com/cmnemoi/skills --skill spec-driven-development
```

### write-unit-tests

Guide for writing and reviewing unit tests. Use when adding tests, fixing brittle test suites, or guiding agents toward maintainable, refactor-safe tests. Covers TDD, AAA pattern, test doubles (mocks, fakes, stubs), test smells, and when to use a business DSL vs. brownfield pragmatism.

```bash
npx skills add https://github.com/cmnemoi/skills --skill write-unit-tests
```

### ralph-loop

Autonomous coding loop with context resets for mechanical, externally verifiable tasks. Use when a task is guided by a short repo spec, involves binary checks (tests pass/fail), and requires iterative progress. Based on "Ralph Wiggum" pattern for avoiding context rot in long-running autonomous sessions.

```bash
npx skills add https://github.com/cmnemoi/skills --skill ralph-loop
```

