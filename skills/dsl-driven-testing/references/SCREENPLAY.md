# The Screenplay Pattern

> Sources: [Serenity/JS](https://serenity-js.org/handbook/design/screenplay-pattern/) | [Serenity BDD](https://serenity-bdd.github.io/docs/screenplay/screenplay_fundamentals) | [Beyond Page Objects (InfoQ)](https://www.infoq.com/articles/Beyond-Page-Objects-Test-Automation-Serenity-Screenplay/)

The Screenplay Pattern is an OO implementation of the Protocol Driver concept. It models tests as a **theatre play**.

---

## The 5 Components

| Component | Role | Analogy |
|-----------|------|---------|
| **Actor** | Represents a user or system | "Bob wants to buy a book" |
| **Ability** | Wrapper around an integration tool | "Bob can use a browser" |
| **Interaction** | Atomic action | "Click the button" |
| **Task** | Group of interactions with business meaning | "Log in", "Place an order" |
| **Question** | Queries system state | "What is the page title?" |

The key insight: **Tasks = DSL layer**, **Abilities = Protocol Drivers**.

---

## TypeScript Example (Serenity/JS)

```typescript
// Ability — wraps the integration tool (= Protocol Driver)
const actor = actorCalled('Apisitt')
    .whoCan(CallAnApi.using(axiosInstance));

// Task — group of interactions with business meaning (= DSL method)
const setupProductCatalogue = (products: Product[]) =>
    Task.where(`#actor sets up the product catalogue`,
        Send.a(PostRequest.to('/products').with(products)),
        Ensure.that(LastResponse.status(), equals(201))
    );

// Question — queries state (= DSL assertion)
const shopTitle = Page.current().title();

// Test — pure business language
await actor.attemptsTo(
    setupProductCatalogue([{ name: 'Apples', price: '£2.50' }]),
    Navigate.to('https://shop.example.org'),
    Ensure.that(shopTitle, endsWith('My Example Shop'))
);
```

---

## Java Example (Serenity BDD)

```java
// Task = reusable DSL unit
public class Login {
    public static Performable as(String username, String password) {
        return Task.where("{0} logs in as " + username,
            Open.url("https://www.saucedemo.com/"),
            Enter.theValue(username).into("#user-name"),
            Enter.theValue(password).into("#password"),
            Click.on("#login-button")
        );
    }
}

// Question = state assertion
Question<Integer> numberOfItems = Question
    .about("the number of todo items")
    .answeredBy(actor ->
        BrowseTheWeb.as(actor).findAll(".todo-list li").size()
    );

// Test — reads like a spec
toby.attemptsTo(Login.as("standard_user", "secret_sauce"));
assertThat(numberOfItems.answeredBy(toby)).isEqualTo(1);
```

---

## Switching Drivers = Switching Abilities

```typescript
// API driver
const apiActor = actorCalled('Apisitt')
    .whoCan(CallAnApi.using(axios));

// Browser driver
const uiActor = actorCalled('Wendy')
    .whoCan(BrowseTheWebWithPlaywright.using(browser));

// Tasks are IDENTICAL — only the Ability changes
await apiActor.attemptsTo(setupProductCatalogue(products));
await uiActor.attemptsTo(openOnlineStore());
```

This is exactly the Protocol Driver substitution principle — same scenario, different driver.

---

## Screenplay vs Page Objects

| | Page Objects | Screenplay Pattern |
|---|---|---|
| Models | A web page | An actor's goal |
| Coupled to | Browser/Selenium | Ability (swappable) |
| Can run in-memory | No | Yes (swap Ability) |
| Vocabulary | UI terms (`fillField`, `clickButton`) | Business terms (`Login`, `PlaceOrder`) |
| Reuse | Low (per-page) | High (Tasks compose) |

---

## When to Use Screenplay vs Bare Protocol Drivers

| Use Screenplay | Use bare Protocol Drivers |
|---|---|
| TypeScript/JavaScript project | Java/Python with existing test infra |
| Serenity/JS or Serenity BDD already in use | Simple system, few drivers |
| Team prefers OO composition | Functional/dataclass approach (Python Scenario) |
| Many actors with different abilities | Single actor, multiple channels |
