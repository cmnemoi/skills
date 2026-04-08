# Kerievsky — Refactoring to Patterns

Based on Joshua Kerievsky's "Refactoring to Patterns" and [Industrial Logic catalog](https://www.industriallogic.com/refactoring-to-patterns/catalog/).

## Philosophy: Not Too Early, Not Too Late

> Introduce patterns when code smells indicate a clear need—not based on anticipated future requirements.

The "antibiotic analogy":
- **Too early** = taking antibiotics prophylactically → unnecessary complexity
- **Too late** = not taking needed antibiotics → code rot and technical debt
- **Just right** = using the right pattern for the actual problem

---

## The 27 Pattern Refactorings

Organized by category from Industrial Logic:

### Creation (6 refactorings)

| Refactoring | Code Smell |
|-------------|------------|
| Chain Constructors | Multiple constructors with similar logic |
| Encapsulate Classes With Factory | Direct construction scattered |
| Encapsulate Composite With Builder | Complex object assembly |
| Introduce Polymorphic Creation With Factory Method | Type-specific creation logic |
| Move Creation Knowledge To Factory | Creation logic in clients |
| Replace Constructors With Creation Methods | Constructors can't express intent |

### Simplification (3 refactorings)

| Refactoring | Code Smell |
|-------------|------------|
| Compose Method | Long method, spaghetti code |
| Replace Conditional Logic With Strategy | Complex conditionals |
| Unify Interfaces | Similar interfaces, different names |

### Generalization (5 refactorings)

| Refactoring | Code Smell |
|-------------|------------|
| Extract Adapter | Incompatible interface |
| Extract Composite | Duplicated tree-like structure |
| Form Template Method | Similar algorithms, different steps |
| Replace One/Many Distinctions With Composite | Special case handling |
| Unify Interfaces With Adapter | Different interfaces, same concept |

### Protection (3 refactorings)

| Refactoring | Code Smell |
|-------------|------------|
| Extract Parameter | Repeated parameter passing |
| Introduce Null Object | Null checks scattered |
| Replace State-Altering Conditionals with State | Complex state transitions |

### Accumulation (6 refactorings)

| Refactoring | Code Smell |
|-------------|------------|
| Move Accumulation To Collecting Parameter | Building collections in loops |
| Move Accumulation To Visitor | Complex traversal logic |
| Move Embellishment To Decorator | Adding behavior via subclassing |
| Replace Conditional Dispatcher With Command | Complex if-else dispatch |
| Replace Hard-Coded Notifications With Observer | Tight coupling |
| Replace Implicit Tree With Composite | Manual tree building |

### Utilities (4 refactorings)

| Refactoring | Code Smell |
|-------------|------------|
| Inline Singleton | Unnecessary Singleton |
| Limit Instantiation With Singleton | Global state |
| Replace Implicit Language With Interpreter | Embedded mini-language |
| Replace Type Code With Class | Type code as primitive |

---

## Detailed Refactorings

### Replace Conditional with Polymorphism

**Code Smell:** Multiple conditionals checking same type, different behavior for each.

**Before:**
```java
class Employee {
    double getPay() {
        if (type.equals("HOURLY")) return hours * rate;
        else if (type.equals("SALARY")) return salary / 12;
        else if (type.equals("COMMISSION")) return baseSalary + (sales * commissionRate);
    }
}
```

**After:**
```java
abstract class Employee { abstract double getPay(); }

class HourlyEmployee extends Employee {
    double getPay() { return hours * rate; }
}
class SalariedEmployee extends Employee {
    double getPay() { return salary / 12; }
}
class CommissionEmployee extends Employee {
    double getPay() { return baseSalary + (sales * commissionRate); }
}
```

---

### Introduce Null Object

**Code Smell:** Repeated null checks throughout codebase.

**Before:**
```java
class Customer {
    void process() {
        if (customer != null && customer.isActive()) {
            customer.sendNotification();
        }
    }
}
// Null checks everywhere...
```

**After:**
```java
interface Customer {
    void sendNotification();
    boolean isActive();
}

class RealCustomer implements Customer {
    void sendNotification() { /* actual logic */ }
    boolean isActive() { return true; }
}

class NullCustomer implements Customer {
    void sendNotification() { /* do nothing */ }
    boolean isActive() { return false; }
}
```

---

### Introduce Parameter Object

**Code Smell:** Methods with many parameters, especially parameters that appear together.

**Before:**
```java
void generateReport(String title, Date start, Date end, 
                   String author, boolean landscape, String format) { }
```

**After:**
```java
class ReportOptions {
    private String title;
    private DateRange period;
    private String author;
    private Orientation orientation;
    private Format format;
}

void generateReport(ReportOptions options) { }
```

---

### Replace Inheritance with Delegation

**Code Smell:** Class inherits methods it doesn't need, or must override most behavior.

**Before:**
```java
class Stack extends ArrayList {
    void push(Object e) { super.add(e); }
    Object pop() { return super.remove(super.size() - 1); }
    // Stack inherits: add(index, e), get(index), set(), etc. - unwanted!
}
```

**After:**
```java
class Stack {
    private ArrayList elements = new ArrayList();
    
    void push(Object e) { elements.add(e); }
    Object pop() { return elements.remove(elements.size() - 1); }
}
```

---

### Extract Method Object

**Code Smell:** Long method where extraction fails due to too many variables.

**Before:**
```java
class Calculator {
    void calculate(Order o, Customer c, TaxRules r, ShippingRules s, 
                   PricingEngine p, InventorySystem i, ...) {
        // 200 lines, 50 local variables - can't extract
    }
}
```

**After:**
```java
class Calculation {
    private Order order;
    private Customer customer;
    // ... all variables become fields
    
    Calculation(Order o, Customer c, ...) {
        this.order = o;
        this.customer = c;
    }
    
    void execute() {
        // All variables now accessible, can refactor further
    }
}
```

---

### Move Method

**Code Smell:** Method uses more features of another class than its own.

**Before:**
```java
class Customer {
    double calculateOrderTotal(Order o) {
        double total = 0;
        for (Item i : o.items) total += i.price * i.quantity;
        return total * this.getDiscountMultiplier();
    }
}
```

**After:**
```java
class Order {
    double calculateTotal(Customer c) {
        double total = 0;
        for (Item i : this.items) total += i.price * i.quantity;
        return total * c.getDiscountMultiplier();
    }
}
```

---

## When NOT to Apply Patterns

| Situation | Why Not |
|-----------|---------|
| **Code is already clear** | Patterns add complexity |
| **No immediate need** | YAGNI — You Aren't Gonna Need It |
| **Team doesn't understand pattern** | Adds cognitive load |
| **Simpler solution exists** | Don't over-engineer |
| **Just for aesthetics** | Refactor for reason, not decoration |
| **Violates YAGNI** | Build when actually needed |

---

## Evolutionary Design

Kerievsky's approach:

1. **Start simple** — Write the simplest code that works
2. **Test-first** — Let tests guide design decisions
3. **Refactor continuously** — Remove duplication, improve structure
4. **Introduce patterns when needed** — Refactor toward patterns as code smells appear

This avoids both over-engineering (applying patterns early) and under-engineering (never improving structure).

---

## Sources

- [Refactoring to Patterns Catalog](https://www.industriallogic.com/refactoring-to-patterns/catalog/) — Industrial Logic
- [Refactoring to Patterns](http://www.tarrani.net/RefactoringToPatterns.pdf) — Joshua Kerievsky (complete book)
- "Stop Over-Engineering" — Kerievsky's article on avoiding premature patterns