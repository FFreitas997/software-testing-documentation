# Accessibility Testing

Accessibility testing (often abbreviated **a11y**) ensures that your application can be used by **everyone**, including people with visual, auditory, motor, or cognitive disabilities. It validates conformance to established standards and catches issues that automated tests or regular QA would miss.

---

## Table of Contents

1. [Why Accessibility Testing Matters](#1-why-accessibility-testing-matters)
2. [WCAG Guidelines](#2-wcag-guidelines)
3. [Types of Accessibility Issues](#3-types-of-accessibility-issues)
4. [Testing Approaches](#4-testing-approaches)
5. [Tools Overview](#5-tools-overview)
6. [Automated Testing with Playwright + axe-core](#6-automated-testing-with-playwright--axe-core)
   - [Full-Page Accessibility Scan](#example-full-page-accessibility-scan)
   - [Scoping the Scan to a Component](#example-scoping-the-scan-to-a-component)
   - [Excluding Known Issues](#example-excluding-known-issues-disable-rules)
   - [Keyboard Navigation](#example-keyboard-navigation)
   - [Colour Contrast Assertion](#example-colour-contrast-assertion)
   - [Built-in Accessibility Assertions](#example-playwright-built-in-accessibility-assertions)
7. [Automated Testing with Selenium + axe-core (Java)](#7-automated-testing-with-selenium--axe-core-java)
8. [Manual Testing Checklist](#8-manual-testing-checklist)
9. [Summary](#9-summary)

---

## 1. Why Accessibility Testing Matters

- **Legal compliance** — Many countries mandate accessibility (e.g., ADA in the US, EN 301 549 in the EU, UK Equality Act).
- **Wider audience** — ~1.3 billion people worldwide live with some form of disability (WHO).
- **SEO benefits** — Semantic HTML and proper ARIA labels improve search engine indexing.
- **Better UX for everyone** — Captions benefit non-native speakers; keyboard navigation helps power users.
- **Ethical responsibility** — Excluding users due to disability is a form of discrimination.

---

## 2. WCAG Guidelines

The **Web Content Accessibility Guidelines (WCAG)**, published by the W3C, are the internationally recognised standard. Tests target one of three conformance levels:

| Level   | Meaning  | Typical Requirement                      |
|---------|----------|------------------------------------------|
| **A**   | Minimum  | Must-pass; blocks access if missing      |
| **AA**  | Standard | Required by most legal frameworks        |
| **AAA** | Enhanced | Best effort; not required for full sites |

WCAG is built on **four principles** (POUR):

| Principle          | Description                                                                                          |
|--------------------|------------------------------------------------------------------------------------------------------|
| **Perceivable**    | Information must be presentable in ways users can perceive (e.g., alt text, captions)                |
| **Operable**       | UI components must be operable by all users (e.g., keyboard navigation, no seizure-inducing content) |
| **Understandable** | Content and operation must be understandable (e.g., clear labels, predictable behaviour)             |
| **Robust**         | Content must be interpreted reliably by assistive technologies (e.g., valid HTML, ARIA roles)        |

---

## 3. Types of Accessibility Issues

| Category           | Examples                                                                  |
|--------------------|---------------------------------------------------------------------------|
| **Visual**         | Missing alt text, low colour contrast, content only conveyed by colour    |
| **Keyboard**       | Elements not reachable by Tab, missing focus indicators, keyboard traps   |
| **Screen reader**  | Missing ARIA labels, incorrect role/state, reading order issues           |
| **Forms**          | Inputs without labels, missing error descriptions, no field grouping      |
| **Motion / Media** | Auto-playing video, no captions, content that flashes > 3 times/sec       |
| **Structure**      | Skipped heading levels, missing landmarks (`<main>`, `<nav>`, `<footer>`) |
| **Timing**         | Session timeouts without warning, time-limited interactions               |

---

## 4. Testing Approaches

A robust accessibility strategy combines **three complementary approaches**:

```
          [Automated tools]       ← catches ~30–40% of issues
                  +
          [Manual testing]        ← human judgment for context and flow
                  +
          [User testing]          ← real users with disabilities
          ════════════════════
          Comprehensive coverage
```

- **Automated** — Fast, repeatable, runs in CI/CD. Cannot evaluate intent, colour meaning, or complex flows.
- **Manual** — Keyboard-only navigation, screen reader walkthroughs, zoom testing.
- **User testing** — Sessions with people using assistive technology in their daily lives.

---

## 5. Tools Overview

### Automated / Browser-Based

| Tool                                                                             | Type              | Notes                                                 |
|----------------------------------------------------------------------------------|-------------------|-------------------------------------------------------|
| [axe-core](https://github.com/dequelabs/axe-core)                                | JS library        | Industry standard engine; zero false positives policy |
| [Playwright + @axe-playwright](https://github.com/abhinaba-ghosh/axe-playwright) | E2E + a11y        | Integrate axe scans into Playwright tests             |
| [Lighthouse](https://developer.chrome.com/docs/lighthouse/)                      | CLI / DevTools    | Google's built-in auditor; gives a score + issues     |
| [WAVE](https://wave.webaim.org/)                                                 | Browser extension | Visual overlay of issues directly on the page         |
| [IBM Equal Access Checker](https://www.ibm.com/able/toolkit/tools/)              | Browser extension | Comprehensive WCAG 2.1 / 2.2 checks                   |

### Screen Readers (Manual)

| Tool      | Platform               |
|-----------|------------------------|
| NVDA      | Windows (free)         |
| JAWS      | Windows (commercial)   |
| VoiceOver | macOS / iOS (built-in) |
| TalkBack  | Android (built-in)     |

### Colour Contrast

| Tool                                                                     | Notes                          |
|--------------------------------------------------------------------------|--------------------------------|
| [Colour Contrast Analyser](https://www.tpgi.com/color-contrast-checker/) | Desktop app                    |
| [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/) | Online                         |
| Browser DevTools                                                         | Chrome/Edge CSS overview panel |

---

## 6. Automated Testing with Playwright + axe-core

### Setup

```bash
npm install --save-dev @axe-core/playwright
```

**`playwright.config.ts`** (no changes required beyond usual config):

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    baseURL: 'http://localhost:3000',
    headless: true,
  },
});
```

---

### Example: Full-Page Accessibility Scan

```typescript
// e2e/tests/accessibility.spec.ts
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test.describe('Accessibility — Home Page', () => {

  test('should have no automatically detectable WCAG 2.1 AA violations', async ({ page }) => {
    await page.goto('/');

    const results = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag21aa'])
      .analyze();

    expect(results.violations).toEqual([]);
  });

  test('should have no violations on the login page', async ({ page }) => {
    await page.goto('/login');

    const results = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa'])
      .analyze();

    // Print a readable summary if there are failures
    if (results.violations.length > 0) {
      const summary = results.violations.map((v) =>
        `[${v.impact}] ${v.id}: ${v.description}\n  Nodes: ${v.nodes.map((n) => n.html).join(', ')}`
      ).join('\n');
      console.error('Accessibility violations:\n' + summary);
    }

    expect(results.violations).toEqual([]);
  });
});
```

---

### Example: Scoping the Scan to a Component

Avoid noise from third-party widgets or cookie banners by scoping the scan to a specific region:

```typescript
// e2e/tests/accessibility.spec.ts
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('checkout form should be accessible', async ({ page }) => {
  await page.goto('/checkout');

  const results = await new AxeBuilder({ page })
    .include('#checkout-form')          // scope scan to this element
    .withTags(['wcag2a', 'wcag2aa'])
    .analyze();

  expect(results.violations).toEqual([]);
});

test('navigation menu should be accessible', async ({ page }) => {
  await page.goto('/');

  const results = await new AxeBuilder({ page })
    .include('nav[aria-label="Main navigation"]')
    .withTags(['wcag2a', 'wcag2aa'])
    .analyze();

  expect(results.violations).toEqual([]);
});
```

---

### Example: Excluding Known Issues (Disable Rules)

When you have a known third-party violation you cannot fix immediately, exclude specific rules and track them as tech debt:

```typescript
test('product page should pass accessibility checks (excluding known vendor issues)', async ({ page }) => {
  await page.goto('/products/1');

  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa'])
    .disableRules(['color-contrast'])   // ⚠️ tracked in issue #42 — vendor widget
    .analyze();

  expect(results.violations).toEqual([]);
});
```

---

### Example: Keyboard Navigation

Automated scanners cannot validate logical keyboard flow. Use Playwright to simulate keyboard-only usage:

```typescript
// e2e/tests/keyboard-navigation.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Keyboard Navigation', () => {

  test('should navigate the main menu with Tab and Enter', async ({ page }) => {
    await page.goto('/');

    // Tab to the first nav link and confirm focus
    await page.keyboard.press('Tab');
    const firstLink = page.getByRole('link', { name: 'Home' });
    await expect(firstLink).toBeFocused();

    // Tab to next item
    await page.keyboard.press('Tab');
    const secondLink = page.getByRole('link', { name: 'Products' });
    await expect(secondLink).toBeFocused();

    // Activate with Enter
    await page.keyboard.press('Enter');
    await expect(page).toHaveURL('/products');
  });

  test('modal should trap focus when open', async ({ page }) => {
    await page.goto('/products');
    await page.getByRole('button', { name: 'Filter' }).click();

    const modal = page.getByRole('dialog', { name: 'Filter Products' });
    await expect(modal).toBeVisible();

    // Tab through all focusable elements and check focus stays in modal
    for (let i = 0; i < 5; i++) {
      await page.keyboard.press('Tab');
      const focusedElement = page.locator(':focus');
      // Focused element must be inside the modal
      await expect(modal.locator(':focus')).toBeAttached();
    }

    // Escape should close the modal
    await page.keyboard.press('Escape');
    await expect(modal).not.toBeVisible();
  });

  test('skip-to-main-content link should be present and functional', async ({ page }) => {
    await page.goto('/');

    // The skip link is typically visually hidden until focused
    await page.keyboard.press('Tab');
    const skipLink = page.getByRole('link', { name: /skip to (main )?content/i });
    await expect(skipLink).toBeFocused();

    await page.keyboard.press('Enter');
    const main = page.getByRole('main');
    await expect(main).toBeFocused();
  });
});
```

---

### Example: Colour Contrast Assertion

```typescript
// e2e/tests/colour-contrast.spec.ts
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('all text should meet WCAG AA colour contrast ratio (4.5:1)', async ({ page }) => {
  await page.goto('/');

  const results = await new AxeBuilder({ page })
    .withRules(['color-contrast'])
    .analyze();

  expect(results.violations).toEqual([]);
});
```

---

### Example: Playwright Built-in Accessibility Assertions

Playwright ships with native accessibility matchers — no extra packages required. These test the **accessible name, role, description, and ARIA tree** of elements directly.

```typescript
// e2e/tests/aria-assertions.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Built-in Playwright Accessibility Assertions', () => {

  // ─── Role & Accessible Name ──────────────────────────────────────────────

  test('primary CTA button should have correct role and accessible name', async ({ page }) => {
    await page.goto('/');

    const cta = page.getByTestId('hero-cta');

    await expect(cta).toHaveRole('button');
    await expect(cta).toHaveAccessibleName('Get Started');
  });

  test('logo image should have a meaningful accessible name', async ({ page }) => {
    await page.goto('/');

    const logo = page.getByRole('img', { name: /company logo/i });

    await expect(logo).toHaveRole('img');
    await expect(logo).toHaveAccessibleName(/company logo/i);
  });

  test('navigation landmark should be labelled', async ({ page }) => {
    await page.goto('/');

    const nav = page.getByRole('navigation');

    await expect(nav).toHaveAccessibleName('Main navigation');
  });

  // ─── Accessible Description ───────────────────────────────────────────────

  test('password input should have an accessible description', async ({ page }) => {
    await page.goto('/register');

    const passwordInput = page.getByLabel('Password');

    // aria-describedby should point to the hint text
    await expect(passwordInput).toHaveAccessibleDescription(
      /at least 8 characters/i
    );
  });

  test('icon-only button should have an accessible name via aria-label', async ({ page }) => {
    await page.goto('/dashboard');

    const closeButton = page.getByTestId('close-notification');

    await expect(closeButton).toHaveRole('button');
    await expect(closeButton).toHaveAccessibleName('Close notification');
  });

  // ─── Form Inputs ─────────────────────────────────────────────────────────

  test('all form inputs on the contact page should be labelled', async ({ page }) => {
    await page.goto('/contact');

    const inputs = page.getByRole('textbox');
    const count  = await inputs.count();

    for (let i = 0; i < count; i++) {
      // Every text input must have a non-empty accessible name
      await expect(inputs.nth(i)).not.toHaveAccessibleName('');
    }
  });

  test('required fields should be marked as such', async ({ page }) => {
    await page.goto('/register');

    const emailInput = page.getByLabel('Email');

    // The input must carry aria-required="true" or required attribute
    await expect(emailInput).toHaveAttribute('required');
  });

  // ─── ARIA Snapshot (Playwright ≥ 1.44) ───────────────────────────────────

  test('product card ARIA structure should match snapshot', async ({ page }) => {
    await page.goto('/products');

    const firstCard = page.getByTestId('product-card').first();

    // Captures the ARIA tree of the component and compares it to a stored
    // snapshot — fails if roles, names, or hierarchy change unexpectedly.
    await expect(firstCard).toMatchAriaSnapshot(`
      - article "Laptop Pro":
        - img "Laptop Pro product image"
        - heading "Laptop Pro" [level=3]
        - text: €999.99
        - button "Add to Cart"
    `);
  });

  test('main navigation ARIA snapshot should be stable', async ({ page }) => {
    await page.goto('/');

    const nav = page.getByRole('navigation', { name: 'Main navigation' });

    await expect(nav).toMatchAriaSnapshot(`
      - navigation "Main navigation":
        - list:
          - listitem:
            - link "Home"
          - listitem:
            - link "Products"
          - listitem:
            - link "About"
          - listitem:
            - link "Contact"
    `);
  });
});
```

> **When to use which approach:**
>
> | Playwright Feature | Best For |
> |--------------------|----------|
> | `toHaveRole()` | Verify semantic element roles |
> | `toHaveAccessibleName()` | Verify labels, `aria-label`, `aria-labelledby` |
> | `toHaveAccessibleDescription()` | Verify hints, `aria-describedby` |
> | `toMatchAriaSnapshot()` | Detect regressions in full ARIA tree structure |
> | **axe-core** (`AxeBuilder`) | Catch a wide range of WCAG rule violations automatically |

---

## 7. Automated Testing with Selenium + axe-core (Java)

### Setup

**Maven dependency:**

```xml
<dependency>
    <groupId>com.deque.html.axe-core</groupId>
    <artifactId>selenium</artifactId>
    <version>4.10.1</version>
    <scope>test</scope>
</dependency>
```

---

### Example: axe-core with Selenium WebDriver

```java
// src/test/java/com/example/accessibility/HomePageAccessibilityTest.java
import com.deque.html.axecore.results.Results;
import com.deque.html.axecore.results.Rule;
import com.deque.html.axecore.selenium.AxeBuilder;
import org.junit.jupiter.api.*;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;

import java.util.List;

import static org.assertj.core.api.Assertions.*;

class HomePageAccessibilityTest {

    private WebDriver driver;

    @BeforeEach
    void setUp() {
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--headless", "--no-sandbox", "--disable-dev-shm-usage");
        driver = new ChromeDriver(options);
    }

    @AfterEach
    void tearDown() {
        if (driver != null) driver.quit();
    }

    @Test
    @DisplayName("Home page should have no WCAG 2.1 AA violations")
    void homePageShouldHaveNoAccessibilityViolations() {
        driver.get("http://localhost:3000/");

        Results results = new AxeBuilder()
            .withTags(List.of("wcag2a", "wcag2aa", "wcag21aa"))
            .analyze(driver);

        List<Rule> violations = results.getViolations();

        if (!violations.isEmpty()) {
            String report = violations.stream()
                .map(v -> String.format("[%s] %s — %s", v.getImpact(), v.getId(), v.getDescription()))
                .reduce("", (a, b) -> a + "\n" + b);
            fail("Accessibility violations found:\n" + report);
        }

        assertThat(violations).isEmpty();
    }

    @Test
    @DisplayName("Login page form should be accessible")
    void loginFormShouldBeAccessible() {
        driver.get("http://localhost:3000/login");

        Results results = new AxeBuilder()
            .include(List.of("#login-form"))          // scope to form element
            .withTags(List.of("wcag2a", "wcag2aa"))
            .analyze(driver);

        assertThat(results.getViolations())
            .as("Login form accessibility violations")
            .isEmpty();
    }
}
```

---

### Example: Parameterized Multi-Page Scan

```java
// src/test/java/com/example/accessibility/SiteWideAccessibilityTest.java
import com.deque.html.axecore.selenium.AxeBuilder;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;

import static org.assertj.core.api.Assertions.*;

class SiteWideAccessibilityTest extends AccessibilityTestBase {

    @ParameterizedTest(name = "Page ''{0}'' should have no WCAG AA violations")
    @ValueSource(strings = { "/", "/login", "/products", "/about", "/contact" })
    void pageShouldHaveNoAccessibilityViolations(String path) {
        driver.get("http://localhost:3000" + path);

        var violations = new AxeBuilder()
            .withTags(List.of("wcag2a", "wcag2aa"))
            .analyze(driver)
            .getViolations();

        assertThat(violations)
            .as("Violations on page: " + path)
            .isEmpty();
    }
}
```

---

## 8. Manual Testing Checklist

Automated tools catch roughly **30–40%** of accessibility issues. Use this checklist for the remaining checks.

### Keyboard Navigation
- [ ] Every interactive element (links, buttons, inputs, modals) is reachable with **Tab** / **Shift+Tab**
- [ ] Focus order is logical and follows visual reading order
- [ ] Focus indicator is clearly visible on all interactive elements
- [ ] No keyboard traps (user can always Tab out of any component)
- [ ] **Escape** closes modals, dropdowns, and overlays
- [ ] Custom widgets (date pickers, sliders) follow [ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/)

### Screen Reader
- [ ] Page has a descriptive `<title>`
- [ ] Headings form a logical hierarchy (`h1` → `h2` → `h3` — no skipped levels)
- [ ] All images have meaningful `alt` attributes (decorative images use `alt=""`)
- [ ] Form inputs are associated with labels (`<label for>` or `aria-label`)
- [ ] Error messages are announced (use `role="alert"` or `aria-live`)
- [ ] Links and buttons have descriptive text (avoid "click here", "read more")
- [ ] Dynamic content changes are announced (`aria-live` regions)
- [ ] Tables have `<caption>` and `<th>` elements with `scope`

### Visual
- [ ] Text contrast ratio ≥ **4.5:1** (normal text) or **3:1** (large text / UI components)
- [ ] Information is not conveyed by colour alone (error states use icons + text)
- [ ] Content is readable and functional at **200% zoom**
- [ ] No content is lost when text size is increased to **200%**
- [ ] Page is usable in **high contrast mode** (Windows / macOS)

### Forms
- [ ] All fields have visible labels
- [ ] Required fields are clearly indicated
- [ ] Error messages identify the specific field and describe the problem
- [ ] Success confirmation is announced to screen readers
- [ ] Autocomplete attributes are set correctly (`autocomplete="email"`, etc.)

### Media
- [ ] Videos have **captions** (auto-generated captions alone are insufficient)
- [ ] Audio content has a **transcript**
- [ ] No content auto-plays audio without user consent
- [ ] No content **flashes more than 3 times per second**

---

## 9. Summary

| Approach                          | Coverage           | Speed       | Effort    | When to Use             |
|-----------------------------------|--------------------|-------------|-----------|-------------------------|
| **Automated (axe-core)**          | ~30–40% of issues  | ⚡ Fast      | Low       | Every CI/CD run         |
| **Keyboard testing**              | Navigation & focus | 🐢 Moderate | Medium    | Sprint review / release |
| **Screen reader testing**         | Semantics & flow   | 🐢 Moderate | High      | Key user journeys       |
| **Colour contrast audit**         | Visual contrast    | ⚡ Fast      | Low       | Design review + CI      |
| **User testing (disabled users)** | Real-world impact  | 🐢🐢 Slow   | Very high | Major releases          |

> **Recommended workflow:**
> 1. Run **axe-core scans** in CI to catch regressions automatically.
> 2. Do a **keyboard walkthrough** of every new feature before merging.
> 3. Schedule a **screen reader session** for each major user journey at least once per quarter.
> 4. Involve **users with disabilities** in usability testing at least once per release cycle.

### Further Reading

- [WCAG 2.2 Quick Reference](https://www.w3.org/WAI/WCAG22/quickref/)
- [ARIA Authoring Practices Guide (APG)](https://www.w3.org/WAI/ARIA/apg/)
- [WebAIM — Introduction to Web Accessibility](https://webaim.org/intro/)
- [Deque University — axe-core](https://www.deque.com/axe/)
- [Playwright Accessibility Testing](https://playwright.dev/docs/accessibility-testing)

