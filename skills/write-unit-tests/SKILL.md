---
name: write-unit-tests
description: Proactively apply when writing or reviewing unit tests. Triggers on unit test, TDD, AAA, test doubles, mocks, fakes, test smells, and DSL-driven testing. Use when adding tests, fixing brittle suites, or guiding agents toward maintainable, refactor-safe tests. Prefer a business DSL by default for new code, with brownfield pragmatism when migration cost is too high.
---

# Write Unit Tests

Write unit tests that catch regressions, stay fast, and survive refactoring. For new code, default to **DSL-driven testing** with an in-memory or domain-level driver, then apply classic-style unit-test rules inside that structure. Fall back to more direct tests mainly in brownfield areas where introducing a DSL would create too much churn for too little payoff.

In this skill, **classic style** means the **Chicago school**: prefer observable outcomes over interaction-heavy verification.

## Default Stance

For **new code**, write unit tests this way:

1. **Business-facing test case or scenario**
2. **Small business DSL that expresses domain intent**
3. **Fast in-memory/domain driver beneath it**
4. **Classic-style assertions on observable outcomes**

This is intentionally aligned with [`dsl-driven-testing`](../dsl-driven-testing/SKILL.md), which should be the default testing architecture by a wide margin.

### Main exception

Use a more direct style mainly in **brownfield** code when all of these are true:

- introducing the DSL would force broad rewrites,
- team adoption cost is high,
- and the near-term payoff is not worth the disruption.

Even in brownfield code, prefer **incremental DSL adoption** around new work, painful tests, and flaky areas rather than copying the old style forever.

## When to Use (and When NOT to)

| Use When | Skip When |
|----------|-----------|
| Adding tests for business logic in new or evolving code | Verifying full system wiring across processes |
| Expressing scenarios in business language with a DSL | Needing confidence in DB/network/framework integration |
| Protecting complex branches or edge cases | Writing broad end-to-end scenarios |
| Replacing brittle mock-heavy tests | Testing trivial getters/setters with no real risk |
| Reviewing AI-generated or over-mocked tests | Brownfield areas where DSL migration cost clearly outweighs the benefit right now |

**Goal:** maximize regression protection **without** coupling tests to implementation details.

## Core Defaults

1. **Default to DSL-driven testing for new code.** Prefer business-language scenarios over framework-shaped tests.
2. **Inside that DSL, prefer classic-style (Chicago school) tests.** Test observable behavior, not internal choreography.
3. **Use a fast in-memory or domain driver first.** Multiple drivers are optional, not required.
4. **Use real in-memory collaborators when they are deterministic and fast.**
5. **Prefer fakes over mocks** for external boundaries you must isolate.
6. **Use mocks only as a last resort** for outbound commands/side effects you cannot observe otherwise.
7. **Structure every test with Arrange, Act, Assert**, even when the scenario is wrapped in a business-language layer.
8. **Treat coverage as a diagnostic, not the goal.**
9. **Try to break the behavior before confirming it.** Look for the input, error, or boundary that makes the code lie.

## Angry Tests: Try to Break the Code

A test earns its place by **finding a defect**. A suite you can never make red proves nothing.

For each important behavior, hunt the input that makes the code lie, the missing rule, the realistic failure. Check that absence, unknown data, unavailability, and invalid input stay distinguishable whenever the spec says they differ.

```python
# ❌ Confirms the author's intuition
def test_returns_products():
    assert search("chair").items == [chair]

# ✅ Attacks the assumption: no match and backend down are different truths
def test_unknown_query_returns_empty_result_without_error():
    assert search("zzz").items == []

def test_unavailable_catalog_fails_explicitly_instead_of_looking_empty():
    with pytest.raises(CatalogUnavailable):
        search("chair", catalog=UnavailableCatalog())
```

### ZOMBIES: choosing which scenarios to write

```text
Z — Zero        : absence, empty input, no result
O — One         : minimal case with a single element
M — Many        : several elements, repetition, volume
B — Boundary    : thresholds, limits, extreme formats
I — Interface   : public contract or external boundary
E — Exceptional : error, timeout, unavailability, invalid input
S — Simple      : the minimal nominal scenario
```

Keep only the dimensions that carry risk: this is a search grid, not a quota. `spec-driven-development` attacks the spec with the same grid, so reuse its conclusions instead of re-deriving them.

### Business edge cases are cheap, technical ones are not

A business edge case costs one test: empty cart, expired membership, duplicate name, a quantity sitting on the threshold. Be liberal with those.

A technical edge case usually costs a **mechanism**: a retry, an optimistic-lock check, a compensating write, a `try/except` that swallows. That is an architecture decision with a permanent cost, and a test demanding it freezes it before anyone chose it.

Before writing a test that forces one, answer three questions:

1. Does this failure actually happen in this system, or is it hypothetical?
2. Does the spec say what the business wants when it happens?
3. Is the honest answer "fail loudly and let the caller decide"?

No answer to (2) means you found a question for the spec or the design, not a test to write. Raise it. Until it is decided, let the error propagate: a `try/except` that hides a failure is worse than the failure.

```python
# ❌ Invents a retry policy nobody specified, and freezes it in the suite
def test_retries_three_times_when_the_catalog_times_out(): ...

# ✅ Pins the decided business outcome; the mechanism stays free to change
def test_catalog_timeout_surfaces_as_unavailable_to_the_caller(): ...
```

### Hostile strategies

Pick one per risky area, not all of them every time.

| Strategy | What it exposes |
|----------|-----------------|
| Irregular values (empty, blanks, unicode, very long, extreme numbers) | Hidden assumptions about shape and size |
| Different data per test, seeded randomization | Tests that only pass on their one fixture |
| Repeating a scenario | Residual state, leaks, order dependence |
| Concurrency, non-deterministic ordering | Races and shared mutable state |
| Degraded boundary (timeout, unavailable, partial response) | Error paths nobody designed |
| Volume, quotas, descriptors, memory | Resource handling invisible in memory |

### Fast and deep

A second axis over unit/integration: it sorts tests by **feedback latency and detection depth**, not by label.

| Dimension | Fast | Deep |
|-----------|------|------|
| Purpose | immediate feedback, localization | integration / environment defects |
| Expected cost | very low | accepted as higher |
| Reality used | controlled, fakes allowed | real or realistic resources |
| Mostly run by | the developer while coding | CI / build / release |
| Typical failure | a local contract broke | an interaction, resource, or system invariant broke |

An in-memory test can prove a function reads a stream and never reveal a file-descriptor leak; opening many real files does. **The cost is sometimes exactly what lets the test see the bug.** Default to the fast loop; go deep only where the real boundary carries the risk.

### Bug, red, fix

```text
observed bug -> minimal reproduction -> RED test -> locate -> fix -> suite GREEN
```

Never fix first: the red test separates the defect's existence from your explanation of it, and leaves a permanent guard. When a deep test reddens, add fast tests around the suspect components until the failure is local instead of reaching for a debugger.

### Make the red diagnostic

A red nobody can act on is half a test: name it like a sentence, make the failure message state the expected business outcome, and keep the local reproduction to one command.

### Angry, not dogmatic

The posture has failure modes of its own. Hold these limits:

- **Overlap** is worth it only when the two tests detect through **independent** paths; two copies of one scenario are pure cost.
- **Flakiness** is acceptable only for a modeled probabilistic phenomenon, with repetition, a policy, and an owner. Otherwise it is a defect of the suite.
- **One assertion per test** is a readability heuristic, not a law; cohesive assertions about one result can improve the diagnostic.
- What matters about fixtures is **state independence**, not a syntactic ban on shared setup.
- Do not requalify every maintainability imperfection as a bug: keep impact, risk, and debt distinct.
- Angry is not random, exhaustive, or slow by default: catch important defects early, and keep a suite the team still runs.

### Beyond unit tests

Add one only when its trigger is present, never as a standing dependency.

| Technique | The angry question | Trigger |
|-----------|--------------------|---------|
| Property-based | Which classes of input break an invariant? | a strong, generic invariant |
| Fuzzing | Which unexpected inputs cause crashes or incoherent state? | parsing, untrusted input |
| Mutation testing | Does the suite notice a deliberately injected fault? | a critical component whose sensitivity you doubt |
| Performance / load | Does the system stay correct under hostile cost or load? | a real latency or volume constraint |
| Architecture / static analysis | Can the build forbid a structure or rule violation? | a boundary worth protecting mechanically |

The reasoning behind all of this, and the parts of the doctrine deliberately not adopted, live in [references/ANGRY-TESTS.md](references/ANGRY-TESTS.md).

## Quick Decision Trees

### "What should I unit test first?"

```
What has the highest payoff?
├─ Important business behavior in new code    → Start with a small business DSL
├─ Complex business rules or branching     → Test first
├─ Code with frequent regressions          → Test first
├─ Public behavior used by many callers    → Test first
├─ Simple pass-through glue                → Lower priority
└─ Trivial code with no real risk          → Maybe skip
```

### "Should I write the test directly or via a DSL?"

```
What context am I in?
├─ New feature or greenfield area            → Use a DSL by default
├─ Long-lived test suite                     → Use a DSL by default
├─ Need scenarios to read like specifications → Use a DSL by default
├─ Brownfield code, high migration cost      → Be pragmatic; direct tests may be fine
└─ Tiny trivial logic with no useful language → Direct test is acceptable
```

### "Should I use a real dependency, fake, or mock?"

```
What kind of dependency is it?
├─ Pure, deterministic, in-memory          → Use the real dependency
├─ External but easy to model in memory    → Use a fake
├─ Need canned answers only                → Use a stub
├─ Need to observe calls after execution   → Use a spy
└─ Need strict interaction verification     → Mock as last resort
```

### "What kind of assertion should I write?"

```
What do I care about?
├─ Final state / returned value            → State assertion
├─ Error condition or boundary case        → Exception / error assertion
├─ Side effect recorded in fake/spy        → Verify observable effect
└─ Internal call order or private methods  → Usually don't test that
```

### "Should this be a fast or a deep test?"

```
What is the risk I am chasing?
├─ Business rule, branch, or contract       → Fast, with a fake at the boundary
├─ Serialization, parsing, real I/O shape   → Deep, no fake for that boundary
├─ Resource leak, volume, concurrency       → Deep, and let it cost what it costs
├─ Reproducing an observed bug              → Fast if it localizes, deep if it needs reality
└─ Both would catch it                      → Fast, and stop there
```

### "Why is this test fragile?"

```
Why does it break too often?
├─ Fails after harmless refactor           → Coupled to implementation
├─ Mocks every collaborator                → Over-mocked, too isolated
├─ Huge setup hides the intent             → Obscure Arrange phase
├─ Many unrelated assertions               → Assertion Roulette
└─ Depends on clock, DB, file system       → Not a true unit test
```

## The Unit-Test Workflow

1. **Pick one behavior** with business value.
2. **Name it in business language first.** Create a small business DSL before writing low-level test code.
3. **Choose the smallest meaningful scope** that still exercises real logic.
4. **Use a fast in-memory or domain driver** as the default execution path.
5. **Keep collaborators real** if they are fast, deterministic, and in-memory.
6. **Replace only unstable boundaries** with a fake, stub, spy, or rare mock.
7. **Write one clear Act step.**
8. **Assert on outcomes visible to the caller or domain.**
9. **Refactor the test for readability** without changing intent.
10. **Check for smells** before adding more tests.

## DSL-First Unit Testing

When the code is new or expected to live a long time, prefer this stack:

```text
Test case / scenario → small business DSL → in-memory driver → system under test
```

Why this should be the default:

- scenarios read like executable specifications,
- tests stay decoupled from HTTP/UI/framework details,
- the same language can often scale beyond unit level later,
- and teams avoid locking their test suite to implementation noise.

If you need the full architecture and migration guidance, use [`dsl-driven-testing`](../dsl-driven-testing/SKILL.md) together with this skill.

## AAA / Four-Phase Structure

```text
Arrange → Act → Assert → (Optional) Teardown
```

- **Arrange:** only the data and collaborators needed for this scenario
- **Act:** ideally one triggering call
- **Assert:** verify the business-relevant outcome
- **Teardown:** only when a shared resource truly needs cleanup

## Test Double Selection

| Double | Use It For | Avoid When |
|--------|------------|------------|
| **Dummy** | Satisfying an unused parameter | You need real behavior |
| **Stub** | Returning fixed answers | You need to inspect effects |
| **Spy** | Recording calls or emitted messages | You want strict pre-programmed expectations |
| **Mock** | Rare verification of unavoidable outbound commands | You are testing queries, internal calls, or domain collaboration |
| **Fake** | Lightweight in-memory implementation of an external contract | The fake would become more complex than the real thing |

**Default order:** real dependency → fake → stub/spy → mock.

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| **Over-mocking** | Tests freeze internal design and break on refactors | Replace mocks with real collaborators or fakes |
| **Assertion Roulette** | Hard to know why the test failed | Use fewer, sharper assertions with clear intent |
| **Obscure Test** | Setup overwhelms the scenario | Use builders/helpers and remove irrelevant setup |
| **Conjoined Twins** | Hidden shared state makes tests flaky | Isolate state and keep everything in-memory |
| **No real assertion** | Coverage rises but confidence does not | Assert outcomes that matter |
| **Testing private methods** | Locks tests to implementation details | Test through public behavior |
| **Multiple behaviors in one test** | Failures are ambiguous | Split into one scenario per test |
| **Skipping a useful DSL in new code** | Tests speak framework details instead of business intent | Add a small scenario/DSL layer first |
| **Happy-path-only suite** | Confirms the intuition that produced the code; the failure modes stay untested | Attack the behavior with the relevant ZOMBIES dimensions |
| **Decorative RED** | A test that fails for the wrong reason proves nothing | Check the failure message says what you expected before making it green |
| **Faking the boundary under test** | The deep test verifies the double, not the reality it exists for | Keep that boundary real; fake the ones you are not testing |
| **Unexplained flaky test** | Reddens without action, and the team stops trusting the suite | Model it with repetition and an owner, or fix the suite |
| **Fix before reproduction** | The wrong mechanism gets repaired, with no lasting guard | Write the failing test first |
| **Test-driven defensive code** | Retries, locks, and swallowed exceptions nobody decided, made permanent by a test | Check the spec decided the outcome; otherwise raise the question |

## The Four Quality Pillars

Evaluate each test against these four properties:

| Pillar | Ask |
|--------|-----|
| **Regression protection** | Would this catch a real bug? |
| **Resistance to refactoring** | Will this stay green after harmless internal changes? |
| **Fast feedback** | Will developers run it constantly? |
| **Maintainability** | Can another developer understand and update it easily? |

If a test scores poorly on **refactoring resistance**, redesign it before adding more assertions.

**Fast feedback** applies to the loop, not to every test: a deep test buys detection with time and belongs in the build's suite, not the one you run on every save.

## Review Checklist

- Does the test describe a business-relevant behavior?
- In new code, should this be expressed through a small business DSL first?
- Is the Act phase a single meaningful trigger?
- Are assertions about outcomes instead of internals?
- Could a real in-memory dependency replace a mock?
- Would the test survive a harmless refactor?
- Is the setup shorter than the behavior being tested?
- Is the test deterministic and fast locally?
- Does it avoid using coverage as the definition of quality?
- Does the test actually try to falsify an assumption, or only confirm one?
- Are the ZOMBIES dimensions that carry risk covered, and the others left out?
- If the test forces a technical mechanism, did someone actually decide that mechanism?
- Would the red be diagnostic: clear name, informative message, one-command reproduction?
- If it overlaps another test, does it detect through an independent path?
- For a bugfix, did the reproduction come before the fix?

## AI-Specific Guidance

Coding agents default to mocking everything. Three instructions fix most of it:

1. Ask for a small business DSL first, then a fast in-memory driver.
2. Forbid verification of private calls and call order unless strictly necessary.
3. Require a written reason for each test double, and for skipping the DSL.

## Reference Documentation

| File | Purpose |
|------|---------|
| [references/CHEATSHEET.md](references/CHEATSHEET.md) | Quick checklist for daily use |
| [references/TEST-DOUBLES.md](references/TEST-DOUBLES.md) | Choosing between real deps, fakes, spies, and mocks |
| [references/ANGRY-TESTS.md](references/ANGRY-TESTS.md) | Why the angry posture works, and which parts of the doctrine to refuse |
| [examples/SERVICE-EXAMPLE.md](examples/SERVICE-EXAMPLE.md) | Example of a DSL-first, classic-style unit test |
| [../dsl-driven-testing/SKILL.md](../dsl-driven-testing/SKILL.md) | Default test architecture for long-lived code |

## Sources

### Primary Source
- User-provided literature review on unit testing — local reference used to synthesize the guidance in this skill

### Key Authors and Concepts Reflected
- Gerard Meszaros — *xUnit Test Patterns* (AAA / Four-Phase, test doubles, test smells)
- Vladimir Khorikov — *Unit Testing: Principles, Practices, and Patterns* (four quality pillars, classic style, refactoring resistance)
- Martin Fowler — test doubles taxonomy and mocking guidance
- Yegor Bugayenko, *Angry Tests* (testing as defect search, hostile inputs, fast/deep, fakes over mock frameworks, regression-first). See [references/ANGRY-TESTS.md](references/ANGRY-TESTS.md)
- Empirical findings summarized in the review: TDD/testing improves quality, excessive mocking increases fragility, and code coverage is useful as a signal but weak as a target
