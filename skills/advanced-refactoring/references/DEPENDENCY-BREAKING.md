# Dependency-Breaking Techniques — Deep Dive

Detailed reference for breaking dependencies in legacy code, based on Michael Feathers' "Working Effectively with Legacy Code".

## The Core Problem

Legacy code is hard to test because:
- Classes create their own dependencies internally
- Dependencies are hidden in constructors
- Global state is accessed directly
- Static methods are called without control

## Sensing vs Separation

### Sensing
Break dependencies to **observe** values your code computes.

```java
// You want to know what the email service sends
class OrderConfirmation {
    private EmailService emailService;
    
    public void send(Order order) {
        // You want to verify this is called with correct params
        emailService.send(order.getEmail(), "Your order...");
    }
}
```

### Separation
Break dependencies to **get code into test harness**.

```java
// Can't even create the object because of hidden deps
class ReportGenerator {
    public ReportGenerator() {
        this.database = new ProductionDatabase();  // Can't mock!
        this.cache = RedisCache.connect("localhost");  // Can't intercept!
    }
}
```

---

## Technique Reference

### 1. Extract Interface

**When:** Class has multiple consumers, need to mock for testing.

**Steps:**
1. Create interface with methods you use
2. Rename original class (e.g., `ProductionXxx`)
3. Have original implement interface
4. Change consumers to depend on interface
5. Inject mock in tests

**Example (Java):**
```java
interface PaymentGateway {
    ChargeResult charge(Amount amount, Currency currency);
}

class StripePaymentGateway implements PaymentGateway { ... }

class OrderService {
    private PaymentGateway gateway;
    public OrderService(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}

// Test
class FakePaymentGateway implements PaymentGateway {
    public ChargeResult lastCharge;
    public ChargeResult charge(Amount a, Currency c) {
        this.lastCharge = new ChargeResult(a, c);
        return ChargeResult.success();
    }
}
```

---

### 2. Parameterize Constructor

**When:** Hidden dependency in constructor, can't replace.

**Steps:**
1. Add parameter for dependency
2. Use parameter instead of creating internally
3. Update all call sites
4. Old constructor can delegate to new with default

**Example:**
```java
// Before
class ReportGenerator {
    private Database db = new ProdDatabase();
    public ReportGenerator() { }
}

// After
class ReportGenerator {
    private Database db;
    public ReportGenerator(Database db) {
        this.db = db;
    }
}

// Backwards compatible
class ReportGenerator {
    public ReportGenerator() {
        this(new ProductionDatabase());
    }
}
```

---

### 3. Subclass and Override Method

**When:** Need to nullify behavior or return test value.

**Steps:**
1. Create test subclass
2. Override problematic method
3. Call from test

**Example:**
```java
// Production
class OrderProcessor {
    protected Money calculateTotal(Order o) {
        return PricingService.calculate(o);  // Can't mock
    }
}

// Test
class TestOrderProcessor extends OrderProcessor {
    protected Money calculateTotal(Order o) {
        return new Money(100, USD);  // Fixed value
    }
}
```

---

### 4. Extract and Override Call

**When:** Static method or global call blocks testing.

**Steps:**
1. Extract call to protected method
2. Subclass and override

```java
// Before
class Inventory {
    public void checkStock(Product p) {
        if (DateUtils.isBusinessDay()) {  // Static, can't control
            // check stock
        }
    }
}

// After: Extract to method
class Inventory {
    protected boolean isBusinessDay() {
        return DateUtils.isBusinessDay();
    }
    
    public void checkStock(Product p) {
        if (isBusinessDay()) {
            // check stock
        }
    }
}

// Test subclass
class TestInventory extends Inventory {
    protected boolean isBusinessDay() { return true; }
}
```

---

### 5. Introduce Instance Delegator

**When:** Class uses only static methods from utility class.

**Steps:**
1. Create instance method that delegates to static
2. Inject instance in class under test
3. In tests, inject fake

```java
// Before
class Calculator {
    public int compute(int x) {
        return MathUtils.round(x * 1.2);  // Static, can't mock
    }
}

// After
class Calculator {
    private MathUtils mathUtils = MathUtils.INSTANCE;
    
    public void setMathUtils(MathUtils m) { this.mathUtils = m; }
    
    public int compute(int x) {
        return mathUtils.round(x * 1.2);
    }
}

// Test
class FakeMathUtils {
    round(x) { return x; }  // Deterministic
}
```

---

### 6. Break Out Method Object

**When:** Long method with many local variables, can't extract.

**Steps:**
1. Create new class with the method
2. Pass all needed data to constructor
3. Execute method in constructor or execute()
4. Replace original call with new object

```java
// Before
class TaxCalculator {
    public Money calculate(Order o, Customer c, TaxRules r, ...) {
        // 200 lines with 15 local variables
    }
}

// After
class TaxCalculation {
    private Order order;
    private Customer customer;
    private TaxRules rules;
    // ... all dependencies
    
    public TaxCalculation(Order o, Customer c, TaxRules r, ...) {
        this.order = o;
        this.customer = c;
        this.rules = r;
    }
    
    public Money execute() {
        // extracted logic
    }
}
```

---

### 7. Encapsulate Global References

**When:** Code uses globals directly.

**Steps:**
1. Create class to hold global references
2. Instance methods access via this class
3. Inject in tests

```java
// Before
class Config {
    static String API_KEY = "prod-key";
}

class Service {
    void call() {
        String key = Config.API_KEY;  // Can't mock
    }
}

// After
class Config {
    private static Config instance;
    static Config get() { return instance; }
    
    String apiKey;
    public void setApiKey(String k) { this.apiKey = k; }
    public String getApiKey() { return apiKey; }
}

class Service {
    private Config config;
    public void setConfig(Config c) { this.config = c; }
    
    void call() {
        String key = config.getApiKey();
    }
}
```

---

### 8. Introduce Static Setter (for Singletons)

**When:** Singleton blocks testing.

```java
class Database {
    private static Database instance;
    public static Database get() { return instance; }
    
    // Add setter
    public static void setInstance(Database d) { instance = d; }
}

// Test
@Before
public void setUp() {
    Database.setInstance(new FakeDatabase());
}

@After
public void tearDown() {
    Database.setInstance(null);
}
```

---

### 9. Link Substitution

**When:** Can't modify dependency, but can control what's linked.

**Steps:**
1. Create fake classes with same names
2. Change classpath to use fakes first
3. Test runs with fakes

---

## Language-Specific Notes

### Java
- Use `mockito` or similar for easy mocking
- `@Mock` annotation for creating test doubles
- `@InjectMocks` for automatic injection

### C#
- Use `Moq`, `NSubstitute`, or `FakeItEasy`
- Interfaces are cheap, use freely

### JavaScript/TypeScript
- Use `sinon`, `jest.mock`, or `proxyquire`
- Dependency injection is more manual

### Python
- Use `unittest.mock` (built-in)
- Patch decorators for globals

---

## Decision Matrix

| Problem | Solution |
|---------|----------|
| Can't create object | Parameterize Constructor |
| Can't mock dependency | Extract Interface |
| Static method call | Extract and Override Call |
| Global variable | Encapsulate Global References |
| Singleton | Introduce Static Setter |
| Long method | Break Out Method Object |
| Utility class | Introduce Instance Delegator |
| Hard to test behavior | Subclass and Override Method |

---

## Sources

- "Working Effectively with Legacy Code" — Michael Feathers, Chapter 9 & 25
- "The Legacy Code Change Algorithm" — Feathers' 5-step process