---
title: "Java Mastery - Part 5: Spring Framework Ecosystem"
date: 2025-07-06 00:00:00 +0530
categories: [Java, Java Mastery]
tags: [Java, Programming, Spring, Spring-Boot, Spring-data, Spring-security, REST-API, Dependency-injection, Micro-services]
---

## Introduction

Welcome to Part 5 of the Complete Java Mastery series. In Parts 1-4, we covered Java fundamentals, JVM internals, modern features, and SDLC practices. Now we dive into the **Spring Framework** - the most popular enterprise Java framework.

Spring revolutionized Java development by introducing **Dependency Injection**, **Aspect-Oriented Programming**, and a comprehensive ecosystem that makes building production applications faster and easier. Spring Boot further simplified this by providing **auto-configuration** and **opinionated defaults**.

### What You'll Learn in Part 5

* **Spring Core**: Dependency Injection, IoC Container, Bean Lifecycle
* **Spring Boot**: Auto-configuration, Starters, Application Configuration
* **Spring Web**: RESTful APIs, Request Mapping, Exception Handling
* **Spring Data JPA**: Database Access, Repositories, Query Methods
* **Spring Security**: Authentication, Authorization, OAuth2, JWT
* **Spring AOP**: Cross-cutting Concerns, Aspects, Advice
* **Production Features**: Actuator, Profiles, Externalized Configuration

This knowledge enables you to:
* Build enterprise-grade Spring applications
* Create production-ready REST APIs
* Implement secure authentication and authorization
* Manage database operations efficiently
* Monitor and maintain Spring applications

---

## 5.1 Spring Core - Dependency Injection and IoC

### Understanding Dependency Injection

**The Problem Without DI:**

```java
// ❌ Tight coupling - hard to test, hard to change
public class UserService {
    private UserRepository repository = new MySQLUserRepository();  // Hardcoded!
    
    public User getUser(Long id) {
        return repository.findById(id);
    }
}

// Problems:
// 1. Can't swap MySQLUserRepository for PostgreSQLUserRepository
// 2. Can't mock repository for testing
// 3. UserService responsible for creating dependencies
```

**Solution: Dependency Injection**

```java
// ✅ Loose coupling - dependencies injected
public class UserService {
    private final UserRepository repository;
    
    // Constructor injection (preferred)
    public UserService(UserRepository repository) {
        this.repository = repository;
    }
    
    public User getUser(Long id) {
        return repository.findById(id);
    }
}

// Now we can:
// 1. Inject any UserRepository implementation
// 2. Mock repository in tests
// 3. Change implementation without modifying UserService
```

### Spring IoC Container

**Application Context:**

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

// Configuration class
@Configuration
public class AppConfig {
    
    @Bean
    public UserRepository userRepository() {
        return new MySQLUserRepository();
    }
    
    @Bean
    public UserService userService(UserRepository repository) {
        return new UserService(repository);
    }
}

// Bootstrap application
public class Application {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        
        UserService userService = context.getBean(UserService.class);
        User user = userService.getUser(1L);
    }
}
```

### Bean Configuration

**Three Ways to Define Beans:**

**1. Java Configuration (Recommended)**

```java
@Configuration
public class DatabaseConfig {
    
    @Bean
    public DataSource dataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
        dataSource.setUsername("root");
        dataSource.setPassword("password");
        dataSource.setMaximumPoolSize(10);
        return dataSource;
    }
    
    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```

**2. Component Scanning (Most Common)**

```java
// Enable component scanning
@Configuration
@ComponentScan(basePackages = "com.example")
public class AppConfig {
}

// Components are auto-detected
@Component
public class EmailService {
    public void sendEmail(String to, String message) {
        System.out.println("Sending email to " + to);
    }
}

@Service  // Specialization of @Component
public class UserService {
    private final UserRepository repository;
    private final EmailService emailService;
    
    @Autowired  // Optional in Spring 4.3+ for constructor injection
    public UserService(UserRepository repository, EmailService emailService) {
        this.repository = repository;
        this.emailService = emailService;
    }
}

@Repository  // Specialization of @Component
public class UserRepositoryImpl implements UserRepository {
    // Implementation
}

@Controller  // Specialization of @Component
public class UserController {
    // Web controller
}
```

**3. XML Configuration (Legacy)**

```xml
<!-- applicationContext.xml -->
<beans>
    <bean id="userRepository" class="com.example.MySQLUserRepository"/>
    
    <bean id="userService" class="com.example.UserService">
        <constructor-arg ref="userRepository"/>
    </bean>
</beans>
```

### Dependency Injection Types

**Constructor Injection (Recommended):**

```java
@Service
public class OrderService {
    private final OrderRepository orderRepository;
    private final PaymentService paymentService;
    private final EmailService emailService;
    
    // Constructor injection - immutable, all dependencies required
    public OrderService(OrderRepository orderRepository,
                       PaymentService paymentService,
                       EmailService emailService) {
        this.orderRepository = orderRepository;
        this.paymentService = paymentService;
        this.emailService = emailService;
    }
}

// Benefits:
// 1. Immutable (final fields)
// 2. All dependencies required (can't create invalid object)
// 3. Easy to test (just pass mocks to constructor)
// 4. No need for @Autowired in Spring 4.3+
```

**Setter Injection:**

```java
@Service
public class ReportService {
    private EmailService emailService;
    
    // Setter injection - optional dependencies
    @Autowired(required = false)
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
}

// Use case: Optional dependencies
```

**Field Injection (Not Recommended):**

```java
@Service
public class NotificationService {
    @Autowired
    private EmailService emailService;  // Field injection
    
    // Problems:
    // 1. Can't make field final (mutable)
    // 2. Hard to test (need reflection)
    // 3. Hides dependencies (no constructor shows what's needed)
}
```

### Bean Scopes

```java
// Singleton (default) - one instance per container
@Service
@Scope("singleton")
public class ConfigService {
    // Shared instance
}

// Prototype - new instance every time
@Service
@Scope("prototype")
public class RequestProcessor {
    // New instance for each injection
}

// Web scopes (Spring Web only)
@Service
@Scope("request")  // New instance per HTTP request
public class RequestContext {
}

@Service
@Scope("session")  // New instance per HTTP session
public class UserSession {
}

@Service
@Scope("application")  // One instance per ServletContext
public class AppConfig {
}
```

**Custom Scope Example:**

```java
@Configuration
public class AppConfig {
    
    @Bean
    @Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
    public TaskExecutor taskExecutor() {
        return new TaskExecutor();
    }
}
```

### Bean Lifecycle

```java
@Component
public class LifecycleBean {
    
    private DataSource dataSource;
    
    // 1. Constructor
    public LifecycleBean() {
        System.out.println("1. Constructor called");
    }
    
    // 2. Dependency injection
    @Autowired
    public void setDataSource(DataSource dataSource) {
        System.out.println("2. Dependencies injected");
        this.dataSource = dataSource;
    }
    
    // 3. Post-construct
    @PostConstruct
    public void init() {
        System.out.println("3. @PostConstruct - bean initialized");
        // Initialization logic here
    }
    
    // 4. Bean ready for use
    
    // 5. Pre-destroy
    @PreDestroy
    public void cleanup() {
        System.out.println("5. @PreDestroy - cleaning up");
        // Cleanup logic here
    }
}
```

**Alternative Lifecycle Methods:**

```java
@Configuration
public class AppConfig {
    
    @Bean(initMethod = "init", destroyMethod = "cleanup")
    public DataProcessor dataProcessor() {
        return new DataProcessor();
    }
}

public class DataProcessor {
    
    public void init() {
        System.out.println("Initialization method");
    }
    
    public void cleanup() {
        System.out.println("Destroy method");
    }
}
```

### Conditional Beans

```java
// Bean created only if class is on classpath
@Bean
@ConditionalOnClass(DataSource.class)
public DatabaseService databaseService() {
    return new DatabaseService();
}

// Bean created only if property is set
@Bean
@ConditionalOnProperty(name = "feature.email.enabled", havingValue = "true")
public EmailService emailService() {
    return new EmailService();
}

// Bean created only if another bean is missing
@Bean
@ConditionalOnMissingBean(DataSource.class)
public DataSource defaultDataSource() {
    return new EmbeddedDataSource();
}

// Custom condition
@Bean
@Conditional(ProductionCondition.class)
public MonitoringService monitoringService() {
    return new MonitoringService();
}

public class ProductionCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        String profile = context.getEnvironment().getActiveProfiles()[0];
        return "production".equals(profile);
    }
}
```

### Profiles

```java
// Development configuration
@Configuration
@Profile("dev")
public class DevConfig {
    
    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .build();
    }
}

// Production configuration
@Configuration
@Profile("prod")
public class ProdConfig {
    
    @Bean
    public DataSource dataSource() {
        HikariDataSource ds = new HikariDataSource();
        ds.setJdbcUrl("jdbc:mysql://prod-db:3306/myapp");
        return ds;
    }
}

// Activate profile
// 1. Command line: -Dspring.profiles.active=prod
// 2. application.properties: spring.profiles.active=prod
// 3. Environment variable: SPRING_PROFILES_ACTIVE=prod
```

---

## 5.2 Spring Boot Fundamentals

Spring Boot simplifies Spring application development with auto-configuration and opinionated defaults.

### Creating a Spring Boot Application

**Using Spring Initializr (start.spring.io):**

```xml
<!-- pom.xml -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.1</version>
</parent>

<dependencies>
    <!-- Web starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- Data JPA starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    
    <!-- PostgreSQL driver -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>
    
    <!-- Test starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

**Main Application Class:**

```java
@SpringBootApplication  // = @Configuration + @EnableAutoConfiguration + @ComponentScan
public class Application {
    
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

**What @SpringBootApplication Does:**

1. **@Configuration**: Marks class as configuration source
2. **@EnableAutoConfiguration**: Enables Spring Boot's auto-configuration
3. **@ComponentScan**: Scans current package and sub-packages for components

### Auto-Configuration

**How Auto-Configuration Works:**

```java
// Spring Boot auto-configures DataSource if:
// 1. spring-boot-starter-data-jpa is on classpath
// 2. DataSource class is available
// 3. No DataSource bean already defined
// 4. Database properties are configured

// application.properties
spring.datasource.url=jdbc:postgresql://localhost:5432/mydb
spring.datasource.username=postgres
spring.datasource.password=secret

// Spring Boot automatically creates:
// - DataSource
// - EntityManagerFactory
// - TransactionManager
// - JPA repositories
```

**Viewing Auto-Configuration:**

```bash
# Run with debug mode
java -jar myapp.jar --debug

# Or in application.properties
debug=true

# Shows:
# - Positive matches (auto-configuration applied)
# - Negative matches (why not applied)
# - Exclusions
```

**Excluding Auto-Configuration:**

```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class Application {
    // DataSource won't be auto-configured
}
```

### Application Configuration

**application.properties:**

```properties
# Server configuration
server.port=8080
server.servlet.context-path=/api

# Database configuration
spring.datasource.url=jdbc:postgresql://localhost:5432/mydb
spring.datasource.username=postgres
spring.datasource.password=secret
spring.datasource.hikari.maximum-pool-size=10

# JPA configuration
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# Logging
logging.level.root=INFO
logging.level.com.example=DEBUG
logging.file.name=app.log

# Custom properties
app.name=My Application
app.version=1.0.0
```

**application.yml (Alternative):**

```yaml
server:
  port: 8080
  servlet:
    context-path: /api

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: postgres
    password: secret
    hikari:
      maximum-pool-size: 10
  
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: true
    properties:
      hibernate:
        format_sql: true

logging:
  level:
    root: INFO
    com.example: DEBUG
  file:
    name: app.log

app:
  name: My Application
  version: 1.0.0
```

**Profile-Specific Configuration:**

```properties
# application.properties (default)
app.environment=default

# application-dev.properties
app.environment=development
spring.datasource.url=jdbc:h2:mem:testdb

# application-prod.properties
app.environment=production
spring.datasource.url=jdbc:postgresql://prod-db:5432/mydb
```

**Reading Configuration Properties:**

```java
// Method 1: @Value annotation
@Service
public class AppService {
    
    @Value("${app.name}")
    private String appName;
    
    @Value("${app.version}")
    private String version;
    
    @Value("${server.port:8080}")  // Default value: 8080
    private int port;
}

// Method 2: @ConfigurationProperties (type-safe)
@ConfigurationProperties(prefix = "app")
@Component
public class AppProperties {
    
    private String name;
    private String version;
    private String environment;
    
    // Getters and setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    // ...
}

// Usage
@Service
public class AppService {
    private final AppProperties appProperties;
    
    public AppService(AppProperties appProperties) {
        this.appProperties = appProperties;
    }
    
    public void printConfig() {
        System.out.println(appProperties.getName());
        System.out.println(appProperties.getVersion());
    }
}
```

**Nested Properties:**

```java
@ConfigurationProperties(prefix = "app")
@Component
public class AppProperties {
    
    private String name;
    private Database database = new Database();
    private Security security = new Security();
    
    public static class Database {
        private int maxConnections;
        private int timeout;
        // Getters/setters
    }
    
    public static class Security {
        private String secretKey;
        private int tokenExpiration;
        // Getters/setters
    }
    
    // Getters/setters for outer properties
}
```

```yaml
app:
  name: My Application
  database:
    max-connections: 20
    timeout: 5000
  security:
    secret-key: my-secret-key
    token-expiration: 3600
```

### Spring Boot Starters

**Common Starters:**

```xml
<!-- Web applications -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- Data JPA -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- Security -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<!-- Validation -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>

<!-- Actuator (monitoring) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

<!-- Test -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<!-- WebFlux (reactive) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>

<!-- Cache -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>

<!-- Redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!-- MongoDB -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>

<!-- Mail -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

**What Each Starter Includes:**

```
spring-boot-starter-web:
  ├── spring-boot-starter
  ├── spring-boot-starter-json
  ├── spring-boot-starter-tomcat (embedded server)
  ├── spring-web
  └── spring-webmvc

spring-boot-starter-data-jpa:
  ├── spring-boot-starter-jdbc
  ├── spring-data-jpa
  ├── hibernate-core
  └── jakarta.persistence-api
```

---

## 5.3 Spring Web - Building REST APIs

Spring Web MVC provides powerful features for building RESTful web services.

### Basic REST Controller

```java
@RestController  // = @Controller + @ResponseBody
@RequestMapping("/api/users")
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    // GET /api/users
    @GetMapping
    public List<User> getAllUsers() {
        return userService.findAll();
    }
    
    // GET /api/users/123
    @GetMapping("/{id}")
    public User getUserById(@PathVariable Long id) {
        return userService.findById(id);
    }
    
    // POST /api/users
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public User createUser(@RequestBody User user) {
        return userService.create(user);
    }
    
    // PUT /api/users/123
    @PutMapping("/{id}")
    public User updateUser(@PathVariable Long id, @RequestBody User user) {
        return userService.update(id, user);
    }
    
    // DELETE /api/users/123
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deleteUser(@PathVariable Long id) {
        userService.delete(id);
    }
}
```

### Request Mapping Variants

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    // Query parameters: /api/products?category=electronics&minPrice=100
    @GetMapping
    public List<Product> getProducts(
        @RequestParam String category,
        @RequestParam(required = false) Double minPrice,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size
    ) {
        return productService.findProducts(category, minPrice, page, size);
    }
    
    // Path variables: /api/products/electronics/123
    @GetMapping("/{category}/{id}")
    public Product getProduct(
        @PathVariable String category,
        @PathVariable Long id
    ) {
        return productService.findByCategoryAndId(category, id);
    }
    
    // Request headers
    @GetMapping("/secure")
    public Product getSecureProduct(
        @RequestHeader("Authorization") String token,
        @RequestHeader(value = "X-Request-ID", required = false) String requestId
    ) {
        return productService.findSecure(token);
    }
    
    // Multiple request methods
    @RequestMapping(value = "/search", method = {RequestMethod.GET, RequestMethod.POST})
    public List<Product> search(@RequestBody SearchCriteria criteria) {
        return productService.search(criteria);
    }
}
```

### Request/Response DTOs

```java
// Request DTO
public class CreateUserRequest {
    private String firstName;
    private String lastName;
    private String email;
    private String password;
    
    // Getters/setters
}

// Response DTO
public class UserResponse {
    private Long id;
    private String firstName;
    private String lastName;
    private String email;
    private LocalDateTime createdAt;
    
    // Constructor, getters/setters
    
    // Factory method
    public static UserResponse from(User user) {
        UserResponse response = new UserResponse();
        response.setId(user.getId());
        response.setFirstName(user.getFirstName());
        response.setLastName(user.getLastName());
        response.setEmail(user.getEmail());
        response.setCreatedAt(user.getCreatedAt());
        return response;
    }
}

// Controller using DTOs
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @PostMapping
    public ResponseEntity<UserResponse> createUser(@RequestBody CreateUserRequest request) {
        User user = userService.create(request);
        UserResponse response = UserResponse.from(user);
        return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(response);
    }
}
```

### Validation

```java
// Request DTO with validation
public class CreateUserRequest {
    
    @NotBlank(message = "First name is required")
    @Size(min = 2, max = 50, message = "First name must be between 2 and 50 characters")
    private String firstName;
    
    @NotBlank(message = "Last name is required")
    @Size(min = 2, max = 50)
    private String lastName;
    
    @NotBlank(message = "Email is required")
    @Email(message = "Email must be valid")
    private String email;
    
    @NotBlank(message = "Password is required")
    @Size(min = 8, message = "Password must be at least 8 characters")
    @Pattern(regexp = "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d).*$", 
             message = "Password must contain uppercase, lowercase, and digit")
    private String password;
    
    @Min(value = 18, message = "Age must be at least 18")
    @Max(value = 120, message = "Age must be at most 120")
    private Integer age;
    
    // Getters/setters
}

// Controller with validation
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @PostMapping
    public ResponseEntity<UserResponse> createUser(
        @Valid @RequestBody CreateUserRequest request,  // @Valid triggers validation
        BindingResult bindingResult
    ) {
        if (bindingResult.hasErrors()) {
            // Handle validation errors manually
            Map<String, String> errors = new HashMap<>();
            bindingResult.getFieldErrors().forEach(error -> 
                errors.put(error.getField(), error.getDefaultMessage())
            );
            return ResponseEntity.badRequest().body(errors);
        }
        
        User user = userService.create(request);
        return ResponseEntity.ok(UserResponse.from(user));
    }
}
```

**Custom Validators:**

```java
// Custom annotation
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = UniqueEmailValidator.class)
public @interface UniqueEmail {
    String message() default "Email already exists";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// Validator implementation
@Component
public class UniqueEmailValidator implements ConstraintValidator<UniqueEmail, String> {
    
    @Autowired
    private UserRepository userRepository;
    
    @Override
    public boolean isValid(String email, ConstraintValidatorContext context) {
        if (email == null) return true;
        return !userRepository.existsByEmail(email);
    }
}

// Usage
public class CreateUserRequest {
    @NotBlank
    @Email
    @UniqueEmail  // Custom validator
    private String email;
}
```

### Exception Handling

**Global Exception Handler:**

```java
@RestControllerAdvice  // Global exception handler
public class GlobalExceptionHandler {
    
    // Handle specific exception
    @ExceptionHandler(UserNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleUserNotFound(UserNotFoundException ex) {
        return new ErrorResponse(
            "USER_NOT_FOUND",
            ex.getMessage(),
            LocalDateTime.now()
        );
    }
    
    // Handle validation errors
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ValidationErrorResponse handleValidationErrors(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage())
        );
        
        return new ValidationErrorResponse(
            "VALIDATION_FAILED",
            "Invalid request data",
            errors,
            LocalDateTime.now()
        );
    }
    
    // Handle generic exceptions
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGenericException(Exception ex) {
        return new ErrorResponse(
            "INTERNAL_ERROR",
            "An unexpected error occurred",
            LocalDateTime.now()
        );
    }
}

// Error response DTOs
public record ErrorResponse(
    String code,
    String message,
    LocalDateTime timestamp
) {}

public record ValidationErrorResponse(
    String code,
    String message,
    Map<String, String> fieldErrors,
    LocalDateTime timestamp
) {}
```

**Custom Exception:**

```java
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(Long id) {
        super("User not found with id: " + id);
    }
}

// In service
@Service
public class UserService {
    
    public User findById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }
}
```

### ResponseEntity for Fine-Grained Control

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    // Return with custom status and headers
    @GetMapping("/{id}")
    public ResponseEntity<UserResponse> getUser(@PathVariable Long id) {
        User user = userService.findById(id);
        
        return ResponseEntity
            .ok()
            .header("X-Custom-Header", "value")
            .cacheControl(CacheControl.maxAge(60, TimeUnit.SECONDS))
            .body(UserResponse.from(user));
    }
    
    // Conditional response
    @GetMapping("/{id}/conditional")
    public ResponseEntity<UserResponse> getUserConditional(@PathVariable Long id) {
        Optional<User> userOpt = userService.findByIdOptional(id);
        
        return userOpt
            .map(user -> ResponseEntity.ok(UserResponse.from(user)))
            .orElse(ResponseEntity.notFound().build());
    }
    
    // Created with location header
    @PostMapping
    public ResponseEntity<UserResponse> createUser(@Valid @RequestBody CreateUserRequest request) {
        User user = userService.create(request);
        UserResponse response = UserResponse.from(user);
        
        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(user.getId())
            .toUri();
        
        return ResponseEntity
            .created(location)
            .body(response);
    }
}
```

### Content Negotiation

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    // Produces JSON by default
    @GetMapping("/{id}")
    public Product getProduct(@PathVariable Long id) {
        return productService.findById(id);
    }
    
    // Produces XML when requested
    @GetMapping(value = "/{id}", produces = "application/xml")
    public Product getProductXml(@PathVariable Long id) {
        return productService.findById(id);
    }
    
    // Consumes JSON
    @PostMapping(consumes = "application/json")
    public Product createProduct(@RequestBody Product product) {
        return productService.create(product);
    }
}
```

### CORS Configuration

```java
// Method-level CORS
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @CrossOrigin(origins = "http://localhost:3000")
    @GetMapping
    public List<User> getAllUsers() {
        return userService.findAll();
    }
}

// Global CORS configuration
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins("http://localhost:3000", "https://myapp.com")
            .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
            .allowedHeaders("*")
            .allowCredentials(true)
            .maxAge(3600);
    }
}
```

---

## 5.4 Spring Data JPA

Spring Data JPA simplifies database access with repository pattern and query methods.

### Entity Mapping

```java
@Entity
@Table(name = "users")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 50)
    private String firstName;
    
    @Column(nullable = false, length = 50)
    private String lastName;
    
    @Column(nullable = false, unique = true)
    private String email;
    
    @Column(nullable = false)
    private String password;
    
    @Column(nullable = false)
    private Integer age;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private UserStatus status;
    
    @CreatedDate
    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    @Column(nullable = false)
    private LocalDateTime updatedAt;
    
    // Relationships
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Order> orders = new ArrayList<>();
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "department_id")
    private Department department;
    
    @ManyToMany
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();
    
    // Constructors, getters, setters
}

public enum UserStatus {
    ACTIVE, INACTIVE, SUSPENDED
}
```

**Composite Primary Key:**

```java
@Embeddable
public class OrderItemId implements Serializable {
    private Long orderId;
    private Long productId;
    
    // Equals, hashCode, constructors
}

@Entity
@Table(name = "order_items")
public class OrderItem {
    
    @EmbeddedId
    private OrderItemId id;
    
    @ManyToOne
    @MapsId("orderId")
    @JoinColumn(name = "order_id")
    private Order order;
    
    @ManyToOne
    @MapsId("productId")
    @JoinColumn(name = "product_id")
    private Product product;
    
    private Integer quantity;
    private BigDecimal price;
}
```

### Repository Pattern

```java
// Basic repository
public interface UserRepository extends JpaRepository<User, Long> {
    // JpaRepository provides:
    // - save(User)
    // - findById(Long)
    // - findAll()
    // - delete(User)
    // - count()
    // - existsById(Long)
    // And many more...
}

// Usage in service
@Service
public class UserService {
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public List<User> findAll() {
        return userRepository.findAll();
    }
    
    public User findById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }
    
    public User create(User user) {
        return userRepository.save(user);
    }
    
    public void delete(Long id) {
        userRepository.deleteById(id);
    }
}
```

### Query Methods

**Derived Query Methods:**

```java
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Find by single field
    Optional<User> findByEmail(String email);
    List<User> findByFirstName(String firstName);
    
    // Find by multiple fields (AND)
    List<User> findByFirstNameAndLastName(String firstName, String lastName);
    
    // Find by multiple fields (OR)
    List<User> findByFirstNameOrLastName(String firstName, String lastName);
    
    // Comparison operators
    List<User> findByAgeGreaterThan(Integer age);
    List<User> findByAgeLessThanEqual(Integer age);
    List<User> findByAgeBetween(Integer minAge, Integer maxAge);
    
    // String operations
    List<User> findByFirstNameStartingWith(String prefix);
    List<User> findByEmailContaining(String substring);
    List<User> findByLastNameIgnoreCase(String lastName);
    
    // Collection operations
    List<User> findByStatusIn(Collection<UserStatus> statuses);
    List<User> findByRolesContaining(Role role);
    
    // Null checks
    List<User> findByDepartmentIsNull();
    List<User> findByDepartmentIsNotNull();
    
    // Ordering
    List<User> findByStatusOrderByCreatedAtDesc(UserStatus status);
    List<User> findAllByOrderByLastNameAscFirstNameAsc();
    
    // Limiting results
    List<User> findTop10ByStatus(UserStatus status);
    List<User> findFirst5ByOrderByCreatedAtDesc();
    
    // Exists check
    boolean existsByEmail(String email);
    
    // Count
    long countByStatus(UserStatus status);
    
    // Delete
    void deleteByStatus(UserStatus status);
    
    // Pagination
    Page<User> findByStatus(UserStatus status, Pageable pageable);
    Slice<User> findByAgeGreaterThan(Integer age, Pageable pageable);
}
```

**@Query Annotation:**

```java
public interface UserRepository extends JpaRepository<User, Long> {
    
    // JPQL query
    @Query("SELECT u FROM User u WHERE u.email = :email")
    Optional<User> findByEmailCustom(@Param("email") String email);
    
    // JPQL with multiple parameters
    @Query("SELECT u FROM User u WHERE u.firstName = ?1 AND u.lastName = ?2")
    List<User> findByName(String firstName, String lastName);
    
    // Native SQL query
    @Query(value = "SELECT * FROM users WHERE age > :age", nativeQuery = true)
    List<User> findUsersOlderThan(@Param("age") Integer age);
    
    // Join query
    @Query("SELECT u FROM User u JOIN u.roles r WHERE r.name = :roleName")
    List<User> findByRoleName(@Param("roleName") String roleName);
    
    // Projection query
    @Query("SELECT new com.example.dto.UserSummary(u.id, u.firstName, u.lastName) " +
           "FROM User u WHERE u.status = :status")
    List<UserSummary> findUserSummaries(@Param("status") UserStatus status);
    
    // Count query
    @Query("SELECT COUNT(u) FROM User u WHERE u.status = :status")
    long countActiveUsers(@Param("status") UserStatus status);
    
    // Update query
    @Modifying
    @Query("UPDATE User u SET u.status = :status WHERE u.id = :id")
    int updateUserStatus(@Param("id") Long id, @Param("status") UserStatus status);
    
    // Delete query
    @Modifying
    @Query("DELETE FROM User u WHERE u.status = :status")
    int deleteByStatus(@Param("status") UserStatus status);
}
```

### Pagination and Sorting

```java
// Repository with pagination
public interface UserRepository extends JpaRepository<User, Long> {
    Page<User> findByStatus(UserStatus status, Pageable pageable);
}

// Controller
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping
    public Page<UserResponse> getUsers(
        @RequestParam(required = false) UserStatus status,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "id") String sortBy,
        @RequestParam(defaultValue = "ASC") String direction
    ) {
        Sort.Direction sortDirection = Sort.Direction.fromString(direction);
        Pageable pageable = PageRequest.of(page, size, Sort.by(sortDirection, sortBy));
        
        Page<User> users = status != null
            ? userRepository.findByStatus(status, pageable)
            : userRepository.findAll(pageable);
        
        return users.map(UserResponse::from);
    }
}

// Example request: GET /api/users?page=0&size=20&sortBy=lastName&direction=DESC
```

### Projections

**Interface Projection:**

```java
public interface UserSummary {
    Long getId();
    String getFirstName();
    String getLastName();
    String getEmail();
}

public interface UserRepository extends JpaRepository<User, Long> {
    List<UserSummary> findAllBy();
    UserSummary findProjectedById(Long id);
}
```

**DTO Projection:**

```java
public class UserSummaryDTO {
    private Long id;
    private String fullName;
    private String email;
    
    public UserSummaryDTO(Long id, String firstName, String lastName, String email) {
        this.id = id;
        this.fullName = firstName + " " + lastName;
        this.email = email;
    }
    
    // Getters
}

public interface UserRepository extends JpaRepository<User, Long> {
    @Query("SELECT new com.example.dto.UserSummaryDTO(u.id, u.firstName, u.lastName, u.email) " +
           "FROM User u")
    List<UserSummaryDTO> findAllSummaries();
}
```

### Transactions

```java
@Service
public class OrderService {
    
    private final OrderRepository orderRepository;
    private final InventoryService inventoryService;
    private final PaymentService paymentService;
    
    // Default: propagation = REQUIRED, isolation = DEFAULT
    @Transactional
    public Order createOrder(CreateOrderRequest request) {
        // All operations in same transaction
        Order order = new Order();
        order.setUserId(request.getUserId());
        order.setItems(request.getItems());
        
        // Save order
        order = orderRepository.save(order);
        
        // Reserve inventory
        inventoryService.reserve(request.getItems());
        
        // Process payment
        paymentService.process(order.getId(), order.getTotal());
        
        return order;
        // If any operation fails, entire transaction rolls back
    }
    
    // Read-only transaction (optimization)
    @Transactional(readOnly = true)
    public List<Order> findAll() {
        return orderRepository.findAll();
    }
    
    // Custom isolation level
    @Transactional(isolation = Isolation.SERIALIZABLE)
    public void updateCriticalData() {
        // Strictest isolation level
    }
    
    // Custom propagation
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logActivity(String activity) {
        // Always starts new transaction
        // Commits independently of calling transaction
    }
}
```

**Transaction Propagation:**

```
REQUIRED (default): Use existing transaction, create new if none exists
REQUIRES_NEW: Always create new transaction, suspend current if exists
NESTED: Execute within nested transaction if exists, REQUIRED otherwise
MANDATORY: Must have existing transaction, throw exception if none
SUPPORTS: Use existing transaction if exists, execute non-transactionally otherwise
NOT_SUPPORTED: Execute non-transactionally, suspend current transaction if exists
NEVER: Execute non-transactionally, throw exception if transaction exists
```

---

## 5.5 Spring Security

Spring Security provides comprehensive security features for authentication, authorization, and protection against common attacks.

### Basic Security Configuration

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/users/**").hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
            )
            .httpBasic(Customizer.withDefaults())
            .csrf(csrf -> csrf.disable());  // Disable for REST APIs
        
        return http.build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### User Details Service

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {
    
    private final UserRepository userRepository;
    
    public CustomUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByEmail(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));
        
        return org.springframework.security.core.userdetails.User.builder()
            .username(user.getEmail())
            .password(user.getPassword())
            .roles(user.getRoles().stream()
                .map(Role::getName)
                .toArray(String[]::new))
            .accountExpired(false)
            .accountLocked(false)
            .credentialsExpired(false)
            .disabled(user.getStatus() != UserStatus.ACTIVE)
            .build();
    }
}
```

### JWT Authentication

**JWT Configuration:**

```java
@Component
public class JwtUtil {
    
    @Value("${jwt.secret}")
    private String secret;
    
    @Value("${jwt.expiration:3600000}")  // 1 hour default
    private Long expiration;
    
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        return createToken(claims, userDetails.getUsername());
    }
    
    private String createToken(Map<String, Object> claims, String subject) {
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + expiration);
        
        return Jwts.builder()
            .setClaims(claims)
            .setSubject(subject)
            .setIssuedAt(now)
            .setExpiration(expiryDate)
            .signWith(SignatureAlgorithm.HS512, secret)
            .compact();
    }
    
    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }
    
    public Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }
    
    public <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }
    
    private Claims extractAllClaims(String token) {
        return Jwts.parser()
            .setSigningKey(secret)
            .parseClaimsJws(token)
            .getBody();
    }
    
    public Boolean validateToken(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        return (username.equals(userDetails.getUsername()) && !isTokenExpired(token));
    }
    
    private Boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }
}
```

**JWT Filter:**

```java
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    private final JwtUtil jwtUtil;
    private final UserDetailsService userDetailsService;
    
    public JwtAuthenticationFilter(JwtUtil jwtUtil, UserDetailsService userDetailsService) {
        this.jwtUtil = jwtUtil;
        this.userDetailsService = userDetailsService;
    }
    
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                   HttpServletResponse response,
                                   FilterChain filterChain) throws ServletException, IOException {
        
        final String authHeader = request.getHeader("Authorization");
        
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }
        
        try {
            final String token = authHeader.substring(7);
            final String username = jwtUtil.extractUsername(token);
            
            if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);
                
                if (jwtUtil.validateToken(token, userDetails)) {
                    UsernamePasswordAuthenticationToken authToken = 
                        new UsernamePasswordAuthenticationToken(
                            userDetails,
                            null,
                            userDetails.getAuthorities()
                        );
                    
                    authToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                    SecurityContextHolder.getContext().setAuthentication(authToken);
                }
            }
        } catch (Exception e) {
            logger.error("JWT authentication failed", e);
        }
        
        filterChain.doFilter(request, response);
    }
}
```

**Security Config with JWT:**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    private final JwtAuthenticationFilter jwtAuthFilter;
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
    
    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config) 
            throws Exception {
        return config.getAuthenticationManager();
    }
}
```

**Authentication Controller:**

```java
@RestController
@RequestMapping("/api/auth")
public class AuthController {
    
    private final AuthenticationManager authenticationManager;
    private final JwtUtil jwtUtil;
    private final UserDetailsService userDetailsService;
    
    @PostMapping("/login")
    public ResponseEntity<AuthResponse> login(@RequestBody LoginRequest request) {
        try {
            authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(
                    request.getEmail(),
                    request.getPassword()
                )
            );
        } catch (BadCredentialsException e) {
            throw new UnauthorizedException("Invalid credentials");
        }
        
        final UserDetails userDetails = userDetailsService.loadUserByUsername(request.getEmail());
        final String token = jwtUtil.generateToken(userDetails);
        
        return ResponseEntity.ok(new AuthResponse(token));
    }
}

public record LoginRequest(String email, String password) {}
public record AuthResponse(String token) {}
```

### Method-Level Security

```java
@Configuration
@EnableMethodSecurity  // Enable method-level security
public class MethodSecurityConfig {
}

@Service
public class UserService {
    
    // Only accessible by ADMIN role
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
    
    // Only accessible by user who owns the resource
    @PreAuthorize("#id == authentication.principal.id or hasRole('ADMIN')")
    public User updateUser(Long id, UpdateUserRequest request) {
        return userRepository.save(user);
    }
    
    // Multiple conditions
    @PreAuthorize("hasRole('USER') and #userId == authentication.principal.id")
    public List<Order> getUserOrders(Long userId) {
        return orderRepository.findByUserId(userId);
    }
    
    // Check return value
    @PostAuthorize("returnObject.userId == authentication.principal.id")
    public Order getOrder(Long orderId) {
        return orderRepository.findById(orderId).orElseThrow();
    }
}
```

### Password Encoding

```java
@Service
public class UserService {
    
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    
    public User registerUser(CreateUserRequest request) {
        User user = new User();
        user.setEmail(request.getEmail());
        user.setPassword(passwordEncoder.encode(request.getPassword()));  // Encode password
        user.setFirstName(request.getFirstName());
        user.setLastName(request.getLastName());
        
        return userRepository.save(user);
    }
    
    public boolean checkPassword(String rawPassword, String encodedPassword) {
        return passwordEncoder.matches(rawPassword, encodedPassword);
    }
}
```

---

## 5.6 Spring AOP (Aspect-Oriented Programming)

AOP enables modularization of cross-cutting concerns like logging, security, and transactions.

### Basic Aspect

```java
@Aspect
@Component
public class LoggingAspect {
    
    private static final Logger logger = LoggerFactory.getLogger(LoggingAspect.class);
    
    // Before advice
    @Before("execution(* com.example.service.*.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        logger.info("Calling method: {}", joinPoint.getSignature().getName());
    }
    
    // After returning advice
    @AfterReturning(
        pointcut = "execution(* com.example.service.*.*(..))",
        returning = "result"
    )
    public void logAfterReturning(JoinPoint joinPoint, Object result) {
        logger.info("Method {} returned: {}", 
            joinPoint.getSignature().getName(), result);
    }
    
    // After throwing advice
    @AfterThrowing(
        pointcut = "execution(* com.example.service.*.*(..))",
        throwing = "error"
    )
    public void logAfterThrowing(JoinPoint joinPoint, Throwable error) {
        logger.error("Method {} threw exception: {}", 
            joinPoint.getSignature().getName(), error.getMessage());
    }
    
    // Around advice (most powerful)
    @Around("execution(* com.example.service.*.*(..))")
    public Object logAround(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        
        logger.info("Method {} starting", joinPoint.getSignature().getName());
        
        try {
            Object result = joinPoint.proceed();  // Execute actual method
            
            long duration = System.currentTimeMillis() - start;
            logger.info("Method {} completed in {}ms", 
                joinPoint.getSignature().getName(), duration);
            
            return result;
        } catch (Throwable e) {
            logger.error("Method {} failed", joinPoint.getSignature().getName(), e);
            throw e;
        }
    }
}
```

### Pointcut Expressions

```java
@Aspect
@Component
public class SecurityAspect {
    
    // All methods in service package
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceMethods() {}
    
    // All public methods
    @Pointcut("execution(public * *(..))")
    public void publicMethods() {}
    
    // Methods with @Transactional
    @Pointcut("@annotation(org.springframework.transaction.annotation.Transactional)")
    public void transactionalMethods() {}
    
    // Combine pointcuts
    @Pointcut("serviceMethods() && publicMethods()")
    public void publicServiceMethods() {}
    
    // Methods with specific annotation
    @Before("@annotation(com.example.annotation.RequirePermission)")
    public void checkPermission(JoinPoint joinPoint) {
        // Check permissions
    }
}
```

### Custom Annotations with AOP

```java
// Custom annotation
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Audited {
    String action() default "";
}

// Aspect
@Aspect
@Component
public class AuditAspect {
    
    private final AuditService auditService;
    
    @Around("@annotation(audited)")
    public Object audit(ProceedingJoinPoint joinPoint, Audited audited) throws Throwable {
        String username = SecurityContextHolder.getContext()
            .getAuthentication()
            .getName();
        
        String action = audited.action();
        String method = joinPoint.getSignature().getName();
        
        try {
            Object result = joinPoint.proceed();
            
            auditService.log(username, action, method, "SUCCESS");
            
            return result;
        } catch (Throwable e) {
            auditService.log(username, action, method, "FAILED");
            throw e;
        }
    }
}

// Usage
@Service
public class UserService {
    
    @Audited(action = "CREATE_USER")
    public User createUser(CreateUserRequest request) {
        // Method automatically audited
        return userRepository.save(user);
    }
    
    @Audited(action = "DELETE_USER")
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}
```

### Performance Monitoring

```java
@Aspect
@Component
public class PerformanceAspect {
    
    private static final Logger logger = LoggerFactory.getLogger(PerformanceAspect.class);
    
    @Around("execution(* com.example.service.*.*(..))")
    public Object monitorPerformance(ProceedingJoinPoint joinPoint) throws Throwable {
        String methodName = joinPoint.getSignature().toShortString();
        
        long start = System.currentTimeMillis();
        Object result = joinPoint.proceed();
        long duration = System.currentTimeMillis() - start;
        
        if (duration > 1000) {  // Log slow methods (> 1 second)
            logger.warn("Slow method detected: {} took {}ms", methodName, duration);
        }
        
        return result;
    }
}
```

---

## 5.7 Spring Boot Actuator

Actuator provides production-ready features for monitoring and managing applications.

### Enable Actuator

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### Configuration

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus,env,loggers
      base-path: /actuator
  
  endpoint:
    health:
      show-details: when-authorized
      show-components: when-authorized
  
  metrics:
    export:
      prometheus:
        enabled: true
```

### Built-in Endpoints

```
GET /actuator                    # List all endpoints
GET /actuator/health            # Application health
GET /actuator/info              # Application info
GET /actuator/metrics           # Application metrics
GET /actuator/metrics/{metric}  # Specific metric
GET /actuator/env               # Environment properties
GET /actuator/loggers           # Logger configuration
POST /actuator/loggers/{name}   # Change log level
GET /actuator/threaddump        # Thread dump
GET /actuator/heapdump          # Heap dump
```

### Custom Health Indicator

```java
@Component
public class DatabaseHealthIndicator implements HealthIndicator {
    
    private final DataSource dataSource;
    
    public DatabaseHealthIndicator(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    
    @Override
    public Health health() {
        try (Connection connection = dataSource.getConnection()) {
            if (connection.isValid(1)) {
                return Health.up()
                    .withDetail("database", "Available")
                    .withDetail("validationQuery", "SELECT 1")
                    .build();
            } else {
                return Health.down()
                    .withDetail("database", "Connection invalid")
                    .build();
            }
        } catch (Exception e) {
            return Health.down()
                .withDetail("database", "Connection failed")
                .withException(e)
                .build();
        }
    }
}
```

### Custom Info Contributor

```java
@Component
public class AppInfoContributor implements InfoContributor {
    
    @Override
    public void contribute(Info.Builder builder) {
        builder.withDetail("app", Map.of(
            "name", "My Application",
            "version", "1.0.0",
            "description", "Spring Boot REST API"
        ));
        
        builder.withDetail("team", Map.of(
            "name", "Engineering Team",
            "email", "team@example.com"
        ));
    }
}
```

### Custom Metrics

```java
@Service
public class OrderService {
    
    private final MeterRegistry meterRegistry;
    private final Counter orderCounter;
    private final Timer orderProcessingTimer;
    
    public OrderService(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        
        // Counter
        this.orderCounter = Counter.builder("orders.created")
            .description("Total orders created")
            .tag("status", "success")
            .register(meterRegistry);
        
        // Timer
        this.orderProcessingTimer = Timer.builder("order.processing.time")
            .description("Order processing time")
            .register(meterRegistry);
    }
    
    public Order createOrder(CreateOrderRequest request) {
        return orderProcessingTimer.record(() -> {
            Order order = processOrder(request);
            orderCounter.increment();  // Increment counter
            return order;
        });
    }
    
    // Gauge example
    @Bean
    public Gauge activeUsersGauge(MeterRegistry registry, UserService userService) {
        return Gauge.builder("users.active", userService, UserService::getActiveUserCount)
            .description("Number of active users")
            .register(registry);
    }
}
```

---

## Summary and Key Takeaways

### Spring Core Mastery

**Dependency Injection:**
* **Constructor injection** (preferred): Immutable, testable
* **Setter injection**: Optional dependencies
* **Field injection** (avoid): Hard to test

**Bean Scopes:**
* **Singleton** (default): One instance per container
* **Prototype**: New instance per injection
* **Request/Session**: Web-specific scopes

**Bean Lifecycle:**
1. Instantiation
2. Dependency injection
3. @PostConstruct
4. Bean ready
5. @PreDestroy

### Spring Boot Benefits

**Auto-Configuration:**
* Automatic configuration based on classpath
* Opinionated defaults
* Override with properties

**Starters:**
* Pre-packaged dependency sets
* spring-boot-starter-web, -data-jpa, -security
* Simplify dependency management

**Production Features:**
* Externalized configuration
* Profiles for environments
* Actuator for monitoring

### REST API Development

**Best Practices:**
* Use DTOs for request/response
* Validate input with @Valid
* Handle exceptions globally
* Return appropriate HTTP status codes
* Use ResponseEntity for fine control

**API Design:**
* RESTful resource naming
* Proper HTTP methods (GET, POST, PUT, DELETE)
* Pagination and sorting
* CORS configuration

### Data Access with Spring Data JPA

**Repository Pattern:**
* JpaRepository for CRUD operations
* Derived query methods
* @Query for custom queries
* Projections for selective fields

**Performance:**
* Lazy loading for associations
* Pagination for large datasets
* Read-only transactions
* Proper indexing

### Security Essentials

**Authentication:**
* UserDetailsService for user loading
* PasswordEncoder (BCrypt)
* JWT for stateless authentication

**Authorization:**
* Role-based access control (RBAC)
* Method-level security (@PreAuthorize)
* HTTP security configuration

**Best Practices:**
* HTTPS in production
* CSRF protection for state-changing operations
* Secure password storage
* Token expiration

### Production Checklist

**Configuration:**
- [ ] Externalized configuration (application.yml)
- [ ] Environment-specific profiles (dev, prod)
- [ ] Secure sensitive data (passwords, keys)
- [ ] Logging configuration

**Security:**
- [ ] Authentication implemented
- [ ] Authorization configured
- [ ] HTTPS enabled
- [ ] CORS configured properly
- [ ] Input validation

**Database:**
- [ ] Connection pooling configured
- [ ] Transaction management
- [ ] Database migrations (Flyway/Liquibase)
- [ ] Proper indexing

**Monitoring:**
- [ ] Actuator endpoints enabled
- [ ] Health checks configured
- [ ] Custom metrics
- [ ] Log aggregation
- [ ] Error tracking

**Testing:**
- [ ] Unit tests for services
- [ ] Integration tests for repositories
- [ ] Controller tests (MockMvc)
- [ ] Security tests

### Frequently Asked Questions

**Q1: When should I use @Component vs @Service vs @Repository?**

**A:** All are specializations of @Component:

* **@Component**: Generic stereotype for any Spring-managed component
* **@Service**: Business logic layer (makes intent clear)
* **@Repository**: Data access layer (adds exception translation)
* **@Controller/@RestController**: Web layer

```java
@Component
public class UtilityHelper { }  // Generic utility

@Service
public class UserService { }    // Business logic

@Repository
public class UserRepositoryImpl { }  // Data access

@RestController
public class UserController { }  // Web layer
```

Use the most specific annotation for clarity and semantics.

---

**Q2: How do I handle transactions across multiple databases?**

**A:** Use **JTA (Java Transaction API)** with a transaction manager like Atomikos:

```java
@Configuration
@EnableTransactionManagement
public class MultiDbConfig {
    
    @Bean
    @Primary
    public DataSource primaryDataSource() {
        // Primary database
    }
    
    @Bean
    public DataSource secondaryDataSource() {
        // Secondary database
    }
    
    @Bean
    public JpaTransactionManager primaryTransactionManager() {
        return new JpaTransactionManager(primaryEntityManagerFactory().getObject());
    }
    
    @Bean
    public JpaTransactionManager secondaryTransactionManager() {
        return new JpaTransactionManager(secondaryEntityManagerFactory().getObject());
    }
}

// Use specific transaction manager
@Service
public class MultiDbService {
    
    @Transactional("primaryTransactionManager")
    public void saveToPrimary() { }
    
    @Transactional("secondaryTransactionManager")
    public void saveToSecondary() { }
}
```

For distributed transactions, consider **eventual consistency** with messaging.

---

**Q3: What's the difference between @Autowired and constructor injection?**

**A:**

**@Autowired (field/setter):**
```java
@Service
public class UserService {
    @Autowired
    private UserRepository repository;  // Field injection
}
```

**Constructor injection:**
```java
@Service
public class UserService {
    private final UserRepository repository;
    
    public UserService(UserRepository repository) {  // No @Autowired needed
        this.repository = repository;
    }
}
```

**Constructor injection is preferred:**
* Immutable (final fields)
* Testable (just pass mocks)
* Required dependencies explicit
* No @Autowired needed (Spring 4.3+)

---

### Interview Questions

**Question 1: Explain the Spring Bean lifecycle.**

**Difficulty:** Mid-Level

**Topics:** Spring Core

**Answer:**

The Spring Bean lifecycle:

1. **Instantiation**: Spring container creates bean instance
2. **Populate properties**: Dependencies injected
3. **BeanNameAware**: setBeanName() called if implemented
4. **BeanFactoryAware**: setBeanFactory() called if implemented
5. **ApplicationContextAware**: setApplicationContext() called
6. **Pre-initialization**: BeanPostProcessor.postProcessBeforeInitialization()
7. **@PostConstruct**: Initialization callback executed
8. **InitializingBean**: afterPropertiesSet() called
9. **Custom init**: init-method executed
10. **Post-initialization**: BeanPostProcessor.postProcessAfterInitialization()
11. **Bean ready**: Bean available for use
12. **@PreDestroy**: Destruction callback
13. **DisposableBean**: destroy() called
14. **Custom destroy**: destroy-method executed

**Example:**
```java
@Component
public class LifecycleBean implements InitializingBean, DisposableBean {
    
    @PostConstruct
    public void postConstruct() {
        System.out.println("@PostConstruct");
    }
    
    @Override
    public void afterPropertiesSet() {
        System.out.println("InitializingBean.afterPropertiesSet()");
    }
    
    @PreDestroy
    public void preDestroy() {
        System.out.println("@PreDestroy");
    }
    
    @Override
    public void destroy() {
        System.out.println("DisposableBean.destroy()");
    }
}
```

---

**Question 2: How would you implement JWT authentication in Spring Boot?**

**Difficulty:** Senior

**Topics:** Spring Security, JWT

**Answer:**

JWT authentication implementation requires:

**1. JWT Utility** for token generation/validation  
**2. Authentication Filter** to validate tokens  
**3. Security Configuration** to integrate filter  
**4. Login endpoint** to issue tokens

**Key Components:**
```java
// 1. JWT Utility
@Component
public class JwtUtil {
    public String generateToken(UserDetails user) {
        return Jwts.builder()
            .setSubject(user.getUsername())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + 3600000))
            .signWith(SignatureAlgorithm.HS512, secret)
            .compact();
    }
    
    public boolean validateToken(String token, UserDetails user) {
        String username = extractUsername(token);
        return username.equals(user.getUsername()) && !isTokenExpired(token);
    }
}

// 2. Filter
public class JwtAuthFilter extends OncePerRequestFilter {
    protected void doFilterInternal(HttpServletRequest request, ...) {
        String token = extractToken(request);
        if (token != null && jwtUtil.validateToken(token, userDetails)) {
            // Set authentication in SecurityContext
        }
    }
}

// 3. Security Config
@Configuration
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) {
        return http
            .csrf().disable()
            .sessionManagement().sessionCreationPolicy(STATELESS)
            .and()
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class)
            .build();
    }
}
```

**Benefits:** Stateless, scalable, mobile-friendly

---

**End of Part 5: Spring Framework Ecosystem**

### What's Next

**Congratulations on Mastering Spring!**

**Completed Parts:**
* Part 1: Java Fundamentals
* Part 2: JVM Internals
* Part 3: Advanced Java Features
* Part 4: SDLC with Java
* Part 5: Spring Framework Ecosystem ✅

**Optional Advanced Topics:**
* **Part 6**: Microservices Architecture (Service discovery, API Gateway, Circuit breakers, Event-driven)
* **Part 7**: Reactive Programming (Spring WebFlux, Project Reactor, Reactive streams)
* **Part 8**: Cloud-Native Java (Docker, Kubernetes, Serverless, Cloud platforms)

### References

* [Spring Framework Documentation](https://docs.spring.io/spring-framework/reference/)
* [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/)
* [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/)
* [Spring Security Reference](https://docs.spring.io/spring-security/reference/)
* [Baeldung Spring Tutorials](https://www.baeldung.com/spring-tutorial)
* [Spring in Action by Craig Walls](https://www.manning.com/books/spring-in-action-sixth-edition)

---
