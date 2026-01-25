---
title: "Java Mastery - Supplement 1: Quick Reference Cheat Sheet"
date: 2025-07-15 00:00:00 +0530
categories: [Java, Java Mastery]
tags: [Java, Programming, Reference, Cheat-Sheet, Quick-reference, Syntax, Patterns, Commands]
---

# 🚀 Complete Java Mastery - Quick Reference Cheat Sheet

## Java Fundamentals (Part 1)

### Data Types
```java
// Primitives
byte b = 127;           // 8-bit  (-128 to 127)
short s = 32767;        // 16-bit (-32,768 to 32,767)
int i = 2147483647;     // 32-bit
long l = 9223372036854775807L;  // 64-bit
float f = 3.14f;        // 32-bit
double d = 3.14159;     // 64-bit (default)
char c = 'A';           // 16-bit Unicode
boolean bool = true;    // true/false

// Wrapper Classes
Integer, Long, Double, Boolean, Character
```

### Collections
```java
List<String> list = new ArrayList<>();
Set<String> set = new HashSet<>();
Map<String, Integer> map = new HashMap<>();
Queue<String> queue = new LinkedList<>();
Deque<String> deque = new ArrayDeque<>();
```

### String Operations
```java
String s = "Hello";
s.length()              // 5
s.charAt(0)             // 'H'
s.substring(1, 4)       // "ell"
s.contains("ell")       // true
s.toUpperCase()         // "HELLO"
s.split(",")            // String[]
String.format("%s %d", "Age", 25)
```

## Modern Java (Part 3)

### Lambda & Streams
```java
// Lambda
(x, y) -> x + y
list.forEach(item -> System.out.println(item));

// Streams
list.stream()
    .filter(x -> x > 10)
    .map(x -> x * 2)
    .sorted()
    .distinct()
    .limit(5)
    .collect(Collectors.toList());
```

### Optional
```java
Optional<String> opt = Optional.ofNullable(value);
opt.isPresent()
opt.ifPresent(v -> System.out.println(v))
opt.orElse("default")
opt.orElseThrow()
opt.map(String::toUpperCase)
```

## JVM & Performance (Parts 2, 9)

### JVM Flags
```bash
# Memory
-Xms4g -Xmx4g              # Heap size
-Xmn1500m                  # Young generation
-XX:MetaspaceSize=512m     # Metaspace

# GC
-XX:+UseG1GC               # G1 garbage collector
-XX:+UseZGC                # ZGC (low latency)
-XX:MaxGCPauseMillis=200   # Target pause time

# Diagnostics
-XX:+HeapDumpOnOutOfMemoryError
-Xlog:gc*:file=gc.log
-XX:NativeMemoryTracking=summary
```

### Common Commands
```bash
# Monitor
jps                        # List Java processes
jstat -gc <pid> 1000       # GC stats every 1s
jmap -heap <pid>           # Heap info
jstack <pid>               # Thread dump

# Profile
jcmd <pid> JFR.start duration=60s filename=recording.jfr
```

## Spring Framework (Parts 5, 7)

### Core Annotations
```java
@SpringBootApplication     // Main class
@RestController           // REST endpoint
@Service                  // Business logic
@Repository               // Data access
@Component                // Generic bean
@Configuration            // Config class
@Bean                     // Bean definition

// Dependency Injection
@Autowired
@Qualifier("beanName")
@Value("${property}")

// Web
@GetMapping("/path")
@PostMapping("/path")
@RequestBody
@PathVariable
@RequestParam

// JPA
@Entity
@Table(name = "users")
@Id
@GeneratedValue
@Column(name = "email")
@OneToMany
@ManyToOne
```

### REST Example
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
    
    @PostMapping
    public User createUser(@Valid @RequestBody CreateUserRequest request) {
        return userService.create(request);
    }
}
```

## Microservices (Part 6)

### Resilience4j Circuit Breaker
```java
@CircuitBreaker(name = "userService", fallbackMethod = "fallback")
public User getUser(Long id) {
    return restTemplate.getForObject("/users/" + id, User.class);
}

public User fallback(Long id, Exception e) {
    return new User("Unknown");
}
```

### Spring Cloud Config
```yaml
spring:
  application:
    name: user-service
  cloud:
    config:
      uri: http://config-server:8888
  config:
    import: optional:configserver:
```

## Cloud-Native (Part 8)

### Dockerfile
```dockerfile
FROM eclipse-temurin:21-jre
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Kubernetes Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:1.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
```

## Security (Part 10)

### Spring Security
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2Login(Customizer.withDefaults());
        
        return http.build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }
}
```

### JWT
```java
// Generate token
String token = Jwts.builder()
    .setSubject(username)
    .setIssuedAt(new Date())
    .setExpiration(new Date(now + 900000)) // 15 min
    .signWith(SignatureAlgorithm.HS512, secret)
    .compact();

// Validate token
Claims claims = Jwts.parser()
    .setSigningKey(secret)
    .parseClaimsJws(token)
    .getBody();
```

## Database (Part 11)

### JPA Query
```java
// Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    @Query("SELECT u FROM User u WHERE u.email = :email")
    Optional<User> findByEmail(@Param("email") String email);
    
    @Query("SELECT u FROM User u JOIN FETCH u.orders WHERE u.id = :id")
    Optional<User> findByIdWithOrders(@Param("id") Long id);
}
```

### MongoDB
```java
@Document(collection = "products")
public class Product {
    @Id
    private String id;
    @Indexed
    private String sku;
    private String name;
    private BigDecimal price;
}

// Query
List<Product> findByPriceBetween(BigDecimal min, BigDecimal max);
```

## Design Patterns (Part 12)

### SOLID Principles
```
S - Single Responsibility: One class, one reason to change
O - Open/Closed: Open for extension, closed for modification
L - Liskov Substitution: Subtypes must be substitutable
I - Interface Segregation: Many specific interfaces > one general
D - Dependency Inversion: Depend on abstractions, not concretions
```

### Common Patterns
```java
// Singleton
public enum Database { INSTANCE; }

// Factory
public interface Shape { void draw(); }
public static Shape create(String type) { ... }

// Builder
User user = new User.Builder("John", "Doe")
    .email("john@example.com")
    .phone("555-1234")
    .build();

// Strategy
public interface PaymentStrategy { void pay(double amount); }
cart.setPaymentStrategy(new CreditCardStrategy());

// Observer
stock.attach(new InvestorObserver("John"));
stock.setPrice(155.0);  // Notifies observers
```

## Common Maven Commands

```bash
# Build
mvn clean install          # Clean and build
mvn clean package         # Create JAR/WAR
mvn clean verify          # Run tests

# Run
mvn spring-boot:run       # Run Spring Boot app
mvn exec:java            # Run main class

# Test
mvn test                 # Run tests
mvn test -Dtest=UserServiceTest  # Run specific test

# Dependencies
mvn dependency:tree      # Show dependency tree
mvn dependency:analyze   # Analyze dependencies

# Deploy
mvn deploy              # Deploy to repository
```

## Git Commands

```bash
# Basic
git clone <url>
git add .
git commit -m "message"
git push origin main

# Branching
git checkout -b feature/new-feature
git merge feature/new-feature
git branch -d feature/new-feature

# Undo
git reset --hard HEAD~1    # Undo last commit
git revert <commit>        # Create revert commit
git stash                  # Temporarily save changes
```

## Docker Commands

```bash
# Build
docker build -t myapp:1.0 .

# Run
docker run -d -p 8080:8080 --name myapp myapp:1.0

# Manage
docker ps                  # List running containers
docker logs myapp         # View logs
docker exec -it myapp bash # Enter container
docker stop myapp         # Stop container
docker rm myapp           # Remove container

# Compose
docker-compose up -d      # Start services
docker-compose down       # Stop services
docker-compose logs -f    # Follow logs
```

## Kubernetes Commands

```bash
# Deployment
kubectl apply -f deployment.yaml
kubectl get pods
kubectl get services
kubectl describe pod <pod-name>

# Logs & Debug
kubectl logs <pod-name>
kubectl logs -f <pod-name>  # Follow
kubectl exec -it <pod-name> -- bash

# Scale
kubectl scale deployment myapp --replicas=5

# Update
kubectl set image deployment/myapp myapp=myapp:2.0
kubectl rollout status deployment/myapp
kubectl rollout undo deployment/myapp

# Config
kubectl create configmap myconfig --from-file=config.properties
kubectl create secret generic mysecret --from-literal=password=secret123
```

## Testing

### JUnit 5
```java
@Test
void testAddition() {
    assertEquals(5, calculator.add(2, 3));
}

@ParameterizedTest
@ValueSource(ints = {1, 2, 3})
void testPositive(int number) {
    assertTrue(number > 0);
}

@BeforeEach
void setUp() { /* Setup before each test */ }

@AfterEach
void tearDown() { /* Cleanup after each test */ }
```

### Mockito
```java
@Mock
private UserRepository userRepository;

@InjectMocks
private UserService userService;

@Test
void testGetUser() {
    when(userRepository.findById(1L))
        .thenReturn(Optional.of(new User("John")));
    
    User user = userService.getUser(1L);
    assertEquals("John", user.getName());
    
    verify(userRepository, times(1)).findById(1L);
}
```

## Regular Expressions

```java
// Common Patterns
String email = "^[A-Za-z0-9+_.-]+@(.+)$";
String phone = "^\\d{3}-\\d{3}-\\d{4}$";  // 555-123-4567
String url = "^https?://.*$";
String ipv4 = "^(?:[0-9]{1,3}\\.){3}[0-9]{1,3}$";

// Usage
Pattern pattern = Pattern.compile(email);
Matcher matcher = pattern.matcher("user@example.com");
if (matcher.matches()) { /* Valid email */ }
```

## Exception Handling

```java
// Try-catch-finally
try {
    riskyOperation();
} catch (SpecificException e) {
    handleSpecific(e);
} catch (Exception e) {
    handleGeneral(e);
} finally {
    cleanup();
}

// Try-with-resources
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    String line = br.readLine();
}

// Custom Exception
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(String message) {
        super(message);
    }
}
```

## Date & Time

```java
LocalDate date = LocalDate.now();
LocalTime time = LocalTime.now();
LocalDateTime dateTime = LocalDateTime.now();
ZonedDateTime zonedDateTime = ZonedDateTime.now();

// Formatting
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String formatted = dateTime.format(formatter);

// Parsing
LocalDate parsed = LocalDate.parse("2024-01-27");

// Manipulation
LocalDate tomorrow = LocalDate.now().plusDays(1);
LocalDateTime inOneHour = LocalDateTime.now().plusHours(1);
```

## File I/O

```java
// Read file
List<String> lines = Files.readAllLines(Paths.get("file.txt"));
String content = Files.readString(Paths.get("file.txt"));

// Write file
Files.writeString(Paths.get("output.txt"), "Hello World");
Files.write(Paths.get("output.txt"), lines);

// Stream lines
try (Stream<String> stream = Files.lines(Paths.get("file.txt"))) {
    stream.filter(line -> line.contains("Java"))
          .forEach(System.out::println);
}
```

## Concurrency

```java
// CompletableFuture
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> fetchData())
    .thenApply(data -> processData(data))
    .thenAccept(result -> System.out.println(result));

// ExecutorService
ExecutorService executor = Executors.newFixedThreadPool(10);
executor.submit(() -> doWork());
executor.shutdown();

// Virtual Threads (Java 21)
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> handleRequest());
}
```

## HTTP Client

```java
HttpClient client = HttpClient.newHttpClient();

HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/users"))
    .header("Content-Type", "application/json")
    .POST(HttpRequest.BodyPublishers.ofString("{\"name\":\"John\"}"))
    .build();

HttpResponse<String> response = client.send(request, 
    HttpResponse.BodyHandlers.ofString());

System.out.println(response.body());
```

## Performance Tips

### DO ✅
- Use StringBuilder for string concatenation in loops
- Pre-size collections when size is known
- Use primitive types instead of wrappers when possible
- Close resources with try-with-resources
- Use lazy loading for expensive operations
- Cache frequently accessed data
- Use parallel streams for CPU-intensive operations
- Profile before optimizing

### DON'T ❌
- Use + for string concatenation in loops
- Create unnecessary objects
- Ignore memory leaks
- Use finalize() (deprecated)
- Synchronize everything
- Optimize without measuring
- Use String.intern() casually
- Ignore warnings

## Common Pitfalls

```java
// ❌ Comparing strings with ==
if (str1 == str2) { }  // Compares references!

// ✅ Use equals()
if (str1.equals(str2)) { }

// ❌ Not checking for null
user.getName().toUpperCase();  // NullPointerException!

// ✅ Use Optional or check
Optional.ofNullable(user).map(User::getName).orElse("Unknown");

// ❌ Modifying collection while iterating
for (String item : list) {
    if (condition) list.remove(item);  // ConcurrentModificationException!
}

// ✅ Use iterator
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    if (condition) it.remove();
}
```

## Quick Interview Answers

**What is JVM?**  
Java Virtual Machine - executes Java bytecode, provides platform independence.

**Difference between JDK and JRE?**  
JDK = Development kit (compiler, tools). JRE = Runtime environment (JVM, libraries).

**What is Spring Boot?**  
Opinionated framework for building production-ready Spring applications quickly.

**What is REST?**  
Representational State Transfer - architectural style for web services using HTTP.

**What is microservices?**  
Architectural style where application is a collection of loosely coupled services.

**What is Docker?**  
Platform for containerization - packages applications with dependencies.

**What is Kubernetes?**  
Container orchestration platform for automating deployment, scaling, management.

---

**Print this sheet and keep it handy!** 📌

---

*Complete Java Mastery Series - Quick Reference*  
*For full details, see Parts 1-12*
