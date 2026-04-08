---
name: dsl-driven-testing
description: Proactively apply when writing acceptance tests, designing test architecture, or separating test concerns. Triggers on DSL tests, protocol drivers, four-layer testing, ATDD, subsecond TDD, acceptance tests, test DSL, business language tests, Scenario builder, Screenplay pattern. Use when creating test suites that must run at multiple levels (unit/integration/E2E), structuring test layers, or when tests are too coupled to infrastructure. Decouples test scenarios from implementation so the same test runs in milliseconds (in-memory) or minutes (real browser).
---

# DSL-Driven Testing — Four Layer Architecture

> Known as: **SimpleDSL**, **Four Layer Testing**, **ATDD with Protocol Drivers**, **Subsecond TDD**
>
> Core idea: write tests in a **pure business DSL**, then plug in different **drivers** (in-memory, HTTP, UI…) — the same scenarios run in milliseconds in-memory and as full E2E tests.

---

## When to Use / Skip

| Use When | Skip When |
|----------|-----------|
| Test suite needs multiple execution levels | Single-level test suite (unit only) |
| Tests are coupled to UI/HTTP/DB | Simple CRUD with no business logic |
| Want same scenario to run fast AND realistically | Throwaway script or prototype |
| Building long-lived system with evolving infra | Quick spike |
| Acceptance tests break on every UI change | Tests already run in < 1s everywhere |

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

**Substitution principle**: same test case + same DSL can connect to any driver. The DSL is the stable interface; drivers are swappable.

---

## Quick Decision Trees

### "Which layer does this code belong to?"

```
Where does this code go?
├─ Describes WHAT the system does (business terms)   → Test Case (Layer 1)
├─ Translates domain vocab to driver calls           → DSL (Layer 2)
├─ Knows about HTTP, Selenium, DB, etc.              → Protocol Driver (Layer 3)
└─ Is the actual application                         → SUT (Layer 4)
```

### "Which driver should I use?"

```
Test goal?
├─ Validate business logic — fast                    → In-memory / Domain driver
├─ Validate HTTP contract / API                      → HTTP driver
├─ Validate UI rendering / user flows                → WebDriver / Playwright
└─ Production smoke test                             → Full E2E driver
```

### "How should I parametrize my DSL?"

```
Language / ecosystem?
├─ Java / need extreme readability                   → "name: value" strings (LMAX style)
├─ Python / TypeScript — type safety matters         → Typed kwargs / typed params
├─ Gherkin / Cucumber                                → Step definitions delegate to DSL object
└─ Need default values everywhere                    → Optional params with sensible defaults
```

### "How do I select the driver at runtime?"

```
Approach?
├─ CI matrix or explicit command                     → Environment variables (codebreaker-js style)
├─ pytest fixture                                    → conftest.py --driver option
├─ Auto-detect from env (creds available?)           → .auto() method on Scenario
└─ Parametrize (same test, all drivers)              → @pytest.mark.parametrize / JUnit params
```

---

## Layer 1: Test Cases

Test cases are **executable specifications** — they describe *what* the system does, never *how*.

### Rules
- Written from the perspective of an external user
- No variables, no control flow (no if/for/while)
- No technical terms (HTTP, SQL, DOM, CSS selectors…)
- Interact with the SUT through **public interfaces only** (no backdoors)
- Express only what's relevant for that specific scenario

```java
// ✅ Business language — no tech references
@Test
public void shouldReceiveCancellationMessageAfterCancellingAnOrder() {
    publicAPI.placeOrder("FTSE100", "side: buy", "quantity: 10", "price: 5000", "order: order1");
    publicAPI.cancelOrder("FTSE100", "order1");
    publicAPI.waitForOrderState("order: order1", "cancelledQuantity: 10");
}

// ❌ Coupled to implementation — cannot run in-memory
@Test
public void shouldReceiveCancellationMessage() {
    driver.findElement(By.id("quantity")).sendKeys("10");
    driver.findElement(By.id("submit-order")).click();
    // ...
}
```

---

## Layer 2: DSL

The DSL is the **lingua franca** between tests and infrastructure.

### DSL Design Rules (Dave Farley)
1. **Business vocabulary only** — never `clickButton()`, always `placeOrder()`
2. **Optional params everywhere** — tests express only what's relevant for the case
3. **Sensible defaults** — default credit card, default user, default item
4. **Encode common setup** — `createUser`, `populateBaseData` are DSL responsibilities
5. **No Java/Python variables in tests** — use aliases stored in `TestContext`
6. **No computed expressions** — values are declared, not calculated

### LMAX-style DSL (Java — string params)

```java
// DSL method — optional params with defaults
public void checkOut(String... args) {
    Params params = new Params(args);
    String item  = params.optional("item",  "Continuous Delivery");
    String price = params.optional("price", "£10.00");
    Card   card  = parseCard(params.optional("card", "1234 5678 9101 0001 12/23 007"));

    driver.checkOut(item, price, card);  // delegate to driver
}

// Test calls only what matters
shopping.checkOut("item: Continuous Delivery");  // price and card use defaults
```

### Typed DSL (Python — kwargs)

```python
# ✅ Type-safe, IDE-friendly, refactorable
flow.when_searching_offres(
    sort=Sort.DATE_CREATION,
    type_contrat=CodeTypeContrat.CDI,
    departement="75",
)

# ❌ LMAX style in Python — works but loses type checking
flow.when_searching_offres("sort: DATE_CREATION", "typeContrat: CDI")
```

### TestContext — shared whiteboard between DSL components

```java
public class DslTestCase {
    private final SystemDriver systemDriver = new SystemDriver();
    private final TestContext  testContext  = new TestContext();

    // DSL fields exposed to tests
    protected final AdminAPI        adminAPI        = new AdminAPI(systemDriver, testContext);
    protected final PublicAPI       publicAPI       = new PublicAPI(systemDriver, testContext);
    protected final TradingUI       tradingUI       = new TradingUI(systemDriver, testContext);
}
// TestContext maps aliases → real IDs: "Bob" → "Bob-83749234"
```

---

## Layer 3: Protocol Drivers

Protocol Drivers are **adapters** in hexagonal architecture terms. Each driver:
- Implements the interface the DSL expects
- Encodes real interactions with the SUT (clicks, HTTP calls, in-memory calls…)
- Isolates **all** infrastructure knowledge

### Same test — two drivers

```java
// WebDriver implementation
@Override
public void assertListedInShoppingBasket(String item) {
    gotoPage("https://www.amazon.co.uk/gp/cart/view.html");
    List<WebElement> found = driver().findElements(
        By.xpath("//span[contains(., \"" + item + "\")]")
    );
    assertEquals(1, found.size());
}

// In-memory implementation — same interface, zero network
@Override
public void assertListedInShoppingBasket(String item) {
    assertTrue(basket.contains(item));
}
```

The test case doesn't change. Only the driver changes.

### Driver selection (JavaScript — env vars)

```javascript
// World.js
function getActor() {
    switch (process.env.ACTOR) {
        case 'DirectActor':    return new DirectActor(makeCodebreaker());
        case 'DomActor':       return new DomActor(makeCodebreaker());
        case 'WebDriverActor': return new WebDriverActor(makeCodebreaker());
    }
}
```

```bash
# Same scenario, 4 execution profiles
ACTOR=DirectActor    API=Codebreaker     cucumber-js  # in-memory, < 10ms
ACTOR=DirectActor    API=HttpCodebreaker cucumber-js  # HTTP, < 500ms
ACTOR=WebDriverActor API=HttpCodebreaker cucumber-js  # real browser, seconds
```

### Driver selection (Python — Scenario builder)

```python
@dataclass
class Scenario:
    def unit(self) -> "Scenario":
        self._http_client = FakeHttpClient()   # no network
        return self

    def integration(self) -> "Scenario":
        self._http_client = HttpClient()       # real HTTP
        return self

    def e2e(self) -> "Scenario":
        self._client = RealClient(os.environ["CLIENT_ID"], os.environ["CLIENT_SECRET"])
        return self

    def auto(self) -> "Scenario":
        """Pick highest available driver from environment."""
        if os.environ.get("CLIENT_ID"):
            return self.e2e()
        if os.environ.get("INTEGRATION"):
            return self.integration()
        return self.unit()
```

### Lazy driver initialization

```java
public class SystemDriver {
    private UIDriver uiDriver;  // null until first access

    public UIDriver getUIDriver() {
        if (uiDriver == null) {
            uiDriver = new UIDriver();  // Selenium starts HERE, only if needed
        }
        return uiDriver;
    }
}
```

---

## Key Patterns

| Pattern | Problem Solved | Example |
|---------|---------------|---------|
| **Alias** | Avoid technical IDs in tests | `"Bob"` → `"Bob-83749234"` in TestContext |
| **Keywords** | Express presence/absence without values | `"status: PRESENT"`, `"fee: ABSENT"` |
| **RememberAs** | Store results without Java variables | `"rememberAs: myOrder"` → `cancelOrder("order: myOrder")` |
| **Parameter Combining** | Group related params | `"bid: 10@49.0"` instead of 2 separate params |
| **Time Machine** | Test time-based logic without sleep | `dsl.waitUntil("marketOpen")` with simulated time |
| **Fake over Mock** | Realistic test doubles | Handcrafted class implementing the same interface, not `MagicMock` |

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| Tech terms in test cases | `fill('#qty', '10')` — can't run in-memory | `trading.placeOrder("quantity: 10")` |
| Page Objects only | Browser-coupled, no in-memory path | Add a DSL layer above Page Objects |
| Backdoor DB setup | `INSERT INTO users…` — bypasses the system | Use DSL: `registrationAPI.createUser("Bob")` |
| Shared test data | Tests depend on global state | Each test creates what it needs in `@Before` |
| Logic in step definitions | `await page.fill('#q', '10')` in When | Delegate to DSL object |
| Single driver type | E2E-only suite — slow and fragile | Add in-memory driver first |
| Mock frameworks over fakes | `MagicMock` hides behavior | Write a real `FakeHttpClient` class with state |

---

## Driver Speed Reference

| Configuration | Driver | External deps | Speed |
|---|---|---|---|
| Unit / in-memory | Direct/Domain | Fakes/Stubs | Milliseconds |
| Integration API | HTTP client | Stubs | Seconds |
| E2E UI | WebDriver/Playwright | Real or stubs | Minutes |
| Production smoke | WebDriver | Real | Minutes |

---

## Growing a DSL — Step by Step

1. Write 2-3 tests covering the most important behaviors
2. Invent the language you need — don't worry about implementation yet
3. Implement the minimal DSL to pass these tests
4. Start with an **in-memory driver** first
5. Add a real driver once the domain is stable
6. Grow the DSL organically as new acceptance criteria arrive

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
| [references/PATTERNS.md](references/PATTERNS.md) | Advanced patterns (LMAX, codebreaker-js, Screenplay) |
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
