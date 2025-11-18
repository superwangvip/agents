---
name: modern-java-features
description: Master Java 21+ modern features including virtual threads, pattern matching, records, sealed classes, and structured concurrency. Use when leveraging latest Java capabilities, optimizing performance, or implementing modern Java patterns.
---

# Modern Java Features

Comprehensive guide to Java 17-21+ modern features including virtual threads, pattern matching, records, sealed classes, text blocks, switch expressions, and structured concurrency for enterprise applications.

## When to Use This Skill

- Implementing virtual threads for high-concurrency applications
- Using pattern matching for type-safe and readable code
- Working with records for immutable data structures
- Designing sealed class hierarchies for better domain modeling
- Implementing structured concurrency for complex async workflows
- Using text blocks for multi-line string handling
- Leveraging switch expressions and pattern matching for control flow
- Optimizing performance with modern JVM features

## Virtual Threads (Project Loom)

### Basic Virtual Thread Usage

```java
import java.util.concurrent.*;
import java.util.stream.*;

public class VirtualThreadExample {

    // Traditional platform thread vs virtual thread comparison
    public void compareThreadTypes() throws InterruptedException {
        // Platform threads (limited by OS)
        ExecutorService platformExecutor = Executors.newFixedThreadPool(100);

        // Virtual threads (lightweight, millions possible)
        ExecutorService virtualExecutor = Executors.newVirtualThreadPerTaskExecutor();

        // Traditional blocking I/O with platform threads
        CountDownLatch platformLatch = new CountDownLatch(1000);
        long platformStart = System.currentTimeMillis();

        for (int i = 0; i < 1000; i++) {
            platformExecutor.submit(() -> {
                try {
                    // Simulate blocking I/O
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    platformLatch.countDown();
                }
            });
        }

        platformLatch.await();
        long platformTime = System.currentTimeMillis() - platformStart;

        // Same work with virtual threads
        CountDownLatch virtualLatch = new CountDownLatch(1000);
        long virtualStart = System.currentTimeMillis();

        for (int i = 0; i < 1000; i++) {
            virtualExecutor.submit(() -> {
                try {
                    // Simulate blocking I/O (same blocking code)
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    virtualLatch.countDown();
                }
            });
        }

        virtualLatch.await();
        long virtualTime = System.currentTimeMillis() - virtualStart;

        System.out.printf("Platform threads: %dms%n", platformTime);
        System.out.printf("Virtual threads: %dms%n", virtualTime);

        platformExecutor.shutdown();
        virtualExecutor.shutdown();
    }

    // Virtual thread factory
    private static final ThreadFactory VIRTUAL_THREAD_FACTORY =
        Thread.ofVirtual()
            .name("worker-", 0)
            .priority(Thread.NORM_PRIORITY)
            .daemon(false)
            .factory();

    public void createVirtualThreadsWithFactory() {
        Thread virtualThread = VIRTUAL_THREAD_FACTORY.newThread(() -> {
            System.out.println("Running in virtual thread: " + Thread.currentThread());
            try {
                Thread.sleep(1000);
                System.out.println("Virtual thread completed");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        virtualThread.start();
    }

    // Structured concurrency with virtual threads
    public CompletableFuture<String> structuredConcurrencyExample() {
        return CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(500); // Simulate network call
                return "Data from service A";
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                throw new RuntimeException(e);
            }
        }, Executors.newVirtualThreadPerTaskExecutor())
        .thenCombine(
            CompletableFuture.supplyAsync(() -> {
                try {
                    Thread.sleep(300); // Simulate network call
                    return "Data from service B";
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    throw new RuntimeException(e);
                }
            }, Executors.newVirtualThreadPerTaskExecutor()),
            (resultA, resultB) -> resultA + " + " + resultB
        );
    }
}
```

### Virtual Thread Web Server

```java
import java.net.*;
import java.io.*;
import java.util.concurrent.*;

public class VirtualThreadWebServer {

    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(8080);
        System.out.println("Server started on port 8080");

        // Accept connections and create virtual threads for each
        while (true) {
            Socket clientSocket = serverSocket.accept();

            // Create virtual thread for each connection
            Thread.startVirtualThread(() -> handleClient(clientSocket));
        }
    }

    private static void handleClient(Socket clientSocket) {
        try (clientSocket;
             BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
             PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true)) {

            String requestLine = in.readLine();
            if (requestLine != null && requestLine.startsWith("GET")) {
                // Process HTTP request
                String response = "HTTP/1.1 200 OK\r\n" +
                                "Content-Type: text/plain\r\n" +
                                "Content-Length: 13\r\n" +
                                "\r\n" +
                                "Hello, World!";

                out.print(response);
            }
        } catch (IOException e) {
            System.err.println("Error handling client: " + e.getMessage());
        }
    }
}
```

## Pattern Matching

### instanceof Pattern Matching (Java 16+)

```java
public class PatternMatchingExample {

    public String processShape(Object shape) {
        // Traditional instanceof with casting
        if (shape instanceof Circle) {
            Circle circle = (Circle) shape;
            return "Circle with radius: " + circle.getRadius();
        } else if (shape instanceof Rectangle) {
            Rectangle rectangle = (Rectangle) shape;
            return "Rectangle with area: " + (rectangle.getWidth() * rectangle.getHeight());
        }
        return "Unknown shape";
    }

    // Modern instanceof pattern matching
    public String processShapeModern(Object shape) {
        if (shape instanceof Circle circle) {
            // 'circle' is automatically cast and available
            return "Circle with radius: " + circle.getRadius();
        } else if (shape instanceof Rectangle rectangle) {
            return "Rectangle with area: " + (rectangle.getWidth() * rectangle.getHeight());
        }
        return "Unknown shape";
    }

    // Nested pattern matching
    public double calculateArea(Object shape) {
        if (shape instanceof Circle circle && circle.getRadius() > 0) {
            return Math.PI * circle.getRadius() * circle.getRadius();
        } else if (shape instanceof Rectangle rectangle &&
                  rectangle.getWidth() > 0 && rectangle.getHeight() > 0) {
            return rectangle.getWidth() * rectangle.getHeight();
        }
        return 0;
    }
}

record Circle(double radius) {}
record Rectangle(double width, double height) {}
```

### Switch Expressions and Pattern Matching (Java 17+)

```java
public class SwitchExpressionExample {

    // Traditional switch statement
    public int getDaysInMonthTraditional(String month) {
        switch (month) {
            case "January":
            case "March":
            case "May":
            case "July":
            case "August":
            case "October":
            case "December":
                return 31;
            case "April":
            case "June":
            case "September":
            case "November":
                return 30;
            case "February":
                return 28; // Simplified
            default:
                throw new IllegalArgumentException("Unknown month: " + month);
        }
    }

    // Modern switch expression
    public int getDaysInMonth(String month) {
        return switch (month) {
            case "January", "March", "May", "July", "August", "October", "December" -> 31;
            case "April", "June", "September", "November" -> 30;
            case "February" -> 28;
            default -> throw new IllegalArgumentException("Unknown month: " + month);
        };
    }

    // Switch expression with pattern matching (Java 21)
    public double getArea(Object shape) {
        return switch (shape) {
            case Circle circle -> Math.PI * circle.radius() * circle.radius();
            case Rectangle rect -> rect.width() * rect.height();
            case Square square -> square.side() * square.side();
            case null -> 0;
            default -> throw new IllegalArgumentException("Unknown shape");
        };
    }

    // Guards in switch patterns
    public String describeShape(Object shape) {
        return switch (shape) {
            case Circle circle when circle.radius() > 10 -> "Large circle";
            case Circle circle -> "Small circle";
            case Rectangle rect when rect.width() == rect.height() -> "Square rectangle";
            case Rectangle rect -> "Rectangle";
            default -> "Unknown shape";
        };
    }

    // Nested patterns
    public String processContainer(Object container) {
        return switch (container) {
            case Box(Circle circle) -> "Box containing circle";
            case Box(Rectangle rect) when rect.width() > rect.height() -> "Box containing wide rectangle";
            case Box(var shape) -> "Box containing: " + shape.getClass().getSimpleName();
            case null -> "Null container";
            default -> "Unknown container";
        };
    }
}

record Box<T>(T content) {}
record Square(double side) {}
```

### Record Patterns (Java 21)

```java
public class RecordPatternsExample {

    // Traditional record deconstruction
    public void processPointTraditional(Point point) {
        int x = point.x();
        int y = point.y();
        System.out.println("Point coordinates: x=" + x + ", y=" + y);
    }

    // Record pattern matching
    public void processPoint(Point point) {
        if (point instanceof Point(int x, int y)) {
            System.out.println("Point coordinates: x=" + x + ", y=" + y);
        }
    }

    // Nested record patterns
    public void processLine(Line line) {
        if (line instanceof Line(Point(int x1, int y1), Point(int x2, int y2))) {
            double distance = Math.sqrt(Math.pow(x2 - x1, 2) + Math.pow(y2 - y1, 2));
            System.out.printf("Line distance: %.2f%n", distance);
        }
    }

    // Switch with record patterns
    public String describeGeometry(Object geometry) {
        return switch (geometry) {
            case Point(int x, int y) -> String.format("Point at (%d, %d)", x, y);
            case Circle(Point(int x, int y), double radius) ->
                String.format("Circle at (%d, %d) with radius %.2f", x, y, radius);
            case Rectangle(Point(int x, int y), int width, int height) ->
                String.format("Rectangle at (%d, %d) with size %dx%d", x, y, width, height);
            default -> "Unknown geometry";
        };
    }
}

record Point(int x, int y) {}
record Circle(Point center, double radius) {}
record Rectangle(Point topLeft, int width, int height) {}
record Line(Point start, Point end) {}
```

## Records and Sealed Classes

### Record Examples

```java
// Simple record
public record User(String id, String name, String email) {

    // Compact constructor for validation
    public User {
        if (id == null || id.isBlank()) {
            throw new IllegalArgumentException("ID cannot be null or blank");
        }
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be null or blank");
        }
        if (email == null || !email.contains("@")) {
            throw new IllegalArgumentException("Invalid email format");
        }
    }

    // Custom methods
    public String displayName() {
        return name + " <" + email + ">";
    }

    // Static factory method
    public static User of(String name, String email) {
        return new User(UUID.randomUUID().toString(), name, email);
    }

    // Override default method
    @Override
    public String toString() {
        return String.format("User[id=%s, name=%s, email=%s]", id, name, email);
    }
}

// Record with generics
public record ApiResponse<T>(boolean success, T data, String message) {

    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(true, data, null);
    }

    public static <T> ApiResponse<T> error(String message) {
        return new ApiResponse<>(false, null, message);
    }

    public boolean isSuccess() {
        return success && data != null;
    }
}

// Record implementing interface
public record Money(BigDecimal amount, Currency currency) implements Comparable<Money> {

    public Money {
        amount = amount.setScale(2, RoundingMode.HALF_UP);
        Objects.requireNonNull(currency, "Currency cannot be null");
    }

    @Override
    public int compareTo(Money other) {
        if (!currency.equals(other.currency)) {
            throw new IllegalArgumentException("Cannot compare different currencies");
        }
        return amount.compareTo(other.amount);
    }

    public Money add(Money other) {
        if (!currency.equals(other.currency)) {
            throw new IllegalArgumentException("Cannot add different currencies");
        }
        return new Money(amount.add(other.amount), currency);
    }
}
```

### Sealed Class Hierarchies

```java
// Sealed interface for payment methods
public sealed interface Payment
    permits CreditCardPayment, BankTransferPayment, DigitalWalletPayment {

    String getPaymentId();
    BigDecimal getAmount();
    PaymentStatus getStatus();

    default boolean isValid() {
        return getAmount().compareTo(BigDecimal.ZERO) > 0;
    }
}

// Sealed classes implementing the interface
public final class CreditCardPayment implements Payment {
    private final String paymentId;
    private final BigDecimal amount;
    private final String cardNumber;
    private final String cardholderName;
    private PaymentStatus status;

    public CreditCardPayment(String paymentId, BigDecimal amount,
                           String cardNumber, String cardholderName) {
        this.paymentId = paymentId;
        this.amount = amount;
        this.cardNumber = maskCardNumber(cardNumber);
        this.cardholderName = cardholderName;
        this.status = PaymentStatus.PENDING;
    }

    @Override
    public String getPaymentId() { return paymentId; }

    @Override
    public BigDecimal getAmount() { return amount; }

    @Override
    public PaymentStatus getStatus() { return status; }

    public String getCardNumber() { return cardNumber; }

    public String getCardholderName() { return cardholderName; }

    private static String maskCardNumber(String cardNumber) {
        if (cardNumber == null || cardNumber.length() < 4) {
            return "****";
        }
        return "****-****-****-" + cardNumber.substring(cardNumber.length() - 4);
    }
}

public non-sealed class BankTransferPayment implements Payment {
    // Implementation for bank transfers
    // non-sealed allows further extension
}

public final class DigitalWalletPayment implements Payment {
    // Implementation for digital wallet payments
}

// Using sealed classes in switch expressions
public class PaymentProcessor {

    public String processPayment(Payment payment) {
        return switch (payment) {
            case CreditCardPayment creditCard ->
                processCreditCard(creditCard);
            case BankTransferPayment bankTransfer ->
                processBankTransfer(bankTransfer);
            case DigitalWalletPayment wallet ->
                processDigitalWallet(wallet);
        };
    }

    private String processCreditCard(CreditCardPayment creditCard) {
        return "Processing credit card payment: " + creditCard.getCardNumber();
    }

    private String processBankTransfer(BankTransferPayment bankTransfer) {
        return "Processing bank transfer: " + bankTransfer.getPaymentId();
    }

    private String processDigitalWallet(DigitalWalletPayment wallet) {
        return "Processing digital wallet: " + wallet.getPaymentId();
    }
}

enum PaymentStatus {
    PENDING, APPROVED, REJECTED, COMPLETED
}
```

## Text Blocks and String Templates

### Text Blocks

```java
public class TextBlockExample {

    // Traditional string concatenation
    public String getHtmlTraditional(String title, String content) {
        return "<html>\n" +
               "  <head>\n" +
               "    <title>" + title + "</title>\n" +
               "  </head>\n" +
               "  <body>\n" +
               "    <h1>" + title + "</h1>\n" +
               "    <p>" + content + "</p>\n" +
               "  </body>\n" +
               "</html>";
    }

    // Text block (Java 15+)
    public String getHtmlWithTextBlock(String title, String content) {
        return """
               <html>
                 <head>
                   <title>%s</title>
                 </head>
                 <body>
                   <h1>%s</h1>
                   <p>%s</p>
                 </body>
               </html>
               """.formatted(title, title, content);
    }

    // Text block with embedded expressions
    public String generateUserReport(User user) {
        return """
               User Report
               ============

               Name: %s
               Email: %s
               ID: %s

               Account Status: %s
               Registration Date: %s

               Thank you for using our service!
               """.formatted(
                   user.name(),
                   user.email(),
                   user.id(),
                   user.isActive() ? "Active" : "Inactive",
                   user.registrationDate()
               );
    }

    // JSON with text blocks
    public String createJsonResponse(User user) {
        return """
               {
                 "id": "%s",
                 "name": "%s",
                 "email": "%s",
                 "active": %b,
                 "roles": [%s]
               }
               """.formatted(
                   user.id(),
                   user.name(),
                   user.email(),
                   user.isActive(),
                   user.roles().stream()
                       .map(role -> "\"" + role + "\"")
                       .collect(Collectors.joining(", "))
               );
    }

    // SQL query with text blocks
    public String createUserTableSql() {
        return """
               CREATE TABLE users (
                   id VARCHAR(36) PRIMARY KEY,
                   name VARCHAR(100) NOT NULL,
                   email VARCHAR(255) UNIQUE NOT NULL,
                   password_hash VARCHAR(255) NOT NULL,
                   created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                   updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
                   active BOOLEAN DEFAULT TRUE
               );

               CREATE INDEX idx_users_email ON users(email);
               CREATE INDEX idx_users_active ON users(active);
               """;
    }
}

record User(String id, String name, String email,
           boolean active, LocalDateTime registrationDate, Set<String> roles) {}
```

### String Templates (Preview Feature)

```java
// Note: String templates are a preview feature in Java 21
public class StringTemplateExample {

    // Basic string template
    public String createGreeting(String name, String title) {
        return STR."Hello \{title} \{name}!";
    }

    // Template with expressions
    public String formatCurrency(BigDecimal amount, Currency currency) {
        return STR."\{currency.getCurrencyCode()} \{amount.setScale(2, RoundingMode.HALF_UP)}";
    }

    // Multi-line template
    public String generateEmail(User user) {
        return STR."""
            Dear \{user.name()},

            Your account \{user.id()} has been created successfully.

            Email: \{user.email()}
            Status: \{user.isActive() ? "Active" : "Inactive"}

            Thank you for registering!

            Best regards,
            The Team
            """;
    }

    // SQL template with parameter binding
    public String createInsertQuery(User user) {
        return STR."""
            INSERT INTO users (id, name, email, active)
            VALUES (
                '\{user.id()}',
                '\{user.name().replace("'", "''")}',
                '\{user.email()}',
                \{user.isActive()}
            );
            """;
    }

    // Custom template processor
    public static class JsonTemplateProcessor implements StringTemplate.Processor {

        @Override
        public String process(StringTemplate template) {
            StringBuilder json = new StringBuilder("{\n");

            for (int i = 0; i < template.fragments().size(); i++) {
                if (i % 2 == 0) {
                    json.append(template.fragments().get(i));
                } else {
                    Object value = template.values().get(i / 2);
                    json.append(quoteValue(value.toString()));
                }
            }

            json.append("\n}");
            return json.toString();
        }

        private String quoteValue(String value) {
            if (value == null) return "null";
            return "\"" + value.replace("\"", "\\\"") + "\"";
        }
    }

    public String createJsonUser(User user) {
        JsonTemplateProcessor jsonProcessor = new JsonTemplateProcessor();
        return jsonProcessor.process(StringTemplate.of("""
            {
                "id": \{user.id()},
                "name": \{user.name()},
                "email": \{user.email()},
                "active": \{user.isActive()}
            }
            """));
    }
}
```

## Modern Collections and APIs

### Enhanced Collections

```java
import java.util.*;
import java.util.stream.*;

public class ModernCollectionsExample {

    // Immutable collections (Java 9+)
    public void demonstrateImmutableCollections() {
        List<String> immutableList = List.of("apple", "banana", "orange");
        Set<Integer> immutableSet = Set.of(1, 2, 3, 4, 5);
        Map<String, Integer> immutableMap = Map.of(
            "one", 1,
            "two", 2,
            "three", 3
        );

        // CopyOf methods for guaranteed immutability
        List<String> guaranteedImmutable = List.copyOf(Arrays.asList("a", "b", "c"));

        // Stream API enhancements
        List<String> filtered = immutableList.stream()
            .takeWhile(s -> !s.equals("orange"))
            .dropWhile(s -> s.equals("apple"))
            .toList(); // Java 16+ direct collector

        // Optional improvements
        Optional<String> optionalValue = Optional.of("hello");
        optionalValue.ifPresentOrElse(
            value -> System.out.println("Value: " + value),
            () -> System.out.println("Value is empty")
        );
    }

    // Collection factory methods
    public Map<String, List<String>> createGroupedData() {
        return Map.ofEntries(
            Map.entry("fruits", List.of("apple", "banana", "orange")),
            Map.entry("vegetables", List.of("carrot", "broccoli", "spinach")),
            Map.entry("grains", List.of("rice", "wheat", "oats"))
        );
    }

    // Stream improvements
    public List<String> processWithStreamEnhancements() {
        return IntStream.rangeClosed(1, 100)
            .filter(n -> n % 2 == 0)
            .mapToObj(Integer::toString)
            .takeWhile(s -> Integer.parseInt(s) <= 50)
            .dropWhile(s -> Integer.parseInt(s) <= 20)
            .collect(Collectors.toList());
    }

    // Optional stream mapping
    public List<String> getActiveUserNames(List<Optional<User>> userOptionals) {
        return userOptionals.stream()
            .flatMap(Optional::stream) // Java 9+
            .filter(User::isActive)
            .map(User::name)
            .toList();
    }
}
```

### HTTP Client (Java 11+)

```java
import java.net.URI;
import java.net.http.*;
import java.net.http.HttpResponse.BodyHandlers;
import java.time.Duration;
import java.util.concurrent.*;

public class ModernHttpClientExample {

    private final HttpClient httpClient;

    public ModernHttpClientExample() {
        this.httpClient = HttpClient.newBuilder()
            .version(HttpClient.Version.HTTP_2)
            .connectTimeout(Duration.ofSeconds(10))
            .executor(Executors.newVirtualThreadPerTaskExecutor())
            .build();
    }

    // Synchronous GET request
    public String get(String url) throws Exception {
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(url))
            .header("Accept", "application/json")
            .timeout(Duration.ofSeconds(30))
            .GET()
            .build();

        HttpResponse<String> response = httpClient.send(
            request, BodyHandlers.ofString()
        );

        if (response.statusCode() >= 200 && response.statusCode() < 300) {
            return response.body();
        } else {
            throw new RuntimeException("HTTP error: " + response.statusCode());
        }
    }

    // Asynchronous GET request
    public CompletableFuture<String> getAsync(String url) {
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(url))
            .header("Accept", "application/json")
            .timeout(Duration.ofSeconds(30))
            .GET()
            .build();

        return httpClient.sendAsync(request, BodyHandlers.ofString())
            .thenApply(response -> {
                if (response.statusCode() >= 200 && response.statusCode() < 300) {
                    return response.body();
                } else {
                    throw new RuntimeException("HTTP error: " + response.statusCode());
                }
            });
    }

    // POST request with JSON body
    public CompletableFuture<String> postJson(String url, String jsonBody) {
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(url))
            .header("Content-Type", "application/json")
            .header("Accept", "application/json")
            .timeout(Duration.ofSeconds(30))
            .POST(HttpRequest.BodyPublishers.ofString(jsonBody))
            .build();

        return httpClient.sendAsync(request, BodyHandlers.ofString())
            .thenApply(response -> {
                if (response.statusCode() >= 200 && response.statusCode() < 300) {
                    return response.body();
                } else {
                    throw new RuntimeException("HTTP error: " + response.statusCode());
                }
            });
    }

    // Concurrent requests
    public CompletableFuture<List<String>> fetchMultipleUrls(List<String> urls) {
        List<CompletableFuture<String>> futures = urls.stream()
            .map(this::getAsync)
            .toList();

        return CompletableFuture.allOf(futures.toArray(new CompletableFuture[0]))
            .thenApply(v -> futures.stream()
                .map(CompletableFuture::join)
                .toList());
    }
}
```

This comprehensive modern Java features guide provides practical examples and best practices for leveraging the latest Java capabilities in enterprise applications, including virtual threads, pattern matching, records, sealed classes, and modern APIs.