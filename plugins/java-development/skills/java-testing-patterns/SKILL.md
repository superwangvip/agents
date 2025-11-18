---
name: java-testing-patterns
description: Implement comprehensive testing strategies with JUnit 5, TestContainers, Mockito, and test-driven development. Use when writing Java tests, setting up test suites, or implementing testing best practices.
---

# Java Testing Patterns

Comprehensive guide to implementing robust testing strategies in Java using JUnit 5, TestContainers, Mockito, integration testing, and test-driven development practices.

## When to Use This Skill

- Writing unit tests for Java code
- Setting up test suites and test infrastructure
- Implementing test-driven development (TDD)
- Creating integration tests for Spring Boot applications
- Testing database operations with TestContainers
- Mocking external dependencies and services
- Setting up continuous testing in CI/CD
- Testing web services and REST APIs
- Performance testing and benchmarking
- Debugging failing tests and test flakiness

## Core Concepts

### 1. Test Types
- **Unit Tests**: Test individual classes/methods in isolation
- **Integration Tests**: Test interaction between components
- **Component Tests**: Test specific components with real dependencies
- **End-to-End Tests**: Test complete application flows
- **Performance Tests**: Measure speed, throughput, and resource usage

### 2. Test Structure (AAA Pattern)
- **Arrange**: Set up test data and preconditions
- **Act**: Execute the code under test
- **Assert**: Verify the results

### 3. Test Isolation
- Tests should be independent and repeatable
- No shared state between tests
- Each test should clean up after itself
- Use proper test lifecycle management

### 4. Test Coverage
- Aim for meaningful coverage, not just high percentages
- Focus on critical business logic and edge cases
- Use coverage tools to identify untested code paths

## Quick Start

```java
// Basic JUnit 5 test
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class CalculatorTest {

    @Test
    void testAddition() {
        // Arrange
        Calculator calculator = new Calculator();

        // Act
        int result = calculator.add(2, 3);

        // Assert
        assertEquals(5, result, "2 + 3 should equal 5");
    }

    @Test
    void testDivisionByZero() {
        Calculator calculator = new Calculator();

        assertThrows(ArithmeticException.class, () -> {
            calculator.divide(10, 0);
        });
    }
}
```

## Fundamental Patterns

### Pattern 1: Basic JUnit 5 Tests

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

class UserServiceTest {

    private UserService userService;
    private UserRepository userRepository;

    @BeforeEach
    void setUp() {
        userRepository = mock(UserRepository.class);
        userService = new UserService(userRepository);
    }

    @Test
    @DisplayName("Should create user successfully with valid data")
    void shouldCreateUserSuccessfully() {
        // Arrange
        CreateUserRequest request = new CreateUserRequest("John Doe", "john@example.com");
        User savedUser = new User(1L, "John Doe", "john@example.com");

        when(userRepository.save(any(User.class))).thenReturn(savedUser);

        // Act
        User result = userService.createUser(request);

        // Assert
        assertNotNull(result);
        assertEquals("John Doe", result.getName());
        assertEquals("john@example.com", result.getEmail());
        verify(userRepository).save(any(User.class));
    }

    @Test
    @DisplayName("Should throw exception for duplicate email")
    void shouldThrowExceptionForDuplicateEmail() {
        // Arrange
        CreateUserRequest request = new CreateUserRequest("Jane Doe", "jane@example.com");

        when(userRepository.existsByEmail("jane@example.com")).thenReturn(true);

        // Act & Assert
        IllegalArgumentException exception = assertThrows(
            IllegalArgumentException.class,
            () -> userService.createUser(request)
        );

        assertEquals("Email already exists", exception.getMessage());
    }
}
```

### Pattern 2: TestContainers for Integration Testing

```java
import org.junit.jupiter.api.Test;
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
class UserRepositoryIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
        registry.add("spring.jpa.hibernate.ddl-auto", () -> "create-drop");
    }

    @Autowired
    private UserRepository userRepository;

    @Test
    void shouldSaveAndRetrieveUser() {
        // Arrange
        User user = new User(null, "Test User", "test@example.com");

        // Act
        User savedUser = userRepository.save(user);
        Optional<User> retrievedUser = userRepository.findById(savedUser.getId());

        // Assert
        assertTrue(retrievedUser.isPresent());
        assertEquals("Test User", retrievedUser.get().getName());
        assertEquals("test@example.com", retrievedUser.get().getEmail());
    }

    @Test
    void shouldFindUsersByEmail() {
        // Arrange
        User user1 = new User(null, "User 1", "user1@example.com");
        User user2 = new User(null, "User 2", "user2@example.com");
        userRepository.saveAll(List.of(user1, user2));

        // Act
        List<User> users = userRepository.findByEmailContaining("@example.com");

        // Assert
        assertEquals(2, users.size());
    }
}
```

### Pattern 3: Mocking with Mockito

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private UserRepository userRepository;

    @Mock
    private ProductRepository productRepository;

    @Mock
    private PaymentService paymentService;

    @InjectMocks
    private OrderService orderService;

    @Test
    void shouldCreateOrderSuccessfully() {
        // Arrange
        Long userId = 1L;
        Long productId = 2L;
        CreateOrderRequest request = new CreateOrderRequest(userId, productId, 2);

        User user = new User(userId, "John Doe", "john@example.com");
        Product product = new Product(productId, "Laptop", 999.99, 10);

        when(userRepository.findById(userId)).thenReturn(Optional.of(user));
        when(productRepository.findById(productId)).thenReturn(Optional.of(product));
        when(paymentService.processPayment(any(PaymentRequest.class)))
            .thenReturn(PaymentResponse.success("payment123"));

        // Act
        Order result = orderService.createOrder(request);

        // Assert
        assertNotNull(result);
        assertEquals(userId, result.getUserId());
        assertEquals(productId, result.getProductId());
        assertEquals(2, result.getQuantity());
        assertEquals(1999.98, result.getTotalAmount());
        assertEquals(OrderStatus.PAID, result.getStatus());

        // Verify interactions
        verify(userRepository).findById(userId);
        verify(productRepository).findById(productId);
        verify(productRepository).save(product); // Stock should be updated
        verify(paymentService).processPayment(any(PaymentRequest.class));
    }

    @Test
    void shouldThrowExceptionWhenProductOutOfStock() {
        // Arrange
        Long userId = 1L;
        Long productId = 2L;
        CreateOrderRequest request = new CreateOrderRequest(userId, productId, 15);

        User user = new User(userId, "John Doe", "john@example.com");
        Product product = new Product(productId, "Laptop", 999.99, 10);

        when(userRepository.findById(userId)).thenReturn(Optional.of(user));
        when(productRepository.findById(productId)).thenReturn(Optional.of(product));

        // Act & Assert
        InsufficientStockException exception = assertThrows(
            InsufficientStockException.class,
            () -> orderService.createOrder(request)
        );

        assertEquals("Insufficient stock for product: Laptop", exception.getMessage());

        // Verify no payment processed
        verify(paymentService, never()).processPayment(any());
    }
}
```

### Pattern 4: Parameterized Tests

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.*;
import static org.junit.jupiter.api.Assertions.*;

class EmailValidatorTest {

    @ParameterizedTest
    @ValueSource(strings = {
        "user@example.com",
        "test.user@domain.co.uk",
        "user+tag@example.org",
        "user123@test-domain.com"
    })
    void shouldValidateCorrectEmails(String email) {
        assertTrue(EmailValidator.isValid(email));
    }

    @ParameterizedTest
    @ValueSource(strings = {
        "invalid.email",
        "@example.com",
        "user@domain",
        "user@.com",
        "",
        " "
    })
    void shouldRejectInvalidEmails(String email) {
        assertFalse(EmailValidator.isValid(email));
    }

    @ParameterizedTest
    @CsvSource({
        "apple, 2, 0.99, 1.98",
        "banana, 5, 0.50, 2.50",
        "orange, 3, 0.75, 2.25"
    })
    void shouldCalculateTotalPriceCorrectly(
            String product, int quantity, double unitPrice, double expectedTotal) {

        PriceCalculator calculator = new PriceCalculator();
        double total = calculator.calculate(product, quantity, unitPrice);

        assertEquals(expectedTotal, total, 0.001);
    }

    @ParameterizedTest
    @EnumSource(UserRole.class)
    void shouldAllowAccessForAllRolesExceptGuest(UserRole role) {
        if (role == UserRole.GUEST) {
            assertFalse(AccessControl.hasFullAccess(role));
        } else {
            assertTrue(AccessControl.hasFullAccess(role));
        }
    }

    @ParameterizedTest
    @MethodSource("provideInvalidPasswordData")
    void shouldRejectInvalidPasswords(String password, String expectedReason) {
        ValidationResult result = PasswordValidator.validate(password);

        assertFalse(result.isValid());
        assertTrue(result.getErrors().contains(expectedReason));
    }

    static Stream<Arguments> provideInvalidPasswordData() {
        return Stream.of(
            Arguments.of("123", "Password must be at least 8 characters"),
            Arguments.of("password123", "Password must contain uppercase letter"),
            Arguments.of("PASSWORD123", "Password must contain lowercase letter"),
            Arguments.of("Password", "Password must contain digit")
        );
    }
}
```

### Pattern 5: Spring Boot Web Layer Testing

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.BDDMockito.given;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(UserController.class)
class UserControllerWebTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void shouldReturnUserWhenGetUserById() throws Exception {
        // Arrange
        Long userId = 1L;
        User user = new User(userId, "John Doe", "john@example.com");
        UserResponse response = new UserResponse(userId, "John Doe", "john@example.com");

        given(userService.getUserById(userId)).willReturn(response);

        // Act & Assert
        mockMvc.perform(get("/api/v1/users/{id}", userId))
            .andExpect(status().isOk())
            .andExpect(content().contentType(MediaType.APPLICATION_JSON))
            .andExpect(jsonPath("$.id").value(userId))
            .andExpect(jsonPath("$.name").value("John Doe"))
            .andExpect(jsonPath("$.email").value("john@example.com"));
    }

    @Test
    void shouldReturnNotFoundWhenUserDoesNotExist() throws Exception {
        // Arrange
        Long userId = 999L;

        given(userService.getUserById(userId))
            .willThrow(new ResourceNotFoundException("User not found"));

        // Act & Assert
        mockMvc.perform(get("/api/v1/users/{id}", userId))
            .andExpect(status().isNotFound())
            .andExpect(jsonPath("$.message").value("User not found"));
    }

    @Test
    void shouldCreateUserSuccessfully() throws Exception {
        // Arrange
        CreateUserRequest request = new CreateUserRequest("Jane Doe", "jane@example.com");
        UserResponse response = new UserResponse(2L, "Jane Doe", "jane@example.com");

        given(userService.createUser(any(CreateUserRequest.class))).willReturn(response);

        // Act & Assert
        mockMvc.perform(post("/api/v1/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(content().contentType(MediaType.APPLICATION_JSON))
            .andExpect(jsonPath("$.id").value(2L))
            .andExpect(jsonPath("$.name").value("Jane Doe"));
    }

    @Test
    void shouldReturnBadRequestWhenInvalidData() throws Exception {
        // Arrange
        CreateUserRequest request = new CreateUserRequest("", "invalid-email");

        // Act & Assert
        mockMvc.perform(post("/api/v1/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.errors").isArray())
            .andExpect(jsonPath("$.errors[?(@.field == 'name')]").exists())
            .andExpect(jsonPath("$.errors[?(@.field == 'email')]").exists());
    }
}
```

### Pattern 6: Dynamic Tests

```java
import org.junit.jupiter.api.DynamicTest;
import org.junit.jupiter.api.TestFactory;
import java.util.stream.Stream;
import static org.junit.jupiter.api.Assertions.*;
import static org.junit.jupiter.api.DynamicTest.dynamicTest;

class StringProcessorTest {

    @TestFactory
    Stream<DynamicTest> shouldProcessStringsWithVariousOperations() {
        StringProcessor processor = new StringProcessor();

        return Stream.of(
            dynamicTest("Should capitalize first letter", () -> {
                String result = processor.capitalize("hello world");
                assertEquals("Hello world", result);
            }),
            dynamicTest("Should reverse string", () -> {
                String result = processor.reverse("hello");
                assertEquals("olleh", result);
            }),
            dynamicTest("Should count words", () -> {
                int count = processor.countWords("hello world test");
                assertEquals(3, count);
            }),
            dynamicTest("Should remove special characters", () -> {
                String result = processor.removeSpecialChars("hello@world!");
                assertEquals("helloworld", result);
            })
        );
    }

    @TestFactory
    Stream<DynamicTest> shouldValidateEmails() {
        return Stream.of("user@example.com", "test.user@domain.co.uk", "user+tag@example.org")
            .map(email -> dynamicTest("Should validate email: " + email, () -> {
                assertTrue(EmailValidator.isValid(email));
            }));
    }
}
```

### Pattern 7: Custom Test Extensions

```java
import org.junit.jupiter.api.extension.*;
import org.junit.jupiter.api.*;

@ExtendWith(DatabaseCleanupExtension.class)
class UserRepositoryCustomTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    void shouldSaveUserWithCleanDatabase() {
        // Database is automatically cleaned before each test
        User user = new User(null, "Test User", "test@example.com");
        User saved = userRepository.save(user);

        assertNotNull(saved.getId());
        assertEquals(1, userRepository.count());
    }
}

// Custom extension for database cleanup
public class DatabaseCleanupExtension implements BeforeEachCallback {

    @Override
    public void beforeEach(ExtensionContext context) {
        // Get test instance
        Object testInstance = context.getRequiredTestInstance();

        // Access database through reflection or implement interface
        if (testInstance instanceof DatabaseCleanable) {
            DatabaseCleanable cleanable = (DatabaseCleanable) testInstance;
            cleanable.cleanupDatabase();
        }
    }
}

public interface DatabaseCleanable {
    void cleanupDatabase();
}
```

### Pattern 8: Performance Testing

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.Timeout;
import java.util.concurrent.TimeUnit;

class PerformanceTest {

    @Test
    @Timeout(value = 5, unit = TimeUnit.SECONDS)
    void shouldProcessLargeDatasetWithinTimeout() {
        DataProcessor processor = new DataProcessor();
        List<DataItem> largeDataset = generateLargeDataset(100000);

        List<Result> results = processor.process(largeDataset);

        assertEquals(100000, results.size());
    }

    @Test
    void shouldMaintainPerformanceUnderLoad() {
        SearchService searchService = new SearchService();

        // Warm up
        for (int i = 0; i < 1000; i++) {
            searchService.search("test query");
        }

        // Measure performance
        long startTime = System.nanoTime();
        for (int i = 0; i < 10000; i++) {
            searchService.search("test query " + i);
        }
        long endTime = System.nanoTime();

        double averageTime = (endTime - startTime) / 10000.0 / 1_000_000.0; // milliseconds

        // Assert average response time is acceptable (< 10ms)
        assertTrue(averageTime < 10.0,
            "Average response time should be less than 10ms, but was " + averageTime + "ms");
    }
}
```

### Pattern 9: Test Data Builders

```java
// Test data builder pattern
public class UserBuilder {
    private Long id = null;
    private String name = "Default Name";
    private String email = "default@example.com";
    private UserRole role = UserRole.USER;
    private boolean active = true;

    public static UserBuilder aUser() {
        return new UserBuilder();
    }

    public UserBuilder withId(Long id) {
        this.id = id;
        return this;
    }

    public UserBuilder withName(String name) {
        this.name = name;
        return this;
    }

    public UserBuilder withEmail(String email) {
        this.email = email;
        return this;
    }

    public UserBuilder withRole(UserRole role) {
        this.role = role;
        return this;
    }

    public UserBuilder active(boolean active) {
        this.active = active;
        return this;
    }

    public User build() {
        return new User(id, name, email, role, active);
    }
}

// Usage in tests
class UserServiceTest {
    @Test
    void shouldActivateUserSuccessfully() {
        // Arrange
        User inactiveUser = UserBuilder.aUser()
            .withId(1L)
            .withName("John Doe")
            .withEmail("john@example.com")
            .active(false)
            .build();

        // Act
        User activatedUser = userService.activateUser(inactiveUser.getId());

        // Assert
        assertTrue(activatedUser.isActive());
    }
}
```

### Pattern 10: Test Configuration and Profiles

```java
@TestConfiguration
public class TestConfig {

    @Bean
    @Primary
    public EmailService emailService() {
        return new MockEmailService();
    }

    @Bean
    @Profile("test")
    public DataSource testDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .addScript("schema.sql")
            .addScript("test-data.sql")
            .build();
    }
}

@SpringBootTest
@ActiveProfiles("test")
class IntegrationTestWithCustomConfig {

    @Test
    void shouldUseTestConfiguration() {
        // Uses MockEmailService and H2 database
        // Test implementation
    }
}
```

## Advanced Testing Scenarios

### Testing with Random Data

```java
import net.jqwik.api.*;
import net.jqwik.api.constraints.*;

@Property
void shouldCalculateCorrectlyWithRandomNumbers(
        @ForAll @IntRange(min = 1, max = 100) int a,
        @ForAll @IntRange(min = 1, max = 100) int b) {

    Calculator calculator = new Calculator();
    int result = calculator.add(a, b);

    assertEquals(a + b, result);
}

@Property
void shouldValidateEmailsWithRandomData(@ForAll @AlphaChars @StringLength(min = 3, max = 20) String name,
                                       @ForAll @Digits @StringLength(min = 10, max = 15) String phone) {

    // Generate random emails and test validation
    String email = name.toLowerCase() + "@" + phone + ".com";

    // Email might not be valid due to random generation
    // Test validation logic
}
```

## Best Practices

### Test Organization

```
src/test/java/
├── unit/                 # Unit tests
│   ├── service/
│   │   └── UserServiceTest.java
│   └── util/
│       └── EmailValidatorTest.java
├── integration/          # Integration tests
│   ├── repository/
│   │   └── UserRepositoryTest.java
│   └── controller/
│       └── UserControllerTest.java
├── e2e/                 # End-to-end tests
│   └── UserWorkflowTest.java
└── configuration/       # Test configuration
    ├── TestConfig.java
    └── TestContainersConfig.java
```

### Test Naming Conventions

```java
// Good test names
@Test
void shouldCreateUserSuccessfully_WhenValidDataProvided() {}

@Test
void shouldThrowException_WhenUserEmailAlreadyExists() {}

@Test
void shouldReturnUserList_WhenUsersExist() {}

@Test
void shouldCalculateTotalPriceCorrectly_WithDiscountApplied() {}
```

### Test Data Management

```java
@TestMethodOrder(OrderAnnotation.class)
class OrderedTestExample {

    @Test
    @Order(1)
    void shouldCreateUser() {
        // Create user for subsequent tests
    }

    @Test
    @Order(2)
    void shouldUpdateUser() {
        // Update user created in previous test
    }

    @Test
    @Order(3)
    void shouldDeleteUser() {
        // Delete user created in first test
    }
}
```

## Resources

- **JUnit 5 Documentation**: https://junit.org/junit5/docs/current/user-guide/
- **TestContainers**: https://www.testcontainers.org/
- **Mockito Documentation**: https://site.mockito.org/
- **AssertJ**: https://assertj.github.io/doc/
- **jqwik**: Property-based testing for Java

## CI/CD Integration

```yaml
# .github/workflows/test.yml
name: Java Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: test
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v3

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Cache Maven dependencies
        uses: actions/cache@v3
        with:
          path: ~/.m2
          key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
          restore-keys: ${{ runner.os }}-m2-

      - name: Run tests
        run: mvn clean verify

      - name: Generate test report
        uses: dorny/test-reporter@v1
        if: success() || failure()
        with:
          name: Maven Tests
          path: target/surefire-reports/*.xml
          reporter: java-junit

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: target/site/jacoco/jacoco.xml
```

This comprehensive Java testing guide provides production-ready patterns and best practices for enterprise Java applications, ensuring robust, maintainable, and reliable test suites.