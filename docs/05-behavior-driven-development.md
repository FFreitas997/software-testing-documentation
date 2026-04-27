# Behavior-Driven Development (BDD)

## What is BDD?

**Behavior-Driven Development (BDD)** is an agile methodology that extends TDD by shifting the focus from individual code units to the *behaviour* of a system from the perspective of its stakeholders. BDD bridges the communication gap between business experts, testers, and developers by using a shared, plain-language specification format.

BDD was introduced by Dan North in 2003 as a response to questions teams had about "what to test" in TDD.

---

## Core Philosophy

BDD encourages teams to:

1. **Collaborate** – Developers, testers, and business analysts write specifications together.
2. **Communicate in a ubiquitous language** – Use business terminology, not technical jargon.
3. **Automate the specification** – Specifications double as executable acceptance tests.
4. **Focus on outcomes** – Describe what the system *should do* for the user, not how it does it internally.

---

## The Three Amigos

The **Three Amigos** is a BDD workshop technique where three perspectives come together to discuss a user story before development begins:

| Role | Perspective |
|------|-------------|
| **Business Analyst / Product Owner** | What is the business need? |
| **Developer** | How will this be implemented? |
| **Tester** | What could go wrong? What are the edge cases? |

This collaborative session produces concrete examples that become the basis for acceptance criteria and automated tests.

---

## Given-When-Then Syntax (Gherkin)

BDD specifications are written in **Gherkin**, a plain-language, structured syntax:

```
Feature: <Short description of the feature>

  Background:
    Given <precondition shared by all scenarios>

  Scenario: <Short description of a specific behaviour>
    Given <an initial context or state>
    When  <an action is taken>
    Then  <an observable outcome>
    And   <additional outcome or step>
    But   <a negated outcome>
```

### Keywords

| Keyword | Purpose |
|---------|---------|
| `Feature` | Groups related scenarios |
| `Background` | Steps shared by all scenarios in a feature |
| `Scenario` | A single concrete example of a behaviour |
| `Scenario Outline` | A scenario template with multiple data rows |
| `Given` | Sets up the initial state |
| `When` | Describes the action or event |
| `Then` | Describes the expected outcome |
| `And` / `But` | Additional steps |
| `Examples` | Data table used with Scenario Outline |

---

## Gherkin Example: E-Commerce Checkout

```gherkin
Feature: Shopping cart checkout

  Background:
    Given the user is logged in as "alice@example.com"

  Scenario: Successful checkout with a valid credit card
    Given the cart contains 1 item priced at $25.00
    And the user has entered valid shipping details
    When the user submits payment with a valid credit card
    Then the order confirmation page is displayed
    And the user receives a confirmation email

  Scenario: Checkout fails with an expired credit card
    Given the cart contains 1 item priced at $25.00
    When the user submits payment with an expired credit card
    Then an error message "Your card has expired" is displayed
    And the order is not placed

  Scenario Outline: Checkout total includes shipping
    Given the cart contains items totalling <subtotal>
    When the user selects "<shipping_method>" shipping
    Then the order total is <expected_total>

    Examples:
      | subtotal | shipping_method | expected_total |
      | $20.00   | Standard        | $24.99         |
      | $20.00   | Express         | $29.99         |
      | $50.00   | Standard        | $54.99         |
```

---

## Connecting Gherkin to Code (Step Definitions)

Each Gherkin step maps to a **step definition** — a function in code that implements the step.

### Java with Cucumber

```java
public class CheckoutSteps {

    @Given("the cart contains {int} item priced at ${double}")
    public void cartContainsItem(int quantity, double price) {
        cart.addItem(new Item(quantity, price));
    }

    @When("the user submits payment with a valid credit card")
    public void submitValidPayment() {
        orderPage = checkoutPage.submitPayment(VALID_CARD);
    }

    @Then("the order confirmation page is displayed")
    public void orderConfirmationIsDisplayed() {
        assertTrue(orderPage.isConfirmationDisplayed());
    }
}
```

### Python with Behave

```python
@given("the cart contains {quantity:d} item priced at ${price:f}")
def step_cart_contains_item(context, quantity, price):
    context.cart.add_item(Item(quantity, price))

@when("the user submits payment with a valid credit card")
def step_submit_valid_payment(context):
    context.order_page = context.checkout_page.submit_payment(VALID_CARD)

@then("the order confirmation page is displayed")
def step_order_confirmation_displayed(context):
    assert context.order_page.is_confirmation_displayed()
```

---

## BDD vs. TDD

| Dimension | TDD | BDD |
|-----------|-----|-----|
| Primary audience | Developers | Developers, testers, business analysts |
| Language | Programming language | Gherkin (natural language) |
| Granularity | Unit / class level | Feature / scenario level |
| Focus | Correctness of code | Correctness of behaviour |
| Documentation | Test method names | Human-readable scenarios |
| Automation layer | Unit test framework | Cucumber, Behave, SpecFlow |

BDD and TDD are **complementary**: BDD drives feature-level acceptance tests from the outside; TDD drives unit-level tests from the inside.

---

## BDD Frameworks by Language

| Language | Framework |
|----------|-----------|
| Java | Cucumber-JVM, JBehave, Serenity BDD |
| Python | Behave, Pytest-BDD, Radish |
| JavaScript / TypeScript | Cucumber.js, Jest + Gherkin via `jest-cucumber` |
| C# | SpecFlow, NUnit with Gherkin |
| Ruby | Cucumber (original), RSpec |
| Go | Godog |
| PHP | Behat |

---

## Living Documentation

When BDD scenarios are automated and kept up-to-date, they become **living documentation**: a human-readable description of the system's behaviour that is verified to be accurate with every CI run. Tools like Serenity BDD and Allure can generate HTML reports from BDD runs that are accessible to non-technical stakeholders.

---

## Benefits and Challenges

### Benefits

- Reduces misunderstandings between business and technical teams.
- Produces regression tests that use business language.
- Makes acceptance criteria explicit and measurable.
- Creates a shared vocabulary (ubiquitous language).

### Challenges

- Maintaining step definitions adds overhead.
- Gherkin can become overly prescriptive and brittle if written poorly.
- Requires discipline and collaboration to be effective.
- Not a substitute for unit or integration tests.

---

## Further Reading

- [Test-Driven Development →](04-test-driven-development.md)
- [Testing Levels →](03-testing-levels.md)
- Dan North, "Introducing BDD" (https://dannorth.net/introducing-bdd/)
- Cucumber documentation (https://cucumber.io/docs)
