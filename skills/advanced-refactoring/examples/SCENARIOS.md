# Refactoring Scenarios — Practical Examples

Real-world examples of applying refactoring techniques from this skill.

---

## Scenario 1: Adding Feature to Untested Code

**Context:** You need to add loyalty points processing to an existing `OrderProcessor` class that has no tests.

### Approach: Sprout Method

```java
// Before: Can't test existing code
class OrderProcessor {
    public void process(Order order) {
        // 500 lines of legacy logic...
        for (Item item : order.getItems()) {
            item.updateInventory();
        }
        transactionService.save(order);
    }
}

// After: Use Sprout Method
class OrderProcessor {
    public void process(Order order) {
        // ... existing logic stays unchanged
        
        // New: Add loyalty points (testable, separate)
        LoyaltyPoints points = calculateLoyaltyPoints(order);
        loyaltyService.award(order.getCustomer(), points);
    }
    
    // New method - fully testable
    public LoyaltyPoints calculateLoyaltyPoints(Order order) {
        int basePoints = order.getTotal().intValue() / 10;
        if (order.isVIP()) basePoints *= 2;
        return new LoyaltyPoints(basePoints);
    }
}
```

---

## Scenario 2: Breaking a Hidden Dependency

**Context:** `ReportGenerator` creates its database connection internally, can't test.

### Approach: Parameterize Constructor

```java
// Before: Hidden dependency
class ReportGenerator {
    private Database db;  // Can't mock!
    
    public ReportGenerator() {
        this.db = new ProductionDatabase("prod-server");
    }
    
    public Report generate(Month month) {
        // Use db to generate report
    }
}

// After: Dependency injectable
class ReportGenerator {
    private Database db;
    
    public ReportGenerator(Database db) {
        this.db = db;
    }
    
    // For backwards compatibility
    public ReportGenerator() {
        this(new ProductionDatabase("prod-server"));
    }
}

// Test now possible
class ReportGeneratorTest {
    @Test
    void shouldGenerateCorrectReport() {
        FakeDatabase db = new FakeDatabase();
        db.setupTestData();
        
        ReportGenerator gen = new ReportGenerator(db);
        Report report = gen.generate(April);
        
        assertEquals(expected, report);
    }
}
```

---

## Scenario 3: Refactoring a Monster Method

**Context:** A 300-line `calculate()` method that nobody understands.

### Approach: Incremental Extraction

```java
// Before: Monster method
public void calculate() {
    // 300 lines mixing validation, computation, formatting...
}

// After: Incremental extraction
public void calculate() {
    validateInput();
    loadData();
    computeResults();
    formatOutput();
}

private void validateInput() {
    // Extracted 50 lines
}

private void loadData() {
    // Extracted 80 lines  
}

private void computeResults() {
    // Extracted 100 lines
}

private void formatOutput() {
    // Extracted 70 lines
}
```

### If too many variables: Use Method Object

```java
// Can't extract because of 30+ local variables
public void calculate() {
    // Mix of: order, customer, pricing, shipping, tax, discounts, rewards...
}

// Use Method Object
class OrderCalculation {
    private Order order;
    private Customer customer;
    // ... all variables become fields
    
    public OrderCalculation(Order o, Customer c, /* all params */) {
        this.order = o;
        this.customer = c;
    }
    
    public Money execute() {
        // Now can refactor incrementally
        return computePricing()
            .addShipping()
            .applyDiscounts()
            .calculateTax();
    }
}
```

---

## Scenario 4: Replacing Complex Conditionals

**Context:** Employee payroll with different calculation rules per type.

### Approach: Replace Conditional with Polymorphism

```java
// Before: Complex switch
class Employee {
    public double calculatePay() {
        switch (type) {
            case "HOURLY": return hours * rate;
            case "SALARY": return salary / 26;
            case "COMMISSION": return base + (sales * 0.15);
            case "CONTRACTOR": return dailyRate * days;
        }
    }
}

// After: Polymorphism
interface PayCalculator {
    double calculatePay();
}

class HourlyPay implements PayCalculator {
    public double calculatePay() { return hours * rate; }
}

class SalariedPay implements PayCalculator {
    public double calculatePay() { return salary / 26; }
}

class CommissionPay implements PayCalculator {
    public double calculatePay() { return base + (sales * 0.15); }
}

class Employee {
    private PayCalculator calculator;
    
    public double calculatePay() {
        return calculator.calculatePay();
    }
}
```

---

## Scenario 5: Handling Null Checks

**Context:** Code full of `if (customer != null)` checks.

### Approach: Introduce Null Object

```java
// Before: Null checks everywhere
class OrderService {
    void process(Order order) {
        if (order.getCustomer() != null) {
            order.getCustomer().sendNotification();
        }
        if (order.getCustomer() != null && order.getCustomer().isActive()) {
            loyaltyService.awardPoints(order);
        }
        // 20 more null checks...
    }
}

// After: Null Object
interface Customer {
    void sendNotification();
    boolean isActive();
    // ...
}

class RealCustomer implements Customer {
    void sendNotification() { emailService.send(this.email); }
    boolean isActive() { return this.status.equals("ACTIVE"); }
}

class NullCustomer implements Customer {
    void sendNotification() { /* no-op */ }
    boolean isActive() { return false; }
}

// Now: No null checks needed
class OrderService {
    void process(Order order) {
        order.getCustomer().sendNotification();
        if (order.getCustomer().isActive()) {
            loyaltyService.awardPoints(order);
        }
    }
}
```

---

## Scenario 6: Parallel Inheritance Hierarchies

**Context:** `SalesRep` and `Manager` both have `ReportFormatter` subclasses.

### Approach: Extract Superclass / Move Method

```java
// Before: Parallel hierarchies
class SalesRep {
    PDFReportFormatter pdfFormatter;
    HTMLReportFormatter htmlFormatter;
}

class Manager {
    PDFReportFormatter pdfFormatter;  // Duplicated!
    HTMLReportFormatter htmlFormatter;  // Duplicated!
}

// After: Pull up to shared superclass
class Employee {
    PDFReportFormatter pdfFormatter;
    HTMLReportFormatter htmlFormatter;
}

class SalesRep extends Employee { ... }
class Manager extends Employee { ... }
```

---

## Scenario 7: Using Approval Testing for Safety Net

**Context:** Need to refactor old report generation with no tests.

### Approach: Approval Testing

```java
// Using ApprovalTests library

@Test
void shouldGenerateSameOutputAsBefore() {
    ReportGenerator gen = new LegacyReportGenerator();
    
    // Run and capture output
    String output = gen.generateMonthlyReport(
        new Date("2024-01-01"), 
        new Date("2024-01-31")
    );
    
    // Verify against approved output
    Approvals.verify(output);
}
```

After running once, approve the output file. Now you have a safety net for refactoring.

---

## Scenario 8: Finding Pinch Points

**Context:** Large system, want to add tests but don't know where.

### Approach: Pinch Point Analysis

```
Effect diagram for OrderSystem:
                  ┌──────────┐
                  │  Order   │
                  └────┬─────┘
                       │
        ┌──────────────┼──────────────┐
        │              │              │
   ┌────▼────┐   ┌─────▼─────┐  ┌───▼────┐
   │Payment  │   │Inventory  │  │Shipping│
   └────┬────┘   └─────┬─────┘  └───┬────┘
        │              │              │
        └──────────────┼──────────────┘
                       │
                  ┌────▼────┐
                  │  Email  │  ← PINCH POINT!
                  └─────────┘

Test at Email - catches issues in all 3 subsystems!
```

Test at the "Email" service (called by all subsystems) to maximize coverage.

---

## Sources

- "Working Effectively with Legacy Code" — Michael Feathers
- [Understand Legacy Code](https://understandlegacycode.com/) — Nicolas Carlo
- [Refactoring to Patterns](https://www.industriallogic.com/refactoring-to-patterns/catalog/) — Joshua Kerievsky
- [Refactoring.guru](https://refactoring.guru/refactoring)