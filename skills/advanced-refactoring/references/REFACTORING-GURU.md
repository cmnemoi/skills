# Refactoring.guru — Code Smells & Techniques Catalog

Based on [Refactoring.guru](https://refactoring.guru/refactoring).

## The 21 Code Smells

### Bloaters

Code that's grown too large or complex.

| Smell | Description | Refactoring |
|-------|-------------|-------------|
| **Long Method** | Method with too many lines | Extract Method, Replace Temp with Query |
| **Large Class** | Class trying to do too much | Extract Class, Extract Subclass |
| **Primitive Obsession** | Using primitives instead of objects | Replace Primitive with Object, Introduce Parameter Object |
| **Long Parameter List** | Too many parameters | Introduce Parameter Object, Preserve Whole Object |
| **Data Clumps** | Same group of parameters appearing together | Introduce Parameter Object, Extract Class |

### Object-Orientation Abusers

Incorrect or incomplete use of OOP.

| Smell | Description | Refactoring |
|-------|-------------|-------------|
| **Switch Statements** | Complex switch/case | Replace Conditional with Polymorphism |
| **Temporary Field** | Fields set only in certain circumstances | Extract Class |
| **Refused Bequest** | Subclass doesn't use parent's methods | Replace Inheritance with Delegation |
| **Alternative Classes with Different Interfaces** | Classes do same thing but different API | Unify Interfaces with Adapter |

### Change Preventers

Code that's hard to change.

| Smell | Description | Refactoring |
|-------|-------------|-------------|
| **Divergent Change** | One class must be changed for different reasons | Extract Class |
| **Shotgun Surgery** | One change requires changes in many classes | Move Method, Inline Class |
| **Parallel Inheritance Hierarchies** | Subclasses mirror each other | Move Method, Push Down Method |

### Dispensables

Code that can be removed.

| Smell | Description | Refactoring |
|-------|-------------|-------------|
| **Comments** | Explanatory comments that hide code smell | - |
| **Duplicate Code** | Same code in multiple places | Extract Method, Pull Up Method |
| **Lazy Class** | Class doing too little | Inline Class |
| **Data Class** | Class with only data, no behavior | Move Method, Introduce Parameter Object |
| **Dead Code** | Unused code | - |
| **Speculative Generality** | "We might need this someday" code | Inline Class |

### Couplers

Excessive coupling.

| Smell | Description | Refactoring |
|-------|-------------|-------------|
| **Feature Envy** | Method uses more of another class than its own | Move Method, Extract Method |
| **Inappropriate Intimacy** | Classes know too much about each other | Move Method, Change Bidirectional to Unidirectional |
| **Message Chains** | Long chain of method calls | Hide Delegate |
| **Middle Man** | Class doing nothing but delegating | Remove Middle Man |

---

## The 66 Refactoring Techniques

### A. Composing Methods (9 techniques)

**1. Extract Method**
```java
// Before
void printOwing() {
    printBanner();
    System.out.println("name: " + name);
    System.out.println("amount: " + getOutstanding());
}

// After
void printOwing() {
    printBanner();
    printDetails();
}

void printDetails() {
    System.out.println("name: " + name);
    System.out.println("amount: " + getOutstanding());
}
```

**2. Inline Method**
```java
// Before
int getRating() { return moreThanFiveDeliveries() ? 2 : 1; }
boolean moreThanFiveDeliveries() { return numberOfDeliveries > 5; }

// After
int getRating() { return numberOfDeliveries > 5 ? 2 : 1; }
```

**3. Extract Variable**
```java
// Before
return order.getQuantity() * order.getItemPrice() 
     - Math.max(0, order.getQuantity() - 500) * order.getItemPrice() * 0.05;

// After
final double basePrice = order.getQuantity() * order.getItemPrice();
final double quantityDiscount = Math.max(0, order.getQuantity() - 500) * order.getItemPrice() * 0.05;
return basePrice - quantityDiscount;
```

**4. Inline Temp**
```java
// Before
final double basePrice = order.getBasePrice();
return basePrice > 1000;

// After
return order.getBasePrice() > 1000;
```

**5. Replace Temp with Query**
```java
// Before
double getPrice() {
    int basePrice = _quantity * _itemPrice;
    double discount = basePrice > 1000 ? basePrice * 0.95 : basePrice;
    return discount;
}

// After
double getPrice() {
    return basePrice() > 1000 ? basePrice() * 0.95 : basePrice();
}

int basePrice() { return _quantity * _itemPrice; }
```

**6. Split Temporary Variable**
```java
// Before
double temp = 2 * (_height + _width);
System.out.println(temp);
temp = _height * _width;
System.out.println(temp);

// After
final double perimeter = 2 * (_height + _width);
System.out.println(perimeter);
final double area = _height * _width;
System.out.println(area);
```

**7. Remove Assignments to Parameters**
```java
// Before
int discount(int inputVal) {
    if (inputVal > 50) inputVal = inputVal - 10;
    return inputVal;
}

// After
int discount(int inputVal) {
    if (inputVal > 50) return inputVal - 10;
    return inputVal;
}
```

**8. Replace Method with Method Object**
```java
// See Kerievsky reference - Break Out Method Object
```

**9. Substitute Algorithm**
```java
// Before
String foundPerson(String[] people) {
    for (String person : people) {
        if (person.equals("Don")) return "Don";
        if (person.equals("John")) return "John";
        if (person.equals("Kent")) return "Kent";
    }
    return "";
}

// After
String foundPerson(String[] people) {
    List candidates = Arrays.asList("Don", "John", "Kent");
    for (String person : people) {
        if (candidates.contains(person)) return person;
    }
    return "";
}
```

---

### B. Moving Features between Objects (8 techniques)

| Technique | Description |
|-----------|-------------|
| **Move Method** | Move method to class where it belongs |
| **Move Field** | Move field to class where it's used |
| **Extract Class** | Split one class into two |
| **Inline Class** | Merge small class into another |
| **Hide Delegate** | Remove chain of calls |
| **Remove Middle Man** | Remove delegation to client |
| **Introduce Foreign Method** | Add method to class you can't modify |
| **Introduce Local Extension** | Create subclass/wrapper |

---

### C. Organizing Data (14 techniques)

| Technique | Description |
|-----------|-------------|
| **Self Encapsulate Field** | Use getters/setters internally |
| **Replace Data Value with Object** | Replace primitive with class |
| **Change Value to Reference** | Object reference instead of copy |
| **Change Reference to Value** | Value object instead of reference |
| **Replace Array with Object** | Named access instead of index |
| **Duplicate Observed Data** | Separate UI from domain |
| **Change Unidirectional Association to Bidirectional** | Add reverse reference |
| **Change Bidirectional Association to Unidirectional** | Remove unused direction |
| **Replace Magic Number with Symbolic Constant** | Named constant |
| **Encapsulate Field** | Make public field private |
| **Encapsulate Collection** | Return read-only collection |
| **Replace Type Code with Class** | Type as class |
| **Replace Type Code with Subclasses** | Inheritance for types |
| **Replace Type Code with State/Strategy** | State/Strategy pattern |
| **Replace Subclass with Fields** | Replace subclass with fields |

---

### D. Simplifying Conditional Expressions (8 techniques)

| Technique | Description |
|-----------|-------------|
| **Decompose Conditional** | Extract conditions to methods |
| **Consolidate Conditional Expression** | Combine conditions |
| **Consolidate Duplicate Conditional Fragments** | Move common code out |
| **Remove Control Flag** | Use break/return instead |
| **Replace Nested Conditional with Guard Clauses** | Early returns |
| **Replace Conditional with Polymorphism** | Use inheritance |
| **Introduce Null Object** | Null object pattern |
| **Introduce Assertion** | Document assumptions |

---

### E. Simplifying Method Calls (14 techniques)

| Technique | Description |
|-----------|-------------|
| **Rename Method** | More descriptive name |
| **Add Parameter** | Add parameter to method |
| **Remove Parameter** | Remove unused parameter |
| **Separate Query from Modifier** | Get vs. side-effect |
| **Parameterize Method** | Combine similar methods |
| **Replace Parameter with Explicit Methods** | Split method |
| **Preserve Whole Object** | Pass object, not fields |
| **Replace Parameter with Method Call** | Call method, not pass result |
| **Introduce Parameter Object** | Group parameters |
| **Remove Setting Method** | Remove setter after construction |
| **Hide Method** | Make private if not used externally |
| **Replace Constructor with Factory Method** | More expressive creation |
| **Replace Error Code with Exception** | Exception instead of error code |
| **Replace Exception with Test** | Check before call |

---

### F. Dealing with Generalization (12 techniques)

| Technique | Description |
|-----------|-------------|
| **Pull Up Field** | Move field to superclass |
| **Pull Up Method** | Move method to superclass |
| **Pull Up Constructor Body** | Share constructor code |
| **Push Down Method** | Move to subclass |
| **Push Down Field** | Move to subclass |
| **Extract Subclass** | Create subclass |
| **Extract Superclass** | Create superclass |
| **Extract Interface** | Create interface |
| **Collapse Hierarchy** | Merge super/subclass |
| **Form Template Method** | Shared algorithm skeleton |
| **Replace Inheritance with Delegation** | Composition over inheritance |
| **Replace Delegation with Inheritance** | When delegation is permanent |

---

## Design Patterns Reference

| Pattern | Category | Purpose |
|---------|----------|---------|
| **Factory Method** | Creational | Interface for creating objects |
| **Abstract Factory** | Creational | Create families of objects |
| **Builder** | Creational | Separate construction from representation |
| **Prototype** | Creational | Clone objects |
| **Singleton** | Creational | One instance |
| **Adapter** | Structural | Convert interface |
| **Bridge** | Structural | Decouple abstraction from implementation |
| **Composite** | Structural | Tree structures |
| **Decorator** | Structural | Add responsibilities dynamically |
| **Facade** | Structural | Unified interface |
| **Flyweight** | Structural | Share fine-grained objects |
| **Proxy** | Structural | Control access |
| **Chain of Responsibility** | Behavioral | Pass request along chain |
| **Command** | Behavioral | Encapsulate request |
| **Iterator** | Behavioral | Sequential access |
| **Mediator** | Behavioral | Encapsulate interaction |
| **Memento** | Behavioral | Capture state |
| **Observer** | Behavioral | Notify changes |
| **State** | Behavioral | Object behavior based on state |
| **Strategy** | Behavioral | Interchangeable algorithms |
| **Template Method** | Behavioral | Define algorithm skeleton |
| **Visitor** | Behavioral | New operation without changing classes |

---

## Sources

- [Refactoring.guru](https://refactoring.guru/refactoring)
- "Refactoring" — Martin Fowler (foundation)