# JUnit 5, Mockito & Spring Boot Testing: Comprehensive Guide

This comprehensive guide covers JUnit 5 testing framework, Mockito for unit testing, including method mocking, conditional returns, exception handling, verification, and Spring Boot testing integration with various testing utilities.

## Table of Contents
1. [JUnit 5 Fundamentals](#1-junit-5-fundamentals)
2. [JUnit 5 Advanced Features](#2-junit-5-advanced-features)
3. [JUnit 5 Assertions and Assumptions](#3-junit-5-assertions-and-assumptions)
4. [Introduction to Mockito](#4-introduction-to-mockito)
5. [Mockito Setup and Configuration](#5-mockito-setup-and-configuration)
6. [Basic Method Mocking](#6-basic-method-mocking)
7. [Conditional Mock Returns](#7-conditional-mock-returns)
8. [Exception Mocking](#8-exception-mocking)
9. [Method Call Verification](#9-method-call-verification)
10. [Verification Count and Timing](#10-verification-count-and-timing)
11. [Spring Boot Testing Integration](#11-spring-boot-testing-integration)
12. [Spring Boot Testing Utilities](#12-spring-boot-testing-utilities)
13. [Best Practices and Advanced Scenarios](#13-best-practices-and-advanced-scenarios)

---

## 1) JUnit 5 Fundamentals

### 1.1 What is JUnit 5?

JUnit 5 is the next generation of the JUnit testing framework for Java. It provides a modern foundation for developer-side testing on the JVM with improved architecture, better extension model, and enhanced features.

```
JUnit 5 Architecture:
┌────────────────────────────────────────────────────────────────────────────────┐
│                           JUnit 5 Platform Architecture                       │
├────────────────────────────────────────────────────────────────────────────────┤
│                                                                                │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐           │
│  │  JUnit Jupiter  │    │  JUnit Vintage  │    │ Third-party     │           │
│  │                 │    │                 │    │ Test Engines    │           │
│  │ • New features  │    │ • JUnit 3 & 4   │    │ • TestNG        │           │
│  │ • Annotations   │    │ • Backward       │    │ • Spock         │           │
│  │ • Assertions    │    │   compatibility  │    │ • Cucumber      │           │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘           │
│           │                       │                       │                   │
│           └───────────────────────┼───────────────────────┘                   │
│                                   │                                           │
│                                   ▼                                           │
│              ┌─────────────────────────────────────────┐                      │
│              │            JUnit Platform               │                      │
│              │                                         │                      │
│              │  • Test Engine API                     │                      │
│              │  • Console Launcher                    │                      │
│              │  • IDE & Build Tool Integration        │                      │
│              │  • Extension Model                     │                      │
│              └─────────────────────────────────────────┘                      │
└────────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Maven Dependencies

```xml
<dependencies>
    <!-- JUnit 5 (Jupiter) - Main testing framework -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.10.1</version>
        <scope>test</scope>
    </dependency>
    
    <!-- JUnit Platform Suite (for test suites) -->
    <dependency>
        <groupId>org.junit.platform</groupId>
        <artifactId>junit-platform-suite</artifactId>
        <version>1.10.1</version>
        <scope>test</scope>
    </dependency>
    
    <!-- AssertJ for fluent assertions (optional but recommended) -->
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <version>3.24.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <!-- Maven Surefire Plugin for running tests -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.2.2</version>
        </plugin>
    </plugins>
</build>
```

---

## 2) JUnit 5 Advanced Features

### 2.1 Basic Test Structure and Annotations

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

/**
 * Basic JUnit 5 test class demonstrating fundamental concepts
 */
class BasicJUnit5Test {
    
    private Calculator calculator;
    
    @BeforeAll
    static void setUpClass() {
        // Executed once before all test methods in the class
        System.out.println("Setting up test class");
    }
    
    @AfterAll
    static void tearDownClass() {
        // Executed once after all test methods in the class
        System.out.println("Tearing down test class");
    }
    
    @BeforeEach
    void setUp() {
        // Executed before each test method
        calculator = new Calculator();
        System.out.println("Setting up test method");
    }
    
    @AfterEach
    void tearDown() {
        // Executed after each test method
        calculator = null;
        System.out.println("Tearing down test method");
    }
    
    @Test
    @DisplayName("Should add two positive numbers correctly")
    void shouldAddTwoPositiveNumbers() {
        // Arrange
        int a = 5;
        int b = 3;
        
        // Act
        int result = calculator.add(a, b);
        
        // Assert
        assertEquals(8, result, "5 + 3 should equal 8");
    }
    
    @Test
    @DisplayName("Should subtract two numbers correctly")
    void shouldSubtractTwoNumbers() {
        // Arrange & Act
        int result = calculator.subtract(10, 4);
        
        // Assert
        assertEquals(6, result);
    }
    
    @Test
    @Disabled("This test is temporarily disabled")
    void disabledTest() {
        // This test will be skipped
        fail("This test should not run");
    }
}

// Simple Calculator class for demonstration
class Calculator {
    public int add(int a, int b) {
        return a + b;
    }
    
    public int subtract(int a, int b) {
        return a - b;
    }
    
    public int multiply(int a, int b) {
        return a * b;
    }
    
    public double divide(int a, int b) {
        if (b == 0) {
            throw new ArithmeticException("Division by zero");
        }
        return (double) a / b;
    }
    
    public boolean isPrime(int number) {
        if (number < 2) return false;
        for (int i = 2; i <= Math.sqrt(number); i++) {
            if (number % i == 0) return false;
        }
        return true;
    }
}
```

---

## 3) JUnit 5 Assertions and Assumptions

### 3.1 Basic Assertions

```java
/**
 * Comprehensive examples of JUnit 5 assertions
 */
class AssertionsExample {
    
    @Test
    @DisplayName("Basic assertion examples")
    void basicAssertions() {
        // Simple assertions
        assertTrue(true, "This should be true");
        assertFalse(false, "This should be false");
        assertNull(null, "This should be null");
        assertNotNull("test", "This should not be null");
        
        // Equality assertions
        assertEquals(2, 1 + 1, "1 + 1 should equal 2");
        assertNotEquals(3, 1 + 1, "1 + 1 should not equal 3");
        
        // Array assertions
        int[] expected = {1, 2, 3};
        int[] actual = {1, 2, 3};
        assertArrayEquals(expected, actual, "Arrays should be equal");
    }
    
    @Test
    @DisplayName("Exception assertions")
    void exceptionAssertions() {
        Calculator calculator = new Calculator();
        
        // Assert that exception is thrown
        ArithmeticException exception = assertThrows(
            ArithmeticException.class,
            () -> calculator.divide(10, 0),
            "Division by zero should throw ArithmeticException"
        );
        
        // Assert exception message
        assertEquals("Division by zero", exception.getMessage());
        
        // Assert that no exception is thrown
        assertDoesNotThrow(
            () -> calculator.divide(10, 2),
            "Division by non-zero should not throw exception"
        );
    }
    
    @Test
    @DisplayName("Grouped assertions")
    void groupedAssertions() {
        Calculator calculator = new Calculator();
        
        // All assertions are executed, even if some fail
        assertAll("Calculator operations",
            () -> assertEquals(8, calculator.add(5, 3), "Addition failed"),
            () -> assertEquals(2, calculator.subtract(5, 3), "Subtraction failed"),
            () -> assertEquals(15, calculator.multiply(5, 3), "Multiplication failed"),
            () -> assertEquals(1.67, calculator.divide(5, 3), 0.01, "Division failed")
        );
    }
}
```

---

## 4) Introduction to Mockito

### 4.1 What is Mockito?

Mockito is a powerful Java testing framework that allows you to create mock objects for unit testing. It enables you to simulate the behavior of dependencies and verify interactions between objects.

```
Mockito Testing Framework Overview:
┌────────────────────────────────────────────────────────────────────────────────┐
│                           Mockito Core Capabilities                           │
├────────────────────────────────────────────────────────────────────────────────┤
│                                                                                │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐           │
│  │   Mock Objects  │    │   Stubbing      │    │  Verification   │           │
│  │                 │    │                 │    │                 │           │
│  │ • Create fakes  │    │ • Define return │    │ • Check calls   │           │
│  │ • Replace deps  │    │ • Set behavior  │    │ • Count invokes │           │
│  │ • Control flow  │    │ • Throw errors  │    │ • Verify order  │           │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘           │
│           │                       │                       │                   │
│           └───────────────────────┼───────────────────────┘                   │
│                                   │                                           │
│                                   ▼                                           │
│              ┌─────────────────────────────────────────┐                      │
│              │         Unit Test Isolation             │                      │
│              │                                         │                      │
│              │  • Test single components              │                      │
│              │  • Mock external dependencies          │                      │
│              │  • Control test environment            │                      │
│              │  • Fast and reliable tests             │                      │
│              └─────────────────────────────────────────┘                      │
└────────────────────────────────────────────────────────────────────────────────┘
```

### 4.2 Benefits of Using Mockito

**Testing Advantages:**
- **Isolation** - Test units in isolation from dependencies
- **Speed** - Fast execution without external resources
- **Control** - Predictable test environment
- **Coverage** - Test edge cases and error conditions

**Development Benefits:**
- **TDD Support** - Write tests before implementation
- **Refactoring Safety** - Detect breaking changes
- **Documentation** - Tests serve as living documentation
- **Quality Assurance** - Ensure code reliability

---

## 5) Mockito Setup and Configuration

### 5.1 Maven Dependencies

```xml
<dependencies>
    <!-- Spring Boot Test Starter (includes Mockito) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    
    <!-- Mockito Core (if not using Spring Boot) -->
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-core</artifactId>
        <version>5.7.0</version>
        <scope>test</scope>
    </dependency>
    
    <!-- Mockito JUnit Jupiter (for JUnit 5) -->
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-junit-jupiter</artifactId>
        <version>5.7.0</version>
        <scope>test</scope>
    </dependency>
    
    <!-- Mockito Inline (for final classes/static methods) -->
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-inline</artifactId>
        <version>5.7.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### 5.2 Basic Test Class Setup

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.InjectMocks;
import org.mockito.junit.jupiter.MockitoExtension;
import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

/**
 * Basic Mockito test setup with JUnit 5
 */
@ExtendWith(MockitoExtension.class)
class BasicMockitoTest {
    
    @Mock
    private ExternalService externalService;
    
    @Mock
    private DatabaseRepository repository;
    
    @InjectMocks
    private UserService userService; // Class under test
    
    @BeforeEach
    void setUp() {
        // Additional setup if needed
        // MockitoAnnotations.openMocks(this); // Alternative to @ExtendWith
    }
    
    @Test
    void testBasicSetup() {
        // Verify mocks are created
        assertNotNull(externalService);
        assertNotNull(repository);
        assertNotNull(userService);
    }
}
```

---

## 6) Basic Method Mocking

### 6.1 Simple Method Stubbing

```java
/**
 * Example classes for demonstration
 */
class User {
    private Long id;
    private String name;
    private String email;
    private boolean active;
    
    // Constructors, getters, setters
    public User(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.active = true;
    }
    
    // Getters and setters...
    public Long getId() { return id; }
    public String getName() { return name; }
    public String getEmail() { return email; }
    public boolean isActive() { return active; }
    public void setActive(boolean active) { this.active = active; }
}

interface UserRepository {
    User findById(Long id);
    User save(User user);
    void deleteById(Long id);
    List<User> findByEmail(String email);
    List<User> findAll();
    boolean existsById(Long id);
}

interface EmailService {
    void sendEmail(String to, String subject, String body);
    boolean validateEmail(String email);
}

/**
 * Service class to test
 */
@Service
class UserService {
    
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
    
    public User getUserById(Long id) {
        if (id == null || id <= 0) {
            throw new IllegalArgumentException("Invalid user ID");
        }
        return userRepository.findById(id);
    }
    
    public User createUser(String name, String email) {
        if (!emailService.validateEmail(email)) {
            throw new IllegalArgumentException("Invalid email format");
        }
        
        User user = new User(null, name, email);
        return userRepository.save(user);
    }
    
    public void deactivateUser(Long id) {
        User user = userRepository.findById(id);
        if (user != null) {
            user.setActive(false);
            userRepository.save(user);
            emailService.sendEmail(user.getEmail(), "Account Deactivated", 
                "Your account has been deactivated.");
        }
    }
    
    public List<User> searchUsersByEmail(String email) {
        return userRepository.findByEmail(email);
    }
}
```

### 6.2 Basic Stubbing Examples

```java
@ExtendWith(MockitoExtension.class)
class BasicStubbingTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void testBasicStubbing() {
        // Arrange: Create test data
        User testUser = new User(1L, "John Doe", "john@example.com");
        
        // Stub the method call
        when(userRepository.findById(1L)).thenReturn(testUser);
        
        // Act: Call the method under test
        User result = userService.getUserById(1L);
        
        // Assert: Verify the result
        assertNotNull(result);
        assertEquals("John Doe", result.getName());
        assertEquals("john@example.com", result.getEmail());
    }
    
    @Test
    void testStubbingWithVoidMethod() {
        // Arrange
        User testUser = new User(1L, "John Doe", "john@example.com");
        when(userRepository.findById(1L)).thenReturn(testUser);
        
        // Stub void method (doesn't throw exception)
        doNothing().when(emailService).sendEmail(anyString(), anyString(), anyString());
        
        // Act
        userService.deactivateUser(1L);
        
        // Verify the void method was called
        verify(emailService).sendEmail(
            eq("john@example.com"), 
            eq("Account Deactivated"), 
            anyString()
        );
    }
    
    @Test
    void testStubbingBooleanReturn() {
        // Arrange
        when(emailService.validateEmail("valid@example.com")).thenReturn(true);
        when(emailService.validateEmail("invalid-email")).thenReturn(false);
        
        // Test valid email
        assertTrue(emailService.validateEmail("valid@example.com"));
        
        // Test invalid email
        assertFalse(emailService.validateEmail("invalid-email"));
    }
    
    @Test
    void testStubbingCollectionReturn() {
        // Arrange
        List<User> users = Arrays.asList(
            new User(1L, "John", "john@example.com"),
            new User(2L, "Jane", "jane@example.com")
        );
        
        when(userRepository.findByEmail("example.com")).thenReturn(users);
        when(userRepository.findByEmail("nonexistent.com")).thenReturn(Collections.emptyList());
        
        // Act & Assert
        List<User> foundUsers = userService.searchUsersByEmail("example.com");
        assertEquals(2, foundUsers.size());
        
        List<User> emptyResult = userService.searchUsersByEmail("nonexistent.com");
        assertTrue(emptyResult.isEmpty());
    }
}
```

---

## 7) Conditional Mock Returns

### 7.1 Argument Matchers

```java
@ExtendWith(MockitoExtension.class)
class ConditionalMockingTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void testArgumentMatchers() {
        // Using argument matchers
        when(userRepository.findById(anyLong())).thenReturn(new User(1L, "Default", "default@example.com"));
        when(userRepository.findById(eq(999L))).thenReturn(null);
        
        // Specific ID returns specific user
        User defaultUser = userService.getUserById(1L);
        assertEquals("Default", defaultUser.getName());
        
        // ID 999 returns null
        User nullUser = userService.getUserById(999L);
        assertNull(nullUser);
    }
    
    @Test
    void testConditionalReturnsWithAnswer() {
        // Using Answer for complex logic
        when(userRepository.findById(anyLong())).thenAnswer(invocation -> {
            Long id = invocation.getArgument(0);
            if (id > 0 && id <= 100) {
                return new User(id, "User" + id, "user" + id + "@example.com");
            }
            return null;
        });
        
        // Test valid ID range
        User validUser = userService.getUserById(50L);
        assertNotNull(validUser);
        assertEquals("User50", validUser.getName());
        
        // Test invalid ID range
        User invalidUser = userService.getUserById(999L);
        assertNull(invalidUser);
    }
    
    @Test
    void testMultipleConditionalReturns() {
        // Multiple when-then combinations
        when(emailService.validateEmail(argThat(email -> email.contains("@gmail.com"))))
            .thenReturn(true);
        when(emailService.validateEmail(argThat(email -> email.contains("@spam.com"))))
            .thenReturn(false);
        when(emailService.validateEmail(argThat(email -> !email.contains("@"))))
            .thenReturn(false);
        
        // Test different email patterns
        assertTrue(emailService.validateEmail("user@gmail.com"));
        assertFalse(emailService.validateEmail("user@spam.com"));
        assertFalse(emailService.validateEmail("invalid-email"));
    }
    
    @Test
    void testSequentialReturns() {
        // Return different values on consecutive calls
        when(userRepository.existsById(1L))
            .thenReturn(true)   // First call
            .thenReturn(false)  // Second call
            .thenReturn(true);  // Third call
        
        assertTrue(userRepository.existsById(1L));  // First call
        assertFalse(userRepository.existsById(1L)); // Second call
        assertTrue(userRepository.existsById(1L));  // Third call
    }
    
    @Test
    void testChainingReturns() {
        // Chain multiple returns
        when(userRepository.findById(anyLong()))
            .thenReturn(new User(1L, "First", "first@example.com"))
            .thenReturn(new User(2L, "Second", "second@example.com"))
            .thenReturn(null);
        
        User first = userService.getUserById(1L);
        assertEquals("First", first.getName());
        
        User second = userService.getUserById(1L);
        assertEquals("Second", second.getName());
        
        User third = userService.getUserById(1L);
        assertNull(third);
    }
}
```

### 7.2 Advanced Conditional Logic

```java
@ExtendWith(MockitoExtension.class)
class AdvancedConditionalTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void testComplexArgumentMatching() {
        // Custom argument matcher
        when(userRepository.save(argThat(user -> 
            user.getName() != null && 
            user.getName().length() > 2 && 
            user.getEmail().contains("@")
        ))).thenAnswer(invocation -> {
            User user = invocation.getArgument(0);
            user.setActive(true);
            return user;
        });
        
        // Mock email validation
        when(emailService.validateEmail(anyString())).thenReturn(true);
        
        // Test valid user creation
        User result = userService.createUser("John Doe", "john@example.com");
        assertNotNull(result);
        assertTrue(result.isActive());
    }
    
    @Test
    void testStateBasedMocking() {
        // Simulate repository state changes
        Map<Long, User> mockDatabase = new HashMap<>();
        
        when(userRepository.save(any(User.class))).thenAnswer(invocation -> {
            User user = invocation.getArgument(0);
            Long id = (long) (mockDatabase.size() + 1);
            user = new User(id, user.getName(), user.getEmail());
            mockDatabase.put(id, user);
            return user;
        });
        
        when(userRepository.findById(anyLong())).thenAnswer(invocation -> {
            Long id = invocation.getArgument(0);
            return mockDatabase.get(id);
        });
        
        when(emailService.validateEmail(anyString())).thenReturn(true);
        
        // Create users and verify state
        User user1 = userService.createUser("John", "john@example.com");
        User user2 = userService.createUser("Jane", "jane@example.com");
        
        assertEquals(1L, user1.getId());
        assertEquals(2L, user2.getId());
        
        // Verify retrieval
        User retrieved = userService.getUserById(1L);
        assertEquals("John", retrieved.getName());
    }
    
    @Test
    void testConditionalBehaviorBasedOnCallCount() {
        AtomicInteger callCount = new AtomicInteger(0);
        
        when(userRepository.findById(1L)).thenAnswer(invocation -> {
            int count = callCount.incrementAndGet();
            if (count == 1) {
                return new User(1L, "Fresh", "fresh@example.com");
            } else if (count <= 3) {
                return new User(1L, "Updated", "updated@example.com");
            } else {
                return null; // Simulate deletion after multiple updates
            }
        });
        
        // First call - fresh user
        User first = userService.getUserById(1L);
        assertEquals("Fresh", first.getName());
        
        // Second and third calls - updated user
        User second = userService.getUserById(1L);
        assertEquals("Updated", second.getName());
        
        User third = userService.getUserById(1L);
        assertEquals("Updated", third.getName());
        
        // Fourth call - user deleted
        User fourth = userService.getUserById(1L);
        assertNull(fourth);
    }
}
```

---

## 8) Exception Mocking

### 8.1 Basic Exception Stubbing

```java
@ExtendWith(MockitoExtension.class)
class ExceptionMockingTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void testThrowingExceptions() {
        // Stub method to throw exception
        when(userRepository.findById(999L))
            .thenThrow(new RuntimeException("User not found"));
        
        // Test exception is thrown
        RuntimeException exception = assertThrows(RuntimeException.class, () -> {
            userService.getUserById(999L);
        });
        
        assertEquals("User not found", exception.getMessage());
    }
    
    @Test
    void testThrowingMultipleExceptionTypes() {
        // Different exceptions for different scenarios
        when(userRepository.findById(-1L))
            .thenThrow(new IllegalArgumentException("Invalid ID"));
        when(userRepository.findById(0L))
            .thenThrow(new RuntimeException("Database error"));
        when(userRepository.findById(999L))
            .thenThrow(new EntityNotFoundException("User not found"));
        
        // Test IllegalArgumentException
        assertThrows(IllegalArgumentException.class, () -> {
            userRepository.findById(-1L);
        });
        
        // Test RuntimeException
        assertThrows(RuntimeException.class, () -> {
            userRepository.findById(0L);
        });
        
        // Test EntityNotFoundException
        assertThrows(EntityNotFoundException.class, () -> {
            userRepository.findById(999L);
        });
    }
    
    @Test
    void testExceptionWithVoidMethods() {
        // Stub void method to throw exception
        doThrow(new RuntimeException("Email service unavailable"))
            .when(emailService)
            .sendEmail(anyString(), anyString(), anyString());
        
        // Setup user retrieval
        when(userRepository.findById(1L))
            .thenReturn(new User(1L, "John", "john@example.com"));
        
        // Test that service method handles the exception
        RuntimeException exception = assertThrows(RuntimeException.class, () -> {
            userService.deactivateUser(1L);
        });
        
        assertEquals("Email service unavailable", exception.getMessage());
    }
    
    @Test
    void testSequentialExceptionThenSuccess() {
        // First call throws exception, second succeeds
        when(userRepository.save(any(User.class)))
            .thenThrow(new RuntimeException("Database temporarily unavailable"))
            .thenReturn(new User(1L, "John", "john@example.com"));
        
        when(emailService.validateEmail(anyString())).thenReturn(true);
        
        // First attempt fails
        assertThrows(RuntimeException.class, () -> {
            userService.createUser("John", "john@example.com");
        });
        
        // Second attempt succeeds
        User result = userService.createUser("John", "john@example.com");
        assertNotNull(result);
        assertEquals("John", result.getName());
    }
}
```

### 8.2 Advanced Exception Scenarios

```java
@ExtendWith(MockitoExtension.class)
class AdvancedExceptionTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void testConditionalExceptions() {
        // Throw exception based on argument
        when(userRepository.findById(anyLong())).thenAnswer(invocation -> {
            Long id = invocation.getArgument(0);
            if (id < 0) {
                throw new IllegalArgumentException("ID cannot be negative");
            } else if (id > 1000) {
                throw new EntityNotFoundException("User not found");
            } else if (id == 500) {
                throw new RuntimeException("Database connection lost");
            }
            return new User(id, "User" + id, "user" + id + "@example.com");
        });
        
        // Test different exception scenarios
        assertThrows(IllegalArgumentException.class, () -> {
            userService.getUserById(-1L);
        });
        
        assertThrows(EntityNotFoundException.class, () -> {
            userService.getUserById(1001L);
        });
        
        assertThrows(RuntimeException.class, () -> {
            userService.getUserById(500L);
        });
        
        // Test successful case
        User user = userService.getUserById(1L);
        assertNotNull(user);
        assertEquals("User1", user.getName());
    }
    
    @Test
    void testExceptionWithCustomMessage() {
        // Create custom exception with detailed message
        when(userRepository.save(any(User.class)))
            .thenThrow(new DataAccessException("Database connection failed") {
                @Override
                public String getMessage() {
                    return "Failed to save user: Database connection timeout after 30 seconds";
                }
            });
        
        when(emailService.validateEmail(anyString())).thenReturn(true);
        
        DataAccessException exception = assertThrows(DataAccessException.class, () -> {
            userService.createUser("John", "john@example.com");
        });
        
        assertTrue(exception.getMessage().contains("Database connection timeout"));
    }
    
    @Test
    void testCascadingExceptions() {
        // Multiple methods in chain can throw exceptions
        when(userRepository.findById(1L))
            .thenReturn(new User(1L, "John", "john@example.com"));
        
        when(userRepository.save(any(User.class)))
            .thenThrow(new RuntimeException("Save operation failed"));
        
        doThrow(new RuntimeException("Email service error"))
            .when(emailService)
            .sendEmail(anyString(), anyString(), anyString());
        
        // Test that both exceptions can occur
        RuntimeException saveException = assertThrows(RuntimeException.class, () -> {
            User user = userService.getUserById(1L);
            user.setActive(false);
            userRepository.save(user); // This will throw
        });
        assertEquals("Save operation failed", saveException.getMessage());
        
        // Reset save to succeed, test email exception
        when(userRepository.save(any(User.class)))
            .thenAnswer(invocation -> invocation.getArgument(0));
        
        RuntimeException emailException = assertThrows(RuntimeException.class, () -> {
            userService.deactivateUser(1L);
        });
        assertEquals("Email service error", emailException.getMessage());
    }
}

// Custom exception for demonstration
class EntityNotFoundException extends RuntimeException {
    public EntityNotFoundException(String message) {
        super(message);
    }
}

abstract class DataAccessException extends RuntimeException {
    public DataAccessException(String message) {
        super(message);
    }
}
```

---

## 9) Method Call Verification

### 9.1 Basic Verification

```java
@ExtendWith(MockitoExtension.class)
class MethodVerificationTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void testBasicVerification() {
        // Arrange
        User testUser = new User(1L, "John", "john@example.com");
        when(userRepository.findById(1L)).thenReturn(testUser);
        
        // Act
        userService.getUserById(1L);
        
        // Verify method was called
        verify(userRepository).findById(1L);
    }
    
    @Test
    void testVerifyWithExactArguments() {
        // Arrange
        when(emailService.validateEmail("john@example.com")).thenReturn(true);
        when(userRepository.save(any(User.class)))
            .thenReturn(new User(1L, "John", "john@example.com"));
        
        // Act
        userService.createUser("John", "john@example.com");
        
        // Verify exact method calls
        verify(emailService).validateEmail("john@example.com");
        verify(userRepository).save(any(User.class));
    }
    
    @Test
    void testVerifyArgumentMatchers() {
        // Arrange
        when(emailService.validateEmail(anyString())).thenReturn(true);
        when(userRepository.save(any(User.class)))
            .thenReturn(new User(1L, "John", "john@example.com"));
        
        // Act
        userService.createUser("John", "john@example.com");
        
        // Verify with argument matchers
        verify(emailService).validateEmail(eq("john@example.com"));
        verify(userRepository).save(argThat(user -> 
            "John".equals(user.getName()) && 
            "john@example.com".equals(user.getEmail())
        ));
    }
    
    @Test
    void testVerifyNoInteractions() {
        // Act - call method that doesn't use mocks
        IllegalArgumentException exception = assertThrows(IllegalArgumentException.class, () -> {
            userService.getUserById(-1L);
        });
        
        // Verify no interactions with mocks
        verifyNoInteractions(userRepository);
        verifyNoInteractions(emailService);
    }
    
    @Test
    void testVerifyNoMoreInteractions() {
        // Arrange
        User testUser = new User(1L, "John", "john@example.com");
        when(userRepository.findById(1L)).thenReturn(testUser);
        
        // Act
        userService.getUserById(1L);
        
        // Verify specific interaction
        verify(userRepository).findById(1L);
        
        // Verify no more interactions occurred
        verifyNoMoreInteractions(userRepository);
        verifyNoMoreInteractions(emailService);
    }
}
```

### 6.2 Complex Verification Scenarios

```java
@ExtendWith(MockitoExtension.class)
class ComplexVerificationTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void testVerifyMultipleMethodCalls() {
        // Arrange
        User testUser = new User(1L, "John", "john@example.com");
        when(userRepository.findById(1L)).thenReturn(testUser);
        when(userRepository.save(any(User.class))).thenReturn(testUser);
        doNothing().when(emailService).sendEmail(anyString(), anyString(), anyString());
        
        // Act
        userService.deactivateUser(1L);
        
        // Verify all expected interactions
        verify(userRepository).findById(1L);
        verify(userRepository).save(testUser);
        verify(emailService).sendEmail(
            eq("john@example.com"),
            eq("Account Deactivated"),
            anyString()
        );
    }
    
    @Test
    void testVerifyMethodCallOrder() {
        // Arrange
        User testUser = new User(1L, "John", "john@example.com");
        when(userRepository.findById(1L)).thenReturn(testUser);
        when(userRepository.save(any(User.class))).thenReturn(testUser);
        doNothing().when(emailService).sendEmail(anyString(), anyString(), anyString());
        
        // Create InOrder verifier
        InOrder inOrder = inOrder(userRepository, emailService);
        
        // Act
        userService.deactivateUser(1L);
        
        // Verify method call order
        inOrder.verify(userRepository).findById(1L);
        inOrder.verify(userRepository).save(any(User.class));
        inOrder.verify(emailService).sendEmail(anyString(), anyString(), anyString());
    }
    
    @Test
    void testVerifyWithCaptor() {
        // Arrange
        ArgumentCaptor<User> userCaptor = ArgumentCaptor.forClass(User.class);
        ArgumentCaptor<String> emailCaptor = ArgumentCaptor.forClass(String.class);
        
        when(emailService.validateEmail(anyString())).thenReturn(true);
        when(userRepository.save(any(User.class)))
            .thenReturn(new User(1L, "John", "john@example.com"));
        
        // Act
        userService.createUser("John", "john@example.com");
        
        // Verify and capture arguments
        verify(userRepository).save(userCaptor.capture());
        verify(emailService).validateEmail(emailCaptor.capture());
        
        // Assert captured values
        User capturedUser = userCaptor.getValue();
        assertEquals("John", capturedUser.getName());
        assertEquals("john@example.com", capturedUser.getEmail());
        
        String capturedEmail = emailCaptor.getValue();
        assertEquals("john@example.com", capturedEmail);
    }
    
    @Test
    void testVerifyWithMultipleCaptures() {
        // Arrange
        ArgumentCaptor<String> emailCaptor = ArgumentCaptor.forClass(String.class);
        when(emailService.validateEmail(anyString())).thenReturn(true);
        when(userRepository.save(any(User.class)))
            .thenAnswer(invocation -> {
                User user = invocation.getArgument(0);
                return new User((long) (Math.random() * 1000), user.getName(), user.getEmail());
            });
        
        // Act - create multiple users
        userService.createUser("John", "john@example.com");
        userService.createUser("Jane", "jane@example.com");
        userService.createUser("Bob", "bob@example.com");
        
        // Verify and capture all email validations
        verify(emailService, times(3)).validateEmail(emailCaptor.capture());
        
        List<String> capturedEmails = emailCaptor.getAllValues();
        assertEquals(3, capturedEmails.size());
        assertTrue(capturedEmails.contains("john@example.com"));
        assertTrue(capturedEmails.contains("jane@example.com"));
        assertTrue(capturedEmails.contains("bob@example.com"));
    }
}
```

---

## 10) Verification Count and Timing

### 10.1 Invocation Count Verification

```java
@ExtendWith(MockitoExtension.class)
class InvocationCountTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void testVerifyExactTimes() {
        // Arrange
        when(userRepository.findById(anyLong()))
            .thenReturn(new User(1L, "John", "john@example.com"));
        
        // Act - call method multiple times
        userService.getUserById(1L);
        userService.getUserById(2L);
        userService.getUserById(3L);
        
        // Verify exact number of calls
        verify(userRepository, times(3)).findById(anyLong());
    }
    
    @Test
    void testVerifyNever() {
        // Act - call method that shouldn't trigger email
        IllegalArgumentException exception = assertThrows(IllegalArgumentException.class, () -> {
            userService.getUserById(null);
        });
        
        // Verify email service was never called
        verify(emailService, never()).sendEmail(anyString(), anyString(), anyString());
        verify(emailService, never()).validateEmail(anyString());
    }
    
    @Test
    void testVerifyAtLeast() {
        // Arrange
        when(emailService.validateEmail(anyString())).thenReturn(true);
        when(userRepository.save(any(User.class)))
            .thenAnswer(invocation -> {
                User user = invocation.getArgument(0);
                return new User((long) (Math.random() * 1000), user.getName(), user.getEmail());
            });
        
        // Act - create multiple users
        userService.createUser("John", "john@example.com");
        userService.createUser("Jane", "jane@example.com");
        userService.createUser("Bob", "bob@example.com");
        userService.createUser("Alice", "alice@example.com");
        
        // Verify at least certain number of calls
        verify(emailService, atLeast(3)).validateEmail(anyString());
        verify(userRepository, atLeast(4)).save(any(User.class));
    }
    
    @Test
    void testVerifyAtMost() {
        // Arrange
        when(userRepository.findById(anyLong()))
            .thenReturn(new User(1L, "John", "john@example.com"));
        
        // Act
        userService.getUserById(1L);
        userService.getUserById(2L);
        
        // Verify at most certain number of calls
        verify(userRepository, atMost(5)).findById(anyLong());
        verify(userRepository, atMost(2)).findById(anyLong());
    }
    
    @Test
    void testVerifyBetween() {
        // Arrange
        when(userRepository.findById(anyLong()))
            .thenReturn(new User(1L, "John", "john@example.com"));
        
        // Act
        userService.getUserById(1L);
        userService.getUserById(2L);
        userService.getUserById(3L);
        
        // Verify calls are within range
        verify(userRepository, atLeast(2)).findById(anyLong());
        verify(userRepository, atMost(5)).findById(anyLong());
    }
}
```

### 10.2 Timeout and Asynchronous Verification

```java
@ExtendWith(MockitoExtension.class)
class TimeoutVerificationTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void testVerifyWithTimeout() throws InterruptedException {
        // Arrange
        when(userRepository.findById(1L))
            .thenReturn(new User(1L, "John", "john@example.com"));
        
        // Act in separate thread to simulate async behavior
        CompletableFuture.runAsync(() -> {
            try {
                Thread.sleep(100); // Simulate some processing time
                userService.getUserById(1L);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        // Verify method was called within timeout period
        verify(userRepository, timeout(1000)).findById(1L);
    }
    
    @Test
    void testVerifyWithTimeoutAndTimes() {
        // Arrange
        when(userRepository.findById(anyLong()))
            .thenReturn(new User(1L, "John", "john@example.com"));
        
        // Act - simulate multiple async calls
        for (int i = 0; i < 3; i++) {
            CompletableFuture.runAsync(() -> {
                try {
                    Thread.sleep(50);
                    userService.getUserById(1L);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }
        
        // Verify exact number of calls within timeout
        verify(userRepository, timeout(1000).times(3)).findById(anyLong());
    }
    
    @Test
    void testVerifyWithTimeoutAtLeast() {
        // Arrange
        when(userRepository.findById(anyLong()))
            .thenReturn(new User(1L, "John", "john@example.com"));
        
        // Act - simulate variable number of async calls
        for (int i = 0; i < 5; i++) {
            CompletableFuture.runAsync(() -> {
                try {
                    Thread.sleep(30);
                    userService.getUserById(1L);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }
        
        // Verify at least certain calls within timeout
        verify(userRepository, timeout(1000).atLeast(3)).findById(anyLong());
    }
    
    @Test
    void testAsynchronousVerification() {
        // Arrange
        AtomicBoolean emailSent = new AtomicBoolean(false);
        User testUser = new User(1L, "John", "john@example.com");
        
        when(userRepository.findById(1L)).thenReturn(testUser);
        when(userRepository.save(any(User.class))).thenReturn(testUser);
        
        doAnswer(invocation -> {
            // Simulate async email sending
            CompletableFuture.runAsync(() -> {
                try {
                    Thread.sleep(200);
                    emailSent.set(true);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
            return null;
        }).when(emailService).sendEmail(anyString(), anyString(), anyString());
        
        // Act
        userService.deactivateUser(1L);
        
        // Verify immediate interactions
        verify(userRepository).findById(1L);
        verify(userRepository).save(testUser);
        
        // Verify async operation with timeout
        verify(emailService, timeout(1000)).sendEmail(anyString(), anyString(), anyString());
    }
}
```

---

## 11) Spring Boot Testing Integration

### 11.1 Basic Spring Boot Test Setup

```java
/**
 * Spring Boot Application for testing
 */
@SpringBootApplication
public class TestApplication {
    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class, args);
    }
}

/**
 * JPA Entity for Spring Boot tests
 */
@Entity
@Table(name = "users")
public class UserEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String name;
    
    @Column(nullable = false, unique = true)
    private String email;
    
    @Column(nullable = false)
    private boolean active = true;
    
    // Constructors, getters, setters
    public UserEntity() {}
    
    public UserEntity(String name, String email) {
        this.name = name;
        this.email = email;
    }
    
    // Getters and setters...
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    public boolean isActive() { return active; }
    public void setActive(boolean active) { this.active = active; }
}

/**
 * Spring Data Repository
 */
@Repository
public interface UserJpaRepository extends JpaRepository<UserEntity, Long> {
    List<UserEntity> findByEmailContaining(String emailPattern);
    Optional<UserEntity> findByEmail(String email);
    boolean existsByEmail(String email);
}

/**
 * Spring Service
 */
@Service
@Transactional
public class SpringUserService {
    
    private final UserJpaRepository userRepository;
    private final SpringEmailService emailService;
    
    public SpringUserService(UserJpaRepository userRepository, SpringEmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
    
    public UserEntity createUser(String name, String email) {
        if (userRepository.existsByEmail(email)) {
            throw new IllegalArgumentException("Email already exists");
        }
        
        if (!emailService.validateEmail(email)) {
            throw new IllegalArgumentException("Invalid email format");
        }
        
        UserEntity user = new UserEntity(name, email);
        UserEntity savedUser = userRepository.save(user);
        
        emailService.sendWelcomeEmail(savedUser.getEmail(), savedUser.getName());
        
        return savedUser;
    }
    
    public UserEntity getUserById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new EntityNotFoundException("User not found with id: " + id));
    }
    
    public List<UserEntity> searchUsersByEmail(String emailPattern) {
        return userRepository.findByEmailContaining(emailPattern);
    }
    
    public void deactivateUser(Long id) {
        UserEntity user = getUserById(id);
        user.setActive(false);
        userRepository.save(user);
        
        emailService.sendAccountDeactivationEmail(user.getEmail(), user.getName());
    }
    
    public UserEntity updateUser(Long id, String name, String email) {
        UserEntity user = getUserById(id);
        
        if (!user.getEmail().equals(email) && userRepository.existsByEmail(email)) {
            throw new IllegalArgumentException("Email already exists");
        }
        
        if (!emailService.validateEmail(email)) {
            throw new IllegalArgumentException("Invalid email format");
        }
        
        user.setName(name);
        user.setEmail(email);
        
        return userRepository.save(user);
    }
}

/**
 * Spring Email Service
 */
@Service
public class SpringEmailService {
    
    private static final String EMAIL_REGEX = "^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$";
    private final Pattern emailPattern = Pattern.compile(EMAIL_REGEX);
    
    public boolean validateEmail(String email) {
        return email != null && emailPattern.matcher(email).matches();
    }
    
    public void sendWelcomeEmail(String email, String name) {
        // Simulate email sending
        System.out.println("Sending welcome email to: " + email);
    }
    
    public void sendAccountDeactivationEmail(String email, String name) {
        // Simulate email sending
        System.out.println("Sending deactivation email to: " + email);
    }
}
```

### 8.2 Spring Boot Unit Tests with Mockito

```java
/**
 * Unit tests for Spring Boot service using Mockito
 */
@ExtendWith(MockitoExtension.class)
class SpringUserServiceTest {
    
    @Mock
    private UserJpaRepository userRepository;
    
    @Mock
    private SpringEmailService emailService;
    
    @InjectMocks
    private SpringUserService userService;
    
    @Test
    void testCreateUser_Success() {
        // Arrange
        String name = "John Doe";
        String email = "john@example.com";
        
        when(userRepository.existsByEmail(email)).thenReturn(false);
        when(emailService.validateEmail(email)).thenReturn(true);
        when(userRepository.save(any(UserEntity.class))).thenAnswer(invocation -> {
            UserEntity user = invocation.getArgument(0);
            user.setId(1L);
            return user;
        });
        doNothing().when(emailService).sendWelcomeEmail(email, name);
        
        // Act
        UserEntity result = userService.createUser(name, email);
        
        // Assert
        assertNotNull(result);
        assertEquals(1L, result.getId());
        assertEquals(name, result.getName());
        assertEquals(email, result.getEmail());
        assertTrue(result.isActive());
        
        // Verify interactions
        verify(userRepository).existsByEmail(email);
        verify(emailService).validateEmail(email);
        verify(userRepository).save(any(UserEntity.class));
        verify(emailService).sendWelcomeEmail(email, name);
    }
    
    @Test
    void testCreateUser_EmailAlreadyExists() {
        // Arrange
        String email = "existing@example.com";
        when(userRepository.existsByEmail(email)).thenReturn(true);
        
        // Act & Assert
        IllegalArgumentException exception = assertThrows(IllegalArgumentException.class, () -> {
            userService.createUser("John", email);
        });
        
        assertEquals("Email already exists", exception.getMessage());
        
        // Verify only email existence check was called
        verify(userRepository).existsByEmail(email);
        verifyNoMoreInteractions(userRepository);
        verifyNoInteractions(emailService);
    }
    
    @Test
    void testCreateUser_InvalidEmail() {
        // Arrange
        String email = "invalid-email";
        when(userRepository.existsByEmail(email)).thenReturn(false);
        when(emailService.validateEmail(email)).thenReturn(false);
        
        // Act & Assert
        IllegalArgumentException exception = assertThrows(IllegalArgumentException.class, () -> {
            userService.createUser("John", email);
        });
        
        assertEquals("Invalid email format", exception.getMessage());
        
        // Verify validation was called but user wasn't saved
        verify(userRepository).existsByEmail(email);
        verify(emailService).validateEmail(email);
        verify(userRepository, never()).save(any(UserEntity.class));
        verify(emailService, never()).sendWelcomeEmail(anyString(), anyString());
    }
    
    @Test
    void testGetUserById_Success() {
        // Arrange
        Long userId = 1L;
        UserEntity user = new UserEntity("John", "john@example.com");
        user.setId(userId);
        
        when(userRepository.findById(userId)).thenReturn(Optional.of(user));
        
        // Act
        UserEntity result = userService.getUserById(userId);
        
        // Assert
        assertNotNull(result);
        assertEquals(userId, result.getId());
        assertEquals("John", result.getName());
        
        verify(userRepository).findById(userId);
    }
    
    @Test
    void testGetUserById_NotFound() {
        // Arrange
        Long userId = 999L;
        when(userRepository.findById(userId)).thenReturn(Optional.empty());
        
        // Act & Assert
        EntityNotFoundException exception = assertThrows(EntityNotFoundException.class, () -> {
            userService.getUserById(userId);
        });
        
        assertEquals("User not found with id: 999", exception.getMessage());
        verify(userRepository).findById(userId);
    }
    
    @Test
    void testDeactivateUser() {
        // Arrange
        Long userId = 1L;
        UserEntity user = new UserEntity("John", "john@example.com");
        user.setId(userId);
        user.setActive(true);
        
        when(userRepository.findById(userId)).thenReturn(Optional.of(user));
        when(userRepository.save(any(UserEntity.class))).thenReturn(user);
        doNothing().when(emailService).sendAccountDeactivationEmail(anyString(), anyString());
        
        // Act
        userService.deactivateUser(userId);
        
        // Assert
        assertFalse(user.isActive());
        
        verify(userRepository).findById(userId);
        verify(userRepository).save(user);
        verify(emailService).sendAccountDeactivationEmail("john@example.com", "John");
    }
}
```

---

## 12) Spring Boot Testing Utilities

### 12.1 @MockBean and @SpyBean

```java
/**
 * Integration tests using Spring Boot Test annotations
 */
@SpringBootTest
@TestPropertySource(properties = {
    "spring.datasource.url=jdbc:h2:mem:testdb",
    "spring.jpa.hibernate.ddl-auto=create-drop"
})
class SpringBootIntegrationTest {
    
    @Autowired
    private SpringUserService userService;
    
    @MockBean  // Replaces the bean in Spring context
    private SpringEmailService emailService;
    
    @Autowired
    private UserJpaRepository userRepository;
    
    @Test
    void testCreateUserWithMockBean() {
        // Arrange
        when(emailService.validateEmail("john@example.com")).thenReturn(true);
        doNothing().when(emailService).sendWelcomeEmail(anyString(), anyString());
        
        // Act
        UserEntity result = userService.createUser("John Doe", "john@example.com");
        
        // Assert
        assertNotNull(result);
        assertEquals("John Doe", result.getName());
        assertEquals("john@example.com", result.getEmail());
        assertTrue(result.isActive());
        
        // Verify mock interactions
        verify(emailService).validateEmail("john@example.com");
        verify(emailService).sendWelcomeEmail("john@example.com", "John Doe");
        
        // Verify database persistence
        Optional<UserEntity> savedUser = userRepository.findById(result.getId());
        assertTrue(savedUser.isPresent());
        assertEquals("John Doe", savedUser.get().getName());
    }
    
    @Test
    void testEmailValidationWithMockBean() {
        // Arrange - Mock different email validation scenarios
        when(emailService.validateEmail("valid@example.com")).thenReturn(true);
        when(emailService.validateEmail("invalid-email")).thenReturn(false);
        
        // Test valid email
        doNothing().when(emailService).sendWelcomeEmail(anyString(), anyString());
        UserEntity validUser = userService.createUser("Valid User", "valid@example.com");
        assertNotNull(validUser);
        
        // Test invalid email
        IllegalArgumentException exception = assertThrows(IllegalArgumentException.class, () -> {
            userService.createUser("Invalid User", "invalid-email");
        });
        assertEquals("Invalid email format", exception.getMessage());
    }
}

/**
 * Testing with @SpyBean for partial mocking
 */
@SpringBootTest
@TestPropertySource(properties = {
    "spring.datasource.url=jdbc:h2:mem:testdb",
    "spring.jpa.hibernate.ddl-auto=create-drop"
})
class SpringBootSpyBeanTest {
    
    @Autowired
    private SpringUserService userService;
    
    @SpyBean  // Wraps the real bean with spy capabilities
    private SpringEmailService emailService;
    
    @Autowired
    private UserJpaRepository userRepository;
    
    @Test
    void testWithSpyBean() {
        // Use real email validation but mock email sending
        doNothing().when(emailService).sendWelcomeEmail(anyString(), anyString());
        
        // Act - email validation uses real implementation
        UserEntity result = userService.createUser("John Doe", "john@example.com");
        
        // Assert
        assertNotNull(result);
        assertEquals("john@example.com", result.getEmail());
        
        // Verify real method was called
        verify(emailService).validateEmail("john@example.com");
        
        // Verify mocked method was called
        verify(emailService).sendWelcomeEmail("john@example.com", "John Doe");
    }
    
    @Test
    void testSpyBeanWithRealAndMockedBehavior() {
        // Partially mock - override only email sending
        doThrow(new RuntimeException("Email service temporarily unavailable"))
            .when(emailService).sendWelcomeEmail(anyString(), anyString());
        
        // Email validation will use real implementation
        // Email sending will throw exception
        
        RuntimeException exception = assertThrows(RuntimeException.class, () -> {
            userService.createUser("John Doe", "john@example.com");
        });
        
        assertEquals("Email service temporarily unavailable", exception.getMessage());
        
        // Verify real validation was called before exception
        verify(emailService).validateEmail("john@example.com");
    }
}
```

### 12.2 @DataJpaTest and @WebMvcTest

```java
/**
 * REST Controller for testing
 */
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private final SpringUserService userService;
    
    public UserController(SpringUserService userService) {
        this.userService = userService;
    }
    
    @PostMapping
    public ResponseEntity<UserEntity> createUser(@RequestBody CreateUserRequest request) {
        try {
            UserEntity user = userService.createUser(request.getName(), request.getEmail());
            return ResponseEntity.ok(user);
        } catch (IllegalArgumentException e) {
            return ResponseEntity.badRequest().build();
        }
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<UserEntity> getUser(@PathVariable Long id) {
        try {
            UserEntity user = userService.getUserById(id);
            return ResponseEntity.ok(user);
        } catch (EntityNotFoundException e) {
            return ResponseEntity.notFound().build();
        }
    }
    
    @PutMapping("/{id}/deactivate")
    public ResponseEntity<Void> deactivateUser(@PathVariable Long id) {
        try {
            userService.deactivateUser(id);
            return ResponseEntity.ok().build();
        } catch (EntityNotFoundException e) {
            return ResponseEntity.notFound().build();
        }
    }
    
    @GetMapping("/search")
    public ResponseEntity<List<UserEntity>> searchUsers(@RequestParam String email) {
        List<UserEntity> users = userService.searchUsersByEmail(email);
        return ResponseEntity.ok(users);
    }
    
    static class CreateUserRequest {
        private String name;
        private String email;
        
        // Constructors, getters, setters
        public CreateUserRequest() {}
        public CreateUserRequest(String name, String email) {
            this.name = name;
            this.email = email;
        }
        
        public String getName() { return name; }
        public void setName(String name) { this.name = name; }
        public String getEmail() { return email; }
        public void setEmail(String email) { this.email = email; }
    }
}

/**
 * Repository layer testing with @DataJpaTest
 */
@DataJpaTest
@TestPropertySource(properties = {
    "spring.jpa.hibernate.ddl-auto=create-drop"
})
class UserJpaRepositoryTest {
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Autowired
    private UserJpaRepository userRepository;
    
    @Test
    void testFindByEmailContaining() {
        // Arrange
        UserEntity user1 = new UserEntity("John", "john@gmail.com");
        UserEntity user2 = new UserEntity("Jane", "jane@yahoo.com");
        UserEntity user3 = new UserEntity("Bob", "bob@gmail.com");
        
        entityManager.persistAndFlush(user1);
        entityManager.persistAndFlush(user2);
        entityManager.persistAndFlush(user3);
        
        // Act
        List<UserEntity> gmailUsers = userRepository.findByEmailContaining("gmail");
        List<UserEntity> yahooUsers = userRepository.findByEmailContaining("yahoo");
        
        // Assert
        assertEquals(2, gmailUsers.size());
        assertEquals(1, yahooUsers.size());
        
        assertTrue(gmailUsers.stream().allMatch(user -> user.getEmail().contains("gmail")));
        assertTrue(yahooUsers.stream().allMatch(user -> user.getEmail().contains("yahoo")));
    }
    
    @Test
    void testExistsByEmail() {
        // Arrange
        UserEntity user = new UserEntity("John", "john@example.com");
        entityManager.persistAndFlush(user);
        
        // Act & Assert
        assertTrue(userRepository.existsByEmail("john@example.com"));
        assertFalse(userRepository.existsByEmail("nonexistent@example.com"));
    }
    
    @Test
    void testFindByEmail() {
        // Arrange
        UserEntity user = new UserEntity("John", "john@example.com");
        entityManager.persistAndFlush(user);
        
        // Act
        Optional<UserEntity> found = userRepository.findByEmail("john@example.com");
        Optional<UserEntity> notFound = userRepository.findByEmail("nonexistent@example.com");
        
        // Assert
        assertTrue(found.isPresent());
        assertEquals("John", found.get().getName());
        
        assertFalse(notFound.isPresent());
    }
}

/**
 * Web layer testing with @WebMvcTest
 */
@WebMvcTest(UserController.class)
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private SpringUserService userService;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    @Test
    void testCreateUser_Success() throws Exception {
        // Arrange
        UserController.CreateUserRequest request = new UserController.CreateUserRequest("John Doe", "john@example.com");
        UserEntity responseUser = new UserEntity("John Doe", "john@example.com");
        responseUser.setId(1L);
        
        when(userService.createUser("John Doe", "john@example.com")).thenReturn(responseUser);
        
        // Act & Assert
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id").value(1))
                .andExpect(jsonPath("$.name").value("John Doe"))
                .andExpect(jsonPath("$.email").value("john@example.com"))
                .andExpect(jsonPath("$.active").value(true));
        
        verify(userService).createUser("John Doe", "john@example.com");
    }
    
    @Test
    void testCreateUser_InvalidInput() throws Exception {
        // Arrange
        UserController.CreateUserRequest request = new UserController.CreateUserRequest("John", "invalid-email");
        
        when(userService.createUser("John", "invalid-email"))
            .thenThrow(new IllegalArgumentException("Invalid email format"));
        
        // Act & Assert
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isBadRequest());
        
        verify(userService).createUser("John", "invalid-email");
    }
    
    @Test
    void testGetUser_Success() throws Exception {
        // Arrange
        UserEntity user = new UserEntity("John Doe", "john@example.com");
        user.setId(1L);
        
        when(userService.getUserById(1L)).thenReturn(user);
        
        // Act & Assert
        mockMvc.perform(get("/api/users/1"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id").value(1))
                .andExpected(jsonPath("$.name").value("John Doe"))
                .andExpect(jsonPath("$.email").value("john@example.com"));
        
        verify(userService).getUserById(1L);
    }
    
    @Test
    void testGetUser_NotFound() throws Exception {
        // Arrange
        when(userService.getUserById(999L))
            .thenThrow(new EntityNotFoundException("User not found"));
        
        // Act & Assert
        mockMvc.perform(get("/api/users/999"))
                .andExpect(status().isNotFound());
        
        verify(userService).getUserById(999L);
    }
    
    @Test
    void testDeactivateUser() throws Exception {
        // Arrange
        doNothing().when(userService).deactivateUser(1L);
        
        // Act & Assert
        mockMvc.perform(put("/api/users/1/deactivate"))
                .andExpect(status().isOk());
        
        verify(userService).deactivateUser(1L);
    }
    
    @Test
    void testSearchUsers() throws Exception {
        // Arrange
        List<UserEntity> users = Arrays.asList(
            new UserEntity("John", "john@gmail.com"),
            new UserEntity("Jane", "jane@gmail.com")
        );
        users.get(0).setId(1L);
        users.get(1).setId(2L);
        
        when(userService.searchUsersByEmail("gmail")).thenReturn(users);
        
        // Act & Assert
        mockMvc.perform(get("/api/users/search")
                .param("email", "gmail"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$", hasSize(2)))
                .andExpect(jsonPath("$[0].name").value("John"))
                .andExpect(jsonPath("$[1].name").value("Jane"));
        
        verify(userService).searchUsersByEmail("gmail");
    }
}
```

### 9.3 TestContainers Integration

```java
/**
 * Integration tests with TestContainers for real database
 */
@SpringBootTest
@Testcontainers
@TestPropertySource(properties = {
    "spring.datasource.driver-class-name=org.postgresql.Driver",
    "spring.jpa.hibernate.ddl-auto=create-drop",
    "spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect"
})
class TestContainersIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:13")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
    
    @Autowired
    private SpringUserService userService;
    
    @MockBean
    private SpringEmailService emailService;
    
    @Autowired
    private UserJpaRepository userRepository;
    
    @Test
    void testUserCreationWithRealDatabase() {
        // Arrange
        when(emailService.validateEmail("john@example.com")).thenReturn(true);
        doNothing().when(emailService).sendWelcomeEmail(anyString(), anyString());
        
        // Act
        UserEntity user = userService.createUser("John Doe", "john@example.com");
        
        // Assert
        assertNotNull(user.getId());
        assertEquals("John Doe", user.getName());
        
        // Verify persistence in real database
        Optional<UserEntity> savedUser = userRepository.findById(user.getId());
        assertTrue(savedUser.isPresent());
        assertEquals("john@example.com", savedUser.get().getEmail());
        
        verify(emailService).validateEmail("john@example.com");
        verify(emailService).sendWelcomeEmail("john@example.com", "John Doe");
    }
    
    @Test
    void testEmailUniquenessConstraint() {
        // Arrange
        when(emailService.validateEmail(anyString())).thenReturn(true);
        doNothing().when(emailService).sendWelcomeEmail(anyString(), anyString());
        
        // Create first user
        userService.createUser("John Doe", "unique@example.com");
        
        // Try to create second user with same email
        IllegalArgumentException exception = assertThrows(IllegalArgumentException.class, () -> {
            userService.createUser("Jane Doe", "unique@example.com");
        });
        
        assertEquals("Email already exists", exception.getMessage());
        
        // Verify only one user exists
        List<UserEntity> users = userRepository.findByEmailContaining("unique@example.com");
        assertEquals(1, users.size());
        assertEquals("John Doe", users.get(0).getName());
    }
}
```

---

## 13) Best Practices and Advanced Scenarios

### 13.1 Testing Best Practices

```java
/**
 * Demonstrates testing best practices with Mockito
 */
@ExtendWith(MockitoExtension.class)
class TestingBestPracticesTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService userService;
    
    // Test data builders for consistent test setup
    private User createTestUser(Long id, String name, String email) {
        User user = new User(id, name, email);
        user.setActive(true);
        return user;
    }
    
    @Test
    @DisplayName("Should create user when valid input is provided")
    void shouldCreateUserWhenValidInput() {
        // Arrange
        String name = "John Doe";
        String email = "john@example.com";
        User expectedUser = createTestUser(1L, name, email);
        
        when(emailService.validateEmail(email)).thenReturn(true);
        when(userRepository.save(any(User.class))).thenReturn(expectedUser);
        
        // Act
        User actualUser = userService.createUser(name, email);
        
        // Assert
        assertThat(actualUser)
            .isNotNull()
            .extracting(User::getId, User::getName, User::getEmail, User::isActive)
            .containsExactly(1L, name, email, true);
        
        // Verify interactions
        verify(emailService).validateEmail(email);
        verify(userRepository).save(argThat(user -> 
            name.equals(user.getName()) && email.equals(user.getEmail())
        ));
    }
    
    @Test
    @DisplayName("Should throw IllegalArgumentException when email is invalid")
    void shouldThrowExceptionWhenEmailIsInvalid() {
        // Arrange
        String invalidEmail = "invalid-email";
        when(emailService.validateEmail(invalidEmail)).thenReturn(false);
        
        // Act & Assert
        assertThatThrownBy(() -> userService.createUser("John", invalidEmail))
            .isInstanceOf(IllegalArgumentException.class)
            .hasMessage("Invalid email format");
        
        // Verify no user was saved
        verify(userRepository, never()).save(any(User.class));
    }
    
    @ParameterizedTest
    @ValueSource(strings = {"", "   ", "a", "ab"})
    @DisplayName("Should reject users with invalid names")
    void shouldRejectInvalidNames(String invalidName) {
        // This test would require adding name validation to the service
        // when(userService.createUser(invalidName, "valid@email.com"))
        //     .thenThrow(IllegalArgumentException.class);
    }
    
    @Test
    @DisplayName("Should handle repository exceptions gracefully")
    void shouldHandleRepositoryExceptions() {
        // Arrange
        when(emailService.validateEmail(anyString())).thenReturn(true);
        when(userRepository.save(any(User.class)))
            .thenThrow(new DataAccessException("Database connection failed") {});
        
        // Act & Assert
        assertThatThrownBy(() -> userService.createUser("John", "john@example.com"))
            .isInstanceOf(DataAccessException.class)
            .hasMessageContaining("Database connection failed");
    }
}
```

### 13.2 Advanced Mocking Patterns

```java
/**
 * Advanced mocking patterns and techniques
 */
@ExtendWith(MockitoExtension.class)
class AdvancedMockingPatternsTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void testMockingStaticMethods() {
        // Mock static methods (requires mockito-inline)
        try (MockedStatic<LocalDateTime> mockedTime = mockStatic(LocalDateTime.class)) {
            LocalDateTime fixedTime = LocalDateTime.of(2023, 1, 1, 12, 0);
            mockedTime.when(LocalDateTime::now).thenReturn(fixedTime);
            
            // Use service that depends on current time
            LocalDateTime currentTime = LocalDateTime.now();
            assertEquals(fixedTime, currentTime);
        }
    }
    
    @Test
    void testMockingFinalClasses() {
        // Mock final classes (requires mockito-inline)
        String mockString = mock(String.class);
        when(mockString.length()).thenReturn(10);
        when(mockString.substring(0, 5)).thenReturn("Hello");
        
        assertEquals(10, mockString.length());
        assertEquals("Hello", mockString.substring(0, 5));
    }
    
    @Test
    void testDeepStubbing() {
        // Create mock with deep stubbing enabled
        UserRepository deepMock = mock(UserRepository.class, RETURNS_DEEP_STUBS);
        
        // Chain method calls on deep stub
        when(deepMock.findById(1L).getName()).thenReturn("John");
        
        // This would normally throw NPE, but deep stubs handle it
        assertEquals("John", deepMock.findById(1L).getName());
    }
    
    @Test
    void testMockingWithCustomAnswer() {
        // Custom Answer implementation
        when(userRepository.save(any(User.class))).thenAnswer(new Answer<User>() {
            private long idCounter = 1;
            
            @Override
            public User answer(InvocationOnMock invocation) throws Throwable {
                User user = invocation.getArgument(0);
                user.setId(idCounter++);
                
                // Simulate some processing time
                Thread.sleep(10);
                
                return user;
            }
        });
        
        when(emailService.validateEmail(anyString())).thenReturn(true);
        
        // Test multiple saves get different IDs
        User user1 = userService.createUser("John", "john@example.com");
        User user2 = userService.createUser("Jane", "jane@example.com");
        
        assertEquals(1L, user1.getId());
        assertEquals(2L, user2.getId());
    }
    
    @Test
    void testBehaviorDrivenDevelopment() {
        // BDD style testing with Mockito
        // Given
        given(emailService.validateEmail("john@example.com")).willReturn(true);
        given(userRepository.save(any(User.class))).willReturn(createTestUser());
        
        // When
        User result = userService.createUser("John", "john@example.com");
        
        // Then
        then(emailService).should().validateEmail("john@example.com");
        then(userRepository).should().save(any(User.class));
        assertThat(result).isNotNull();
    }
    
    @Test
    void testMockResetting() {
        // Setup initial mock behavior
        when(userRepository.findById(1L)).thenReturn(createTestUser());
        
        // Use the mock
        User user = userService.getUserById(1L);
        assertNotNull(user);
        
        // Reset the mock
        reset(userRepository);
        
        // Mock behavior is cleared
        when(userRepository.findById(1L)).thenReturn(null);
        User nullUser = userService.getUserById(1L);
        assertNull(nullUser);
    }
    
    @Test
    void testPartialMocking() {
        // Spy on real object
        UserService spyService = spy(userService);
        
        // Mock specific method while keeping others real
        doReturn(createTestUser()).when(spyService).getUserById(1L);
        
        // Call real method internally, but mocked method returns controlled value
        User user = spyService.getUserById(1L);
        assertNotNull(user);
        
        // Verify real method wasn't called due to mocking
        verify(userRepository, never()).findById(1L);
    }
    
    private User createTestUser() {
        return new User(1L, "Test User", "test@example.com");
    }
}
```

### 13.3 Complex Integration Scenarios

```java
/**
 * Complex integration testing scenarios
 */
@SpringBootTest
@TestPropertySource(properties = {
    "spring.datasource.url=jdbc:h2:mem:testdb",
    "spring.jpa.hibernate.ddl-auto=create-drop"
})
class ComplexIntegrationTest {
    
    @Autowired
    private SpringUserService userService;
    
    @MockBean
    private SpringEmailService emailService;
    
    @Autowired
    private UserJpaRepository userRepository;
    
    @Test
    @Transactional
    @Rollback
    void testTransactionalBehavior() {
        // Setup email service to fail after user is saved
        when(emailService.validateEmail(anyString())).thenReturn(true);
        doThrow(new RuntimeException("Email service failed"))
            .when(emailService).sendWelcomeEmail(anyString(), anyString());
        
        // This should rollback the user creation due to email failure
        assertThrows(RuntimeException.class, () -> {
            userService.createUser("John", "john@example.com");
        });
        
        // Verify user was not persisted due to rollback
        List<UserEntity> users = userRepository.findAll();
        assertTrue(users.isEmpty());
    }
    
    @Test
    void testConcurrentUserCreation() throws InterruptedException {
        // Setup email service
        when(emailService.validateEmail(anyString())).thenReturn(true);
        doNothing().when(emailService).sendWelcomeEmail(anyString(), anyString());
        
        int threadCount = 10;
        CountDownLatch startLatch = new CountDownLatch(1);
        CountDownLatch endLatch = new CountDownLatch(threadCount);
        List<Exception> exceptions = Collections.synchronizedList(new ArrayList<>());
        
        // Create multiple threads trying to create users simultaneously
        for (int i = 0; i < threadCount; i++) {
            final int userId = i;
            new Thread(() -> {
                try {
                    startLatch.await();
                    userService.createUser("User" + userId, "user" + userId + "@example.com");
                } catch (Exception e) {
                    exceptions.add(e);
                } finally {
                    endLatch.countDown();
                }
            }).start();
        }
        
        // Start all threads simultaneously
        startLatch.countDown();
        endLatch.await(5, TimeUnit.SECONDS);
        
        // Verify results
        assertTrue(exceptions.isEmpty(), "No exceptions should occur during concurrent creation");
        
        List<UserEntity> users = userRepository.findAll();
        assertEquals(threadCount, users.size());
        
        // Verify all email validations occurred
        verify(emailService, times(threadCount)).validateEmail(anyString());
        verify(emailService, times(threadCount)).sendWelcomeEmail(anyString(), anyString());
    }
    
    @Test
    void testEventualConsistency() throws InterruptedException {
        // Setup async email behavior
        CompletableFuture<Void> emailFuture = new CompletableFuture<>();
        when(emailService.validateEmail(anyString())).thenReturn(true);
        doAnswer(invocation -> {
            // Simulate async email processing
            CompletableFuture.runAsync(() -> {
                try {
                    Thread.sleep(100);
                    emailFuture.complete(null);
                } catch (InterruptedException e) {
                    emailFuture.completeExceptionally(e);
                }
            });
            return null;
        }).when(emailService).sendWelcomeEmail(anyString(), anyString());
        
        // Create user
        UserEntity user = userService.createUser("John", "john@example.com");
        assertNotNull(user);
        
        // Wait for async email processing
        emailFuture.get(1, TimeUnit.SECONDS);
        
        // Verify eventual consistency
        verify(emailService, timeout(1000)).sendWelcomeEmail("john@example.com", "John");
    }
}
```

---

This comprehensive Mockito and Spring Boot testing guide provides:

✅ **Complete Setup** - Dependencies, annotations, and configuration  
✅ **Method Mocking** - Basic to advanced stubbing techniques  
✅ **Conditional Returns** - Argument matchers and complex logic  
✅ **Exception Handling** - Throwing and handling exceptions in tests  
✅ **Verification** - Method calls, arguments, and timing  
✅ **Spring Integration** - @MockBean, @SpyBean, testing utilities  
✅ **Best Practices** - Clean test structure and advanced patterns  
✅ **Real-world Scenarios** - Complex integration and concurrent testing  

Each section includes complete, runnable examples with detailed explanations, making this guide both a learning resource and practical reference for implementing comprehensive test suites with JUnit 5, Mockito, and Spring Boot.

---

## 14) Advanced Interview Questions for Experienced Developers

This section contains challenging questions that experienced developers often encounter in technical interviews, along with comprehensive answers and practical examples.

### 14.1 JUnit 5 Advanced Concepts

#### Q1: Explain the difference between @ExtendWith and @RegisterExtension in JUnit 5. When would you use each?

**Answer:**

@ExtendWith and @RegisterExtension are two ways to integrate extensions in JUnit 5, each with distinct advantages:

**@ExtendWith - Declarative Approach:**
```java
@ExtendWith(MockitoExtension.class)
@ExtendWith(TestInfoParameterResolver.class)
class DeclarativeExtensionTest {
    // Extensions applied at class level
    // No configuration options
    // Applied to all test methods
}
```

**@RegisterExtension - Programmatic Approach:**
```java
class ProgrammaticExtensionTest {
    
    @RegisterExtension
    static DatabaseExtension database = DatabaseExtension.builder()
        .withCredentials("test", "password")
        .withSchema("test_schema")
        .build();
    
    @RegisterExtension
    TempDirectoryExtension tempDir = TempDirectoryExtension.builder()
        .withCleanup(true)
        .withPrefix("test-")
        .build();
}
```

**Key Differences:**
- **@ExtendWith**: Simple, declarative, no configuration
- **@RegisterExtension**: Programmatic, configurable, flexible

**When to use each:**
- Use **@ExtendWith** for standard extensions without configuration
- Use **@RegisterExtension** when you need configuration, conditional logic, or per-test customization

#### Q2: How does JUnit 5's test instance lifecycle work? What are the implications of @TestInstance(Lifecycle.PER_CLASS)?

**Answer:**

JUnit 5 supports two test instance lifecycles:

**PER_METHOD (Default):**
```java
class DefaultLifecycleTest {
    private int counter = 0;
    
    @Test
    void firstTest() {
        counter++;
        assertEquals(1, counter); // Always 1 - fresh instance
    }
    
    @Test
    void secondTest() {
        counter++;
        assertEquals(1, counter); // Always 1 - fresh instance
    }
}
```

**PER_CLASS:**
```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class PerClassLifecycleTest {
    private int counter = 0;
    
    @BeforeAll // Can be non-static
    void setUpAll() {
        System.out.println("Setup once");
    }
    
    @Test
    void firstTest() {
        counter++;
        assertEquals(1, counter);
    }
    
    @Test
    void secondTest() {
        counter++;
        assertEquals(2, counter); // Counter persists
    }
}
```

**Implications of PER_CLASS:**
- Single instance shared across all tests
- Instance variables persist between tests
- @BeforeAll/@AfterAll can be non-static
- Often requires @TestMethodOrder for predictable behavior
- Useful for integration tests and expensive setup/teardown

### 14.2 Mockito Advanced Techniques

#### Q3: Explain the difference between @Mock, @Spy, and @InjectMocks. What are the potential pitfalls?

**Answer:**

**@Mock - Pure Mock:**
```java
@Mock
private EmailService emailService; // All methods return null/defaults

@Test
void testMock() {
    // Returns null unless stubbed
    assertNull(emailService.validateEmail("test@example.com"));
    
    // Stub behavior
    when(emailService.validateEmail(anyString())).thenReturn(true);
    assertTrue(emailService.validateEmail("any@email.com"));
}
```

**@Spy - Partial Mock:**
```java
@Spy
private ValidationService validationService = new ValidationService();

@Test
void testSpy() {
    // Calls real method
    assertTrue(validationService.isValidEmail("real@email.com"));
    
    // Partial stubbing
    doReturn(false).when(validationService).isValidEmail("invalid");
    assertFalse(validationService.isValidEmail("invalid"));
}
```

**@InjectMocks - Dependency Injection:**
```java
@InjectMocks
private UserService userService; // Mocks injected here

@Test
void testInjectMocks() {
    // Setup mocks
    when(emailService.validateEmail(anyString())).thenReturn(true);
    
    // Service uses injected mocks
    User user = userService.createUser("John", "john@example.com");
    assertNotNull(user);
}
```

**Common Pitfalls:**

1. **Spy Stubbing:** Use `doReturn()` instead of `when()` to avoid calling real methods during stubbing
2. **InjectMocks Limitations:** Doesn't work with static methods or complex constructors
3. **Injection Strategy:** Ambiguity when multiple injection strategies are possible

#### Q4: How do you test static methods, final classes, and private methods? What are the trade-offs?

**Answer:**

**Static Method Mocking (requires mockito-inline):**
```java
@Test
void testStaticMethodMocking() {
    try (MockedStatic<StaticUtility> mockedStatic = mockStatic(StaticUtility.class)) {
        mockedStatic.when(() -> StaticUtility.formatCurrency(anyDouble()))
                   .thenReturn("$100.00");
        
        String result = PaymentService.processPayment(123.45);
        assertEquals("Payment processed: $100.00", result);
        
        mockedStatic.verify(() -> StaticUtility.formatCurrency(123.45));
    }
}
```

**Final Class Mocking:**
```java
@Test
void testFinalClassMocking() {
    FinalService finalService = mock(FinalService.class);
    when(finalService.process("input")).thenReturn("mocked output");
    
    assertEquals("mocked output", finalService.process("input"));
}
```

**Private Method Testing:**
```java
@Test
void testPrivateMethodTesting() {
    // Preferred: Test through public interface
    ServiceWithPrivateMethod service = new ServiceWithPrivateMethod();
    String result = service.publicMethodThatCallsPrivate("test");
    assertEquals("PROCESSED: TEST", result);
    
    // Alternative: Use reflection (not recommended)
    Method privateMethod = ServiceWithPrivateMethod.class
        .getDeclaredMethod("privateMethod", String.class);
    privateMethod.setAccessible(true);
    String directResult = (String) privateMethod.invoke(service, "direct");
}
```

**Trade-offs:**
- **Static Mocking**: Enables legacy testing but has performance overhead and global state issues
- **Final Mocking**: Works with third-party libraries but requires mockito-inline
- **Private Testing**: Direct access breaks encapsulation; prefer testing through public interface

### 14.3 Spring Boot Testing Strategies

#### Q5: Compare @SpringBootTest, @WebMvcTest, @DataJpaTest, and @JsonTest. When would you use each?

**Answer:**

**@SpringBootTest - Full Integration:**
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class FullIntegrationTest {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @MockBean // Only mock external dependencies
    private ExternalEmailService emailService;
    
    @Test
    void shouldPerformEndToEndTest() {
        // Tests entire application stack
        ResponseEntity<User> response = restTemplate.postForEntity(
            "/api/users", request, User.class);
        assertEquals(HttpStatus.CREATED, response.getStatusCode());
    }
}
```

**@WebMvcTest - Web Layer:**
```java
@WebMvcTest(UserController.class)
class WebLayerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService; // Mock service layer
    
    @Test
    void shouldTestControllerLogic() throws Exception {
        when(userService.createUser(anyString(), anyString()))
            .thenReturn(new User(1L, "John", "john@example.com"));
        
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(requestJson))
                .andExpect(status().isCreated())
                .andExpected(jsonPath("$.name").value("John"));
    }
}
```

**@DataJpaTest - Persistence Layer:**
```java
@DataJpaTest
class DataLayerTest {
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void shouldTestRepositoryOperations() {
        User user = entityManager.persistAndFlush(
            new User("John", "john@example.com"));
        
        Optional<User> found = userRepository.findById(user.getId());
        assertTrue(found.isPresent());
    }
}
```

**@JsonTest - Serialization:**
```java
@JsonTest
class JsonSerializationTest {
    
    @Autowired
    private JacksonTester<User> userJsonTester;
    
    @Test
    void shouldSerializeCorrectly() throws Exception {
        User user = new User(1L, "John", "john@example.com");
        JsonContent<User> json = userJsonTester.write(user);
        
        assertThat(json).extractingJsonPathStringValue("$.name")
            .isEqualTo("John");
    }
}
```

**When to Use Each:**
- **@SpringBootTest**: End-to-end integration, full system behavior
- **@WebMvcTest**: Controller logic, request/response handling, security
- **@DataJpaTest**: Repository methods, query logic, database constraints
- **@JsonTest**: Serialization/deserialization, custom converters

#### Q6: How do you test asynchronous operations, scheduled tasks, and event-driven architectures?

**Answer:**

**Async Operations:**
```java
@SpringBootTest
class AsyncOperationTest {
    
    @Autowired
    private AsyncUserService asyncUserService;
    
    @MockBean
    private EmailService emailService;
    
    @Test
    void shouldHandleAsyncOperations() throws Exception {
        when(emailService.sendWelcomeEmail(anyString())).thenReturn(true);
        
        CompletableFuture<User> future = asyncUserService.createUserAsync(
            "John", "john@example.com");
        
        // Wait with timeout
        User result = future.get(5, TimeUnit.SECONDS);
        assertNotNull(result);
        
        // Verify async interaction
        verify(emailService, timeout(2000)).sendWelcomeEmail("john@example.com");
    }
}
```

**Scheduled Tasks:**
```java
@SpringBootTest
class ScheduledTaskTest {
    
    @Autowired
    private ScheduledUserService scheduledService;
    
    @MockBean
    private UserRepository userRepository;
    
    @Test
    void shouldExecuteScheduledTask() {
        when(userRepository.findInactiveUsers(any(LocalDateTime.class)))
            .thenReturn(Arrays.asList(new User("Inactive", "inactive@example.com")));
        
        // Trigger manually for testing
        scheduledService.cleanupInactiveUsers();
        
        verify(userRepository).findInactiveUsers(any(LocalDateTime.class));
        verify(userRepository).delete(any(User.class));
    }
}
```

**Event-Driven Architecture:**
```java
@SpringBootTest
class EventDrivenTest {
    
    @Autowired
    private ApplicationEventPublisher eventPublisher;
    
    @SpyBean
    private UserEventListener userEventListener;
    
    @Test
    void shouldHandleEvents() {
        User user = new User(1L, "John", "john@example.com");
        UserCreatedEvent event = new UserCreatedEvent(this, user);
        
        eventPublisher.publishEvent(event);
        
        // Verify event processing
        verify(userEventListener, timeout(1000))
            .handleUserCreated(any(UserCreatedEvent.class));
    }
}
```

**Testing Strategies:**
- **Async**: Use timeouts and CompletableFuture.get() with proper exception handling
- **Scheduled**: Test logic separately from scheduling, use manual triggering
- **Events**: Test publication and handling separately, verify with timeout for async processing

### 14.4 Additional Advanced Questions

#### Q7: How would you implement a custom test slice annotation for your specific domain?

**Answer:**

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = {ServiceLayerTestConfiguration.class})
@TestPropertySource(locations = "classpath:service-test.properties")
@Transactional
@interface ServiceLayerTest {
}

@TestConfiguration
class ServiceLayerTestConfiguration {
    
    @Bean
    @Primary
    public Clock testClock() {
        return Clock.fixed(Instant.parse("2023-01-01T12:00:00Z"), ZoneId.systemDefault());
    }
    
    @Bean
    public ValidationService validationService() {
        return new ValidationService();
    }
}
```

This comprehensive Q&A section provides experienced developers with deep insights into advanced testing scenarios and best practices for JUnit 5, Mockito, and Spring Boot testing.
