---
name: spring-boot-patterns
description: Master Spring Boot 3.x patterns, microservices architecture, reactive programming, and production deployment. Use when building Spring Boot applications, designing microservices, or implementing enterprise Spring patterns.
---

# Spring Boot Patterns

Comprehensive guide to Spring Boot 3.x patterns, microservices architecture, reactive programming, security, testing, and production deployment strategies.

## When to Use This Skill

- Building Spring Boot microservices and web applications
- Implementing reactive programming with WebFlux
- Setting up enterprise-grade Spring Boot applications
- Configuring authentication and authorization with Spring Security
- Designing data access layers with Spring Data
- Implementing test strategies for Spring Boot applications
- Optimizing Spring Boot applications for production
- Setting up monitoring and observability
- Configuring multi-environment deployments

## Core Patterns

### 1. Application Architecture Patterns
- **Layered Architecture**: Controller → Service → Repository → Entity
- **Hexagonal Architecture**: Ports and adapters for clean separation
- **Domain-Driven Design**: Bounded contexts and aggregate roots
- **CQRS**: Command Query Responsibility Segregation
- **Event-Driven Architecture**: Asynchronous communication with events

### 2. Configuration Management
- **Profiles**: Environment-specific configurations
- **External Configuration**: Environment variables and config servers
- **Property Validation**: @Validated configuration classes
- **Encrypted Properties**: Sensitive data protection

### 3. Data Access Patterns
- **Repository Pattern**: Spring Data JPA abstraction
- **DTO Pattern**: Data transfer objects for API communication
- **Entity Mapping**: JPA entities with proper relationships
- **Database Migration**: Flyway or Liquibase for schema changes

## Application Structure Patterns

### Standard Spring Boot Structure

```
src/main/java/com/example/myapp/
├── MyappApplication.java              # Main application class
├── config/                           # Configuration classes
│   ├── SecurityConfig.java          # Security configuration
│   ├── DatabaseConfig.java          # Database configuration
│   ├── WebConfig.java               # Web layer configuration
│   └── CacheConfig.java             # Cache configuration
├── controller/                      # Web layer
│   ├── UserController.java          # REST controllers
│   ├── Advice/                      # Global exception handlers
│   │   └── GlobalExceptionHandler.java
│   └── Filter/                      # Custom filters
│       └── LoggingFilter.java
├── service/                         # Business logic layer
│   ├── UserService.java            # Service interfaces
│   └── impl/                       # Service implementations
│       └── UserServiceImpl.java
├── repository/                      # Data access layer
│   ├── UserRepository.java         # Spring Data repositories
│   └── custom/                     # Custom repository methods
│       └── UserRepositoryCustom.java
├── domain/                         # Business domain models
│   ├── entity/                    # JPA entities
│   │   └── User.java
│   ├── dto/                       # Data transfer objects
│   │   ├── UserCreateRequest.java
│   │   └── UserResponse.java
│   └── exception/                 # Custom exceptions
│       ├── ResourceNotFoundException.java
│       └── BusinessException.java
├── security/                       # Security-related classes
│   ├── JwtTokenProvider.java      # JWT token management
│   ├── UserDetailsServiceImpl.java # User details service
│   └── SecurityUtils.java         # Security utilities
├── util/                          # Utility classes
│   ├── DateUtil.java              # Date/time utilities
│   └── ValidationUtil.java        # Validation utilities
└── integration/                   # External service integrations
    ├── PaymentClient.java         # External API clients
    └── NotificationService.java   # Notification services
```

### Clean Architecture Pattern

```java
// Controller Layer
@RestController
@RequestMapping("/api/v1/users")
@Validated
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @PostMapping
    public ResponseEntity<UserResponse> createUser(
            @Valid @RequestBody CreateUserRequest request) {

        UserResponse response = userService.createUser(request);
        return new ResponseEntity<>(response, HttpStatus.CREATED);
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserResponse> getUser(@PathVariable Long id) {
        UserResponse response = userService.getUserById(id);
        return ResponseEntity.ok(response);
    }
}

// Service Layer
@Service
@Transactional
@Slf4j
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;
    private final UserMapper userMapper;
    private final PasswordEncoder passwordEncoder;
    private final ApplicationEventPublisher eventPublisher;

    @Override
    public UserResponse createUser(CreateUserRequest request) {
        log.info("Creating user with email: {}", request.getEmail());

        // Check if user already exists
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new UserAlreadyExistsException(
                "User with email " + request.getEmail() + " already exists");
        }

        // Create entity
        User user = userMapper.toEntity(request);
        user.setPassword(passwordEncoder.encode(request.getPassword()));
        user.setRole(UserRole.USER);
        user.setEnabled(true);

        // Save entity
        User savedUser = userRepository.save(user);

        // Publish event
        eventPublisher.publishEvent(new UserCreatedEvent(savedUser.getId()));

        log.info("User created successfully with ID: {}", savedUser.getId());
        return userMapper.toResponse(savedUser);
    }
}

// Repository Layer
@Repository
public interface UserRepository extends JpaRepository<User, Long>, UserRepositoryCustom {

    boolean existsByEmail(String email);

    @Query("SELECT u FROM User u WHERE u.email = :email")
    Optional<User> findByEmail(@Param("email") String email);

    @Query("SELECT u FROM User u WHERE u.role = :role AND u.enabled = true")
    List<User> findByRoleAndEnabled(@Param("role") UserRole role, boolean enabled);
}

// Domain Entity
@Entity
@Table(name = "users")
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode(of = "id")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true, length = 255)
    private String email;

    @Column(nullable = false, length = 255)
    private String password;

    @Column(name = "first_name", length = 100)
    private String firstName;

    @Column(name = "last_name", length = 100)
    private String lastName;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private UserRole role;

    @Column(nullable = false)
    private boolean enabled = true;

    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    @Version
    private Long version;
}
```

## Configuration Patterns

### Security Configuration

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity(prePostEnabled = true)
@RequiredArgsConstructor
@Slf4j
public class SecurityConfig {

    private final JwtTokenProvider jwtTokenProvider;
    private final UserDetailsService userDetailsService;
    private final CorsConfigurationSource corsConfigurationSource;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf.disable())
            .cors(cors -> cors.configurationSource(corsConfigurationSource))
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/v1/auth/**").permitAll()
                .requestMatchers("/api/v1/public/**").permitAll()
                .requestMatchers("/actuator/health").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/v1/users/**").hasAnyRole("USER", "ADMIN")
                .requestMatchers(HttpMethod.POST, "/api/v1/users/**").hasRole("ADMIN")
                .requestMatchers(HttpMethod.PUT, "/api/v1/users/**").hasRole("ADMIN")
                .requestMatchers(HttpMethod.DELETE, "/api/v1/users/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider),
                           UsernamePasswordAuthenticationFilter.class)
            .addFilterBefore(new JwtAuthorizationFilter(jwtTokenProvider, userDetailsService),
                           UsernamePasswordAuthenticationFilter.class)
            .exceptionHandling(exception -> exception
                .authenticationEntryPoint(new JwtAuthenticationEntryPoint())
                .accessDeniedHandler(new CustomAccessDeniedHandler())
            )
            .build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }

    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration authConfig) throws Exception {
        return authConfig.getAuthenticationManager();
    }
}
```

### Database Configuration

```java
@Configuration
@EnableJpaAuditing(auditorAwareRef = "auditorProvider")
@EnableTransactionManagement
@Slf4j
public class DatabaseConfig {

    @Bean
    @Primary
    public DataSource primaryDataSource(
            @Value("${spring.datasource.url}") String url,
            @Value("${spring.datasource.username}") String username,
            @Value("${spring.datasource.password}") String password,
            @Value("${spring.datasource.hikari.maximum-pool-size}") int maxPoolSize) {

        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(url);
        config.setUsername(username);
        config.setPassword(password);
        config.setMaximumPoolSize(maxPoolSize);
        config.setMinimumIdle(5);
        config.setIdleTimeout(300000);
        config.setConnectionTimeout(20000);
        config.setMaxLifetime(1200000);
        config.setLeakDetectionThreshold(60000);

        config.setPoolName("PrimaryHikariPool");

        // Performance optimizations
        config.addDataSourceProperty("cachePrepStmts", "true");
        config.addDataSourceProperty("prepStmtCacheSize", "250");
        config.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");

        return new HikariDataSource(config);
    }

    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> {
            Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
            if (authentication == null || !authentication.isAuthenticated()) {
                return Optional.of("system");
            }
            return Optional.of(authentication.getName());
        };
    }

    @Bean
    public JpaTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {
        JpaTransactionManager transactionManager = new JpaTransactionManager();
        transactionManager.setEntityManagerFactory(entityManagerFactory);
        return transactionManager;
    }
}
```

### Cache Configuration

```java
@Configuration
@EnableCaching
@Slf4j
public class CacheConfig {

    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(Caffeine.newBuilder()
            .expireAfterWrite(30, TimeUnit.MINUTES)
            .maximumSize(1000)
            .recordStats());

        // Pre-populate cache definitions
        cacheManager.setCacheNames(Arrays.asList(
            "users", "products", "categories", "permissions"
        ));

        return cacheManager;
    }

    @Bean
    public CacheKeyGenerator cacheKeyGenerator() {
        return new SimpleKeyGenerator() {
            @Override
            public Object generate(Object target, Method method, Object... params) {
                return method.getName() + "_" + Arrays.toString(params);
            }
        };
    }
}
```

## Reactive Patterns

### Reactive WebFlux Controller

```java
@RestController
@RequestMapping("/api/v1/reactive/users")
@RequiredArgsConstructor
@Slf4j
public class ReactiveUserController {

    private final ReactiveUserService userService;

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Mono<UserResponse> createUser(
            @Valid @RequestBody Mono<CreateUserRequest> requestMono) {

        return requestMono
            .flatMap(userService::createUser)
            .doOnNext(user -> log.info("Created user: {}", user.getId()))
            .onErrorResume(e -> {
                log.error("Error creating user", e);
                return Mono.error(new BusinessException("Failed to create user", e));
            });
    }

    @GetMapping("/{id}")
    public Mono<UserResponse> getUser(@PathVariable Long id) {
        return userService.getUserById(id)
            .switchIfEmpty(Mono.error(new ResourceNotFoundException("User not found: " + id)));
    }

    @GetMapping
    public Flux<UserResponse> getAllUsers(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size,
            @RequestParam(defaultValue = "id") String sortBy,
            @RequestParam(defaultValue = "desc") String sortDir) {

        return userService.getAllUsers(PageRequest.of(page, size,
            Sort.by(Sort.Direction.fromString(sortDir), sortBy)));
    }

    @GetMapping("/stream")
    public Flux<UserResponse> streamUsers() {
        return userService.getAllUsers()
            .delayElements(Duration.ofMillis(100)); // Simulate real-time data
    }
}
```

### Reactive Service Implementation

```java
@Service
@Transactional
@Slf4j
public class ReactiveUserServiceImpl implements ReactiveUserService {

    private final ReactiveUserRepository userRepository;
    private final UserMapper userMapper;
    private final PasswordEncoder passwordEncoder;
    private final ApplicationEventPublisher eventPublisher;

    @Override
    public Mono<UserResponse> createUser(CreateUserRequest request) {
        return userRepository.existsByEmail(request.getEmail())
            .flatMap(exists -> {
                if (exists) {
                    return Mono.error(new UserAlreadyExistsException(
                        "User with email " + request.getEmail() + " already exists"));
                }
                return Mono.just(exists);
            })
            .flatMap(exists -> {
                User user = userMapper.toEntity(request);
                user.setPassword(passwordEncoder.encode(request.getPassword()));
                user.setRole(UserRole.USER);
                user.setEnabled(true);

                return userRepository.save(user);
            })
            .doOnSuccess(savedUser -> {
                eventPublisher.publishEvent(new UserCreatedEvent(savedUser.getId()));
                log.info("User created successfully: {}", savedUser.getId());
            })
            .map(userMapper::toResponse);
    }

    @Override
    public Mono<UserResponse> getUserById(Long id) {
        return userRepository.findById(id)
            .map(userMapper::toResponse)
            .switchIfEmpty(Mono.error(new ResourceNotFoundException("User not found: " + id)));
    }

    @Override
    public Flux<UserResponse> getAllUsers(PageRequest pageRequest) {
        return userRepository.findAll()
            .skip(pageRequest.getOffset())
            .take(pageRequest.getPageSize())
            .sort(createComparator(pageRequest.getSort()))
            .map(userMapper::toResponse);
    }

    private Comparator<User> createComparator(Sort sort) {
        return (user1, user2) -> {
            for (Sort.Order order : sort) {
                int comparison = compareUsers(user1, user2, order.getProperty());
                if (comparison != 0) {
                    return order.isAscending() ? comparison : -comparison;
                }
            }
            return 0;
        };
    }

    private int compareUsers(User user1, User user2, String property) {
        return switch (property.toLowerCase()) {
            case "id" -> Long.compare(user1.getId(), user2.getId());
            case "email" -> user1.getEmail().compareTo(user2.getEmail());
            case "name" -> user1.getFirstName().compareTo(user2.getFirstName());
            case "createdat" -> user1.getCreatedAt().compareTo(user2.getCreatedAt());
            default -> 0;
        };
    }
}
```

## Testing Patterns

### Unit Testing with @WebMvcTest

```java
@WebMvcTest(UserController.class)
@Import({SecurityConfig.class, JwtTokenProvider.class})
@ExtendWith(MockitoExtension.class)
class UserControllerWebTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    @WithMockUser(roles = "USER")
    void shouldReturnUserWhenGetUserById() throws Exception {
        // Given
        Long userId = 1L;
        UserResponse userResponse = new UserResponse(userId, "John Doe", "john@example.com", UserRole.USER);

        given(userService.getUserById(userId)).willReturn(Mono.just(userResponse));

        // When & Then
        mockMvc.perform(get("/api/v1/users/{id}", userId))
            .andExpect(status().isOk())
            .andExpect(content().contentType(MediaType.APPLICATION_JSON))
            .andExpect(jsonPath("$.id").value(userId))
            .andExpect(jsonPath("$.name").value("John Doe"))
            .andExpect(jsonPath("$.email").value("john@example.com"));
    }

    @Test
    void shouldReturnUnauthorizedWhenAccessWithoutAuthentication() throws Exception {
        mockMvc.perform(get("/api/v1/users/1"))
            .andExpect(status().isUnauthorized());
    }

    @Test
    @WithMockUser(roles = "ADMIN")
    void shouldCreateUserSuccessfully() throws Exception {
        // Given
        CreateUserRequest request = new CreateUserRequest("Jane Doe", "jane@example.com", "password123");
        UserResponse response = new UserResponse(2L, "Jane Doe", "jane@example.com", UserRole.USER);

        given(userService.createUser(any(CreateUserRequest.class))).willReturn(Mono.just(response));

        // When & Then
        mockMvc.perform(post("/api/v1/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(content().contentType(MediaType.APPLICATION_JSON))
            .andExpect(jsonPath("$.id").value(2L))
            .andExpect(jsonPath("$.name").value("Jane Doe"));
    }
}
```

### Integration Testing with @SpringBootTest

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
@Transactional
class UserControllerIntegrationTest {

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
    private TestRestTemplate restTemplate;

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private JwtTokenProvider jwtTokenProvider;

    private String adminToken;

    @BeforeEach
    void setUp() {
        // Create admin user and get token
        User admin = userRepository.save(User.builder()
            .email("admin@test.com")
            .password(passwordEncoder.encode("password"))
            .role(UserRole.ADMIN)
            .enabled(true)
            .build());

        adminToken = jwtTokenProvider.generateToken(admin.getEmail(), List.of("ROLE_ADMIN"));
    }

    @Test
    void shouldCreateAndRetrieveUser() {
        // Create user request
        CreateUserRequest request = new CreateUserRequest("John Doe", "john@example.com", "password123");

        // Create user
        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth(adminToken);
        HttpEntity<CreateUserRequest> createEntity = new HttpEntity<>(request, headers);

        ResponseEntity<UserResponse> createResponse = restTemplate.postForEntity(
            "/api/v1/users", createEntity, UserResponse.class);

        assertEquals(HttpStatus.CREATED, createResponse.getStatusCode());
        UserResponse createdUser = createResponse.getBody();
        assertNotNull(createdUser);
        assertEquals("John Doe", createdUser.getName());

        // Retrieve user
        ResponseEntity<UserResponse> getResponse = restTemplate.getForEntity(
            "/api/v1/users/" + createdUser.getId(), UserResponse.class);

        assertEquals(HttpStatus.OK, getResponse.getStatusCode());
        UserResponse retrievedUser = getResponse.getBody();
        assertNotNull(retrievedUser);
        assertEquals(createdUser.getId(), retrievedUser.getId());
        assertEquals("John Doe", retrievedUser.getName());
    }
}
```

## Production Patterns

### Health Check Configuration

```java
@Component
public class CustomHealthIndicator implements HealthIndicator {

    private final ExternalServiceClient externalServiceClient;

    @Override
    public Health health() {
        try {
            // Check external service availability
            boolean externalServiceUp = externalServiceClient.isHealthy();

            if (externalServiceUp) {
                return Health.up()
                    .withDetail("externalService", "Available")
                    .withDetail("timestamp", Instant.now())
                    .build();
            } else {
                return Health.down()
                    .withDetail("externalService", "Unavailable")
                    .withDetail("timestamp", Instant.now())
                    .build();
            }
        } catch (Exception e) {
            return Health.down()
                .withDetail("error", e.getMessage())
                .withDetail("timestamp", Instant.now())
                .build();
        }
    }
}
```

### Metrics Configuration

```java
@Configuration
public class MetricsConfig {

    @Bean
    public TimedAspect timedAspect(MeterRegistry registry) {
        return new TimedAspect(registry);
    }

    @Bean
    public CountedAspect countedAspect(MeterRegistry registry) {
        return new CountedAspect(registry);
    }

    @Bean
    public MeterRegistryCustomizer<MeterRegistry> metricsCommonTags() {
        return registry -> registry.config()
            .commonTags(
                "application", "my-spring-boot-app",
                "region", System.getenv().getOrDefault("REGION", "unknown")
            );
    }
}

// Custom metrics in services
@Service
public class UserServiceImpl implements UserService {

    private final Counter userCreationCounter;
    private final Timer userProcessingTimer;

    public UserServiceImpl(MeterRegistry meterRegistry, UserRepository userRepository) {
        this.userRepository = userRepository;
        this.userCreationCounter = Counter.builder("user.creation.total")
            .description("Total number of users created")
            .register(meterRegistry);
        this.userProcessingTimer = Timer.builder("user.processing.time")
            .description("Time taken to process user operations")
            .register(meterRegistry);
    }

    @Timed(value = "user.creation.time", description = "Time taken to create a user")
    @Counted(value = "user.creation.attempts", description = "Number of user creation attempts")
    public UserResponse createUser(CreateUserRequest request) {
        return Timer.Sample.start()
            .stop(userProcessingTimer, () -> {
                UserResponse response = processUserCreation(request);
                userCreationCounter.increment();
                return response;
            });
    }
}
```

## Deployment Patterns

### Docker Configuration

```dockerfile
# Multi-stage Dockerfile
FROM openjdk:21-jdk-slim as builder

WORKDIR /app

# Copy Maven wrapper and pom
COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .

# Download dependencies
RUN ./mvnw dependency:go-offline -B

# Copy source code
COPY src ./src

# Build application
RUN ./mvnw clean package -DskipTests

# Runtime stage
FROM openjdk:21-jre-slim

# Install curl for health checks
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

# Create app user
RUN groupadd -r spring && useradd -r -g spring spring

WORKDIR /app

# Copy the built jar
COPY --from=builder /app/target/*.jar app.jar

# Change ownership
RUN chown spring:spring app.jar

USER spring:spring

EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"]
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-boot-app
  labels:
    app: spring-boot-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: spring-boot-app
  template:
    metadata:
      labels:
        app: spring-boot-app
    spec:
      containers:
      - name: spring-boot-app
        image: my-registry/spring-boot-app:latest
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "prod"
        - name: SPRING_DATASOURCE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
        - name: SPRING_DATASOURCE_USERNAME
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: username
        - name: SPRING_DATASOURCE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: spring-boot-service
spec:
  selector:
    app: spring-boot-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: ClusterIP
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: spring-boot-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: spring-boot-app
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

This comprehensive Spring Boot patterns guide provides production-ready patterns, reactive programming techniques, security implementations, and deployment strategies for modern Spring Boot applications.