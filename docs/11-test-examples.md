# Test Examples

This document provides practical, real-world examples of **Unit Tests** (Java), **Integration Tests** (Java), and **End-to-End Tests** (Playwright).

---

## Table of Contents

1. [Unit Tests — Java](#1-unit-tests--java)
   - [Setup](#setup)
   - [Example: Testing a Service Class](#example-testing-a-service-class)
   - [Example: Testing Utility Methods](#example-testing-utility-methods)
   - [Example: Testing with Mocks (Mockito)](#example-testing-with-mocks-mockito)
2. [Integration Tests — Java](#2-integration-tests--java)
   - [Setup](#setup-1)
   - [Example: Testing a REST Controller (Spring Boot)](#example-testing-a-rest-controller-spring-boot)
   - [Example: Testing the Repository Layer (JPA + H2)](#example-testing-the-repository-layer-jpa--h2)
3. [Testcontainers — Java](#3-testcontainers--java)
   - [Setup](#setup-2)
   - [Example: Testing with a Real PostgreSQL Container](#example-testing-with-a-real-postgresql-container)
   - [Example: Testing with Redis Container](#example-testing-with-redis-container)
   - [Example: Reusable Container Configuration](#example-reusable-container-configuration)
4. [End-to-End Tests — Playwright](#4-end-to-end-tests--playwright)
   - [Setup](#setup-2)
   - [Example: Login Flow](#example-login-flow)
   - [Example: Form Submission](#example-form-submission)
   - [Example: API Mocking](#example-api-mocking)

---

## 1. Unit Tests — Java

Unit tests validate a **single unit of logic** (a method or class) in complete isolation from external dependencies such as databases, APIs, or file systems.

### Setup

**Dependencies (`pom.xml` — Maven):**

```xml
<dependencies>
    <!-- JUnit 5 -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.10.2</version>
        <scope>test</scope>
    </dependency>

    <!-- Mockito -->
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-core</artifactId>
        <version>5.11.0</version>
        <scope>test</scope>
    </dependency>

    <!-- AssertJ (fluent assertions) -->
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <version>3.25.3</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

---

### Example: Testing a Service Class

**Production code:**

```java
// src/main/java/com/example/shop/OrderService.java
public class OrderService {

    public double calculateTotal(List<Double> prices, double discountPercent) {
        if (prices == null || prices.isEmpty()) {
            throw new IllegalArgumentException("Price list must not be empty");
        }
        double subtotal = prices.stream().mapToDouble(Double::doubleValue).sum();
        return subtotal - (subtotal * discountPercent / 100);
    }

    public boolean isEligibleForFreeShipping(double orderTotal) {
        return orderTotal >= 50.0;
    }
}
```

**Test code:**

```java
// src/test/java/com/example/shop/OrderServiceTest.java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;

import java.util.List;

import static org.assertj.core.api.Assertions.*;

class OrderServiceTest {

    private OrderService orderService;

    @BeforeEach
    void setUp() {
        orderService = new OrderService();
    }

    @Test
    @DisplayName("Should calculate total with discount correctly")
    void shouldCalculateTotalWithDiscount() {
        List<Double> prices = List.of(10.0, 20.0, 30.0); // subtotal = 60.0
        double discount = 10.0; // 10%

        double total = orderService.calculateTotal(prices, discount);

        assertThat(total).isEqualTo(54.0);
    }

    @Test
    @DisplayName("Should throw exception when price list is empty")
    void shouldThrowExceptionWhenPriceListIsEmpty() {
        assertThatThrownBy(() -> orderService.calculateTotal(List.of(), 0))
            .isInstanceOf(IllegalArgumentException.class)
            .hasMessage("Price list must not be empty");
    }

    @Test
    @DisplayName("Should throw exception when price list is null")
    void shouldThrowExceptionWhenPriceListIsNull() {
        assertThatThrownBy(() -> orderService.calculateTotal(null, 0))
            .isInstanceOf(IllegalArgumentException.class);
    }

    @ParameterizedTest(name = "Total {0} → free shipping: {1}")
    @CsvSource({
        "49.99, false",
        "50.00, true",
        "100.00, true"
    })
    @DisplayName("Should determine free shipping eligibility correctly")
    void shouldDetermineShippingEligibility(double total, boolean expected) {
        assertThat(orderService.isEligibleForFreeShipping(total)).isEqualTo(expected);
    }
}
```

---

### Example: Testing Utility Methods

**Production code:**

```java
// src/main/java/com/example/utils/StringUtils.java
public class StringUtils {

    public static String capitalize(String input) {
        if (input == null || input.isBlank()) return input;
        return Character.toUpperCase(input.charAt(0)) + input.substring(1).toLowerCase();
    }

    public static boolean isPalindrome(String input) {
        if (input == null) return false;
        String cleaned = input.replaceAll("\\s+", "").toLowerCase();
        return cleaned.equals(new StringBuilder(cleaned).reverse().toString());
    }
}
```

**Test code:**

```java
// src/test/java/com/example/utils/StringUtilsTest.java
import org.junit.jupiter.api.Test;

import static org.assertj.core.api.Assertions.*;

class StringUtilsTest {

    @Test
    void shouldCapitalizeFirstLetter() {
        assertThat(StringUtils.capitalize("hello")).isEqualTo("Hello");
    }

    @Test
    void shouldReturnNullWhenInputIsNull() {
        assertThat(StringUtils.capitalize(null)).isNull();
    }

    @Test
    void shouldReturnBlankStringUnchanged() {
        assertThat(StringUtils.capitalize("   ")).isEqualTo("   ");
    }

    @Test
    void shouldDetectPalindrome() {
        assertThat(StringUtils.isPalindrome("racecar")).isTrue();
        assertThat(StringUtils.isPalindrome("race car")).isTrue(); // ignores spaces
    }

    @Test
    void shouldReturnFalseForNonPalindrome() {
        assertThat(StringUtils.isPalindrome("hello")).isFalse();
    }
}
```

---

### Example: Testing with Mocks (Mockito)

**Production code:**

```java
// src/main/java/com/example/shop/UserService.java
public class UserService {

    private final UserRepository userRepository;
    private final EmailService emailService;

    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }

    public User registerUser(String email, String name) {
        if (userRepository.existsByEmail(email)) {
            throw new IllegalStateException("Email already in use: " + email);
        }
        User user = new User(email, name);
        userRepository.save(user);
        emailService.sendWelcomeEmail(email, name);
        return user;
    }
}
```

**Test code:**

```java
// src/test/java/com/example/shop/UserServiceTest.java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.assertj.core.api.Assertions.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @Mock
    private EmailService emailService;

    @InjectMocks
    private UserService userService;

    @Test
    void shouldRegisterUserSuccessfully() {
        when(userRepository.existsByEmail("jane@example.com")).thenReturn(false);

        User result = userService.registerUser("jane@example.com", "Jane");

        assertThat(result.getEmail()).isEqualTo("jane@example.com");
        verify(userRepository).save(any(User.class));
        verify(emailService).sendWelcomeEmail("jane@example.com", "Jane");
    }

    @Test
    void shouldThrowExceptionWhenEmailAlreadyExists() {
        when(userRepository.existsByEmail("jane@example.com")).thenReturn(true);

        assertThatThrownBy(() -> userService.registerUser("jane@example.com", "Jane"))
            .isInstanceOf(IllegalStateException.class)
            .hasMessageContaining("Email already in use");

        verify(userRepository, never()).save(any());
        verify(emailService, never()).sendWelcomeEmail(anyString(), anyString());
    }
}
```

---

## 2. Integration Tests — Java

Integration tests validate how multiple components work together, typically involving a **real (or embedded) database**, HTTP stack, or other infrastructure.

### Setup

**Dependencies (`pom.xml`):**

```xml
<dependencies>
    <!-- Spring Boot Test (includes JUnit 5, Mockito, AssertJ) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <!-- H2 in-memory database for tests -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

**Test application properties (`src/test/resources/application-test.yml`):**

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1
    driver-class-name: org.h2.Driver
    username: sa
    password:
  jpa:
    hibernate:
      ddl-auto: create-drop
    database-platform: org.hibernate.dialect.H2Dialect
```

---

### Example: Testing a REST Controller (Spring Boot)

**Production code:**

```java
// src/main/java/com/example/shop/ProductController.java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    private final ProductService productService;

    public ProductController(ProductService productService) {
        this.productService = productService;
    }

    @GetMapping("/{id}")
    public ResponseEntity<Product> getById(@PathVariable Long id) {
        return productService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Product create(@RequestBody @Valid ProductRequest request) {
        return productService.create(request);
    }
}
```

**Test code:**

```java
// src/test/java/com/example/shop/ProductControllerIntegrationTest.java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import static org.hamcrest.Matchers.*;

@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
class ProductControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ProductRepository productRepository;

    @Test
    void shouldReturnProductWhenItExists() throws Exception {
        Product saved = productRepository.save(new Product("Laptop", 999.99));

        mockMvc.perform(get("/api/products/" + saved.getId()))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name", is("Laptop")))
            .andExpect(jsonPath("$.price", is(999.99)));
    }

    @Test
    void shouldReturn404WhenProductDoesNotExist() throws Exception {
        mockMvc.perform(get("/api/products/99999"))
            .andExpect(status().isNotFound());
    }

    @Test
    void shouldCreateProductAndReturn201() throws Exception {
        String requestBody = """
            {
                "name": "Mechanical Keyboard",
                "price": 149.99
            }
            """;

        mockMvc.perform(post("/api/products")
                .contentType(MediaType.APPLICATION_JSON)
                .content(requestBody))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id", notNullValue()))
            .andExpect(jsonPath("$.name", is("Mechanical Keyboard")));
    }

    @Test
    void shouldReturn400WhenRequestBodyIsInvalid() throws Exception {
        String invalidBody = """
            {
                "name": "",
                "price": -10
            }
            """;

        mockMvc.perform(post("/api/products")
                .contentType(MediaType.APPLICATION_JSON)
                .content(invalidBody))
            .andExpect(status().isBadRequest());
    }
}
```

---

### Example: Testing the Repository Layer (JPA + H2)

**Production code:**

```java
// src/main/java/com/example/shop/OrderRepository.java
public interface OrderRepository extends JpaRepository<Order, Long> {
    List<Order> findByCustomerEmail(String email);
    long countByStatus(OrderStatus status);
}
```

**Test code:**

```java
// src/test/java/com/example/shop/OrderRepositoryIntegrationTest.java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.ActiveProfiles;

import java.util.List;

import static org.assertj.core.api.Assertions.*;

@DataJpaTest        // spins up only JPA context + H2
@ActiveProfiles("test")
class OrderRepositoryIntegrationTest {

    @Autowired
    private OrderRepository orderRepository;

    @BeforeEach
    void setUp() {
        orderRepository.deleteAll();
        orderRepository.saveAll(List.of(
            new Order("alice@example.com", OrderStatus.PENDING),
            new Order("alice@example.com", OrderStatus.SHIPPED),
            new Order("bob@example.com",   OrderStatus.PENDING)
        ));
    }

    @Test
    void shouldFindOrdersByCustomerEmail() {
        List<Order> orders = orderRepository.findByCustomerEmail("alice@example.com");

        assertThat(orders).hasSize(2);
        assertThat(orders).extracting(Order::getCustomerEmail)
            .containsOnly("alice@example.com");
    }

    @Test
    void shouldCountOrdersByStatus() {
        long pendingCount = orderRepository.countByStatus(OrderStatus.PENDING);

        assertThat(pendingCount).isEqualTo(2);
    }

    @Test
    void shouldReturnEmptyListForUnknownEmail() {
        List<Order> orders = orderRepository.findByCustomerEmail("unknown@example.com");

        assertThat(orders).isEmpty();
    }
}
```

---

## 3. Testcontainers — Java

Testcontainers is a Java library that spins up **real Docker containers** (PostgreSQL, Redis, Kafka, etc.) during tests. Unlike H2, you test against the exact same database engine used in production.

### Setup

**Dependencies (`pom.xml`):**

```xml
<dependencies>
    <!-- Testcontainers BOM (manages versions) -->
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>testcontainers-bom</artifactId>
        <version>1.19.7</version>
        <type>pom</type>
        <scope>import</scope>
    </dependency>

    <!-- Core -->
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>testcontainers</artifactId>
        <scope>test</scope>
    </dependency>

    <!-- PostgreSQL module -->
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>postgresql</artifactId>
        <scope>test</scope>
    </dependency>

    <!-- Redis module -->
    <dependency>
        <groupId>com.redis</groupId>
        <artifactId>testcontainers-redis</artifactId>
        <version>2.2.2</version>
        <scope>test</scope>
    </dependency>

    <!-- JUnit 5 integration -->
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>junit-jupiter</artifactId>
        <scope>test</scope>
    </dependency>

    <!-- Spring Boot Test -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

> **Prerequisite:** Docker must be running on the machine executing the tests.

---

### Example: Testing with a Real PostgreSQL Container

**Production code:**

```java
// src/main/java/com/example/shop/ProductRepository.java
public interface ProductRepository extends JpaRepository<Product, Long> {
    List<Product> findByCategory(String category);
    Optional<Product> findByNameIgnoreCase(String name);
}
```

**Test code:**

```java
// src/test/java/com/example/shop/ProductRepositoryContainerTest.java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.jdbc.AutoConfigureTestDatabase;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

import java.util.List;

import static org.assertj.core.api.Assertions.*;

@DataJpaTest
@Testcontainers
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE) // don't replace with H2
class ProductRepositoryContainerTest {

    // Container is started once and shared across all tests in this class
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine")
        .withDatabaseName("shop_test")
        .withUsername("test")
        .withPassword("test");

    // Wire the container's dynamic URL/credentials into Spring's datasource config
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url",      postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private ProductRepository productRepository;

    @BeforeEach
    void setUp() {
        productRepository.deleteAll();
        productRepository.saveAll(List.of(
            new Product("Laptop",   "Electronics", 999.99),
            new Product("Phone",    "Electronics", 699.99),
            new Product("T-Shirt",  "Clothing",     29.99)
        ));
    }

    @Test
    void shouldFindProductsByCategory() {
        List<Product> electronics = productRepository.findByCategory("Electronics");

        assertThat(electronics).hasSize(2);
        assertThat(electronics).extracting(Product::getName)
            .containsExactlyInAnyOrder("Laptop", "Phone");
    }

    @Test
    void shouldFindProductByNameCaseInsensitive() {
        var product = productRepository.findByNameIgnoreCase("laptop");

        assertThat(product).isPresent();
        assertThat(product.get().getPrice()).isEqualTo(999.99);
    }

    @Test
    void shouldReturnEmptyWhenCategoryHasNoProducts() {
        List<Product> products = productRepository.findByCategory("Books");

        assertThat(products).isEmpty();
    }
}
```

---

### Example: Testing with Redis Container

**Production code:**

```java
// src/main/java/com/example/cache/SessionService.java
@Service
public class SessionService {

    private final StringRedisTemplate redisTemplate;
    private static final Duration SESSION_TTL = Duration.ofMinutes(30);

    public SessionService(StringRedisTemplate redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

    public void createSession(String sessionId, String userId) {
        redisTemplate.opsForValue().set("session:" + sessionId, userId, SESSION_TTL);
    }

    public Optional<String> getSession(String sessionId) {
        return Optional.ofNullable(redisTemplate.opsForValue().get("session:" + sessionId));
    }

    public void invalidateSession(String sessionId) {
        redisTemplate.delete("session:" + sessionId);
    }
}
```

**Test code:**

```java
// src/test/java/com/example/cache/SessionServiceContainerTest.java
import com.redis.testcontainers.RedisContainer;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;
import org.testcontainers.utility.DockerImageName;

import static org.assertj.core.api.Assertions.*;

@SpringBootTest
@Testcontainers
class SessionServiceContainerTest {

    @Container
    static RedisContainer redis = new RedisContainer(
        DockerImageName.parse("redis:7-alpine")
    );

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.data.redis.host", redis::getHost);
        registry.add("spring.data.redis.port", redis::getFirstMappedPort);
    }

    @Autowired
    private SessionService sessionService;

    @AfterEach
    void tearDown() {
        // Clean up Redis state between tests
        sessionService.invalidateSession("session-abc");
        sessionService.invalidateSession("session-xyz");
    }

    @Test
    void shouldCreateAndRetrieveSession() {
        sessionService.createSession("session-abc", "user-42");

        var result = sessionService.getSession("session-abc");

        assertThat(result).isPresent().contains("user-42");
    }

    @Test
    void shouldReturnEmptyForNonExistentSession() {
        var result = sessionService.getSession("session-xyz");

        assertThat(result).isEmpty();
    }

    @Test
    void shouldInvalidateSession() {
        sessionService.createSession("session-abc", "user-42");
        sessionService.invalidateSession("session-abc");

        var result = sessionService.getSession("session-abc");

        assertThat(result).isEmpty();
    }
}
```

---

### Example: Reusable Container Configuration

For larger projects with many test classes, starting a new container per class is slow. Use a **shared base class** or a **singleton pattern** to reuse a single container across the entire test suite.

```java
// src/test/java/com/example/config/PostgresContainerBase.java
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.containers.PostgreSQLContainer;

/**
 * Base class that starts a single PostgreSQL container for the whole test suite.
 * Extend this class in any integration test that needs a real database.
 */
public abstract class PostgresContainerBase {

    // static → container is started once and shared across all subclasses
    static final PostgreSQLContainer<?> POSTGRES;

    static {
        POSTGRES = new PostgreSQLContainer<>("postgres:16-alpine")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test")
            .withReuse(true); // reuse across Gradle/Maven runs (requires ~/.testcontainers.properties)
        POSTGRES.start();
    }

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url",      POSTGRES::getJdbcUrl);
        registry.add("spring.datasource.username", POSTGRES::getUsername);
        registry.add("spring.datasource.password", POSTGRES::getPassword);
    }
}
```

**Usage in test classes:**

```java
// src/test/java/com/example/shop/OrderRepositoryTest.java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class OrderRepositoryTest extends PostgresContainerBase {

    @Autowired
    private OrderRepository orderRepository;

    @Test
    void shouldPersistAndRetrieveOrder() {
        Order saved = orderRepository.save(new Order("alice@example.com", OrderStatus.PENDING));

        assertThat(orderRepository.findById(saved.getId())).isPresent();
    }
}

// src/test/java/com/example/shop/ProductRepositoryTest.java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class ProductRepositoryTest extends PostgresContainerBase {

    @Autowired
    private ProductRepository productRepository;

    @Test
    void shouldSaveProduct() {
        Product saved = productRepository.save(new Product("Monitor", "Electronics", 399.99));

        assertThat(saved.getId()).isNotNull();
    }
}
```

> **Enable container reuse** by adding the following to `~/.testcontainers.properties`:
> ```properties
> testcontainers.reuse.enable=true
> ```
> This keeps the container alive between test runs, dramatically speeding up local development feedback loops.

---

## 4. End-to-End Tests — Playwright

End-to-end (E2E) tests simulate real user interactions in an actual browser, validating the entire stack from UI to backend.

### Setup

**Install Playwright (Node.js):**

```bash
npm init playwright@latest
```

**Project structure:**

```
e2e/
├── tests/
│   ├── login.spec.ts
│   ├── checkout.spec.ts
│   └── api-mock.spec.ts
├── playwright.config.ts
└── package.json
```

**`playwright.config.ts`:**

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  timeout: 30_000,
  retries: 1,
  use: {
    baseURL: 'http://localhost:3000',
    headless: true,
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox',  use: { ...devices['Desktop Firefox'] } },
  ],
});
```

---

### Example: Login Flow

```typescript
// e2e/tests/login.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Login Page', () => {

  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
  });

  test('should log in with valid credentials', async ({ page }) => {
    await page.getByLabel('Email').fill('user@example.com');
    await page.getByLabel('Password').fill('secret123');
    await page.getByRole('button', { name: 'Sign In' }).click();

    await expect(page).toHaveURL('/dashboard');
    await expect(page.getByRole('heading', { name: 'Welcome back' })).toBeVisible();
  });

  test('should show error message for wrong password', async ({ page }) => {
    await page.getByLabel('Email').fill('user@example.com');
    await page.getByLabel('Password').fill('wrongpassword');
    await page.getByRole('button', { name: 'Sign In' }).click();

    await expect(page.getByText('Invalid email or password')).toBeVisible();
    await expect(page).toHaveURL('/login'); // stays on login page
  });

  test('should show validation errors when fields are empty', async ({ page }) => {
    await page.getByRole('button', { name: 'Sign In' }).click();

    await expect(page.getByText('Email is required')).toBeVisible();
    await expect(page.getByText('Password is required')).toBeVisible();
  });

  test('should navigate to forgot password page', async ({ page }) => {
    await page.getByRole('link', { name: 'Forgot password?' }).click();

    await expect(page).toHaveURL('/forgot-password');
  });
});
```

---

### Example: Form Submission

```typescript
// e2e/tests/checkout.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Checkout Flow', () => {

  test.beforeEach(async ({ page }) => {
    // Seed session / login state
    await page.goto('/login');
    await page.getByLabel('Email').fill('user@example.com');
    await page.getByLabel('Password').fill('secret123');
    await page.getByRole('button', { name: 'Sign In' }).click();
    await page.waitForURL('/dashboard');
  });

  test('should complete a full checkout successfully', async ({ page }) => {
    // Add item to cart
    await page.goto('/products');
    await page.getByTestId('product-card-1').getByRole('button', { name: 'Add to Cart' }).click();
    await expect(page.getByTestId('cart-count')).toHaveText('1');

    // Go to checkout
    await page.getByRole('link', { name: 'Checkout' }).click();
    await expect(page).toHaveURL('/checkout');

    // Fill shipping details
    await page.getByLabel('Full Name').fill('Jane Doe');
    await page.getByLabel('Address').fill('123 Main Street');
    await page.getByLabel('City').fill('Lisbon');
    await page.getByLabel('Postal Code').fill('1000-001');

    // Fill payment details
    await page.getByLabel('Card Number').fill('4242424242424242');
    await page.getByLabel('Expiry').fill('12/26');
    await page.getByLabel('CVV').fill('123');

    // Submit order
    await page.getByRole('button', { name: 'Place Order' }).click();

    // Assert confirmation
    await expect(page).toHaveURL(/\/orders\/\d+\/confirmation/);
    await expect(page.getByRole('heading', { name: 'Order Confirmed!' })).toBeVisible();
    await expect(page.getByTestId('order-id')).not.toBeEmpty();
  });

  test('should show error when card is declined', async ({ page }) => {
    await page.goto('/checkout');

    await page.getByLabel('Card Number').fill('4000000000000002'); // decline card
    await page.getByLabel('Expiry').fill('12/26');
    await page.getByLabel('CVV').fill('123');

    await page.getByRole('button', { name: 'Place Order' }).click();

    await expect(page.getByText('Your card was declined')).toBeVisible();
  });

  test('should display correct order summary', async ({ page }) => {
    await page.goto('/cart');

    const itemNames  = page.getByTestId('cart-item-name');
    const itemPrices = page.getByTestId('cart-item-price');

    await expect(itemNames.first()).toBeVisible();
    await expect(itemPrices.first()).toContainText('€');

    const totalText = await page.getByTestId('cart-total').innerText();
    expect(Number(totalText.replace(/[^0-9.]/g, ''))).toBeGreaterThan(0);
  });
});
```

---

### Example: API Mocking

Playwright can intercept network requests to mock API responses, making tests fast and deterministic.

```typescript
// e2e/tests/api-mock.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Product Listing with API Mocking', () => {

  test('should display products returned by the API', async ({ page }) => {
    // Intercept the products API call and return mock data
    await page.route('**/api/products', async (route) => {
      await route.fulfill({
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify([
          { id: 1, name: 'Mocked Laptop',   price: 999.99 },
          { id: 2, name: 'Mocked Keyboard', price: 149.99 },
        ]),
      });
    });

    await page.goto('/products');

    await expect(page.getByText('Mocked Laptop')).toBeVisible();
    await expect(page.getByText('Mocked Keyboard')).toBeVisible();
    await expect(page.getByTestId('product-card')).toHaveCount(2);
  });

  test('should show empty state when API returns no products', async ({ page }) => {
    await page.route('**/api/products', async (route) => {
      await route.fulfill({
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify([]),
      });
    });

    await page.goto('/products');

    await expect(page.getByText('No products available')).toBeVisible();
    await expect(page.getByTestId('product-card')).toHaveCount(0);
  });

  test('should show error banner when API fails', async ({ page }) => {
    await page.route('**/api/products', async (route) => {
      await route.fulfill({
        status: 500,
        contentType: 'application/json',
        body: JSON.stringify({ message: 'Internal Server Error' }),
      });
    });

    await page.goto('/products');

    await expect(page.getByRole('alert')).toBeVisible();
    await expect(page.getByText('Something went wrong')).toBeVisible();
  });

  test('should handle slow API responses gracefully', async ({ page }) => {
    await page.route('**/api/products', async (route) => {
      await new Promise((resolve) => setTimeout(resolve, 2000)); // simulate 2s delay
      await route.fulfill({
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify([{ id: 1, name: 'Slow Product', price: 10.00 }]),
      });
    });

    await page.goto('/products');

    // Loading skeleton should be visible while waiting
    await expect(page.getByTestId('loading-skeleton')).toBeVisible();
    // After load, product should appear
    await expect(page.getByText('Slow Product')).toBeVisible({ timeout: 5000 });
  });
});
```

---

## Summary

| Type | Scope | Tools | Speed | Goal |
|---|---|---|---|---|
| **Unit** | Single class / method | JUnit 5, Mockito, AssertJ | ⚡ Very fast | Validate isolated business logic |
| **Integration** | Multiple layers (HTTP + DB) | Spring Boot Test, MockMvc, H2, DataJpaTest | 🐢 Moderate | Validate components working together |
| **Testcontainers** | Multiple layers with real infra | Testcontainers, PostgreSQL, Redis, JUnit 5 | 🐢 Moderate–Slow | Validate against production-identical infrastructure |
| **End-to-End** | Full application in a browser | Playwright | 🐢🐢 Slower | Validate real user journeys |

> **Tip:** Follow the [Testing Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html) — write many unit tests, fewer integration tests, and even fewer E2E tests to maintain a fast and maintainable test suite.

