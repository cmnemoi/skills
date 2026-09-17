---
name: dsl-driven-testing
description: Proactively apply when writing acceptance tests, designing test architecture, or separating test concerns. Triggers on DSL tests, protocol drivers, four-layer testing, ATDD, subsecond TDD, acceptance tests, test DSL, business language tests, Scenario builder, Screenplay pattern. Use when creating test suites that must run at multiple levels (unit/integration/E2E), or when you want tests to express business intent clearly even at a single level. Decouples test scenarios from implementation and turns tests into readable executable specifications.
---

# DSL-Driven Testing — Default Way to Write Tests

> Known as: **SimpleDSL**, **Four Layer Testing**, **ATDD with Protocol Drivers**, **Subsecond TDD**

Write tests in a **pure business DSL**, then plug in different **drivers** (in-memory, HTTP, UI...) when needed.

## Why a DSL at All

In production code, you do not scatter `if user.role_id == 3` across the codebase. You write `user.is_admin()`. You build abstractions that **name the business problem**, so the next reader grasps the intent instead of decoding the mechanism.

Test code is code. It deserves the same treatment.

| | Mechanism | Business problem |
|---|---|---|
| Production | `if user.role_id == 3` | `if user.is_admin()` |
| Test | `page.fill("#qty", "10")` | `trading.place_order(quantity=10)` |

The DSL is nothing more than that familiar abstraction layer, applied to tests. The whole four-layer architecture follows from this one idea:

- **Layer 1** (test cases) — the business problem, stated
- **Layer 2** (DSL) — the abstraction that names it
- **Layer 3** (drivers) — the mechanism, hidden

If you already model your domain in production code, you already accept this skill. Building abstractions in production while writing tests against raw selectors and URLs means holding your tests to a lower standard than your code. Tests are read more often than they are written, and they break more often than production code.

## Default Stance

Use this as the default way to write tests unless there is a concrete reason not to. It produces clearer executable specs, better design pressure, and lower maintenance cost than implementation-coupled tests.

- Use it for new features, bug fixes, and any suite expected to outlive a short spike
- Use it even when only one execution level exists today — **one driver is enough to start**. Multiple drivers are a payoff, not a prerequisite
- Fall back to direct tests mainly in **brownfield** areas where migration cost or churn clearly outweighs the benefit

| Use When | Skip / Relax When |
|----------|---------------------|
| Almost always for new product code | Brownfield area where retrofitting broadly is too expensive right now |
| Test suite needs multiple execution levels | Throwaway spike or short-lived prototype |
| Tests are coupled to UI/HTTP/DB | Truly trivial code where a DSL adds ceremony without clarity |
| Want the same scenario to run fast AND realistically | Team constraints make migration unrealistic for now |
| Want tests to read like executable specs | Simple CRUD with no business language to preserve |

---

## The 4-Layer Architecture

```
┌──────────────────────────────────────────────────┐
│  Test Case   Test Case   Test Case   Test Case   │  ← Layer 1: Executable specs
│ (business language — no tech references)         │    Written by devs, QA, or PO
└──────────────────────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────┐
│           Domain Specific Language (DSL)         │  ← Layer 2: Business vocabulary
│     Translates domain to driver calls            │    Stable interface between layers
└──────────────────────────────────────────────────┘
                         │
           ┌─────────────┼─────────────┐
           ▼             ▼             ▼
┌──────────────┐ ┌──────────────┐ ┌────────────────┐
│  Protocol    │ │  Protocol    │ │ External Sys.  │  ← Layer 3: Translators
│  Driver (UI) │ │  Driver (API)│ │     Stub       │    One driver per access channel
└──────────────┘ └──────────────┘ └────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────┐
│              System Under Test (SUT)             │  ← Layer 4: The real system
└──────────────────────────────────────────────────┘
```

**Substitution principle**: the same test case and the same DSL can connect to any driver. The DSL is the stable interface; drivers are swappable.

### Which layer does this code belong to?

```
├─ Describes WHAT the system does (business terms)   → Test Case (Layer 1)
├─ Translates domain vocab to driver calls           → DSL (Layer 2)
├─ Knows about HTTP, Selenium, DB, etc.              → Protocol Driver (Layer 3)
└─ Is the actual application                         → SUT (Layer 4)
```

### Which driver should I use?

```
├─ Only one useful driver today          → Use it directly; runtime selection can wait
├─ Validate business logic — fast        → In-memory / Domain driver
├─ Validate HTTP contract / API          → HTTP driver
├─ Validate UI rendering / user flows    → WebDriver / Playwright
└─ Production smoke test                 → Full E2E driver
```

| Configuration | Driver | External deps | Speed |
|---|---|---|---|
| Unit / in-memory | Direct/Domain | Fakes/Stubs | Milliseconds |
| Integration API | HTTP client | Stubs | Seconds |
| E2E UI | WebDriver/Playwright | Real or stubs | Minutes |
| Production smoke | WebDriver | Real | Minutes |

---

## Layer 1: Test Cases

Test cases are **executable specifications** — they describe *what* the system does, never *how*.

- Written from the perspective of an external user
- No variables, no control flow (no `if`/`for`/`while`)
- No technical terms (HTTP, SQL, DOM, CSS selectors…)
- Interact with the SUT through **public interfaces only** — no backdoors
- Express only what is relevant for that scenario

```python
# ✅ Business language — runs on any driver
def test_cancelling_an_order_releases_the_quantity():
    trading.place_order(symbol="FTSE100", side=Side.BUY, quantity=10, price=5000, alias="order1")
    trading.cancel_order(alias="order1")
    trading.then_order_is_cancelled(alias="order1", cancelled_quantity=10)

# ❌ Coupled to the browser — cannot run in-memory
def test_cancelling_an_order():
    page.fill("#quantity", "10")
    page.click("#submit-order")
```

---

## Layer 2: DSL

The DSL is the **lingua franca** between tests and infrastructure.

### Design rules (Dave Farley)

1. **Business vocabulary only** — never `click_button()`, always `place_order()`
2. **Optional params everywhere** — tests express only what the case needs
3. **Sensible defaults** — default credit card, default user, default item
4. **Encode common setup** — `create_user`, `populate_base_data` belong to the DSL
5. **No host-language variables in tests** — use aliases stored in a `TestContext`
6. **No computed expressions** — values are declared, not calculated

### Inputs and outputs must be business terms

> **Critical**: a true business DSL takes **inputs and outputs directly in business terms**. If your assertions reference URLs, HTTP status codes, or driver internals, the DSL is not business-focused enough.

| | Implementation-coupled DSL | True business DSL |
|-----------|----------------------------|-------------------|
| **Inputs** | `.with_http_response(status=200, body={...})` | `.given_offers(offers=[...])` |
| **Outputs** | `.then_last_get_url_contains("...")` | `.then_offers_found(count=N)` |
| **Assertions** | HTTP status, headers, URL params | Business outcomes, domain entities |
| **Coupling** | Tied to driver internals | Driver is an implementation detail |

This rule holds in brownfield too. Existing tests may check URLs; the **new** DSL methods you write must not.

Parametrization styles per language, and the `TestContext` pattern: see [references/PATTERNS.md](references/PATTERNS.md).

---

## Layer 3: Protocol Drivers

Protocol Drivers are **adapters** in hexagonal architecture terms. Each driver:

- Implements the interface the DSL expects
- Encodes real interactions with the SUT (clicks, HTTP calls, in-memory calls…)
- Isolates **all** infrastructure knowledge

Driver implementations, runtime driver selection, and lazy initialization: see [references/PATTERNS.md](references/PATTERNS.md).

---

## Key Patterns

| Pattern | Problem Solved | Example |
|---------|---------------|---------|
| **Alias** | Avoid technical IDs in tests | `"Bob"` → `"Bob-83749234"` in TestContext |
| **Keywords** | Express presence/absence without values | `"status: PRESENT"`, `"fee: ABSENT"` |
| **RememberAs** | Store results without host-language variables | `"rememberAs: myOrder"` → `cancel_order("order: myOrder")` |
| **Parameter Combining** | Group related params | `"bid: 10@49.0"` instead of 2 params |
| **Time Machine** | Test time-based logic without sleep | `dsl.wait_until("marketOpen")` with simulated time |
| **Fake over Mock** | Realistic test doubles | Hand-written class implementing the interface, not `MagicMock` |

Full code for each: [references/PATTERNS.md](references/PATTERNS.md).

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| Tech terms in test cases | `fill('#qty', '10')` — cannot run in-memory | `trading.place_order(quantity=10)` |
| Page Objects only | Browser-coupled, no in-memory path | Add a DSL layer above Page Objects |
| Backdoor DB setup | `INSERT INTO users…` bypasses the system | Use the DSL: `registration.create_user("Bob")` |
| Shared test data | Tests depend on global state | Each test creates what it needs |
| Logic in step definitions | `await page.fill('#q', '10')` in a `When` | Delegate to the DSL object |
| Single slow driver only | UI/E2E-only suite — slow and fragile | Keep the DSL, add an in-memory driver first |
| Mock frameworks over fakes | `MagicMock` hides behavior | Write a real `FakeHttpClient` with state |
| DSL exposes driver internals | Test checks `last_get_url` or status codes | Assert business outcomes: `then_offers_found()` |
| DSL input is driver-specific | `.with_http_response(...)` leaks HTTP | Use domain inputs: `.given_offers([...])` |
| DSL not reusable | Each new scenario needs new DSL methods | Cover the common business scenarios |

---

## Growing a DSL — Step by Step

1. Write 2-3 tests covering the most important behaviors
2. Invent the language you need — ignore implementation for now
3. Implement the minimal DSL to pass these tests
4. Start with a **single driver** giving the fastest useful feedback (often in-memory)
5. Add other drivers later, only if they buy confidence at another level
6. Grow the DSL as new acceptance criteria arrive

### Brownfield adoption rule

Prefer **incremental adoption** over heroic rewrites.

- Add the DSL around new behavior, flaky areas, or tests painful to read
- Do not rewrite a legacy suite for aesthetic consistency alone
- But for new tests in a long-lived area, bias strongly toward the DSL

### Ownership model

```
Test Cases   → Anyone (QA, BA, PO, Dev)
DSL + PDs    → Developers (own the plumbing)
```

> *"If a test breaks, a Dev should notice first. Devs own the DSL and Protocol Drivers."*
> — Dave Farley

---

## Reference Documentation

| File | Purpose |
|------|---------|
| [references/PATTERNS.md](references/PATTERNS.md) | Advanced patterns, per-language DSL mechanics, driver selection |
| [references/SCREENPLAY.md](references/SCREENPLAY.md) | Screenplay Pattern — OO approach to protocol drivers |
| [examples/PYTHON-SCENARIO.md](examples/PYTHON-SCENARIO.md) | Full Python example (france-travail-api style) |

---

## Sources

- **[ATDD How-to Guide](https://dojoconsortium.org/assets/ATDD%20-%20How%20to%20Guide.pdf)** — Dave Farley / Continuous Delivery Ltd. (2020)
- **[LMAX SimpleDSL Wiki](https://github.com/LMAX-Exchange/Simple-DSL/wiki)** — LMAX Exchange
- **[subsecondtdd/codebreaker-js](https://github.com/subsecondtdd/codebreaker-js)** — Nat Pryce
- **[Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/)** — Alistair Cockburn
- **[Serenity/JS Screenplay Pattern](https://serenity-js.org/handbook/design/screenplay-pattern/)** — TypeScript
- **[Growing Object-Oriented Software, Guided by Tests](https://growing-object-oriented-software.com/)** — Freeman & Pryce
