# Types of Testing

Software testing can be categorised along several dimensions. Understanding the taxonomy helps teams build a balanced testing portfolio that catches different classes of defects.

---

## 1. Functional Testing

Functional testing validates that the software behaves according to specified requirements. It treats the system as a black box: inputs go in, outputs come out, and the tester checks whether the outputs are correct.

### Subtypes

| Type | Description |
|------|-------------|
| **Unit Testing** | Tests an individual function or class in isolation |
| **Integration Testing** | Tests interactions between two or more components |
| **System Testing** | Tests the complete, integrated system end-to-end |
| **Acceptance Testing** | Validates the system against user requirements |
| **Regression Testing** | Verifies that existing features still work after a change |
| **Smoke Testing** | A quick sanity check to see if the build is stable enough for further testing |
| **Sanity Testing** | Narrow regression check on a specific area after a minor change |

---

## 2. Non-Functional Testing

Non-functional testing evaluates *how* the system performs, rather than *what* it does.

| Type | What it measures |
|------|-----------------|
| **Performance Testing** | Response time, throughput, and resource usage under normal load |
| **Load Testing** | Behaviour under anticipated peak load |
| **Stress Testing** | Behaviour beyond normal load limits |
| **Spike Testing** | Behaviour during sudden, extreme load increases |
| **Endurance (Soak) Testing** | Behaviour over a sustained period |
| **Scalability Testing** | Ability to scale up or out to handle increased load |
| **Usability Testing** | Ease of use and user experience |
| **Accessibility Testing** | Compliance with accessibility standards (e.g., WCAG 2.1) |
| **Security Testing** | Resistance to attacks and data protection |
| **Compatibility Testing** | Behaviour across different browsers, OS, devices |
| **Reliability Testing** | Ability to perform without failure over time |
| **Recovery Testing** | Ability to recover from crashes, hardware failures, or errors |
| **Localisation / Internationalisation Testing** | Correct behaviour in different languages and locales |

---

## 3. Structural (White-Box) Testing

Structural testing examines the internal structure of the code. Tests are designed with knowledge of the implementation.

### Common Techniques

- **Statement Coverage** – ensure every line of code is executed at least once.
- **Branch Coverage** – ensure every branch (if/else) is taken at least once.
- **Path Coverage** – ensure every possible execution path is tested.
- **Condition Coverage** – ensure every Boolean sub-expression evaluates to both `true` and `false`.

### When to Use

White-box testing is most valuable during unit and integration testing, and during security audits where understanding the code path helps identify vulnerabilities.

---

## 4. Change-Related Testing

When code changes, targeted testing ensures that the changes work correctly and do not break anything else.

| Type | Trigger | Description |
|------|---------|-------------|
| **Regression Testing** | Any code change | Re-run existing tests to catch unintended side effects |
| **Re-testing (Confirmation Testing)** | A bug fix is applied | Verify that the specific defect is resolved |
| **Impact Analysis** | Before a change | Identify which parts of the system a change might affect |

---

## 5. Black-Box vs. White-Box vs. Grey-Box Testing

| Approach | Knowledge of Internals | Typical Testers |
|----------|------------------------|-----------------|
| **Black-Box** | None | QA engineers, business analysts, end users |
| **White-Box** | Full | Developers, security researchers |
| **Grey-Box** | Partial | QA engineers with access to architectural diagrams or APIs |

---

## 6. Manual vs. Automated Testing

| Dimension | Manual Testing | Automated Testing |
|-----------|---------------|-------------------|
| Speed | Slow | Fast |
| Reliability | Prone to human error | Consistent |
| Upfront cost | Low | High (tooling, scripting) |
| Maintenance cost | Low | Can be high as UI changes |
| Best for | Exploratory, usability, ad-hoc testing | Regression, load, CI/CD pipelines |

### The Testing Pyramid

The **testing pyramid** (popularised by Mike Cohn) recommends having many fast unit tests at the base, fewer integration tests in the middle, and a small number of slow end-to-end tests at the top:

```
          /\
         /  \
        / E2E\        ← Few, slow, costly
       /------\
      /        \
     /Integration\   ← Some, medium speed
    /------------\
   /              \
  /   Unit Tests   \ ← Many, fast, cheap
 /------------------\
```

Inverting the pyramid (many E2E tests, few unit tests) is an anti-pattern that leads to slow, flaky, and expensive CI pipelines.

---

## 7. Exploratory Testing

Exploratory testing is simultaneous learning, test design, and test execution. Rather than following a predefined script, the tester explores the application guided by curiosity and domain knowledge.

**Key characteristics:**
- Sessions are time-boxed (e.g., 90 minutes).
- A **charter** defines the scope: "Explore the payment module using credit cards to discover security and validation defects."
- Findings are logged in a session sheet.

Exploratory testing is particularly effective at finding defects that scripted tests miss, because it adapts to what is discovered during execution.

---

## 8. Acceptance Testing

Acceptance testing verifies that the system satisfies business requirements and is ready for delivery.

| Subtype | Description |
|---------|-------------|
| **User Acceptance Testing (UAT)** | End users validate the system in a production-like environment |
| **Business Acceptance Testing (BAT)** | Business stakeholders confirm the system meets business goals |
| **Alpha Testing** | Conducted by internal teams before release |
| **Beta Testing** | Conducted by a selected group of external users before GA release |
| **Contract Acceptance Testing** | Verifies that the system meets contractual requirements |
| **Regulation Acceptance Testing** | Verifies compliance with laws and regulations |

---

## Further Reading

- [Testing Levels →](03-testing-levels.md)
- [Performance Testing →](07-performance-testing.md)
- [Security Testing →](08-security-testing.md)
