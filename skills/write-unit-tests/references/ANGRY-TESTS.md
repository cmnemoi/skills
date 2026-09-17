# Angry Tests

Yegor Bugayenko's testing philosophy, reconstructed. [The main skill](../SKILL.md) carries the rules and the tables; this file carries the reasoning behind them and the parts of the doctrine deliberately refused.

## The inversion

Testing means executing a program **with the intent of finding errors** (Myers). Proving that it works selects examples that confirm your intuition; trying to show how it breaks pushes you toward boundaries, rare states, and unplanned interactions.

Hence the formula: **a passing test is a weak test**. A suite you can never make red gives no evidence of sensitivity to defects. A test's quality is its ability to fail when it should and to stay stable when it should not.

## The safety net

The function of automated tests is economic: they lower the expected cost of future errors (regressions, incidents, debugging, fear of refactoring), which is what makes change faster as well as safer.

So coverage is a proxy, never the goal, and a bug that reaches a user is a hole in the net. A test nobody runs automatically before integration guards nothing.

## Why reproduction comes first

A red test separates the *existence* of a defect from your *explanation* of it. Fixing first is how the wrong mechanism gets repaired and declared fixed because the incident stopped reproducing by hand.

The same logic drives diagnosis: when a deep test reddens, add fast tests around the suspect components until the failure is local. The test becomes an instrument of localization, not only a final guard.

## Why hostility works

If the system only passes because the test hands it exactly the environment it hopes for, the test confirms the design instead of challenging it. Irregular values, varied data, repetition, concurrency, and degraded boundaries all do the same job: remove the crutches and make implicit assumptions visible.

## Why fast/deep beats unit/integration

A test's label matters less than its cost and its reach. An in-memory test can prove a function reads a stream and never reveal a file-descriptor leak; a test that opens many real files can. **The slowness is sometimes exactly what lets the test see the bug**, which is why deep tests belong in the build instead of being optimized away.

## Test design and doubles

The prescriptions (short tests, one main assertion, sentence-like names, explicit failure messages, independent data, public interface only) all serve one goal: when it reddens, you should know what broke without reading the test.

On doubles, Bugayenko is not anti-double; he is against mock frameworks and against tests that end up verifying the mock's expectations instead of the software's behavior. His catalogue calls **Mockery** a test so saturated with doubles that the real unit is never exercised. The alternative is a fake as a first-class object: coherent, testable, sometimes shared. The reverse reflex is refused too: do not fake the file system, and do not let doubles replace reality everywhere, since the deep layer exists to check what doubles cannot guarantee.

## Why mutation testing fits

It stops asking "were the lines executed?" and starts asking "would the suite notice a fault?". That is a direct measure of the net's sensitivity, which is what coverage only gestures at. The same reasoning admits fuzzing, property-based testing, load tests, and architecture checks: anything that turns a quality property into a red signal belongs to the net.

## Tensions and limits

The corpus is provocative and, in places, self-contradictory. Adopt the **objective**, negotiate the **prescriptions**.

| Prescription | Limit kept here |
|--------------|-----------------|
| Overlapping tests are welcome; one fault may redden several | Only when the detection modes are **independent**. Two copies of one scenario are pure cost, and the same excerpts also recommend detecting redundant tests |
| Some flaky tests are acceptable | Only for a **modeled probabilistic phenomenon**, with repetition, a policy, and an owner. A CI that reddens without action destroys trust |
| One assertion per test | A forcing function for readability, not a law. Cohesive assertions about one result can improve the diagnostic |
| Ban `setUp`/`tearDown` and shared fixtures | What matters is **state independence**. A syntactic ban produces duplication or artificial test abstractions |
| Tests and production code in separate pull requests | Real point (nobody should loosen a test and change the code in one move), but the coordination cost is high and it leaves disabled tests in trunk. **Not adopted here** |
| Any maintainability imperfection is a "bug" | Makes quality visible, blurs prioritization. Keep impact, risk, and debt distinct |
| Fuzzing, mutation, property-based testing | Available techniques, never a mandatory dependency |
| A multi-thousand-line test class is fine, it is only a script container | Length is not structural by itself, but it usually signals a production unit that is too big |

**The strongest version** is an optimization objective: maximize the probability of detecting an important defect early, minimize the time to understand a red, and keep the suite fast and reliable enough that the team goes on using it.

## A practicable version

1. A very fast local loop: no useless I/O, explicit fakes where they buy speed.
2. A deep layer on the technical boundaries that matter: database, file system, broker, network, process, real parsing and serialization.
3. Every observed bug starts with a minimal automated reproduction, merged before the fix.
4. Every risky area gets at least one hostile strategy.
5. Periodically check that the suite actually detects faults: mutation testing on important components, analysis of escaped defects.
6. Every red is an exploitable diagnostic: precise name, informative assertion, simple local reproduction.
7. Every quality invariant stable enough to codify goes into CI.
8. Overlap when two tests protect one risk by independent paths; delete duplication that changes neither risk nor diagnostic.
9. Every flaky test gets an owner and a policy, or gets fixed.
10. Revise the rules from escaped defects: the suite is a product that learns from its failures.

## Sources

| Ref | Source | Contribution |
|-----|--------|--------------|
| [1] | `yegor256/at` — *Best Practices for Automated Testing* (official book excerpts) | test shape, hostile data, fast/deep, fakes, redundancy, fuzzing, mutation, architecture |
| [3] | *TDD Misbeliefs* (2019) | "a passing test is a weak test"; misuse of test doubles |
| [4] | *Automated Tests Are the Safety Net that Saves You* (2022) | the safety-net metaphor; a user-facing bug is a hole in the net |
| [5] | *The TDD That Works for Me* (2017) | bug to test to fix; economics of regression tests |
| [6] | *Fast Tests Help Humans, Deep Tests Help Servers* (2023) | the fast/deep axis; the file-descriptor leak example |
| [7] | *Well-Connected Test Suites Catch More Bugs* (2026) | deliberate redundancy as density of the net |
| [8] | *Single Statement Unit Tests* (2017) | declarative tests, one assertion, objects and matchers for setup |
| [9] | *On the Layout of Tests* (2023) | layout, failure messages, fakes as first-class objects |
| [12] | *Any Program Has an Unlimited Number of Bugs* (2017) | unbounded number of potential defects |
| [13] | *When Do You Stop Testing?* (2015) | testing as destructive search; the stopping question |
| [15] | *Does Code Review Involve Testing?* (2019) | the merge pipeline as an automatic barrier |
| [16] | *Fixing Broken Systems with Unit Tests* (Vol. 2) | diagnosing a deep failure by adding fast tests |
| [17] | *Unit Testing Anti-Patterns, Full List* (2018) | happy path, Mockery, Inspector, Giant, shared leftovers |
| [21] | *Wikipedia's Definition of a Software Bug Is Wrong* (2015) | non-functional defects, maintainability |

Synthesized from a critical review of these public sources, not from the book text.

## See also

- [Main skill](../SKILL.md): the rules, the ZOMBIES grid, and the hostile-strategy and fast/deep tables
- [Test doubles](TEST-DOUBLES.md)
- [Spec-driven development](../../spec-driven-development/SKILL.md): the adversarial spec review that precedes these tests
