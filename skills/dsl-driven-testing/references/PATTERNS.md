# Advanced DSL Patterns

## LMAX Exchange Patterns (Java)

LMAX Exchange runs ~6000 acceptance tests on 30-40 machines in ~35 minutes. Their patterns are battle-tested on asynchronous, high-performance trading systems.

### Alias Pattern

Avoid technical IDs leaking into test cases. The DSL generates unique IDs internally and maps them to human-readable aliases stored in `TestContext`.

```java
// ❌ Unreadable — ID is an implementation detail
registrationAPI.createUser("user-83749234");
publicAPI.login("user-83749234");

// ✅ Alias — DSL generates "Bob-83749234" internally
registrationAPI.createUser("Bob");
publicAPI.login("Bob");
// TestContext maps: "Bob" → "Bob-83749234"
```

### Keywords Pattern

Express presence/absence without caring about exact values.

```java
adminAPI.checkInstrumentStatus("instrument: FTSE100", "status: PRESENT");
adminAPI.checkFee("instrument: FTSE100", "fee: ABSENT");
adminAPI.checkOrderCount("orderCount: ANY");     // exists, value irrelevant
adminAPI.checkOrderCount("orderCount: NONE");    // must be empty
```

### RememberAs Pattern

Store results across DSL calls without Java variables in the test.

```java
// Store the created order ID under alias "myOrder"
publicAPI.placeOrder("FTSE100", "side: buy", "quantity: 10",
                     "price: 5000", "rememberAs: myOrder");

// Reference it later — no variable needed in the test
publicAPI.cancelOrder("FTSE100", "order: myOrder");
publicAPI.waitForOrderState("order: myOrder", "cancelledQuantity: 10");
```

### Parameter Combining Pattern

Group related values to reduce noise:

```java
// ❌ Verbose — 4 params for one logical thing
fixAPI.placeMassOrder("bidQuantity: 10", "bidPrice: 38", "askQuantity: 10", "askPrice: 42");

// ✅ Combined — reads like a real order book
fixAPI.placeMassOrder("instrumentToClose", "bid: 10@38", "ask: 10@42");
```

### Time Machine Pattern

Test time-based functionality without `Thread.sleep()` or flaky timing.

```java
// Setup: enable simulated time
dsl.enableTimeMachine("timeTravelTo: <next weekday>");
dsl.createTimePoint("name: open",  "value: origin plus 1 weekdayCalendarOpenOffset");
dsl.createTimePoint("name: close", "value: origin plus 1 weekdayCalendarCloseOffset");

// Advance time deterministically
dsl.waitUntil("open");

// ... test actions ...

dsl.waitUntil("close");
publicAPI.waitForOrderBookStatus("instrumentToClose", "Closed");
```

### DSL Sub-domain Navigation (LMAX style)

For large systems, organize the DSL by sub-domain using nested objects:

```java
// Navigation through sub-domains
tradingUI.dealTicket.open("instrument: FTSE100");
tradingUI.allInstruments.list.waitForBestBidAndAskPrices(
    "instrument: FTSE100", "bid: 49.0", "ask: 51.0");
tradingUI.watchlist.list.checkInstrumentList("instrument: FTSE100");
```

---

## codebreaker-js Patterns (JavaScript)

Nat Pryce's reference implementation — same Gherkin scenario, 5 drivers.

### Step Definitions as Thin DSL Wrappers

```javascript
// Steps delegate immediately to the DSL actor — no logic here
Given('{actor} has started a game with the secret {string}', async function(actor, secret) {
    await this.synchronized();  // wait for all actors to see same state
    await actor.startGame(secret);
});

When('{actor} joins the game', async function(actor) {
    await this.synchronized();
    await actor.joinGame();
});

Then('{actor} must guess a {int}-letter word', async function(actor, letterCount) {
    const actual = await actor.getCurrentGameLetterCount();
    assert.equal(actual, letterCount);
});
```

### Synchronized State Between Actors

`this.synchronized()` ensures that two actors (Molly and Benny) see the same state:
- **In-memory**: synchronous, immediate
- **HTTP with SSE**: polls/waits for Server-Sent Events to arrive

```javascript
// World.js — synchronized implementation depends on driver
async synchronized() {
    if (this._pubSub instanceof MemoryPubSub) {
        return;  // already synchronous in-memory
    }
    await this._pubSub.waitForSync();  // wait for SSE message
}
```

### Adaptive PubSub

```javascript
function makeActorSub() {
    if (process.env.API === 'HttpCodebreaker') {
        const server = new WebServer(port);
        _stoppables.push(server);
        return new EventSourcePubSub(port);  // real SSE via HTTP
    }
    return new MemoryPubSub();               // in-memory, no network
}
```

---

## Python Scenario Builder Pattern

From [cmnemoi/france-travail-api](https://github.com/cmnemoi/france-travail-api).

### Key Differences from LMAX

| Aspect | LMAX (Java) | france-travail-api (Python) |
|--------|-------------|----------------------------|
| DSL params | `"name: value"` strings parsed at runtime | Typed kwargs — IDE autocompletion |
| Drivers | Separate classes injected via SystemDriver | Methods on the Scenario dataclass |
| Test doubles | Protocol Driver implementations | `FakeHttpClient` hand-crafted class |
| Assertions | Direct `assert` | Chained on Scenario + `expect()` DSL |
| Async | No | Full async support (`when_xxx_async`) |

### Fake over Mock

```python
# ✅ Hand-crafted Fake — real class with state, no magic
class FakeHttpClient:
    def __init__(self) -> None:
        self.responses: list[HTTPResponse] = []
        self.last_get_url: str | None = None

    def add_response(self, response: HTTPResponse) -> None:
        self.responses.append(response)

    def get(self, url: str, headers: dict | None = None) -> HTTPResponse:
        self.last_get_url = url       # record for assertions
        return self.responses.pop(0)  # return pre-programmed response

# ❌ Mock — magic, hides behavior, brittle
from unittest.mock import MagicMock
http_client = MagicMock()
http_client.get.return_value = MagicMock(status_code=200)
```

### Time Control Without Patching

```python
@dataclass
class Scenario:
    now: datetime.datetime = field(
        default_factory=lambda: datetime.datetime(2025, 12, 25, 10, 0, 0, tzinfo=datetime.UTC)
    )

# Token expiry test — deterministic, no datetime.now() patching
flow = Scenario(now=datetime.datetime(2026, 1, 1, tzinfo=datetime.UTC))
# DSL uses self.now internally instead of datetime.now()
```

### Single-Suite Driver Selection (conftest.py)

```python
# conftest.py
def pytest_addoption(parser):
    parser.addoption("--driver", default="unit",
                     choices=["unit", "integration", "e2e"])

@pytest.fixture
def scenario(request):
    driver = request.config.getoption("--driver")
    s = Scenario()
    return getattr(s, driver)()  # s.unit(), s.integration(), or s.e2e()
```

```bash
pytest                       # unit — FakeHttpClient, milliseconds
pytest --driver=integration  # real HTTP
pytest --driver=e2e          # full client with real credentials
```

### Parametrize Over Available Drivers

```python
# conftest.py — run same test on all available drivers
def available_drivers():
    drivers = ["unit"]
    if os.environ.get("INTEGRATION"):
        drivers.append("integration")
    if os.environ.get("CLIENT_ID"):
        drivers.append("e2e")
    return drivers

@pytest.fixture(params=available_drivers())
def scenario(request):
    s = Scenario()
    return getattr(s, request.param)()
```

---

## Hexagonal Architecture Alignment

The 4-layer testing model maps directly onto hexagonal architecture:

```
Driver Ports  (left/primary)  ← Test Driver, HTTP Driver, UI Driver
    │
    ▼
HEXAGON (business logic)
    │
    ▼
Driven Ports (right/secondary) → Real DB, External API, Stub/Mock
```

### Recommended Progression

1. **Test driver + driven stubs** → validates pure business logic
2. **Real driver (HTTP/UI) + driven stubs** → validates API/UI contract  
3. **Test driver + real driven adapters** → validates DB, external services
4. **Everything real** → full E2E smoke test

---

## Business DSL Anti-Patterns

These patterns indicate your DSL is too coupled to implementation:

### 1. Driver Internals in Assertions

```python
# ❌ Anti-pattern: checks URL (driver detail)
flow.then_last_get_url_contains("api/offres")

# ✅ Business DSL: checks domain outcome
flow.then_offers_found(count=5)
flow.then_first_offer_has(titre="Développeur")
```

### 2. HTTP-Specific Setup Methods

```python
# ❌ Anti-pattern: HTTP response setup leaks into DSL
flow.with_http_response(status=200, body={"resultats": [...]})

# ✅ Business DSL: domain entities directly
flow.given_offers(offers=[Offre(...), Offre(...)])
```

### 3. Test Accesses Driver State

```python
# ❌ Anti-pattern: test reads driver internals
assert flow._http_client.last_get_url == "https://api.example.com/..."

# ✅ Business DSL: all state hidden behind DSL methods
flow.then_order_confirmed()
flow.then_total_is(100.00)
```

### 4. Multiple Drivers Can't Share Tests

If your test case needs to change when switching from in-memory to HTTP driver, your DSL is implementation-coupled.

```python
# ❌ Anti-pattern: different assertions per driver
if driver == "http":
    assert response.status_code == 200
else:
    assert response.body == expected

# ✅ Business DSL: same test, any driver
flow.then_offers_count(10)  # Works with all drivers
```

### 5. Over-Hiding Scenario Data

A DSL abstracts away **how** the SUT is reached (transport, fakes, serialization, wiring). It must not abstract away **which data** makes the scenario meaningful. Over-hiding turns a test into a shape with no content — readable, but no longer able to tell scenarios apart or say what actually failed.

```python
# ❌ Cheats — hides scenario data, not just mechanics.
# Cannot tell what was sent, what existed beforehand, what output is expected,
# or whether two such tests differ at all.
self.scenario.given_customer_in_paris()
await self.scenario.when_customer_searches()
self.scenario.then_results_are_correct()

# ✅ Hides transport/fake/serialization/wiring, keeps business data explicit
self.scenario.given_ice_trucks_near(
    place="San Francisco",
    ice_trucks=["Bay Area - San Francisco", "San Francisco"],
)
await self.scenario.when_customer_searches_near("San Francisco")
self.scenario.then_ice_trucks_are_presented(
    ["Bay Area - San Francisco", "San Francisco"]
)
```

- **Do not hide relevant inputs behind parameterless DSL methods.** If the value drives the behavior under test — place, product, price, status, quantity, a returned list, a business rule — it is a parameter, not a default baked into a method name.
- **Do not hide expected outputs behind generic assertions** such as `then_result_is_correct()` or `then_results_are_correct()`. Assert the actual business outcome: `then_ice_trucks_are_presented([...])`, `then_order_status_is("CANCELLED")`.

The exception: hide a value when it is genuinely **incidental to the behavior under test**. `given_authenticated_customer()` may hide a JWT, a UUID, an expiry date — none of that is what the test is about. If in doubt, keep the data visible: an over-explicit test is easier to trim than an over-hidden one is to debug.

### Refactoring to Business DSL

| From (Implementation) | To (Business) |
|-----------------------|---------------|
| `.then_last_url_contains("...")` | `.then_request_made_for(resource)` |
| `.with_http_response(status, body)` | `.given_offers(offers)` or `.given_orders(orders)` |
| `.then_status_code_is(401)` | `.then_authentication_fails()` |
| Check `last_get_url` | `.then_search_performed_with(query)` |

The goal: **same test case should run identically across all drivers** — the only thing that changes is the driver implementation.

---

## Language-Specific DSL Mechanics

### LMAX-style DSL (Java — string params with defaults)

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

### Choosing a parametrization style

| Language / need | Style |
|---|---|
| Java / extreme readability | `"name: value"` strings (LMAX) |
| Python / TypeScript, type safety matters | Typed kwargs / typed params |
| Gherkin / Cucumber | Step definitions delegate to a DSL object |
| Defaults needed everywhere | Optional params with sensible defaults |

### TestContext — shared whiteboard between DSL components

```java
public class DslTestCase {
    private final SystemDriver systemDriver = new SystemDriver();
    private final TestContext  testContext  = new TestContext();

    // DSL fields exposed to tests
    protected final AdminAPI  adminAPI  = new AdminAPI(systemDriver, testContext);
    protected final PublicAPI publicAPI = new PublicAPI(systemDriver, testContext);
    protected final TradingUI tradingUI = new TradingUI(systemDriver, testContext);
}
// TestContext maps aliases → real IDs: "Bob" → "Bob-83749234"
```

---

## Protocol Driver Mechanics

### Same test — two drivers (Java)

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

The test case does not change. Only the driver changes.

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
# Same scenario, 3 execution profiles
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

Full walkthrough: [examples/PYTHON-SCENARIO.md](../examples/PYTHON-SCENARIO.md).

### How to select the driver at runtime

| Situation | Approach |
|---|---|
| Only one driver exists today | No runtime selection yet |
| CI matrix or explicit command | Environment variables (codebreaker-js style) |
| pytest | `conftest.py` with a `--driver` option |
| Auto-detect from env (creds available?) | `.auto()` method on `Scenario` |
| Same test on all drivers | `@pytest.mark.parametrize` / JUnit params |

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
