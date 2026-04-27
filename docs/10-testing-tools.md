# Testing Tools

## Overview

The testing ecosystem is rich with frameworks and utilities for every language, test level, and testing type. This document provides a curated reference of the most widely used tools, organised by category.

---

## Unit Testing Frameworks

### Java

| Tool | Description |
|------|-------------|
| **JUnit 5** | The de-facto standard for Java unit testing. Supports parameterised tests, extensions, and `@Nested` test classes. |
| **TestNG** | Feature-rich alternative to JUnit with support for data providers, test groups, and parallel execution. |
| **Kotest** | Kotlin-first test framework with rich matchers and property-based testing. |
| **Spock** | Groovy-based BDD/TDD framework with expressive specification-style tests. |

### Python

| Tool | Description |
|------|-------------|
| **pytest** | The most popular Python test framework. Powerful fixture system, parameterisation, and a huge plugin ecosystem. |
| **unittest** | Python's built-in test framework, inspired by JUnit. |
| **Hypothesis** | Property-based testing library that generates test cases automatically. |

### JavaScript / TypeScript

| Tool | Description |
|------|-------------|
| **Jest** | All-in-one testing framework by Meta. Built-in mocking, coverage, and snapshot testing. |
| **Vitest** | Fast, Vite-native test runner, compatible with Jest API. Excellent for modern frontend projects. |
| **Mocha** | Flexible test runner paired with assertion libraries like Chai. |
| **Jasmine** | Behaviour-driven testing framework, no external dependencies. |

### C#

| Tool | Description |
|------|-------------|
| **xUnit** | Modern, community-driven test framework for .NET. |
| **NUnit** | Mature and widely used .NET test framework. |
| **MSTest** | Microsoft's official test framework, integrated with Visual Studio. |

### Go

| Tool | Description |
|------|-------------|
| **`testing` package** | Go's built-in test framework; run with `go test`. |
| **Testify** | Adds assertions, mocks, and test suites to Go's testing package. |
| **Ginkgo + Gomega** | BDD-style framework for Go. |

### Ruby

| Tool | Description |
|------|-------------|
| **RSpec** | BDD-style framework; the most popular Ruby test tool. |
| **Minitest** | Lightweight, fast, built into Ruby's standard library. |

---

## Assertion Libraries

| Language | Library | Example |
|----------|---------|---------|
| Java | **AssertJ** | `assertThat(user.getName()).isEqualTo("Alice");` |
| Java | **Hamcrest** | `assertThat(list, hasSize(3));` |
| Python | **pytest assertions** | `assert response.status_code == 200` |
| JavaScript | **Chai** | `expect(result).to.equal(42)` |
| .NET | **FluentAssertions** | `result.Should().Be(42);` |

---

## Mocking Libraries

| Language | Library | Description |
|----------|---------|-------------|
| Java | **Mockito** | Industry-standard mocking framework for Java |
| Java | **EasyMock** | Older mocking framework, still widely used |
| Python | **`unittest.mock`** | Built-in mocking support |
| Python | **pytest-mock** | Thin pytest wrapper around `unittest.mock` |
| JavaScript | **Jest Mocks** | Built-in mocking in Jest |
| JavaScript | **Sinon.js** | Standalone spies, stubs, and mocks |
| .NET | **Moq** | The most popular mocking library for .NET |
| .NET | **NSubstitute** | Friendly, readable .NET mock framework |

---

## Integration & API Testing

| Tool | Language | Description |
|------|----------|-------------|
| **REST Assured** | Java | Fluent API for testing REST services |
| **Spring MockMvc** | Java | Test Spring MVC controllers without a server |
| **Testcontainers** | Java, Python, Go, .NET | Runs real dependencies (DB, MQ) in Docker for integration tests |
| **Supertest** | JavaScript | HTTP assertions for Node.js apps |
| **HTTPX / requests** | Python | HTTP client used in integration tests |
| **Postman / Newman** | Any | API testing with collections; CLI runner for CI |
| **Pact** | Multiple | Consumer-driven contract testing |

---

## End-to-End (E2E) Testing

| Tool | Description |
|------|-------------|
| **Selenium** | The original browser automation framework. Supports all major browsers and languages. |
| **Playwright** | Modern, fast, multi-browser automation by Microsoft. Supports Chromium, Firefox, and WebKit. |
| **Cypress** | JavaScript-native E2E framework. Excellent developer experience, real-time reloading. |
| **Puppeteer** | Google's Node.js library for controlling Chrome/Chromium. |
| **WebdriverIO** | Node.js WebDriver framework with Selenium and devtools support. |
| **Appium** | Mobile app automation for iOS and Android. |

### Playwright Example

```javascript
import { test, expect } from '@playwright/test';

test('user can log in', async ({ page }) => {
  await page.goto('https://example.com/login');
  await page.fill('#username', 'alice');
  await page.fill('#password', 'password123');
  await page.click('button[type="submit"]');
  await expect(page).toHaveURL('/dashboard');
  await expect(page.locator('h1')).toHaveText('Welcome, Alice');
});
```

---

## Performance Testing Tools

| Tool | Language | Description |
|------|----------|-------------|
| **Apache JMeter** | GUI / XML | Mature, broad protocol support |
| **k6** | JavaScript | Developer-friendly, CI-ready |
| **Gatling** | Scala / Java | Continuous performance testing with HTML reports |
| **Locust** | Python | Distributed load testing with a Python API |
| **Artillery** | YAML / JS | API and microservice load testing |

*For a detailed comparison, see [Performance Testing →](07-performance-testing.md).*

---

## Security Testing Tools

| Tool | Type | Description |
|------|------|-------------|
| **OWASP ZAP** | DAST | Free, powerful web application scanner |
| **Burp Suite** | DAST | Industry-standard web security testing platform |
| **SonarQube** | SAST | Static code analysis with security rules |
| **Semgrep** | SAST | Fast, customisable static analysis |
| **OWASP Dependency-Check** | SCA | Identifies vulnerable third-party dependencies |
| **Snyk** | SCA | Finds and fixes vulnerabilities in code and containers |
| **Trivy** | SCA / Container | Scans containers, filesystems, and IaC |
| **TruffleHog** | Secrets | Scans repository history for leaked secrets |

*For more detail, see [Security Testing →](08-security-testing.md).*

---

## BDD / Specification Testing

| Tool | Language | Description |
|------|----------|-------------|
| **Cucumber** | Java, JS, Ruby, Python | Most widely used BDD framework with Gherkin |
| **Behave** | Python | BDD framework using Gherkin |
| **SpecFlow** | C# | Cucumber for .NET |
| **JBehave** | Java | Original Java BDD framework |
| **Serenity BDD** | Java | Cucumber/JBehave wrapper with rich reports |
| **Godog** | Go | Cucumber for Go |

*For details on BDD concepts, see [Behavior-Driven Development →](05-behavior-driven-development.md).*

---

## Code Coverage Tools

| Language | Tool | Notes |
|----------|------|-------|
| Java | **JaCoCo** | Maven/Gradle plugin; generates HTML reports |
| Python | **Coverage.py / pytest-cov** | Standard Python coverage tool |
| JavaScript | **Istanbul (nyc) / c8** | Built into Jest; standalone for other runners |
| C# | **Coverlet** | Cross-platform .NET coverage tool |
| Go | `go test -cover` | Built-in coverage in the Go toolchain |
| Ruby | **SimpleCov** | Coverage for Ruby/Rails |

*For a detailed guide, see [Test Coverage →](06-test-coverage.md).*

---

## Mutation Testing Tools

| Language | Tool | Description |
|----------|------|-------------|
| Java | **Pitest (PIT)** | Fast, bytecode-level mutation testing |
| Python | **mutmut** | Simple mutation testing for Python |
| JavaScript | **Stryker** | Multi-framework mutation testing platform |
| .NET | **Stryker.NET** | Mutation testing for C# and VB.NET |

---

## CI/CD Integration

Most test tools integrate seamlessly with popular CI/CD platforms:

| Platform | Native Test Support |
|----------|-------------------|
| **GitHub Actions** | JUnit XML reports, coverage upload actions |
| **GitLab CI/CD** | Built-in test report widgets |
| **Jenkins** | JUnit plugin, HTML Publisher plugin |
| **CircleCI** | Test splitting and parallelism |
| **Azure DevOps** | Test Plans, test result publishing |

**GitHub Actions example — Java with JUnit and JaCoCo:**

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
      - name: Run tests
        run: mvn verify
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v4
        with:
          files: target/site/jacoco/jacoco.xml
```

---

## Test Reporting and Dashboards

| Tool | Description |
|------|-------------|
| **Allure Report** | Rich HTML reports with screenshots, steps, and history trends |
| **ReportPortal** | Centralised test reporting and analytics platform |
| **Codecov** | Hosted code coverage reporting with PR integration |
| **Coveralls** | Coverage tracking with GitHub/GitLab integration |
| **SonarQube** | Code quality and coverage dashboard |

---

## Further Reading

- [Best Practices →](09-best-practices.md)
- [Test Coverage →](06-test-coverage.md)
- [Performance Testing →](07-performance-testing.md)
- [Security Testing →](08-security-testing.md)
