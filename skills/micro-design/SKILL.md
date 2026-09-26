---
name: micro-design
description: Personal micro-design heuristics for writing and refactoring code - naming conditions, placing responsibilities, value objects, rich entities, predicates, duplication, names, comments, and how far a refactoring should go. Use whenever writing new code or refactoring existing code, in any language or repository, even if the user does not mention clean code or design. Does not cover tests or architecture.
---

# Micro-design

Covers methods, classes and names. Architecture, tests and working practices (TDD, commits, running checks) are out of scope. Before/after examples: [references/examples.md](references/examples.md).

## Core principle

**Prefer changes that reduce the interpretation a reader needs to understand the domain. Never add indirection, types, methods or abstractions merely to satisfy a rule.** When a rule below conflicts with this principle in a given case, the principle wins.

## Relation to the codebase

- **A floor against habits, not against decisions.** Apply these rules to the code you touch even if the codebase is looser by habit. Follow a stricter codebase. An explicit convention (linter rule, ADR, CONTRIBUTING, the repo's own AGENTS.md/CLAUDE.md) wins over these rules.
- **Propose and wait** before changing macro design (splitting a service, passing an ID instead of an object…), introducing a shared abstraction (including mutualising code across files), or changing an API consumed outside the repository.

## Extraction

- **Name a condition when the name carries domain knowledge the expression hides**: `isFirstContact($rebelBase)` rather than `$rebelBase === null`. The expression's shape is irrelevant (null check, comparison, input guard).
- Do not name general knowledge (`if ($amount <= 0)` stays) and do not wrap a single domain call (`$rebelBase->isDecoded()`).
- Prefer an explaining variable to a method when the expression is used once, is not a condition, and stays at the method's level of abstraction.
- **Public methods read as a recipe** at one level of abstraction. A clean recipe can hide two responsibilities: suspect it when private methods do not share collaborators, steps change for different reasons, the honest class name contains "and", or an `else` is as long as its `if`. Write the recipe anyway, then propose the split.
- **Command/query separation.** Operations on immutable values that return a new instance count as queries.
- **Narrowest scope**: declare variables at first use; give functions minimal visibility and define them right below their first caller.
- No threshold on method length or parameter count; the other rules find the right size.

## Placement

Default order: on the object that holds the information, then pass what is needed, then a local private method. A value object is an alternative at every step.

- Put behaviour on the object only when it is **obvious**: it reads the object's direct state, it is a natural question in the domain's vocabulary, and it is transversal across features.
- **A feature's policy stays out of the object**, even when it only reads that object's state (`canApplyForPsychologicalSupport()` belongs to the action).
- **Never add to a god object**: extract locally or introduce the missing object.
- **Chaining is fine on a single abstraction** (fluent API, operations returning the same type). Descending into different types, or reaching another aggregate (`$player->getDaedalus()`), is suspect. An inherited object graph is macro design: tolerate it.

For intrinsic properties vs policies, central actors, invariants and varying rules, read [references/placement.md](references/placement.md).

## Value objects

- If the codebase models values as value objects everywhere, do the same. Otherwise, the signals are an invariant, behaviour, a unit (`Duration`, not an int of minutes) or two same-type primitives easy to confuse. A domain name alone is not enough.
- **Values only, never a reference to an entity.** Pass the configuration values it needs, or a small value object carrying them (`GameCalendar` for `GameDate`).
- *Exploratory*: a value object is often a better home for behaviour than an entity or a service, especially for a cluster of values that travel together with logic, and for what crosses aggregates (`$learner->book($lesson->slot())` rather than `book($lesson->schedule(), $lessonId)`). Create it if it stays local; propose it if it would be shared.

## Entities and predicates

- Entities expose **domain verbs**, never public setters. Intrinsic behaviour goes in the entity; behaviour needing other aggregates, infrastructure or events goes in a service.
- **Negative predicates**: provide an exact domain antonym (`isHuman()` / `isMush()`), otherwise use `!`. Do not invent `isNotX()`, and do not use an approximate antonym: it silently changes the meaning. Implement the negative from the positive, and add it only when a caller needs it. Existing `isNotX()` methods are listed, not renamed.

## Names

- Domain names, not technical ones (`doesNotUpdate`, not `isNoOp`). A name that would fit any other object says nothing.
- **Banned** — classes: `Manager`, `Helper`, `Util(s)`, `Processor`, `Data`, `Info`, `Wrapper`, `Common`, `Misc`, `Stuff`; methods: `handle()` outside framework conventions, `process()`, `manage()`, `perform()`, `doX()`, bare `compute()`; variables: `$data`, `$info`, `$result`, `$tmp`, `$obj`, `$item` unless it is a domain item.
- No abbreviations unless better known than the full word (`http`, `url`, `id`, `json`).
- **No single-letter or short names, including in lambdas and loops**: `fn (Planet $planet) => ...`, not `fn ($p) => ...`. Agents tend to slip here.
- Ecosystem suffixes are fine (`Service`, `Repository`, `Subscriber`, `CommandHandler`…), and so is `execute()` when the class name carries the verb.
- Absence contracts (`OrThrow`/`OrNull`, nullable, Option): follow the repository's convention; ask if there is none.

## Duplication

Duplication is about knowledge, not shape. Mutualise only when you are sure it is the same knowledge: the copies would change together, dispatch on the same source of truth, or carry the same domain name once extracted. When in doubt, don't; list it instead, since a wrong abstraction costs more than duplication. Within a file, extract; across files, propose.

## Details

- **Named arguments** for literals, consecutive same-type parameters and constructor arguments of entities, value objects and DTOs. Never when the variable already carries the parameter's name; that rule wins.
- **Magic numbers** become constants, unless self-explanatory in context (bounds of an invariant encapsulated in a value object or entity). A number with a unit becomes a value object.
- **Comments**: every comment that says *what* goes (a formula becomes a method, a unit becomes a value object, or simply delete it). Keep *why* comments and type annotations used by tooling.

## Scope of changes

- New code: apply every rule.
- Existing code: bring the method you touch in line, then list the other violations and offer to fix them. Do not over-refactor: a change that reveals nothing more of the domain is churn.

## Final check

1. Did every change reduce interpretation, with nothing added just to satisfy a rule?
2. Is each responsibility on its owner (intrinsic) or near its use case (policy), without feeding a god object?
3. Are names specific, unabbreviated, never single-letter, free of banned words and invented `isNotX()`?
4. Are "what" comments gone and magic numbers named unless self-explanatory?
5. Did you stay in scope, list remaining violations, and propose anything macro or shared?
